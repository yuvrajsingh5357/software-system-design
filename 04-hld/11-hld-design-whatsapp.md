# HLD Case Study: Design WhatsApp / Chat System

A real-time messaging system is one of the most technically challenging systems to design. It tests WebSocket architecture, message ordering, read receipts, and end-to-end encryption.

## Table of Contents
- [Requirements and Clarifications](#requirements-and-clarifications)
- [Capacity Estimation](#capacity-estimation)
- [API Design](#api-design)
- [Data Model](#data-model)
- [High-Level Architecture](#high-level-architecture)
- [Deep Dive: Real-Time Messaging](#deep-dive-real-time-messaging)
- [Deep Dive: Message Ordering and Delivery](#deep-dive-message-ordering-and-delivery)
- [Deep Dive: Group Chats](#deep-dive-group-chats)
- [Deep Dive: Media Sharing](#deep-dive-media-sharing)
- [⚡ Flashcard Summary](#-flashcard-summary)

---

## Requirements and Clarifications

**Ask these questions:**

1. "One-to-one messaging and group chats?" → Both
2. "Maximum group size?" → 1000 members
3. "Media: images, videos, documents?" → Yes
4. "Read receipts (sent/delivered/read indicators)?" → Yes
5. "Online/last seen status?" → Yes
6. "End-to-end encryption?" → Mention, but don't design in detail

**Functional Requirements:**
1. One-to-one messaging
2. Group messaging (up to 1000 members)
3. Message delivery receipts (sent ✓, delivered ✓✓, read ✓✓ blue)
4. Online/offline status and last seen
5. Media messages (images, videos, documents)
6. Message history and search

**Non-Functional Requirements:**
- 2 billion users (WhatsApp scale), 100M DAU for our design
- 100 billion messages/day = 1.16M messages/sec
- Message delivery: < 100ms (same region)
- High availability: 99.99%
- Messages stored for 30 days; media stored for 90 days

---

## Capacity Estimation

```
Messages:
  100M DAU × 40 messages/day = 4B messages/day
  4B / 86,400 = 46,296 messages/sec → ~50K messages/sec

Storage:
  Text message: 100 bytes
  50K/sec × 100B = 5MB/sec = 432GB/day
  
  Media (20% of messages): 50K × 20% × 100KB (avg) = 1GB/sec → too much
  Media handled separately: S3 + CDN; links in messages

Connections:
  100M DAU, all connected via WebSocket
  100M persistent connections → need many chat servers
  Each server handles 50K connections → 100M / 50K = 2000 chat servers
```

---

## API Design

```
WebSocket (real-time channel):
  // Client connects
  WS wss://chat.example.com/ws
  Auth: token in header or query param

  // Send message
  Client → Server:
  {
    "type": "MESSAGE",
    "chatId": "chat-abc123",
    "messageId": "msg-uuid-client-generated",  // client-generated for idempotency
    "content": "Hello!",
    "contentType": "TEXT",
    "timestamp": 1705312200000  // client timestamp
  }

  // Server ACK
  Server → Client:
  {
    "type": "ACK",
    "messageId": "msg-uuid-client-generated",
    "serverMessageId": "msg-server-generated",
    "serverTimestamp": 1705312200100
  }

  // Deliver to recipient
  Server → Recipient:
  {
    "type": "NEW_MESSAGE",
    "message": { ... }
  }

  // Read receipt
  Client → Server:
  { "type": "READ_RECEIPT", "chatId": "...", "lastReadMessageId": "..." }

REST APIs:
  GET /api/chats/{chatId}/messages?before={messageId}&count=50  // message history
  POST /api/chats/group  // create group
  PUT /api/chats/{chatId}/members  // add/remove member
  POST /api/media/upload  // get pre-signed S3 URL
```

---

## Data Model

### Messages Table (Cassandra — write-optimized, time-series)

```cql
CREATE TABLE messages (
    chat_id     UUID,
    message_id  BIGINT,   -- Snowflake ID (time-sortable)
    sender_id   UUID,
    content     TEXT,
    content_type TEXT,    -- TEXT, IMAGE, VIDEO, DOCUMENT
    media_url   TEXT,     -- S3 URL for media
    created_at  TIMESTAMP,
    PRIMARY KEY (chat_id, message_id)
) WITH CLUSTERING ORDER BY (message_id DESC);

-- Query: last 50 messages in a chat
SELECT * FROM messages WHERE chat_id = ? LIMIT 50;

-- Query: messages before a cursor
SELECT * FROM messages WHERE chat_id = ? AND message_id < ? LIMIT 50;
```

### Chats Table (PostgreSQL)

```sql
CREATE TABLE chats (
    chat_id    UUID PRIMARY KEY,
    chat_type  VARCHAR(10) NOT NULL,  -- DIRECT, GROUP
    created_at TIMESTAMP DEFAULT NOW(),
    -- For group chats:
    group_name VARCHAR(100),
    group_icon TEXT
);

CREATE TABLE chat_members (
    chat_id   UUID,
    user_id   UUID,
    joined_at TIMESTAMP,
    role      VARCHAR(10),  -- ADMIN, MEMBER
    PRIMARY KEY (chat_id, user_id)
);
CREATE INDEX idx_chat_members_user ON chat_members(user_id);  -- "get all chats for a user"

CREATE TABLE message_status (
    chat_id    UUID,
    message_id BIGINT,
    user_id    UUID,        -- whose status this is
    status     VARCHAR(10), -- SENT, DELIVERED, READ
    updated_at TIMESTAMP,
    PRIMARY KEY (chat_id, message_id, user_id)
);
```

### User Presence (Redis)

```
key:   "presence:{userId}"
value: { "status": "ONLINE", "lastSeen": 1705312200000 }
TTL:   30 seconds (refreshed by heartbeat every 10s)

If key expired → user is OFFLINE; lastSeen = last timestamp before expiry
```

---

## High-Level Architecture

```
                         Client A                    Client B
                           |                              |
                           | WebSocket                    | WebSocket
                           ▼                              ▼
                    ┌─────────────┐                ┌─────────────┐
                    │ Chat Server │                │ Chat Server │
                    │    (A's     │                │    (B's     │
                    │ connection) │                │ connection) │
                    └──────┬──────┘                └──────┬──────┘
                           │                              │
              ┌────────────▼──────────────────────────────▼──────────┐
              │                   Message Router                       │
              │              (discovers which server a user is on)     │
              └────────────────────────────────────────────────────────┘
                                         │
              ┌──────────────────────────▼──────────────────────────┐
              │                    Message Bus                        │
              │                      (Kafka)                          │
              └──────────────────────────────────────────────────────┘
                    │                  │                    │
         ┌──────────▼────┐  ┌──────────▼────┐   ┌──────────▼──────┐
         │  Message Store │  │  Notification │   │  Media Service  │
         │  (Cassandra)   │  │  Service      │   │  (S3 + CDN)     │
         └────────────────┘  └───────────────┘   └─────────────────┘
              │
         ┌────▼──────────┐
         │  PostgreSQL   │
         │  (chats,      │
         │   members,    │
         │   status)     │
         └───────────────┘
         ┌───────────────┐
         │  Redis        │
         │  (presence,   │
         │   routing)    │
         └───────────────┘
```

---

## Deep Dive: Real-Time Messaging

**How does message reach recipient in real-time?**

```
1. Client A (connected to Server 1) sends message

2. Server 1 receives message via WebSocket

3. Server 1 checks: which server is Client B connected to?
   → Redis: GET "user_server:{clientB_id}" = "server-42"

4a. If Client B is on this server (Server 1):
    → Push directly via WebSocket connection

4b. If Client B is on Server 42:
    → Publish to Redis pub/sub channel: PUBLISH "user:{clientB_id}" message
    → Server 42 is subscribed to "user:{clientB_id}"
    → Server 42 receives message → pushes to Client B via WebSocket

4c. If Client B is OFFLINE:
    → Store message in Cassandra
    → Send push notification (FCM for Android, APNs for iOS)
    → When Client B comes online: receive missed messages

5. Persist message to Cassandra (async via Kafka)

6. Send ACK to Client A: message delivered to server (✓ grey)
7. When Client B receives: send DELIVERED receipt (✓✓ grey)
8. When Client B opens chat: send READ receipt (✓✓ blue)
```

### WebSocket Connection Management

```
Each chat server:
  - Maintains 50K WebSocket connections
  - In-memory map: userId → WebSocket connection

  - Heartbeat: client pings every 10s; server closes if no ping for 30s
  - Reconnect: client auto-reconnects with exponential backoff

On server failure:
  - Client detects connection drop → reconnects to any available server
  - New server registers user: SET "user_server:{userId}" "new-server-id"
  - Missed messages fetched from Cassandra
```

---

## Deep Dive: Message Ordering and Delivery

**Problem:** Messages may arrive out of order due to network conditions.

**Solution: Sequence numbers per chat**

```
Cassandra message_id = Snowflake ID (time-sortable)

At sender: client generates optimistic message_id
At server: server assigns authoritative message_id (Snowflake)

Gaps detected: client requests missing range from server
Re-ordering: client sorts by server message_id before display
```

**At-least-once delivery:**
```
Client retries if no ACK within 5 seconds
Server deduplicates using client_message_id:
  INSERT INTO messages (client_message_id, ...) 
  IF NOT EXISTS  -- Cassandra lightweight transaction
```

---

## Deep Dive: Group Chats

**Challenge:** Group of 1000 members. One message → deliver to 1000 users.

```
Naive approach: fanout to all 1000 members
  1000 × 50K groups × 1 message/sec = 50M writes/sec → impractical

Hybrid approach (similar to Twitter celebrity):
  Small groups (<100): fanout on write
  Large groups (>100): fanout on read (members pull when they open the group)

Message stored once in Cassandra for the group chat_id
Members pull on open (pagination):
  SELECT * FROM messages WHERE chat_id = '{group-id}' LIMIT 50;
```

**Delivery receipts for groups:**
```
Group of 500:
  Each member's READ receipt → 500 status updates per message
  
Optimization:
  Don't show individual receipts for large groups
  Show aggregate: "Read by 342 of 500 members"
  Store last_read_message_id per member; aggregate in DB periodically
```

---

## Deep Dive: Media Sharing

```
Flow for image upload:
  1. Client: POST /api/media/upload → server returns pre-signed S3 URL (15 min expiry)
  2. Client: PUT {presigned-url} → upload directly to S3 (bypasses our servers!)
  3. Client: send message with media_url = S3 URL
  4. Server stores message; recipient downloads from S3 via CDN

CDN caching:
  Images/videos cached at CDN edge
  Cache-Control: max-age=86400 (1 day for immutable media)

Thumbnail generation:
  S3 upload → Lambda trigger → resize to thumbnail → store back to S3
  Message includes both full_url and thumbnail_url

Video:
  Client-side compression before upload (WhatsApp auto-compresses)
  Async transcoding: original S3 → transcoding service → multiple resolutions
```

---

## ⚡ Flashcard Summary

| # | Question | Answer |
|---|----------|--------|
| 1 | Messages/sec (100M DAU) | ~50K messages/sec |
| 2 | Connections per server | ~50K WebSocket connections |
| 3 | Total chat servers needed | 100M / 50K = 2000 chat servers |
| 4 | User-to-server routing | Redis: GET "user_server:{userId}" |
| 5 | Cross-server delivery | Redis pub/sub or Kafka to target server |
| 6 | Offline delivery | Store in Cassandra + FCM/APNs push notification |
| 7 | Message storage DB | Cassandra (partition by chat_id, cluster by message_id) |
| 8 | Presence storage | Redis with 30s TTL; heartbeat refreshes every 10s |
| 9 | Message ordering | Snowflake message_id; client sorts by server ID |
| 10 | At-least-once delivery | Client retries; server deduplicates by client_message_id |
| 11 | Group chat large groups | Fanout on read; store once, members pull on open |
| 12 | Media upload flow | Pre-signed S3 URL → client uploads directly to S3 |
| 13 | Read receipt (✓✓ blue) | Client sends READ_RECEIPT; stored in message_status |
| 14 | Read receipt for groups | Aggregate count; don't show individual for large groups |
| 15 | Reconnect strategy | Client reconnects with exponential backoff; fetches missed msgs |

---

*Next: [HLD Case Study — Design YouTube](12-hld-design-youtube.md)*

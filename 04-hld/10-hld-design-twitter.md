# HLD Case Study: Design Twitter / Social News Feed

This is the most comprehensive HLD case study — it covers fan-out, feed generation, database at scale, and real-time notifications.

## Table of Contents
- [Requirements and Clarifications](#requirements-and-clarifications)
- [Capacity Estimation](#capacity-estimation)
- [API Design](#api-design)
- [Data Model](#data-model)
- [High-Level Architecture](#high-level-architecture)
- [Deep Dive: Feed Generation](#deep-dive-feed-generation)
- [Deep Dive: Timeline Fanout](#deep-dive-timeline-fanout)
- [Deep Dive: Search and Trends](#deep-dive-search-and-trends)
- [Scalability Challenges](#scalability-challenges)
- [⚡ Flashcard Summary](#-flashcard-summary)

---

## Requirements and Clarifications

**Ask these questions:**

1. "What are the core features?" → Post tweet, follow/unfollow, view home feed, like/retweet
2. "Do we need real-time notifications?" → Yes (likes, retweets, mentions)
3. "What's the expected scale?" → 300M DAU, 500M tweets/day
4. "Do we need trending topics?" → Yes, basic top 10

**Functional Requirements:**
1. Post tweets (text, images, videos)
2. Follow/unfollow users
3. View home timeline (tweets from followed users)
4. View user profile timeline
5. Like and retweet
6. Trending topics
7. Search tweets

**Non-Functional Requirements:**
- 300M DAU, 500M tweets/day → ~5,800 tweets/sec
- Read heavy: 10:1 read/write
- Home feed latency: < 200ms
- Post tweet latency: < 500ms
- High availability: 99.99%
- Eventually consistent for feeds (1-2 second lag is acceptable)

---

## Capacity Estimation

```
Writes:
  500M tweets/day = 5,787 tweets/sec
  Peak: 10× = 57,870 tweets/sec (World Cup finals)

Reads:
  Home feed: each DAU checks feed 5× = 1.5B feed views/day = 17,361 feed reads/sec
  Peak: ~100K read requests/sec

Storage:
  Tweet: 280 chars = 280 bytes + metadata ≈ 1KB
  500M × 1KB = 500GB/day
  5 years: 500GB × 365 × 5 = ~900TB (tweets)
  
  Images (30% of tweets): 500M × 30% × 100KB = 15TB/day → CDN + S3
  Videos (5% of tweets): 500M × 5% × 5MB = 125TB/day → CDN + S3

Social Graph:
  300M users × average 200 follows = 60B follow relationships
  60B × (8 bytes follower + 8 bytes followee) = ~1TB
```

---

## API Design

```
POST /api/tweets
Headers: Authorization: Bearer {token}
Body: { "text": "Hello world! #Twitter", "mediaUrls": ["https://..."] }
Response: 201 Created → { "tweetId": "...", "text": "...", "createdAt": "..." }

GET /api/home/timeline?cursor={cursor}&count=20
Response: 200 OK
{
  "tweets": [
    {
      "tweetId": "...",
      "author": { "userId": "...", "username": "alice", "displayName": "Alice" },
      "text": "Hello world!",
      "likes": 1523,
      "retweets": 45,
      "createdAt": "2024-01-15T10:30:00Z"
    }
  ],
  "nextCursor": "..."
}

GET /api/users/{userId}/timeline?cursor={cursor}&count=20

POST /api/tweets/{tweetId}/likes
DELETE /api/tweets/{tweetId}/likes

POST /api/follow
Body: { "targetUserId": "..." }

DELETE /api/follow/{targetUserId}

GET /api/search/tweets?q=keyword&type=RECENT

GET /api/trends
Response: [{ "topic": "#WorldCup", "tweetCount": 1250000 }, ...]
```

---

## Data Model

### Users Table (PostgreSQL)

```sql
CREATE TABLE users (
    user_id       BIGINT PRIMARY KEY,  -- Snowflake ID
    username      VARCHAR(50) UNIQUE NOT NULL,
    display_name  VARCHAR(100),
    email         VARCHAR(255) UNIQUE NOT NULL,
    bio           TEXT,
    profile_image TEXT,
    created_at    TIMESTAMP DEFAULT NOW(),
    follower_count BIGINT DEFAULT 0,
    following_count BIGINT DEFAULT 0
);
```

### Tweets Table (PostgreSQL, sharded by user_id)

```sql
CREATE TABLE tweets (
    tweet_id      BIGINT PRIMARY KEY,  -- Snowflake ID (time-sortable)
    user_id       BIGINT NOT NULL,
    text          VARCHAR(280),
    media_urls    TEXT[],              -- array of S3 URLs
    like_count    BIGINT DEFAULT 0,
    retweet_count BIGINT DEFAULT 0,
    created_at    TIMESTAMP DEFAULT NOW(),
    FOREIGN KEY (user_id) REFERENCES users(user_id)
);

CREATE INDEX idx_tweets_user_created ON tweets(user_id, created_at DESC);
```

### Social Graph (Separate service — Cassandra)

```cql
-- Follow relationships: scalable, write-optimized
CREATE TABLE follows (
    follower_id UUID,
    followee_id UUID,
    followed_at TIMESTAMP,
    PRIMARY KEY (follower_id, followee_id)
);

-- Reverse lookup: who follows user X?
CREATE TABLE followers (
    followee_id UUID,
    follower_id UUID,
    followed_at TIMESTAMP,
    PRIMARY KEY (followee_id, follower_id)
);
```

### Timeline / Feed (Redis + Cassandra)

```
Redis (hot timeline cache):
  key: "timeline:{userId}"
  type: Sorted Set
  score: tweet_id (Snowflake, time-sortable)
  member: tweet payload JSON (or just tweet_id for hybrid)

Cassandra (persistent timeline):
  CREATE TABLE home_timeline (
      user_id UUID,
      tweet_id BIGINT,
      author_id UUID,
      PRIMARY KEY (user_id, tweet_id)
  ) WITH CLUSTERING ORDER BY (tweet_id DESC);
```

---

## High-Level Architecture

```
                    ┌──────────────────────────────────────┐
                    │          API Gateway / Load Balancer  │
                    └──────┬───────────────────────┬────────┘
                           │                       │
              ┌────────────▼────────┐  ┌───────────▼──────────────┐
              │    Read Services     │  │    Write Services         │
              │  - Timeline Service  │  │  - Tweet Service          │
              │  - Search Service    │  │  - Follow Service         │
              └──────┬──────────────┘  └───────────┬──────────────┘
                     │                             │
              ┌──────▼────────────┐    ┌───────────▼─────────────┐
              │  Redis Timeline   │    │        Kafka            │
              │  Cache (hot users)│    │  (tweet-created events) │
              └──────┬────────────┘    └───────────┬─────────────┘
                     │ (miss)                      │
         ┌───────────▼──────────┐   ┌──────────────▼───────────────┐
         │  Cassandra Timeline  │   │       Fanout Service          │
         │  (persistent)        │   │  (writes to followers' feeds) │
         └──────────────────────┘   └──────────────┬───────────────┘
                                                    │
                    ┌───────────────────────────────▼──────────────┐
                    │       Cassandra + Redis (timeline stores)     │
                    └──────────────────────────────────────────────┘
                    ┌───────────────────────────────────────────────┐
                    │   PostgreSQL (tweets, users, likes)           │
                    │   + Read replicas                             │
                    └───────────────────────────────────────────────┘
                    ┌───────────────────────────────────────────────┐
                    │   S3 + CloudFront CDN (media storage)         │
                    └───────────────────────────────────────────────┘
```

---

## Deep Dive: Feed Generation

Two approaches to generating a user's home timeline:

### Approach 1: Pull (Fan-out on Read)

```
When user views feed:
  1. Get list of users they follow (from social graph)
  2. Fetch recent tweets from each followed user
  3. Merge and sort by timestamp
  4. Return top 20

Pros: No write amplification; celebrities with 100M followers don't cause 100M writes
Cons: High read latency; complex merge; expensive for users with many follows

Time complexity: O(n × T) where n = follows, T = tweets to merge
```

### Approach 2: Push (Fan-out on Write) ← Twitter's historical approach

```
When user posts a tweet:
  1. Kafka event: tweet-created
  2. Fanout Service reads event
  3. For each of author's followers:
     → Write tweet to follower's timeline (Redis + Cassandra)

When user views feed:
  1. Read pre-computed timeline from Redis (very fast!)

Pros: O(1) feed read; very fast
Cons: Write amplification for celebrities; 100M followers = 100M writes per tweet!
```

### Approach 3: Hybrid (Twitter's current approach)

```
Regular users (< 1M followers): Fan-out on Write
  - Tweet → Kafka → Fanout → write to all followers' feeds

Celebrities (> 1M followers): Fan-out on Read
  - Mark as "celebrity" flag
  - When user opens feed:
    1. Fetch pre-computed timeline (push-based follows)
    2. Fetch recent tweets from followed celebrities (pull)
    3. Merge (small merge — few celebrities)

This way:
  - Most feeds: O(1) read
  - Celebrity tweets: only fetched when needed (no 100M writes)
```

---

## Deep Dive: Timeline Fanout

```
Fanout Service (after user posts tweet):

1. Kafka Consumer receives tweet-created event
2. Look up followers of author (Social Graph Service)
3. For each follower:
   a. Add tweet to Redis sorted set: ZADD timeline:{followerId} {tweetId} {tweetId}
   b. Trim to last 800 tweets: ZREMRANGEBYRANK timeline:{followerId} 0 -801
   c. If follower has no Redis timeline (cold): persist to Cassandra only

4. In parallel: index tweet for search (Elasticsearch)
5. In parallel: process for trending topics

For 5,800 tweets/sec with avg 200 followers:
  5,800 × 200 = 1.16M timeline writes/sec
  Redis: 1M ops/sec per instance → need 2-3 Redis instances
```

---

## Deep Dive: Search and Trends

### Full-Text Search

```
Elasticsearch cluster:
  - Inverted index: term → list of tweet IDs
  - Near real-time (NRT): indexed within 1 second of post
  - Horizontal scaling: shard tweets by tweet_id range

Tweet indexing (via Kafka → Elasticsearch connector):
  {
    "tweet_id": "...",
    "text": "Hello world #Twitter",
    "author": "alice",
    "timestamp": "2024-01-15T10:30:00Z",
    "hashtags": ["Twitter"]
  }

Trending topics:
  Count hashtag occurrences in last 24 hours
  Sliding window using Redis sorted set:
    ZINCRBY trending:hashtags:{window} 1 "#WorldCup"
  Top 10: ZREVRANGE trending:hashtags:{window} 0 9 WITHSCORES
```

---

## Scalability Challenges

### Challenge 1: Celebrity Problem (Thunderbird Problem)

**Situation:** Justin Bieber has 100M followers. He posts → 100M writes.

**Solution:** Hybrid approach above. Celebrity tweets fetched separately at read time.

### Challenge 2: Hot Partition in Kafka

**Situation:** All tweets from a viral moment go to the same Kafka partition → one partition overloaded.

**Solution:** Partition by tweet_id (not user_id) for uniform distribution.

### Challenge 3: Database Write Contention on Likes

**Situation:** Viral tweet gets millions of likes/sec → UPDATE tweets SET likes = likes + 1 → contention.

**Solution:**
1. Counter in Redis: `INCR tweet:like_count:{tweetId}` (atomic, fast)
2. Periodic batch sync to PostgreSQL (every 10 seconds)

---

## ⚡ Flashcard Summary

| # | Question | Answer |
|---|----------|--------|
| 1 | Tweet/sec (500M/day) | ~5,800 writes/sec |
| 2 | Read/write ratio | ~10:1 (feed reads dominate) |
| 3 | Fan-out on write | Push tweet to all followers' timelines at write time |
| 4 | Fan-out on read | Pull tweets from all follows at read time |
| 5 | Hybrid approach | Regular users: push; celebrities: pull at read |
| 6 | Celebrity threshold | ~1M followers; fetch their tweets at read time |
| 7 | Timeline storage | Redis sorted set (hot) + Cassandra (persistent) |
| 8 | Redis timeline ops | ZADD + ZREMRANGEBYRANK (keep last 800 tweets) |
| 9 | Fanout write throughput | 5,800 × 200 followers = 1.16M timeline writes/sec |
| 10 | Search indexing | Kafka → Elasticsearch; inverted index; NRT |
| 11 | Trending topics | Redis sorted set with 24h sliding window |
| 12 | Like counter | Redis INCR → periodic batch sync to PostgreSQL |
| 13 | Snowflake for tweet_id | Time-sortable → feed sorted by ID = sorted by time |
| 14 | Media storage | S3 + CDN (CloudFront) for images/videos |
| 15 | Viral event handling | Circuit breaker + rate limiting; autoscale fanout workers |

---

*Next: [HLD Case Study — Design WhatsApp](11-hld-design-whatsapp.md)*

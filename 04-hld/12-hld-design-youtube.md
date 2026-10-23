# HLD Case Study: Design YouTube (Video Streaming Service)

Design a global video streaming platform like YouTube or Netflix — tests video transcoding pipelines, CDN distribution, and adaptive bitrate streaming.

## Table of Contents
- [Requirements and Clarifications](#requirements-and-clarifications)
- [Capacity Estimation](#capacity-estimation)
- [High-Level Architecture](#high-level-architecture)
- [Deep Dive: Video Upload & Transcoding Pipeline](#deep-dive-video-upload--transcoding-pipeline)
- [Deep Dive: Video Streaming & Adaptive Bitrate](#deep-dive-video-streaming--adaptive-bitrate)
- [Deep Dive: CDN and Cache Optimizations](#deep-dive-cdn-and-cache-optimizations)
- [Interview Tips](#interview-tips)
- [⚡ Flashcard Summary](#-flashcard-summary)

---

## Requirements and Clarifications

**Functional Requirements:**
1. Users can upload videos.
2. Users can watch videos (real-time streaming).
3. Search videos by title/tags.
4. Track video analytics (views, likes, dislikes).
5. Support comments and sharing.

**Non-Functional Requirements:**
1. **Low Latency:** Smooth video playback, minimal buffering.
2. **High Availability:** 99.99% system availability.
3. **High Scalability:** 2 billion active users daily, millions of uploads.
4. **Reliability:** No uploaded video should ever be lost.

---

## Capacity Estimation

```
Scale:
  Daily Active Users (DAU) = 100M
  Average watch time = 30 minutes/user/day
  Average video size = 10 minutes, 1080p, 30fps = ~200MB

Writes (Uploads):
  Assume 1% of DAU uploads 1 video per day = 1M videos/day
  1M videos / 86400 seconds = ~12 uploads/sec
  Daily Storage: 1M × 200MB = 200 TB/day
  5-Year Storage (including replication and transcoding multiple formats ~5×):
    200 TB × 365 × 5 × 5 = 18.25 PB (Petabytes)

Reads (Streaming):
  Average watch time: 30 minutes/user/day
  100M users × 30 minutes = 3B minutes watched/day
  3B / 10 mins (average video length) = 300M videos watched/day
  Average QPS (Reads) = 300M / 86400 = ~3,500 plays/sec
  Peak QPS = 3,500 × 2 = ~7,000 plays/sec

Egress Bandwidth:
  Assume average stream quality is 720p (~1 Mbps)
  7,000 concurrent plays × 1 Mbps = 7 Gbps egress bandwidth
```

---

## High-Level Architecture

```
                    ┌────────────────────────────┐
                    │     DNS / Load Balancer    │
                    └───────┬────────────┬───────┘
                            │            │
            ┌───────────────▼──┐      ┌──▼────────────────┐
            │   API Gateway    │      │  Upload Service   │
            │ (Metadata/Auth)  │      │ (Chunked Uploads) │
            └───────┬──────────┘      └──┬────────────────┘
                    │                    │
            ┌───────▼──────────┐      ┌──▼────────────────┐
            │   User/Video     │      │   Raw S3 Bucket   │
            │   Metadata DB    │      └──┬────────────────┘
            └──────────────────┘         │
                                      ┌──▼────────────────┐
                                      │   Transcoder &    │
                                      │   Processor       │
                                      └──┬────────────────┘
                                         │
            ┌──────────────────┐      ┌──▼────────────────┐
            │    CDN Edge      │◄─────│  Encoded S3/CDN   │
            │    (Akamai)      │      └───────────────────┘
            └────────▲─────────┘
                     │ (Stream)
            ┌────────┴─────────┐
            │    Client App    │
            └──────────────────┘
```

**Workflow for Uploading:**
1. Client uploads raw video to `Upload Service`.
2. Raw video is saved in temporary object storage (Raw S3).
3. `Transcoding Service` is notified, splits video into chunks, encodes into multiple formats and resolutions, and stores them in `Encoded S3`.
4. Metadata DB is updated with video path, duration, and user details.

**Workflow for Streaming:**
1. Client fetches video metadata and stream URL from `API Gateway`.
2. Client sends request to closest `CDN Edge` to stream video chunks.
3. If CDN cache misses, CDN fetches chunks from `Encoded S3` and caches them locally.

---

## Deep Dive: Video Upload & Transcoding Pipeline

Directly uploading a single 10 GB file is prone to failure (network timeouts). 

### 1. Chunked Upload
- The client splits the video into small chunks (e.g., 2-5 MB).
- Chunks are uploaded sequentially. If one fails, only that chunk is re-uploaded.

### 2. Transcoding Architecture
Raw video must be compressed and formatted to run on different devices (iOS, Android, SmartTVs) and network speeds.
- **DAG (Directed Acyclic Graph) Workflow:**
  ```
  [ Raw Video ] ──► [ Splitter ] ──┬──► [ 360p Encoder ] ──► [ Joiner ] ──► [ S3 ]
                                    ├──► [ 720p Encoder ] ──► [ Joiner ] ──► [ S3 ]
                                    └──► [ 1080p Encoder ] ─► [ Joiner ] ──► [ S3 ]
  ```
- **Encoding Formats:** H.264 (universal support), VP9/AV1 (better compression, modern).

---

## Deep Dive: Video Streaming & Adaptive Bitrate

Streaming protocols download video sequentially instead of downloading the entire file first.

### 1. Adaptive Bitrate (ABR) Streaming
The client player automatically adjusts video quality (e.g., drops from 1080p to 360p) based on real-time network bandwidth.
- **How it works:** The transcoded video is stored in multiple resolutions, split into 2-second segments. Along with the segments, an index file (manifest) is generated.
- **Manifest File (e.g., `.m3u8` for HLS):** Lists all available quality streams and the URLs of all 2-second chunks. The client player reads this manifest and chooses which chunk URL to fetch next.

### 2. Streaming Protocols
- **HLS (HTTP Live Streaming):** Apple's protocol. Widely supported, runs over standard HTTP, easy to cache on CDNs.
- **DASH (Dynamic Adaptive Streaming over HTTP):** Open standard, similar to HLS.

---

## Deep Dive: CDN and Cache Optimizations

Streaming 7 Gbps directly from origin databases is cost-prohibitive and slow.

### 1. Geo-distributed CDNs
We place edge servers (CDNs) close to users globally. 99% of streaming traffic is served from the CDN.

### 2. Cache Tiering & Eviction
- **Hot Videos:** Popular/newly uploaded videos are pushed to CDN edge caches immediately.
- **Long-tail (Cold) Videos:** Infrequently watched videos are not cached on edges; they stay in object storage (S3) and are fetched on-demand.
- **Eviction:** CDNs use Least Recently Used (LRU) cache eviction.

---

## Interview Tips

1. **Understand Transcoding:** Emphasize that transcoding is a GPU/CPU-heavy async process. Never do it synchronously on the web servers.
2. **Explain ABR:** Show that you understand how HLS/DASH manifest files work (e.g., why Netflix doesn't buffer forever).
3. **Clarify storage:** Videos are never stored in databases; they live in Object Storage (S3/GCS). Only URLs, titles, and tags are stored in SQL/NoSQL.

---

## ⚡ Flashcard Summary

| # | Question | Answer |
|---|----------|--------|
| 1 | Egress traffic nature | Highly read-heavy; requires global CDN caching |
| 2 | Video storage | Object storage (S3/GCS) with CDN distribution |
| 3 | Chunked upload benefit | Resumes upload from failed chunk instead of restarting |
| 4 | Transcoding necessity | Encodings (H.264/VP9) & resolutions (360p..1080p) fit all screens |
| 5 | Adaptive Bitrate (ABR) | Player dynamically adjusts quality based on network bandwidth |
| 6 | Streaming protocols | HLS (Apple) and DASH (Open Standard) |
| 7 | Manifest file (.m3u8) | Index file linking to all segment URLs of different qualities |
| 8 | CDN cache eviction | LRU (Least Recently Used) keeps hot videos on edges |
| 9 | Long-tail content | Unpopular videos served directly from S3 (bypassing CDN) |
| 10 | Metadata DB | Sharded MySQL or Cassandra (scale video indexes) |

---

*Next: [HLD Case Study — Design Uber](13-hld-design-uber.md)*

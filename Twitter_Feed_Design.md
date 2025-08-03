# System Design: Twitter's Feed Fetching System

This document outlines the high-level design for a scalable system to fetch a user's Twitter feed, focusing on the read-heavy nature of the service.

---

### 1. Understanding the Problem & Clarifying Questions

The core task is to design the read-path for Twitter's timeline. Before starting, it's crucial to establish the scope with clarifying questions:

*   **Scale:** The system must operate at Twitter's scale, handling hundreds of millions of Daily Active Users (DAU) and millions of read requests per second. We'll assume a **90% read, 10% write** traffic pattern.
*   **Feed Content:** The primary goal is to fetch a list of `tweet_ids`. A separate service would then be responsible for "hydrating" these IDs into full tweet objects containing text, media, user info, etc.
*   **Feed Type:** The design will focus on a **chronological feed** as the foundational model. An algorithmic feed would be an extension built on top of this base.

---

### 2. QPS (Queries Per Second) Calculation

To justify our design decisions, we must estimate the load on the system.

#### Assumptions
*   **Daily Active Users (DAU):** 300 million
*   **User Read Activity:** The average user fetches their feed (or a new chunk of it) 30 times per day.

#### Read QPS Calculation
1.  **Total Daily Reads:** 300,000,000 users × 30 reads/user = **9,000,000,000 (9 billion) reads/day**.
2.  **Seconds in a Day:** 24 × 60 × 60 = **86,400 seconds**.
3.  **Average Read QPS:** 9,000,000,000 / 86,400 ≈ **104,000 QPS**.
4.  **Peak Read QPS:** Traffic is not uniform. Assuming peak traffic is **2x** the average: 104,000 × 2 = **~210,000 QPS**.

Our system must be able to handle **~210,000 read requests per second.** This high read load dictates our architectural choices.

---

### 3. High-Level Architecture Diagram

```mermaid
graph TD
    subgraph "User & Edge"
        UserClient["User's Client (Phone/Web)"] -->|1. GET /v2/timeline| APIGateway["API Gateway<br/>(Auth & Rate Limiting)"];
    end

    APIGateway -->|2. Forward Request| FeedService["Feed Generation Service"];

    subgraph "Core Read Path"
        FeedService -->|3. Fetch pre-computed feed| RedisCache["<b>Redis Cache</b><br/>(User Timelines)"];
        RedisCache --> FeedService;
        FeedService -->|4. Get celebrity tweets (if any)| CassandraDB["<b>Cassandra DB</b><br/>(Tweets, Follows, Users)"];
        CassandraDB --> FeedService;
    end
    
    FeedService -->|5. Merge feeds & return IDs| UserClient;

    subgraph "Background Write Path (Fan-out)"
        OtherServices["Other Services (e.g., Post a Tweet)"] -->|New Tweet| FanoutWorker["Fan-out Workers"];
        FanoutWorker -->|Inject tweet into follower feeds| RedisCache;
    end
```

---

### 4. Database Choice and Schema

A relational database would not scale for this read volume. A **NoSQL, wide-column store like Cassandra or DynamoDB** is the ideal choice due to its horizontal scalability and high availability.

*   **`tweets` table:** `tweet_id` (PK), `user_id`, `text`, `media_urls`, `created_at`
*   **`followers` table:** `user_id` (PK), `follower_id`

---

### 5. Feed Generation: A Hybrid Push/Pull Model

This is the most critical design decision.

1.  **Pull Model (Read-time generation):** Fetch tweets from everyone a user follows when they request their feed. This is too slow and doesn't scale.
2.  **Push Model (Write-time generation):** When a user tweets, "push" the new tweet into the timeline of all their followers. This makes reads incredibly fast but suffers from the **"fan-out" problem** for celebrities with millions of followers.

**Solution: Hybrid Model**

*   **For most users (< 5,000 followers):** Use the **push model**. When they tweet, a background worker injects the tweet ID into a cached timeline for each follower.
*   **For "celebrities" (> 5,000 followers):** Use the **pull model**. We do *not* fan-out their tweets. Instead, their tweets are merged into a user's feed at read-time.

This balances fast reads for everyone with manageable writes.

---

### 6. Caching Layers

Caching is critical for low latency.

*   **CDN for Static Assets:** All media (images, videos) is served from a CDN.
*   **In-Memory Cache for Feeds (Redis):** A Redis cluster stores the pre-computed timelines.
    *   **Data Structure:** A Redis List for each user: `timeline:<user_id>`.
    *   **Operation:** Reads are a simple `LRANGE` command. Writes are `LPUSH` (to add to the front) and `LTRIM` (to keep the list at a max size, e.g., 800 tweets). This makes the core read operation extremely fast.

---

### 7. Rate Limiting

To protect the system from abuse, we implement rate limiting at the edge (in the API Gateway).

*   **Algorithm:** The **Token Bucket** algorithm is suitable, as it allows for brief bursts while enforcing a sustained rate.
*   **Implementation:** Track request counts per user in a distributed cache like Redis.

---

### 8. Summary of the Read-Path Flow

1.  A user's client requests their timeline.
2.  The request hits the **API Gateway**, which handles authentication and checks rate limits.
3.  The request is routed to the **Feed Generation Service**.
4.  The service fetches the user's pre-computed timeline from the **Redis Cache**.
5.  It identifies any **celebrities** the user follows and fetches their recent tweets from the main database (this can also be cached).
6.  The service **merges** the two lists.
7.  A final, sorted list of `tweet_ids` is returned.
8.  The client then fetches the full content for those specific tweets from a separate service.

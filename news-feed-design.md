# News Feed Service Design

## Patterns

- [Fanout-on-Write Pattern(pull)](#fanout-on-write-pattern)
- [Fanout-on-Read Pattern(pull)](#fanout-on-read-pattern)


## Fanout-on-Write Pattern

Fan-out-on-write is an architectural design pattern in which data is immediately pushed or pre-distributed to all target consumers' inboxes or storage locations at the moment it is created (written). The primary goal is to optimize for very fast read operations by performing the heavy computational work upfront.

### How It Works

When a user creates a new piece of content (e.g., a social media post):

1. The system identifies all relevant recipients (e.g., all followers of the user)
2. The content is instantly written into each recipient's individual, precomputed feed or timeline
3. When a user views their feed, the data is already there and can be retrieved with simple, fast lookups

### Pros and Cons

| Aspect | Description |
|--------|-------------|
| **Pros** | Ultra-fast reads with very low latency; instant feed display |
| | Consistent read performance, regardless of follower count |
| | Reduced read-time computation cost |
| **Cons** | Write amplification from single posts to thousands/millions of followers |
| | Expensive for high-follower "celebrity" accounts (hotkey problem) |
| | Increased storage overhead from data duplication |

### Use Cases and Mitigation

Ideal for systems where read frequency significantly exceeds write frequency. Twitter uses this approach for most user timelines.

A hybrid approach addresses high-follower account issues:
- Fanout-on-write for regular users
- Fanout-on-read for celebrity accounts



## Fanout-on-Read Pattern

Fan-out-on-read is an architectural design pattern where data is not pre-distributed to consumers but is instead fetched at the time of reading. This approach is beneficial in scenarios where the write frequency is high, and the read frequency is variable.

### How It Works

When a user wants to view their feed:

1. The system retrieves the latest content from a central storage location.
2. The content is compiled and delivered to the user in real-time.
3. This allows for dynamic content updates and reduces storage overhead.

### Pros and Cons

| Aspect | Description |
|--------|-------------|
| **Pros** | Lower storage costs due to no data duplication; flexibility in content delivery |
| | Real-time updates for users, ensuring they see the latest content |
| **Cons** | Slower read operations compared to fanout-on-write; potential for increased latency during peak times |

### Use Cases

Ideal for systems with high write activity or where content is frequently updated, such as news feeds or live event updates.



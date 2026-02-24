## Why To Consider Replacing Redis

The pain points:

* Two databases to backup (PostgreSQL as main database)
* Redis uses RAM (expensive at scale)
* Redis persistence is complicated
* Network hop between Postgres and Redis

### Two databases. Two things to manage. Two points of failure.

### Realization: PostgreSQL can do everything Redis does.

## Reasons to Replace Redis

| Reason | with Redis | without Redis |
|---------|-------|------------|
| Cost at Scale | High | Lower (same PostgreSQL for cache) |
| Operational Complexity | 1. Postgres backup<br>2. Redis backup<br>3. Postgres monitoring<br>4. Redis monitoring<br>5. Postgres failover<br>6. Redis Sentinel/Cluster | 1. Postgres backup<br>2. Postgres monitoring<br>3. Postgres failover |
| Data Consistency | `await redis.del(user:${id});`<br>⚠️ What if Redis is down?<br>⚠️ What if this fails?<br>Now cache and DB are out of sync<br><br>**Issues:**<br>- Cache invalidation complexity<br>- Network dependency<br>- Sync failures possible | `await db.query('UPDATE users SET name = $1 WHERE id = $2', [name, id]);`<br>Everything in Postgres<br>Transactions solve consistency<br><br>**Benefits:**<br>- Single source of truth<br>- ACID guarantees<br>- No sync issues |

## PostgreSQL Features for Cache

| Feature | Redis | PostgreSQL |
|---------|-------|------------|
| Caching with UNLOGGED Tables | `await redis.set('session:abc123', JSON.stringify(sessionData), 'EX', 3600);` | `CREATE UNLOGGED TABLE cache (`<br>`key TEXT PRIMARY KEY,`<br>`value JSONB NOT NULL,`<br>`expires_at TIMESTAMPTZ NOT NULL`<br>`);`<br><br>`CREATE INDEX idx_cache_expires`<br>`ON cache(expires_at);` <br><br>### INSERT ###<br> `INSERT INTO cache (key, value, expires_at)' `<br>` 'VALUES ($1, $2, NOW,() + INTERVAL '1 hour')` <br> `ON CONFLICT (key)`<br> `DO UPDATE SET value = EXCLUDED.value, expires_at = EXCLUDED.expires_at;` <br><br>###READ###<br> `SELECT value FROM cache` <br> `WHERE key = $1 AND expires_at > NOW();`<br><br>###Cleanup (run periodically)### <br> `DELETE FROM cache WHERE expires_at < NOW();`|
| Pub/Sub with LISTEN/NOTIFY | // Publisher<br> `redis.publish('notifications', JSON.stringify({ userId: 123, msg: 'Hello' }));` <br><br>// Subscriber `redis.subscribe('notifications');` <br>`redis.on ('message', (channel, message) => { console.log(message);});`| -- Publisher<br> `NOTIFY otifications, '{"userId": 123, "msg": "Hello"}';` <br><br>// Subscriber (Node.js with pg)<br> `const client = new Client({ connectionString: process.env.DATABASE_URL });`<br>`await client.connect();`<br>`await client.query('LISTEN notifications');`<br>`client.on('notification', (msg) => {`<br>`const payload = JSON.parse(msg.payload);`<br>`console.log(payload);`<br>`});`|
| Job Queues with SKIP LOCKED | `queue.add('send-email', { to, subject, body });`<br> `queue.process ('send-email', async (job) => {`<br> `await sendEmail(job.data);`<br>`});` |`CREATE TABLE jobs (`<br>`id BIGSERIAL PRIMARY KEY,`<br>`queue TEXT NOT NULL,`<br>`payload JSONB NOT NULL,`<br>`  attempts INT DEFAULT 0,`<br>`  max_attempts INT DEFAULT 3,`<br>`  scheduled_at TIMESTAMPTZ `<br>`DEFAULT NOW(),`<br>`  created_at TIMESTAMPTZ DEFAULT NOW()`<br>`);`<br><br>`CREATE INDEX idx_jobs_queue ON jobs(queue, scheduled_at)`<br>`WHERE attempts < max_attempts;` <br><br>###Enqueue### <br> `INSERT INTO jobs (queue, payload)`<br>`VALUES ('send-email', '{"to": "user@example.com", "subject": "Hi"}');`<br><br>###Worker (dequeue)###<br>`WITH next_job AS (`<br>`  SELECT id FROM jobs`<br>`  WHERE queue = $1`<br>`    AND attempts < max_attempts`<br>`    AND scheduled_at <= NOW()`<br>`  ORDER BY scheduled_at`<br>`  LIMIT 1`<br>`  FOR UPDATE SKIP LOCKED)`<br>`UPDATE jobs`<br>`SET attempts = attempts + 1`<br>`FROM next_job`<br>`WHERE jobs.id = `<br>`next_job.id`<br>`RETURNING *;`|
| Rate Limiting | `const key = `ratelimit:${userId}`;`<br>`const count = await redis.incr(key);`<br>`if (count === 1) {`<br>`  await redis.expire(key, 60);`<br> // 60 seconds<br>`}`<br><br>`if (count > 100) {`<br>`  throw new Error('Rate limit exceeded');`<br>`}` <Br><br><br>##When Redis is better:##<br>1. Need sub-millisecond rate limiting<br>2. Extremely high throughput (millions of requests/sec)| `CREATE TABLE api_requests (`<br>`  user_id INT NOT NULL,`<br>`  created_at TIMESTAMPTZ DEFAULT NOW()`<br>`);`<br><br>-- Check rate limit <br>`SELECT COUNT(*) FROM api_requests`<br>`WHERE user_id = $1`<br>`  AND created_at > OW() - INTERVAL '1 minute';`<br><br>-- If under limit, insert<br>`INSERT INTO api_requests (user_id) VALUES ($1);`<br><br>-- Cleanup old requests periodically<br>`DELETE FROM api_requests WHERE created_at < NOW() - INTERVAL '5 minutes';` <br><br><br> ##When Postgres is better:## <br>1. Need to rate limit based on complex logic (not just counts)<br>2. Want rate limit data in same transaction as business logic|
| Sessions with JSONB | `await redis.set(`session:${sessionId}`, JSON.stringify(sessionData), 'EX', 86400);`| `CREATE TABLE sessions (`<br>`  id TEXT PRIMARY KEY,`<br>`  data JSONB NOT NULL,`<br>`expires_at TIMESTAMPTZ NOT NULL`<br>`);`<br><br>`CREATE INDEX idx_sessions_expires ON sessions(expires_at);`<br><br>-- Insert/Update<br>`INSERT INTO sessions (id, data, expires_at)`<br>`VALUES ($1, $2, NOW() + INTERVAL '24 hours')`<br>`ON CONFLICT (id) DO UPDATE`<br>`  SET data = EXCLUDED.data, expires_at = EXCLUDED.expires_at;`<br><br>-- Read<br>`SELECT data FROM sessions`<br>`WHERE id = $1 AND expires_at > NOW();`<br><br><br>##bonus##<br>-- Find all sessions for a specific user<br>`SELECT * FROM sessions`<br>`WHERE data->>'userId' = '123';`<br><br>-- Find sessions with specific role<br>`SELECT * FROM sessions`<br>`WHERE data->'user'->>'role' = 'admin';` |

## When to Keep Redis
1. When Need Extreme Performance
2. When Using Redis-Specific Data Structures(Sorted sets/HyperLogLog/advanced pub/sub)
3. When Application Have a Separate Caching Layer Requirement

## Decision Matrix

| Replace Redis with Postgres | Keep Redis |
|---------|-------|
| You're using Redis for simple caching/sessions | You need 100k+ ops/second
Cache hit rate is < 95% (lots of writes) | You use Redis data structures (sorted sets, etc.)
You want transactional consistency | You have dedicated ops team
You're okay with 0.1-1ms slower operations | Sub-millisecond latency is critical
You're a small team with limited ops resources | You're doing geo-replication






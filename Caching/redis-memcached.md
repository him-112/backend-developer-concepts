# Redis & Memcached - Caching Technologies Guide 🔥

## 🎯 What You'll Learn
Master Redis and Memcached for high-performance caching and data storage!

## 🌟 Caching Fundamentals

### Redis vs Memcached Overview
Think of them like **different types of storage lockers**:
- **Memcached** = Simple storage locker (key-value only)
- **Redis** = Smart storage unit with compartments (data structures, persistence)

| Feature | Redis | Memcached |
|---------|-------|-----------|
| **Data Types** | Strings, Lists, Sets, Hashes, etc. | Strings only |
| **Persistence** | Yes (RDB, AOF) | No |
| **Replication** | Master-slave | No |
| **Transactions** | Yes | No |
| **Memory Usage** | Higher | Lower |
| **Performance** | Slightly slower | Slightly faster |

### Key Interview Points ⭐
1. "Redis offers rich data structures, Memcached is simpler"
2. "Redis provides persistence, Memcached is memory-only"
3. "Both are excellent for caching, Redis for more complex scenarios"
4. "Choose based on use case complexity and requirements"
5. "Redis can replace databases for certain use cases"

## 🔧 Redis Deep Dive

### 1. **Data Structures & Use Cases**

```javascript
const redis = require('redis');
const client = redis.createClient();

// STRING: Simple key-value caching
await client.set('user:1001', JSON.stringify({ name: 'John', email: 'john@example.com' }));
const user = JSON.parse(await client.get('user:1001'));

// SET expiration
await client.setex('session:abc123', 3600, 'user:1001'); // Expire in 1 hour

// HASH: User profile data
await client.hset('user:1001:profile', {
  name: 'John Doe',
  email: 'john@example.com',
  age: 30,
  city: 'New York'
});

// Get specific fields
const name = await client.hget('user:1001:profile', 'name');
const profile = await client.hgetall('user:1001:profile');

// LIST: Recent activity, queues
await client.lpush('user:1001:activities', 'logged_in', 'viewed_profile', 'updated_settings');
const recentActivities = await client.lrange('user:1001:activities', 0, 4); // Get 5 most recent

// SET: Unique tags, followers
await client.sadd('user:1001:tags', 'developer', 'javascript', 'nodejs');
const tags = await client.smembers('user:1001:tags');

// Check if user has specific tag
const hasTag = await client.sismember('user:1001:tags', 'javascript');

// SORTED SET: Leaderboard, rankings
await client.zadd('game:leaderboard', 1500, 'player1', 1200, 'player2', 1800, 'player3');
const topPlayers = await client.zrevrange('game:leaderboard', 0, 9); // Top 10
const playerRank = await client.zrevrank('game:leaderboard', 'player1');

// HYPERLOGLOG: Unique visitors count
await client.pfadd('page:home:unique_visitors', 'user1', 'user2', 'user3');
const uniqueCount = await client.pfcount('page:home:unique_visitors');
```

### 2. **Advanced Redis Patterns**

```javascript
// Distributed Locking
class RedisLock {
  constructor(client, key, ttl = 10000) {
    this.client = client;
    this.key = `lock:${key}`;
    this.ttl = ttl;
    this.value = Math.random().toString(36);
  }
  
  async acquire() {
    const result = await this.client.set(this.key, this.value, 'PX', this.ttl, 'NX');
    return result === 'OK';
  }
  
  async release() {
    const script = `
      if redis.call('get', KEYS[1]) == ARGV[1] then
        return redis.call('del', KEYS[1])
      else
        return 0
      end
    `;
    
    const result = await this.client.eval(script, 1, this.key, this.value);
    return result === 1;
  }
}

// Usage
const lock = new RedisLock(client, 'user:1001:update');
if (await lock.acquire()) {
  try {
    // Critical section - only one process can execute
    await updateUserProfile(userId, newData);
  } finally {
    await lock.release();
  }
} else {
  throw new Error('Could not acquire lock');
}

// Rate Limiting
class RedisRateLimiter {
  constructor(client) {
    this.client = client;
  }
  
  async isAllowed(key, limit, window) {
    const current = Date.now();
    const pipeline = this.client.pipeline();
    
    // Remove old entries
    pipeline.zremrangebyscore(key, 0, current - window);
    
    // Count current requests
    pipeline.zcard(key);
    
    // Add current request
    pipeline.zadd(key, current, `${current}-${Math.random()}`);
    
    // Set expiration
    pipeline.expire(key, Math.ceil(window / 1000));
    
    const results = await pipeline.exec();
    const count = results[1][1];
    
    return count < limit;
  }
}

// Usage: 100 requests per minute
const rateLimiter = new RedisRateLimiter(client);
const allowed = await rateLimiter.isAllowed('user:1001:api', 100, 60000);
```

### 3. **Redis Pub/Sub**

```javascript
// Publisher
class EventPublisher {
  constructor(client) {
    this.client = client;
  }
  
  async publishEvent(channel, event) {
    const message = JSON.stringify({
      ...event,
      timestamp: Date.now(),
      id: Math.random().toString(36)
    });
    
    await this.client.publish(channel, message);
  }
}

// Subscriber
class EventSubscriber {
  constructor(client) {
    this.subscriber = client.duplicate();
    this.handlers = new Map();
  }
  
  async subscribe(channel, handler) {
    this.handlers.set(channel, handler);
    await this.subscriber.subscribe(channel);
    
    this.subscriber.on('message', (channel, message) => {
      const handler = this.handlers.get(channel);
      if (handler) {
        const event = JSON.parse(message);
        handler(event);
      }
    });
  }
}

// Usage
const publisher = new EventPublisher(client);
const subscriber = new EventSubscriber(client);

await subscriber.subscribe('user-events', (event) => {
  console.log('User event received:', event);
  // Handle user event (send email, update cache, etc.)
});

await publisher.publishEvent('user-events', {
  type: 'user.created',
  userId: 1001,
  email: 'john@example.com'
});
```

## 🔧 Memcached Deep Dive

### 1. **Basic Operations**

```javascript
const memcached = require('memcached');
const client = new memcached('localhost:11211');

// Basic get/set operations
client.set('user:1001', JSON.stringify({ name: 'John', email: 'john@example.com' }), 3600, (err) => {
  if (err) console.error('Set failed:', err);
});

client.get('user:1001', (err, data) => {
  if (err) console.error('Get failed:', err);
  else {
    const user = JSON.parse(data);
    console.log('User:', user);
  }
});

// Multiple operations
client.getMulti(['user:1001', 'user:1002', 'user:1003'], (err, data) => {
  if (err) console.error('GetMulti failed:', err);
  else {
    console.log('Users:', data);
  }
});

// Atomic operations
client.cas('counter', 1, 0, 3600, (err, result) => {
  if (err) console.error('CAS failed:', err);
  else if (result === true) {
    console.log('Counter updated successfully');
  } else {
    console.log('Counter was modified by another client');
  }
});
```

### 2. **Memcached Best Practices**

```javascript
class MemcachedService {
  constructor(servers = ['localhost:11211']) {
    this.client = new memcached(servers, {
      maxKeySize: 250,        // Key size limit
      maxExpiration: 2592000, // 30 days max
      maxValue: 1048576,      // 1MB value limit
      poolSize: 10,           // Connection pool
      algorithm: 'crc32',     // Hash algorithm
      reconnect: 18000,       // Reconnect timeout
      timeout: 5000,          // Operation timeout
      retries: 5,             // Retry attempts
      retry: 30000            // Retry timeout
    });
  }
  
  async set(key, value, ttl = 3600) {
    return new Promise((resolve, reject) => {
      // Validate key length
      if (key.length > 250) {
        return reject(new Error('Key too long'));
      }
      
      // Serialize value
      const serialized = JSON.stringify(value);
      
      // Check value size
      if (Buffer.byteLength(serialized) > 1048576) {
        return reject(new Error('Value too large'));
      }
      
      this.client.set(key, serialized, ttl, (err) => {
        if (err) reject(err);
        else resolve(true);
      });
    });
  }
  
  async get(key) {
    return new Promise((resolve, reject) => {
      this.client.get(key, (err, data) => {
        if (err) reject(err);
        else if (data === undefined) resolve(null);
        else {
          try {
            resolve(JSON.parse(data));
          } catch (parseErr) {
            reject(parseErr);
          }
        }
      });
    });
  }
  
  async getMulti(keys) {
    return new Promise((resolve, reject) => {
      this.client.getMulti(keys, (err, data) => {
        if (err) reject(err);
        else {
          const result = {};
          for (const [key, value] of Object.entries(data)) {
            try {
              result[key] = JSON.parse(value);
            } catch (parseErr) {
              result[key] = null;
            }
          }
          resolve(result);
        }
      });
    });
  }
}
```

## 🚀 Performance Optimization

### 1. **Connection Pooling**

```javascript
// Redis connection pooling
class RedisPool {
  constructor(config) {
    this.pool = [];
    this.config = config;
    this.maxConnections = config.maxConnections || 10;
    this.currentConnections = 0;
  }
  
  async getConnection() {
    if (this.pool.length > 0) {
      return this.pool.pop();
    }
    
    if (this.currentConnections < this.maxConnections) {
      this.currentConnections++;
      return redis.createClient(this.config);
    }
    
    // Wait for available connection
    return new Promise((resolve) => {
      const checkPool = () => {
        if (this.pool.length > 0) {
          resolve(this.pool.pop());
        } else {
          setTimeout(checkPool, 10);
        }
      };
      checkPool();
    });
  }
  
  releaseConnection(connection) {
    this.pool.push(connection);
  }
}
```

### 2. **Caching Strategies**

```javascript
class SmartCache {
  constructor(redisClient, memcachedClient) {
    this.redis = redisClient;
    this.memcached = memcachedClient;
  }
  
  // Cache-aside pattern
  async get(key, fetcher, ttl = 3600) {
    // Try cache first
    let data = await this.redis.get(key);
    
    if (data === null) {
      // Cache miss - fetch from source
      data = await fetcher();
      
      // Store in cache
      await this.redis.setex(key, ttl, JSON.stringify(data));
    } else {
      data = JSON.parse(data);
    }
    
    return data;
  }
  
  // Write-through pattern
  async set(key, value, ttl = 3600) {
    // Write to cache and database simultaneously
    const [cacheResult, dbResult] = await Promise.all([
      this.redis.setex(key, ttl, JSON.stringify(value)),
      this.saveToDatabase(key, value)
    ]);
    
    return { cache: cacheResult, db: dbResult };
  }
  
  // Write-behind pattern
  async setAsync(key, value, ttl = 3600) {
    // Write to cache immediately
    await this.redis.setex(key, ttl, JSON.stringify(value));
    
    // Queue database write for later
    await this.redis.lpush('write_queue', JSON.stringify({ key, value }));
    
    return true;
  }
  
  // Multi-level caching
  async getMultiLevel(key, fetcher, ttl = 3600) {
    // Level 1: Try Memcached (fastest)
    let data = await this.memcached.get(key);
    
    if (data === null) {
      // Level 2: Try Redis (persistent)
      data = await this.redis.get(key);
      
      if (data !== null) {
        // Found in Redis, promote to Memcached
        await this.memcached.set(key, data, ttl);
        data = JSON.parse(data);
      } else {
        // Level 3: Fetch from source
        data = await fetcher();
        
        // Store in both caches
        const serialized = JSON.stringify(data);
        await Promise.all([
          this.redis.setex(key, ttl, serialized),
          this.memcached.set(key, serialized, ttl)
        ]);
      }
    } else {
      data = JSON.parse(data);
    }
    
    return data;
  }
}
```

## 💡 Common Interview Questions & Answers

### Q1: "When would you choose Redis over Memcached?"
**Answer:**
"Choose Redis when you need:
- **Rich data structures** (lists, sets, sorted sets)
- **Persistence** (data survives restarts)
- **Pub/Sub messaging**
- **Transactions and atomic operations**
- **Built-in replication**
- **Lua scripting**

Choose Memcached for:
- **Simple key-value caching**
- **Lower memory overhead**
- **Slightly better performance for simple operations**
- **Multi-threaded architecture**"

### Q2: "How do you handle cache invalidation?"
**Answer:**
"Several strategies:
1. **TTL-based** - Set expiration times
2. **Event-driven** - Invalidate on data changes
3. **Tag-based** - Group related cache entries
4. **Version-based** - Include version in cache key
5. **Manual** - Explicit invalidation APIs

I prefer event-driven invalidation for consistency with fallback TTL for safety."

### Q3: "How do you prevent cache stampede?"
**Answer:**
"Cache stampede occurs when many processes try to regenerate the same cache entry simultaneously. Solutions:
1. **Locking** - Use distributed locks
2. **Probabilistic early expiration** - Refresh before expiry
3. **Background refresh** - Async cache warming
4. **Semaphore pattern** - Limit concurrent regenerations
5. **Stale-while-revalidate** - Serve stale data while refreshing"

### Q4: "How do you monitor cache performance?"
**Answer:**
"Key metrics to monitor:
- **Hit rate** (cache hits / total requests)
- **Response time** (average cache operation time)
- **Memory usage** (used vs available memory)
- **Connection count** (active connections)
- **Error rate** (failed operations)
- **Eviction rate** (items removed due to memory pressure)

Use tools like Redis CLI, Memcached stats, or monitoring platforms like DataDog."

## 🔧 Production Best Practices

### 1. **High Availability Setup**

```javascript
// Redis Sentinel for high availability
const redis = require('redis');

const sentinelClient = redis.createClient({
  sentinels: [
    { host: 'sentinel1', port: 26379 },
    { host: 'sentinel2', port: 26379 },
    { host: 'sentinel3', port: 26379 }
  ],
  name: 'mymaster'
});

// Automatic failover handling
sentinelClient.on('sentinel:connection', () => {
  console.log('Connected to Redis Sentinel');
});

sentinelClient.on('sentinel:failover', (master) => {
  console.log('Redis failover detected:', master);
});
```

### 2. **Memory Management**

```javascript
// Memory optimization strategies
class CacheManager {
  constructor(client) {
    this.client = client;
    this.maxMemory = 1024 * 1024 * 1024; // 1GB
  }
  
  async checkMemoryUsage() {
    const info = await this.client.info('memory');
    const memoryUsed = parseInt(info.match(/used_memory:(\d+)/)[1]);
    const memoryRatio = memoryUsed / this.maxMemory;
    
    if (memoryRatio > 0.8) {
      console.warn('High memory usage:', memoryRatio * 100 + '%');
      await this.evictExpiredKeys();
    }
  }
  
  async evictExpiredKeys() {
    // Force eviction of expired keys
    await this.client.eval('return redis.call("EVAL", "for i=1,1000 do if redis.call(\\"RANDOMKEY\\") then redis.call(\\"EXPIRE\\", redis.call(\\"RANDOMKEY\\"), 0) end end return 1", 0)', 0);
  }
}
```

## 🔧 Quick Checklist

- [ ] Understand Redis vs Memcached trade-offs
- [ ] Know Redis data structures and use cases
- [ ] Implement proper error handling and timeouts
- [ ] Handle cache invalidation strategies
- [ ] Monitor cache performance metrics
- [ ] Set up high availability configurations

## 🎉 You're Redis & Memcached Ready!

**Key Message**: Redis and Memcached are **powerful caching solutions** that can dramatically improve application performance when used correctly!

**Interview Golden Rule**: Always discuss the specific use case requirements when choosing between Redis and Memcached, and explain caching strategies clearly.

Perfect! Now you can confidently discuss Redis and Memcached! 🔥 
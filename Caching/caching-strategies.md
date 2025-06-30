# Caching Strategies

## 🔍 Overview
Caching stores frequently accessed data in fast storage to improve performance and reduce database load.

## 📋 Types of Cache

### 1. In-Memory Cache
- **Location**: Same server as application
- **Examples**: Node.js Map, Redis on same server
- **Pros**: Fastest access, no network latency
- **Cons**: Limited by RAM, lost on restart

### 2. Distributed Cache  
- **Location**: Separate cache servers
- **Examples**: Redis, Memcached
- **Pros**: Shared across instances, scalable
- **Cons**: Network latency, additional infrastructure

## 🎯 Caching Patterns

### 1. Cache-Aside (Lazy Loading)
Application manages cache directly. Most common pattern.

```javascript
async getUser(userId) {
  // Try cache first
  let user = await cache.get(`user:${userId}`);
  if (user) return user;
  
  // Cache miss - get from database
  user = await database.getUser(userId);
  if (user) {
    await cache.set(`user:${userId}`, user, 300); // 5 min TTL
  }
  return user;
}
```

### 2. Write-Through
Cache updated immediately when database is updated.

```javascript
async updateUser(userId, data) {
  const user = await database.updateUser(userId, data);
  await cache.set(`user:${userId}`, user); // Update cache
  return user;
}
```

### 3. Write-Behind
Cache updated immediately, database updated asynchronously.

```javascript
async updateUser(userId, data) {
  await cache.set(`user:${userId}`, data); // Update cache first
  writeQueue.push({userId, data}); // Queue for DB write
  return data;
}
```

## 🔄 Cache Invalidation

### 1. TTL (Time-To-Live)
```javascript
// Different TTL based on data type
await cache.set('user:profile', user, 3600);    // 1 hour
await cache.set('session', session, 900);       // 15 minutes  
await cache.set('prices', prices, 300);         // 5 minutes
```

### 2. Event-Based Invalidation
```javascript
eventBus.on('user.updated', async (event) => {
  await cache.delete(`user:${event.userId}`);
});
```

## 🌊 Cache Warming
Pre-populate cache with frequently accessed data.

```javascript
async warmCache() {
  const popularUsers = await database.getPopularUsers();
  for (const user of popularUsers) {
    await cache.set(`user:${user.id}`, user, 3600);
  }
}
```

## 🚨 Common Problems

### 1. Cache Stampede
Multiple requests regenerate same expired cache.

**Solution**: Use locks
```javascript
const locks = new Map();

async getWithLock(key, loader) {
  if (locks.has(key)) {
    return await locks.get(key); // Wait for existing request
  }
  
  const promise = loader();
  locks.set(key, promise);
  
  try {
    return await promise;
  } finally {
    locks.delete(key);
  }
}
```

### 2. Hot Key Problem
One key gets too much traffic.

**Solution**: Local caching for hot keys
```javascript
// Cache popular data locally for short time
if (isHotKey(key)) {
  localCache.set(key, data, 30); // 30 seconds local cache
}
```

## ❓ Interview Questions

1. **Q: Cache-Aside vs Read-Through difference?**
   A: Cache-Aside: App manages cache. Read-Through: Cache loads data automatically.

2. **Q: How handle cache consistency?**
   A: Event-driven invalidation, TTL, or accept eventual consistency.

3. **Q: What is cache stampede?**
   A: Multiple requests regenerating expired cache. Use locks to prevent.

4. **Q: When use Write-Behind vs Write-Through?**
   A: Write-Behind for high throughput, Write-Through for consistency.

## 💡 Best Practices

- Design for cache misses
- Use appropriate TTL based on data change frequency
- Monitor cache hit rates
- Plan for cache failures
- Use consistent key naming conventions

---

**Key Takeaway**: Choose caching strategy based on consistency requirements, data patterns, and performance needs. 
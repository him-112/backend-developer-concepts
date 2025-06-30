# Performance Optimization Techniques

## 🔍 Overview
Performance optimization is crucial for scalable backend systems. This covers key optimization techniques and common bottlenecks.

## 🗄️ Database Performance

### Indexing Strategies
```sql
-- Check query performance
EXPLAIN SELECT * FROM users WHERE email = 'john@example.com';

-- Create indexes for frequently queried columns
CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_orders_user_id ON orders(user_id);

-- Composite index for multiple column queries
CREATE INDEX idx_orders_user_status ON orders(user_id, status);
```

### Query Optimization
```javascript
// BAD - N+1 Query Problem
async function getOrdersWithUsers() {
  const orders = await Order.findAll();
  for (const order of orders) {
    order.user = await User.findByPk(order.userId); // N+1 queries!
  }
  return orders;
}

// GOOD - Use joins
async function getOrdersWithUsers() {
  return await Order.findAll({
    include: [{ model: User }] // Single query with JOIN
  });
}
```

### Connection Pooling
```javascript
const pool = new Pool({
  host: 'localhost',
  user: 'postgres',
  max: 20, // Maximum connections
  min: 5,  // Minimum connections
  idleTimeoutMillis: 30000
});

async function getUserById(id) {
  const client = await pool.connect();
  try {
    const result = await client.query('SELECT * FROM users WHERE id = $1', [id]);
    return result.rows[0];
  } finally {
    client.release(); // Always release
  }
}
```

## ⚡ Application Performance

### Async Operations
```javascript
// BAD - Sequential processing
async function processOrdersSequential(orders) {
  const results = [];
  for (const order of orders) {
    const result = await processOrder(order); // Slow!
    results.push(result);
  }
  return results;
}

// GOOD - Parallel processing
async function processOrdersParallel(orders) {
  const promises = orders.map(order => processOrder(order));
  return await Promise.all(promises);
}
```

### Memory Management
```javascript
// BAD - Memory leak
class UserService {
  constructor() {
    this.cache = new Map(); // Never cleaned up!
  }
  
  async getUser(id) {
    if (!this.cache.has(id)) {
      this.cache.set(id, await this.fetchUser(id)); // Grows forever
    }
    return this.cache.get(id);
  }
}

// GOOD - Limited cache with cleanup
class UserService {
  constructor() {
    this.cache = new Map();
    this.maxSize = 1000;
  }
  
  async getUser(id) {
    if (!this.cache.has(id)) {
      if (this.cache.size >= this.maxSize) {
        const firstKey = this.cache.keys().next().value;
        this.cache.delete(firstKey);
      }
      this.cache.set(id, await this.fetchUser(id));
    }
    return this.cache.get(id);
  }
}
```

## 📊 Monitoring & Profiling

### Performance Middleware
```javascript
function performanceMiddleware(req, res, next) {
  const startTime = process.hrtime.bigint();
  
  res.on('finish', () => {
    const duration = Number(process.hrtime.bigint() - startTime) / 1000000;
    console.log(`${req.method} ${req.path} - ${duration.toFixed(2)}ms`);
    
    if (duration > 1000) {
      console.warn(`Slow request: ${req.method} ${req.path} - ${duration}ms`);
    }
  });
  
  next();
}
```

### Memory Monitoring
```javascript
function logMemoryUsage() {
  const usage = process.memoryUsage();
  console.log('Memory Usage:', {
    rss: `${Math.round(usage.rss / 1024 / 1024)} MB`,
    heapUsed: `${Math.round(usage.heapUsed / 1024 / 1024)} MB`
  });
}

setInterval(logMemoryUsage, 30000); // Every 30 seconds
```

## 🚀 Scaling Techniques

### Load Balancing
```javascript
class LoadBalancer {
  constructor(servers) {
    this.servers = servers;
    this.currentIndex = 0;
  }
  
  getNextServer() {
    const server = this.servers[this.currentIndex];
    this.currentIndex = (this.currentIndex + 1) % this.servers.length;
    return server;
  }
}
```

### Circuit Breaker
```javascript
class CircuitBreaker {
  constructor(failureThreshold = 5, resetTimeout = 60000) {
    this.failureThreshold = failureThreshold;
    this.resetTimeout = resetTimeout;
    this.state = 'CLOSED';
    this.failureCount = 0;
  }
  
  async execute(operation) {
    if (this.state === 'OPEN') {
      throw new Error('Circuit breaker is OPEN');
    }
    
    try {
      const result = await operation();
      this.onSuccess();
      return result;
    } catch (error) {
      this.onFailure();
      throw error;
    }
  }
  
  onSuccess() {
    this.failureCount = 0;
    this.state = 'CLOSED';
  }
  
  onFailure() {
    this.failureCount++;
    if (this.failureCount >= this.failureThreshold) {
      this.state = 'OPEN';
    }
  }
}
```

## 🔧 Common Bottlenecks

### 1. Blocking I/O
```javascript
// BAD - Blocking
const fs = require('fs');
const data = fs.readFileSync('file.txt'); // Blocks event loop!

// GOOD - Non-blocking
const fs = require('fs').promises;
const data = await fs.readFile('file.txt');
```

### 2. Inefficient Algorithms
```javascript
// BAD - O(n²)
function findDuplicates(arr1, arr2) {
  const duplicates = [];
  for (const item1 of arr1) {
    for (const item2 of arr2) {
      if (item1 === item2) duplicates.push(item1);
    }
  }
  return duplicates;
}

// GOOD - O(n)
function findDuplicates(arr1, arr2) {
  const set2 = new Set(arr2);
  return arr1.filter(item => set2.has(item));
}
```

## ❓ Interview Questions

1. **Q: How identify performance bottlenecks?**
   A: Use profiling tools, monitor metrics (response time, memory), analyze logs, implement APM.

2. **Q: What's N+1 query problem?**
   A: Making N additional queries in a loop. Fix with eager loading or joins.

3. **Q: How optimize database performance?**
   A: Proper indexing, query optimization, connection pooling, caching.

4. **Q: Horizontal vs vertical scaling?**
   A: Vertical adds power to existing machines, horizontal adds more machines.

5. **Q: How handle memory leaks?**
   A: Monitor usage, use profiling tools, clean up listeners, proper garbage collection.

## 💡 Best Practices

### Optimization Principles
- Measure before optimizing
- Focus on actual bottlenecks
- Use appropriate data structures
- Implement proper caching
- Monitor continuously

### Performance Checklist
- [ ] Response time tracking
- [ ] Memory usage monitoring  
- [ ] Database query optimization
- [ ] Proper indexing
- [ ] Connection pooling
- [ ] Async operations
- [ ] Error handling

---

**Key Takeaway**: Measure first, optimize the bottleneck, and monitor continuously. Performance is about balancing speed, resources, and maintainability. 
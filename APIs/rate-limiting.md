# Rate Limiting - Interview Guide ⚡

## 🎯 What You'll Learn
Master rate limiting to protect APIs from abuse and ensure fair usage!

## 🌟 Rate Limiting Basics

### What is Rate Limiting?
Think of it like a **bouncer at a club** - controls how many people can enter per hour to prevent overcrowding.

**Key Benefits:**
- Prevents API abuse
- Protects server resources  
- Ensures fair usage
- Prevents DDoS attacks

## 🛠️ Common Algorithms

### 1. **Token Bucket**
- Allows bursts of traffic
- Refills tokens at steady rate
- Good for APIs with variable load

```javascript
const rateLimit = require('express-rate-limit');

const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100, // 100 requests per window
  message: 'Too many requests'
});

app.use('/api/', limiter);
```

### 2. **Fixed Window**
- Simple counter reset at fixed intervals
- Easy to implement
- Can allow bursts at window boundaries

### 3. **Sliding Window**
- More accurate than fixed window
- Smooths out traffic spikes
- Higher memory usage

## 🎯 Implementation Strategies

### Multi-Level Limiting:
```javascript
// Global limit
const globalLimiter = rateLimit({ max: 1000 });

// Per-IP limit
const ipLimiter = rateLimit({ max: 100 });

// User-based limit
const userLimiter = async (req, res, next) => {
  const limit = req.user.plan === 'premium' ? 1000 : 100;
  // Check user-specific limit
  next();
};

app.use(globalLimiter);
app.use(ipLimiter);
app.use(userLimiter);
```

### Redis for Distributed Systems:
```javascript
const redis = require('redis');
const client = redis.createClient();

async function checkRateLimit(key, limit, windowSeconds) {
  const count = await client.incr(key);
  if (count === 1) {
    await client.expire(key, windowSeconds);
  }
  return count <= limit;
}
```

## 💡 Interview Questions & Answers

### Q: "What rate limiting algorithms do you know?"
**Answer:** "Token bucket for allowing bursts, fixed window for simplicity, sliding window for accuracy, and leaky bucket for smoothing traffic."

### Q: "How do you implement rate limiting in microservices?"
**Answer:** "Use Redis as centralized store, implement at API gateway level, and have per-service limits with circuit breakers."

### Q: "What happens when rate limit is exceeded?"
**Answer:** "Return 429 status code, include Retry-After header, log the incident, and implement exponential backoff on client side."

## 🚀 Best Practices

1. **Multiple layers** (global, per-IP, per-user)
2. **Proper error responses** (429 status, retry headers)
3. **Different limits per endpoint**
4. **Monitoring and alerting**
5. **Graceful degradation**

## 🔧 Quick Checklist

- [ ] Understand token bucket vs fixed window
- [ ] Know how to implement with Redis
- [ ] Explain multi-level rate limiting
- [ ] Describe proper error handling
- [ ] Discuss monitoring strategies

Perfect! Now you can confidently discuss rate limiting! ⚡ 
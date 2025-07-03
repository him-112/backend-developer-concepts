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

# Rate Limiting in APIs - Explanation

## 💡 **Rate Limiting Kya Hai?**

**Rate Limiting** ek technique hai jo APIs use karti hain taaki ek **user** ek defined **time period** mein kitni baar request bhej sakta hai, yeh control kar sake. Isse **server overload** nahi hota, aur **fair usage** maintain hoti hai.

---

## **Rate Limiting Kaise Kaam Karta Hai?**

- API me ek **limit** set hoti hai ki user ko **ek specific time window** (jaise 15 minutes, 1 hour, etc.) mein kitni requests bhejni ki permission hai.
- Agar user limit cross kar leta hai, toh **usse agle time window tak wait karna padta hai**.

---

## **Common Rate Limiting Strategies**

### 1️⃣ **Fixed Window:**  
- Ek fixed time window (jaise 15 minutes) ke andar ek fixed number of requests allowed hain (for example, 100 requests).
- Agar user 100 requests bhej deta hai, toh **next 15-minute window** ke start hone tak user ko request karne ki permission nahi milegi.

### 2️⃣ **Sliding Window:**  
- Ye approach thoda flexible hota hai. Time window **continuously move karta rehta hai**, jisme user ki last request ko dhyan me rakha jata hai.
- Agar user ne 100 requests last 15 minutes mein ki hain, toh usko aur requests tabhi milengi jab pehle ki requests time window se bahar ho jayein.

### 3️⃣ **Token Bucket:**  
- Har request ko ek **token** ke through process kiya jata hai.
- Jab tokens khatam ho jaate hain, user ko aur requests nahi bhejni milti jab tak naye tokens available na ho jayein.

### 4️⃣ **Leaky Bucket:**  
- Isme ek fixed **outflow rate** hota hai, jisme excess requests discard ho jati hain.

---

## **Agar 15-Minute Window Time Up Ho Jaye, Toh Kya Hota Hai?**

Maan lo, ek user ne **100 requests** bheje hain **15-minute window** mein, aur window ka limit bhi **100 requests** hai.

- **Window ka time khatam hone ke baad** (jaise 15 minutes ka time complete ho gaya), user ko **dobara requests bhejne ka access milta hai**.
- Agar **Fixed Window Rate Limiting** hai:
  - **Counter reset ho jata hai** aur user **phir se requests bhej sakta hai** jaise hi 15-minute window reset hota hai.
  - Jaise, agar **1:00 PM ko limit reach ho gayi thi**, toh user **1:15 PM ke baad** fir se requests bhej sakta hai.

- Agar **Sliding Window Rate Limiting** hai:
  - User tabhi request bhej sakta hai jab **unki purani requests ka time window khatam ho jaye**.

---

## **Example Scenario** (Fixed Window)

- **Limit:** 100 requests every **15 minutes**.
- **User ne 1:00 PM tak 100 requests bheji hain.**
- User **1:00 PM se 1:15 PM tak** request nahi bhej sakta.
- **1:15 PM par** window reset ho jata hai, aur **user fir se 100 requests bhej sakta hai.**

---

## **Agar User Exceeds Rate Limit?**

Agar user **limit cross kar leta hai**, toh:

- API **HTTP 429** status code return karta hai, jo hota hai **Too Many Requests**.
- Usse **error message** milta hai jaise:
   - `"Rate limit exceeded. Try again later."`
   - `"Too many requests. Please wait a while before trying again."`
- **Retry-After Header** bhi ho sakta hai, jo bataata hai ki user ko kitna time wait karna hai.

---

## **Summary**

- **Agar 15-minute window reset hota hai**: User fir se request bhej sakta hai, kyunki rate limit reset ho chuki hoti hai.
- **Agar limit exceed hota hai**: User ko HTTP 429 error milta hai aur wo **next window** tak wait karega ya **retry period** ke baad hi request kar sakta hai.

Isse **API abuse** nahi hota aur sab users ko fair chance milta hai!

---

Agar aur koi confusion ho ya **example** chahiye ho toh batana! 🚀
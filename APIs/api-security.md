# API Security - Interview Guide 🔒

## 🎯 What You'll Learn
Master API security to protect applications from common threats and vulnerabilities!

## 🌟 API Security in Simple Terms

### Why API Security Matters?
Think of APIs like **doors to your house**:
- **Unlocked door** = Insecure API (anyone can enter)
- **Door with good locks** = Secure API (only authorized people enter)
- **Security camera** = Logging and monitoring
- **Alarm system** = Rate limiting and threat detection

APIs are often the **biggest attack surface** in modern applications!

### Key Interview Points ⭐
**What interviewers want to hear:**
1. "Authentication vs Authorization distinction"
2. "Input validation and sanitization"
3. "Rate limiting and DDoS protection"
4. "HTTPS and data encryption"
5. "Common vulnerabilities like injection attacks"

## 🔐 Core Security Principles

### 1. **Authentication & Authorization**
*Who are you? What can you do?*

```javascript
// JWT Authentication
const jwt = require('jsonwebtoken');

function authenticateToken(req, res, next) {
  const token = req.headers['authorization']?.replace('Bearer ', '');
  
  if (!token) {
    return res.status(401).json({ error: 'Access token required' });
  }
  
  jwt.verify(token, process.env.JWT_SECRET, (err, user) => {
    if (err) {
      return res.status(403).json({ error: 'Invalid token' });
    }
    req.user = user;
    next();
  });
}

// Role-based authorization
function requireRole(allowedRoles) {
  return (req, res, next) => {
    if (!allowedRoles.includes(req.user.role)) {
      return res.status(403).json({ error: 'Insufficient permissions' });
    }
    next();
  };
}

// Usage
app.delete('/admin/users/:id', 
  authenticateToken, 
  requireRole(['admin']), 
  deleteUser
);
```

### 2. **Input Validation**
*Never trust user input!*

```javascript
const { body, validationResult } = require('express-validator');

// Prevent SQL injection
const validateUser = [
  body('email').isEmail().normalizeEmail(),
  body('password').isLength({ min: 8 }),
  (req, res, next) => {
    const errors = validationResult(req);
    if (!errors.isEmpty()) {
      return res.status(400).json({ errors: errors.array() });
    }
    next();
  }
];

// SQL Injection prevention
app.get('/users/:id', [
  param('id').isUUID().withMessage('Invalid user ID format')
], async (req, res) => {
  // Use parameterized queries
  const user = await db.query(
    'SELECT * FROM users WHERE id = $1', 
    [req.params.id]
  );
  res.json(user);
});

// XSS prevention
const helmet = require('helmet');
app.use(helmet());

// Sanitize input
const DOMPurify = require('dompurify');
const clean = DOMPurify.sanitize(userInput);
```

### 3. **Rate Limiting**
*Prevent abuse and maintain availability*

```javascript
const rateLimit = require('express-rate-limit');

const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100 // limit each IP to 100 requests per windowMs
});

app.use('/api/', limiter);

// Advanced: User-based rate limiting
const userLimiter = async (req, res, next) => {
  if (req.user) {
    const key = `rate_limit_${req.user.id}`;
    const requests = await redis.incr(key);
    
    if (requests === 1) {
      await redis.expire(key, 3600); // 1 hour window
    }
    
    if (requests > 1000) { // 1000 requests per hour per user
      return res.status(429).json({ error: 'Rate limit exceeded' });
    }
  }
  next();
};
```

## 🛡️ Common API Vulnerabilities & Prevention

### 🚨 Why These Vulnerabilities Exist
Most API security vulnerabilities share a common root cause: **trusting user input without proper validation**. When developers:
- Accept data from users without validation
- Use that data directly in database queries, HTML output, or system commands
- Don't implement proper authentication/authorization checks

The result is that attackers can manipulate your application to do things it wasn't designed to do!

### 1. **SQL Injection**
**What is SQL Injection?**
SQL Injection occurs when an attacker can insert malicious SQL code into your database queries through user input. This happens when you directly concatenate user input into SQL strings without proper sanitization.

**Real-world example:**
```javascript
// If user inputs: email = "user@test.com'; DROP TABLE users; --"
// This becomes: SELECT * FROM users WHERE email = 'user@test.com'; DROP TABLE users; --'
// Result: Your entire users table gets deleted! 💀
```

**How it works:**
1. Attacker finds an input field (like login form)
2. Instead of normal data, they input SQL commands
3. Your application executes these commands thinking they're part of the query
4. Attacker can read, modify, or delete your entire database

**Prevention:**
```javascript
// ❌ Vulnerable - String concatenation
const query = `SELECT * FROM users WHERE email = '${email}'`;

// ✅ Safe - Parameterized query
const query = 'SELECT * FROM users WHERE email = $1';
const result = await db.query(query, [email]);

// ✅ Safe - ORM/Query Builder
const user = await User.findOne({ where: { email } });
```

### 2. **NoSQL Injection**
**What is NoSQL Injection?**
Similar to SQL injection, but targets NoSQL databases like MongoDB. Attackers manipulate query objects to bypass authentication or access unauthorized data.

**Real-world example:**
```javascript
// If user sends: { "email": { "$ne": null } }
// This becomes: db.users.findOne({ email: { $ne: null } })
// Result: Returns the first user in the database, bypassing authentication! 🚨
```

**How it works:**
1. Attacker sends JSON objects with MongoDB operators (like `$ne`, `$gt`, `$where`)
2. Your application directly uses this object in database queries
3. The operators change the query logic
4. Attacker can bypass authentication or access unauthorized data

**Prevention:**
```javascript
// ❌ Vulnerable
const user = await User.findOne({ email: req.body.email });

// ✅ Safe with validation
const userSchema = {
  email: { type: 'string', format: 'email' }
};

if (!validate(req.body, userSchema)) {
  return res.status(400).json({ error: 'Invalid input' });
}

const user = await User.findOne({ email: req.body.email });
```

### 3. **Cross-Site Scripting (XSS)**
**What is XSS?**
XSS attacks occur when an attacker injects malicious scripts into web pages that are then executed in other users' browsers. The malicious script runs with the same permissions as the legitimate website.

**Real-world example:**
```javascript
// User submits a comment: "<script>alert('XSS Attack!');</script>"
// If you display this directly: <div>${userComment}</div>
// Result: The script executes in every visitor's browser! 🚨

// More dangerous example:
// "<script>fetch('/api/transfer', {method: 'POST', body: JSON.stringify({amount: 1000, to: 'attacker'})})</script>"
// This could transfer money from victim's account!
```

**How it works:**
1. Attacker finds an input field that displays user content (comments, profiles, etc.)
2. They submit malicious JavaScript code instead of normal text
3. Your application stores and displays this code without sanitization
4. When other users view the page, the malicious script executes in their browsers
5. The script can steal cookies, session tokens, or perform actions on behalf of the victim

**Types of XSS:**
- **Stored XSS**: Malicious script is stored in your database (most dangerous)
- **Reflected XSS**: Script is reflected back in URL parameters or form data
- **DOM-based XSS**: Script manipulates the DOM directly in the browser

**Prevention:**
```javascript
const helmet = require('helmet');
const DOMPurify = require('dompurify');

// Content Security Policy
app.use(helmet({
  contentSecurityPolicy: {
    directives: {
      defaultSrc: ["'self'"],
      scriptSrc: ["'self'", "'unsafe-inline'"],
      styleSrc: ["'self'", "'unsafe-inline'"],
      imgSrc: ["'self'", "data:", "https:"],
    },
  },
}));

// Sanitize HTML content
app.post('/content', (req, res) => {
  const clean = DOMPurify.sanitize(req.body.html);
  // Save clean content
});
```

### 4. **Cross-Site Request Forgery (CSRF)**
**What is CSRF?**
CSRF attacks trick users into performing unwanted actions on websites where they're authenticated. The attacker leverages the user's existing session to perform actions without their knowledge.

**Real-world example:**
```html
<!-- Attacker creates a malicious webpage with this form: -->
<form action="https://yourbank.com/transfer" method="POST" style="display:none;">
  <input name="amount" value="10000">
  <input name="to_account" value="attacker_account">
  <input type="submit" value="Transfer">
</form>
<script>document.forms[0].submit();</script>

<!-- When logged-in user visits this page, money gets transferred automatically! 💸 -->
```

**How it works:**
1. User logs into a legitimate website (like their bank)
2. Attacker tricks user into visiting a malicious webpage
3. The malicious page contains hidden forms that submit to the legitimate website
4. Since the user is still logged in, their browser automatically sends cookies
5. The legitimate website sees a valid request and processes it

**Key insight**: The attack works because browsers automatically include cookies with requests, even from malicious websites!

**Prevention:**
```javascript
const csrf = require('csurf');

// CSRF protection
const csrfProtection = csrf({ cookie: true });

app.use(csrfProtection);

app.get('/form', (req, res) => {
  res.render('form', { csrfToken: req.csrfToken() });
});

// For API-only applications, use SameSite cookies
app.use(session({
  cookie: {
    sameSite: 'strict',
    secure: true, // HTTPS only
    httpOnly: true
  }
}));
```

## 🔒 Advanced Security Measures

### 1. **API Keys & Scoping**
```javascript
// API Key with scopes
app.post('/api-keys', authenticateToken, async (req, res) => {
  const apiKey = {
    key: crypto.randomBytes(32).toString('hex'),
    userId: req.user.id,
    scopes: req.body.scopes || ['read'],
    rateLimit: req.body.rateLimit || 1000,
    expiresAt: new Date(Date.now() + 365 * 24 * 60 * 60 * 1000) // 1 year
  };
  
  await ApiKey.create(apiKey);
  res.json({ apiKey: apiKey.key });
});

// Validate API key and scopes
async function validateApiKey(requiredScope) {
  return async (req, res, next) => {
    const apiKey = req.headers['x-api-key'];
    
    if (!apiKey) {
      return res.status(401).json({ error: 'API key required' });
    }
    
    const keyData = await ApiKey.findOne({ 
      key: apiKey, 
      active: true,
      expiresAt: { $gt: new Date() }
    });
    
    if (!keyData) {
      return res.status(401).json({ error: 'Invalid or expired API key' });
    }
    
    if (!keyData.scopes.includes(requiredScope)) {
      return res.status(403).json({ error: `Scope '${requiredScope}' required` });
    }
    
    req.apiKey = keyData;
    next();
  };
}

// Usage
app.get('/api/data', validateApiKey('read'), getData);
app.post('/api/data', validateApiKey('write'), createData);
```

### 2. **Request Signing**
```javascript
const crypto = require('crypto');

// Generate signature
function generateSignature(payload, secret) {
  return crypto
    .createHmac('sha256', secret)
    .update(payload)
    .digest('hex');
}

// Verify request signature
function verifySignature(req, res, next) {
  const signature = req.headers['x-signature'];
  const timestamp = req.headers['x-timestamp'];
  
  // Prevent replay attacks - check timestamp
  const now = Math.floor(Date.now() / 1000);
  if (Math.abs(now - timestamp) > 300) { // 5 minutes
    return res.status(401).json({ error: 'Request too old' });
  }
  
  // Verify signature
  const payload = timestamp + JSON.stringify(req.body);
  const expectedSignature = generateSignature(payload, process.env.WEBHOOK_SECRET);
  
  if (!crypto.timingSafeEqual(Buffer.from(signature), Buffer.from(expectedSignature))) {
    return res.status(401).json({ error: 'Invalid signature' });
  }
  
  next();
}
```

### 3. **Content Type Validation**
```javascript
// Prevent content type confusion attacks
function validateContentType(allowedTypes) {
  return (req, res, next) => {
    const contentType = req.headers['content-type'];
    
    if (!contentType) {
      return res.status(400).json({ error: 'Content-Type header required' });
    }
    
    const baseType = contentType.split(';')[0].trim();
    if (!allowedTypes.includes(baseType)) {
      return res.status(415).json({ 
        error: `Unsupported Content-Type: ${baseType}` 
      });
    }
    
    next();
  };
}

// Usage
app.post('/api/json-data', 
  validateContentType(['application/json']), 
  handleJsonData
);

app.post('/api/upload', 
  validateContentType(['multipart/form-data']), 
  handleFileUpload
);
```

## 💡 Common Interview Questions & Perfect Answers

### Q1: "How do you prevent SQL injection in APIs?"
**Perfect Answer:**
"I prevent SQL injection using several methods:
1. **Parameterized queries** - Never concatenate user input into SQL strings
2. **Input validation** - Validate all inputs before processing
3. **ORM/Query builders** - Use tools that handle parameterization automatically
4. **Principle of least privilege** - Database users have minimal required permissions
5. **Input sanitization** - Remove or escape dangerous characters"

### Q2: "What's the difference between authentication and authorization?"
**Perfect Answer:**
"Authentication verifies WHO you are (like showing your ID), while authorization determines WHAT you can do (like having VIP access). Authentication comes first - you must prove your identity before we can check your permissions. For example, JWT tokens handle authentication, while role-based access control handles authorization."

### Q3: "How do you implement rate limiting?"
**Perfect Answer:**
"I implement rate limiting at multiple levels:
1. **API Gateway level** - Protect against DDoS attacks
2. **Application level** - Using middleware like express-rate-limit
3. **User-based limits** - Different limits for different user tiers
4. **Endpoint-specific limits** - Stricter limits for sensitive operations
5. **Progressive delays** - Slow down repeated requests instead of blocking"

### Q4: "How do you secure API keys?"
**Perfect Answer:**
"API key security involves:
1. **Never expose in client-side code** - Use environment variables
2. **Scope-based permissions** - Keys only access what they need
3. **Regular rotation** - Implement key expiration and renewal
4. **Rate limiting per key** - Prevent abuse
5. **Audit logging** - Track key usage
6. **Secure storage** - Hash keys in database like passwords"

## 🚀 Security Best Practices Checklist

### **Basic Security (Must Have):**
- [ ] HTTPS everywhere (TLS 1.2+)
- [ ] Input validation and sanitization
- [ ] Parameterized SQL queries
- [ ] Authentication on all protected endpoints
- [ ] Rate limiting implementation
- [ ] Error handling (don't leak sensitive info)
- [ ] Security headers (Helmet.js)

### **Advanced Security (Nice to Have):**
- [ ] Request signing for webhooks
- [ ] API versioning strategy
- [ ] Content Security Policy
- [ ] CORS configuration
- [ ] Audit logging and monitoring
- [ ] Penetration testing
- [ ] Security scanning in CI/CD

### **Monitoring & Response:**
- [ ] Failed authentication alerting
- [ ] Rate limit violation tracking
- [ ] Unusual traffic pattern detection
- [ ] Security incident response plan
- [ ] Regular security audits

## 🔧 Security Tools & Libraries

### **Node.js Security Stack:**
```javascript
// Essential security packages
const helmet = require('helmet');           // Security headers
const rateLimit = require('express-rate-limit'); // Rate limiting
const validator = require('validator');     // Input validation
const bcrypt = require('bcrypt');          // Password hashing
const jwt = require('jsonwebtoken');       // JWT tokens
const crypto = require('crypto');          // Cryptographic functions

// Security middleware setup
app.use(helmet());
app.use(rateLimit({ windowMs: 15 * 60 * 1000, max: 100 }));
app.use(express.json({ limit: '10mb' })); // Limit payload size
```

## 🎯 Interview Success Tips

### **Always Mention:**
- "Security is a process, not a product"
- "Defense in depth - multiple security layers"
- "Principle of least privilege"
- "Never trust user input"
- "Security by design, not afterthought"

### **Show Real-World Understanding:**
- "OWASP Top 10 awareness"
- "Security headers implementation"
- "Monitoring and alerting setup"
- "Incident response planning"
- "Regular security audits"

## 🎉 You're API Security Ready!

**Key Message**: API security is about **layered defense** - multiple security measures working together to protect your application and data!

**Interview Golden Rule**: Always emphasize that security is an ongoing process, not a one-time implementation. Show you understand both common vulnerabilities and prevention strategies.

Now you can confidently discuss API security in any interview! 🔒 
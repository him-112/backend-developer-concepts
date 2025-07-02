# Authentication Strategies - Interview Guide 🔐

## 🎯 What You'll Learn
Master different authentication strategies and know when to use each approach in modern applications!

## 🌟 Authentication Strategies Overview

### Authentication vs Authorization
- **Authentication**: "Who are you?" (Identity verification)
- **Authorization**: "What can you do?" (Permission checking)

Think of it like a **nightclub**:
- **Authentication** = Checking your ID at the door
- **Authorization** = VIP areas you're allowed to enter

## 🛠️ Common Authentication Strategies

### 1. **Session-Based Authentication** (Traditional)
*Think: Hotel room key that works until checkout*

```javascript
// Login process
app.post('/login', async (req, res) => {
  const { email, password } = req.body;
  
  // Verify credentials
  const user = await User.findOne({ email });
  const isValid = await bcrypt.compare(password, user.passwordHash);
  
  if (!isValid) {
    return res.status(401).json({ error: 'Invalid credentials' });
  }
  
  // Create session
  req.session.userId = user.id;
  req.session.user = {
    id: user.id,
    email: user.email,
    role: user.role
  };
  
  res.json({ message: 'Login successful' });
});

// Protected middleware
function requireAuth(req, res, next) {
  if (!req.session.userId) {
    return res.status(401).json({ error: 'Authentication required' });
  }
  next();
}

// Logout
app.post('/logout', (req, res) => {
  req.session.destroy();
  res.json({ message: 'Logged out' });
});
```

**Pros:**
- Simple to implement
- Server controls session state
- Automatic cleanup on server restart

**Cons:**
- Not stateless (harder to scale)
- Requires sticky sessions in load balancers
- CSRF vulnerable

### 2. **Token-Based Authentication (JWT)**
*Think: Airline boarding pass with your info*

```javascript
const jwt = require('jsonwebtoken');

// Login process
app.post('/login', async (req, res) => {
  const { email, password } = req.body;
  
  // Verify credentials
  const user = await User.findOne({ email });
  const isValid = await bcrypt.compare(password, user.passwordHash);
  
  if (!isValid) {
    return res.status(401).json({ error: 'Invalid credentials' });
  }
  
  // Create JWT token
  const token = jwt.sign(
    { 
      userId: user.id,
      email: user.email,
      role: user.role 
    },
    process.env.JWT_SECRET,
    { expiresIn: '24h' }
  );
  
  res.json({ token });
});

// Protected middleware
function requireAuth(req, res, next) {
  const token = req.headers.authorization?.replace('Bearer ', '');
  
  if (!token) {
    return res.status(401).json({ error: 'No token provided' });
  }
  
  try {
    const decoded = jwt.verify(token, process.env.JWT_SECRET);
    req.user = decoded;
    next();
  } catch (error) {
    res.status(401).json({ error: 'Invalid token' });
  }
}
```

**Pros:**
- Stateless (easy to scale)
- Works across microservices
- No server-side storage needed

**Cons:**
- Can't revoke tokens easily
- Larger payload than session IDs
- Token expiration management

### 3. **API Key Authentication**
*Think: Building access card*

```javascript
// Generate API key for user
app.post('/api-keys', requireAuth, async (req, res) => {
  const apiKey = crypto.randomBytes(32).toString('hex');
  
  await ApiKey.create({
    key: apiKey,
    userId: req.user.id,
    name: req.body.name,
    scopes: req.body.scopes || ['read']
  });
  
  res.json({ apiKey });
});

// API key middleware
async function requireApiKey(req, res, next) {
  const apiKey = req.headers['x-api-key'];
  
  if (!apiKey) {
    return res.status(401).json({ error: 'API key required' });
  }
  
  const keyRecord = await ApiKey.findOne({ 
    key: apiKey, 
    active: true 
  }).populate('user');
  
  if (!keyRecord) {
    return res.status(401).json({ error: 'Invalid API key' });
  }
  
  req.user = keyRecord.user;
  req.apiKey = keyRecord;
  next();
}

// Usage
app.get('/api/data', requireApiKey, (req, res) => {
  res.json({ data: 'Protected data' });
});
```

### 4. **Multi-Factor Authentication (MFA)**
*Think: Bank vault requiring multiple keys*

```javascript
const speakeasy = require('speakeasy');

// Enable TOTP for user
app.post('/mfa/enable', requireAuth, async (req, res) => {
  const secret = speakeasy.generateSecret({
    name: 'YourApp',
    account: req.user.email
  });
  
  // Save secret to user record (encrypted)
  await User.findByIdAndUpdate(req.user.id, {
    mfaSecret: encrypt(secret.base32),
    mfaEnabled: false  // Enable after verification
  });
  
  res.json({
    qrCodeUrl: secret.otpauth_url,
    secret: secret.base32
  });
});

// Verify TOTP and enable MFA
app.post('/mfa/verify', requireAuth, async (req, res) => {
  const { token } = req.body;
  const user = await User.findById(req.user.id);
  
  const secret = decrypt(user.mfaSecret);
  const verified = speakeasy.totp.verify({
    secret,
    encoding: 'base32',
    token,
    window: 2
  });
  
  if (!verified) {
    return res.status(400).json({ error: 'Invalid MFA token' });
  }
  
  await User.findByIdAndUpdate(req.user.id, {
    mfaEnabled: true
  });
  
  res.json({ message: 'MFA enabled successfully' });
});

// Login with MFA
app.post('/login', async (req, res) => {
  const { email, password, mfaToken } = req.body;
  
  const user = await User.findOne({ email });
  const isValid = await bcrypt.compare(password, user.passwordHash);
  
  if (!isValid) {
    return res.status(401).json({ error: 'Invalid credentials' });
  }
  
  // Check MFA if enabled
  if (user.mfaEnabled) {
    if (!mfaToken) {
      return res.status(200).json({ 
        requiresMfa: true,
        message: 'MFA token required' 
      });
    }
    
    const secret = decrypt(user.mfaSecret);
    const mfaValid = speakeasy.totp.verify({
      secret,
      encoding: 'base32',
      token: mfaToken,
      window: 2
    });
    
    if (!mfaValid) {
      return res.status(401).json({ error: 'Invalid MFA token' });
    }
  }
  
  // Generate JWT token
  const token = jwt.sign({ userId: user.id }, process.env.JWT_SECRET);
  res.json({ token });
});
```

### 5. **Biometric Authentication**
*Think: Fingerprint door locks*

```javascript
// WebAuthn (Browser API for biometrics)
const { generateRegistrationOptions, verifyRegistrationResponse } = require('@simplewebauthn/server');

// Start biometric registration
app.post('/auth/biometric/register/start', requireAuth, async (req, res) => {
  const options = generateRegistrationOptions({
    rpName: 'Your App',
    rpID: 'yourapp.com',
    userID: req.user.id,
    userName: req.user.email,
    userDisplayName: req.user.name,
    attestationType: 'none',
    authenticatorSelection: {
      authenticatorAttachment: 'platform', // Built-in biometrics
      userVerification: 'required',
    },
  });
  
  // Store challenge in session
  req.session.currentChallenge = options.challenge;
  
  res.json(options);
});

// Complete biometric registration
app.post('/auth/biometric/register/finish', requireAuth, async (req, res) => {
  const { body } = req;
  
  const verification = await verifyRegistrationResponse({
    response: body,
    expectedChallenge: req.session.currentChallenge,
    expectedOrigin: 'https://yourapp.com',
    expectedRPID: 'yourapp.com',
  });
  
  if (verification.verified) {
    // Save authenticator info
    await User.findByIdAndUpdate(req.user.id, {
      $push: {
        authenticators: {
          credentialID: verification.registrationInfo.credentialID,
          credentialPublicKey: verification.registrationInfo.credentialPublicKey,
          counter: verification.registrationInfo.counter,
        }
      }
    });
    
    res.json({ verified: true });
  } else {
    res.status(400).json({ error: 'Verification failed' });
  }
});
```

## 💡 Common Interview Questions & Perfect Answers

### Q1: "When would you use sessions vs JWT tokens?"
**Perfect Answer:**
"Use sessions for:
- Traditional web apps with server-side rendering
- When you need immediate token revocation
- When security is paramount (server controls state)

Use JWT for:
- Microservices architectures
- Mobile apps and SPAs
- When you need stateless scaling
- Cross-domain authentication"

### Q2: "How do you implement secure password storage?"
**Perfect Answer:**
"Never store plain text passwords. Use strong hashing algorithms:
1. **bcrypt** (most common) - adaptive, slow by design
2. **scrypt** or **Argon2** (newer, more secure)
3. Add **salt** to prevent rainbow table attacks
4. Use high **cost factor** (12+ for bcrypt)
5. Consider **pepper** (server-side secret) for extra security"

```javascript
const bcrypt = require('bcrypt');

// Registration
const saltRounds = 12;
const hashedPassword = await bcrypt.hash(password, saltRounds);

// Login verification
const isValid = await bcrypt.compare(password, hashedPassword);
```

### Q3: "How do you handle authentication in microservices?"
**Perfect Answer:**
"Several approaches:
1. **JWT tokens** - Self-contained, no shared state
2. **API Gateway** - Central auth, forwards user context
3. **Service mesh** - Infrastructure-level auth
4. **Shared auth service** - Centralized token validation

I prefer JWT with API Gateway for simplicity and performance."

### Q4: "What are the security considerations for authentication?"
**Perfect Answer:**
"Key security measures:
1. **HTTPS everywhere** - Encrypt in transit
2. **Strong password policies** - Complexity requirements
3. **Rate limiting** - Prevent brute force attacks
4. **Account lockout** - Temporary blocks after failures
5. **Session management** - Proper timeout and invalidation
6. **CSRF protection** - For session-based auth
7. **Input validation** - Sanitize all inputs"

## 🎯 Real-World Implementation Patterns

### Authentication Middleware Pattern:
```javascript
// Flexible auth middleware supporting multiple strategies
function authenticate(strategies = ['jwt']) {
  return async (req, res, next) => {
    let authenticated = false;
    let user = null;
    
    for (const strategy of strategies) {
      try {
        switch (strategy) {
          case 'jwt':
            user = await authenticateJWT(req);
            break;
          case 'apikey':
            user = await authenticateApiKey(req);
            break;
          case 'session':
            user = await authenticateSession(req);
            break;
        }
        
        if (user) {
          authenticated = true;
          break;
        }
      } catch (error) {
        // Try next strategy
        continue;
      }
    }
    
    if (!authenticated) {
      return res.status(401).json({ error: 'Authentication required' });
    }
    
    req.user = user;
    next();
  };
}

// Usage
app.get('/api/data', authenticate(['jwt', 'apikey']), (req, res) => {
  res.json({ data: 'protected' });
});
```

### Role-Based Authorization:
```javascript
function requireRole(roles) {
  return (req, res, next) => {
    if (!req.user) {
      return res.status(401).json({ error: 'Authentication required' });
    }
    
    if (!roles.includes(req.user.role)) {
      return res.status(403).json({ error: 'Insufficient permissions' });
    }
    
    next();
  };
}

// Usage
app.delete('/admin/users/:id', 
  authenticate(['jwt']), 
  requireRole(['admin', 'super_admin']), 
  deleteUser
);
```

## 🚀 Best Practices for Interviews

### **Always Mention Security:**
- "Hash passwords with bcrypt"
- "Use HTTPS in production"
- "Implement rate limiting"
- "Validate and sanitize inputs"
- "Consider MFA for sensitive operations"

### **Show Architecture Understanding:**
- "JWT for stateless scaling"
- "Sessions for traditional web apps"
- "API keys for service-to-service"
- "OAuth for third-party integration"

### **Demonstrate Real-World Knowledge:**
- "Password reset flows with time-limited tokens"
- "Remember me functionality with refresh tokens"
- "Social login integration"
- "Progressive authentication (guest → basic → premium)"

## 🔧 Quick Interview Checklist

- [ ] Explain sessions vs tokens vs API keys
- [ ] Describe secure password storage
- [ ] Know MFA implementation approaches
- [ ] Understand OAuth/OIDC basics
- [ ] Discuss authentication in microservices
- [ ] Explain common security vulnerabilities
- [ ] Know rate limiting and account lockout strategies

## 🎉 You're Authentication Ready!

**Key Message**: Choose the right authentication strategy based on your architecture, security requirements, and user experience needs!

**Interview Golden Rule**: Always discuss security implications and trade-offs when explaining authentication approaches.

Now you have a comprehensive understanding of authentication strategies! 🚀 
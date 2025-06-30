# Common Security Vulnerabilities

## 🔍 Overview
Understanding security vulnerabilities is crucial for backend developers. This covers the most common security issues and prevention strategies.

## 🎯 Top Security Vulnerabilities

### 1. SQL Injection
**Description**: Malicious SQL code inserted into queries.

**Vulnerable Code**:
```javascript
// BAD - Direct string concatenation
const query = `SELECT * FROM users WHERE id = ${userId}`;
```

**Prevention**:
```javascript
// GOOD - Parameterized queries
const query = 'SELECT * FROM users WHERE id = ?';
db.query(query, [userId], callback);

// Input validation
const { param } = require('express-validator');
app.get('/users/:id', 
  param('id').isInt(),
  (req, res) => { /* safe to proceed */ }
);
```

### 2. Cross-Site Scripting (XSS)
**Description**: Attackers inject malicious scripts into web applications.

**Prevention**:
```javascript
const validator = require('validator');

// Sanitize user input
const comment = validator.escape(req.body.comment);

// Security headers
app.use((req, res, next) => {
  res.setHeader('X-XSS-Protection', '1; mode=block');
  res.setHeader('Content-Security-Policy', "default-src 'self'");
  next();
});
```

### 3. Cross-Site Request Forgery (CSRF)
**Description**: Tricks users into performing unwanted actions.

**Prevention**:
```javascript
const csrf = require('csurf');

// CSRF protection
app.use(csrf({ cookie: true }));

// Double Submit Cookie pattern
app.use((req, res, next) => {
  if (['POST', 'PUT', 'DELETE'].includes(req.method)) {
    const token = req.headers['x-csrf-token'];
    const cookieToken = req.cookies['csrf-token'];
    
    if (!token || token !== cookieToken) {
      return res.status(403).json({ error: 'Invalid CSRF token' });
    }
  }
  next();
});
```

### 4. Weak Authentication
**Description**: Poor authentication mechanisms allow unauthorized access.

**Prevention**:
```javascript
const bcrypt = require('bcrypt');
const rateLimit = require('express-rate-limit');

// Password requirements
function validatePassword(password) {
  if (password.length < 8) return false;
  if (!/[a-z]/.test(password)) return false;
  if (!/[A-Z]/.test(password)) return false;
  if (!/\d/.test(password)) return false;
  if (!/[!@#$%^&*]/.test(password)) return false;
  return true;
}

// Rate limiting
const loginLimiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 5, // 5 attempts per window
  message: 'Too many login attempts'
});

// Secure password hashing
async function hashPassword(password) {
  return await bcrypt.hash(password, 12);
}
```

### 5. Insecure Direct Object References (IDOR)
**Description**: Accessing objects without proper authorization.

**Prevention**:
```javascript
// BAD - No authorization check
app.get('/users/:id/profile', (req, res) => {
  const profile = getUserProfile(req.params.id); // Anyone can access!
  res.json(profile);
});

// GOOD - Proper authorization
app.get('/users/:id/profile', authenticateToken, (req, res) => {
  const requestedUserId = req.params.id;
  const currentUserId = req.user.id;
  
  if (requestedUserId !== currentUserId && !req.user.isAdmin) {
    return res.status(403).json({ error: 'Access denied' });
  }
  
  const profile = getUserProfile(requestedUserId);
  res.json(profile);
});
```

## 🔐 Security Best Practices

### Input Validation
```javascript
const { body, validationResult } = require('express-validator');

const validateUser = [
  body('email').isEmail().normalizeEmail(),
  body('password').isLength({ min: 8 })
    .matches(/^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[@$!%*?&])/),
  body('age').isInt({ min: 0, max: 150 }).optional(),
  
  (req, res, next) => {
    const errors = validationResult(req);
    if (!errors.isEmpty()) {
      return res.status(400).json({ errors: errors.array() });
    }
    next();
  }
];
```

### Secure Headers
```javascript
const helmet = require('helmet');

app.use(helmet({
  contentSecurityPolicy: {
    directives: {
      defaultSrc: ["'self'"],
      scriptSrc: ["'self'"]
    }
  },
  hsts: {
    maxAge: 31536000,
    includeSubDomains: true
  }
}));

// Remove server info
app.disable('x-powered-by');
```

### Secure File Upload
```javascript
const multer = require('multer');

const upload = multer({
  limits: {
    fileSize: 5 * 1024 * 1024 // 5MB
  },
  fileFilter: (req, file, cb) => {
    const allowedTypes = ['image/jpeg', 'image/png'];
    if (allowedTypes.includes(file.mimetype)) {
      cb(null, true);
    } else {
      cb(new Error('Invalid file type'), false);
    }
  }
});
```

### API Rate Limiting
```javascript
const generalLimiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100, // requests per window
  message: 'Too many requests'
});

const authLimiter = rateLimit({
  windowMs: 15 * 60 * 1000,
  max: 5, // stricter for auth
  message: 'Too many auth attempts'
});

app.use('/api/', generalLimiter);
app.use('/auth/', authLimiter);
```

## ❓ Interview Questions

1. **Q: What is SQL injection and how to prevent it?**
   A: Malicious SQL in queries. Prevent with parameterized queries and input validation.

2. **Q: Types of XSS attacks?**
   A: Stored (persistent), Reflected (non-persistent), DOM-based. Prevent with input sanitization and CSP.

3. **Q: How implement secure authentication?**
   A: Strong passwords, bcrypt hashing, rate limiting, secure sessions, MFA.

4. **Q: What is CSRF?**
   A: Cross-Site Request Forgery. Prevent with CSRF tokens and SameSite cookies.

5. **Q: How secure API endpoints?**
   A: Authentication, authorization, input validation, rate limiting, HTTPS.

## 🛡️ Security Checklist

### Authentication
- [ ] Strong password policies  
- [ ] Password hashing (bcrypt ≥ 10 rounds)
- [ ] Rate limiting on auth endpoints
- [ ] Secure session management
- [ ] MFA implementation

### Input Validation
- [ ] Server-side validation
- [ ] Input sanitization
- [ ] Parameterized queries
- [ ] File upload restrictions

### Security Headers
- [ ] CSP (Content Security Policy)
- [ ] XSS Protection headers
- [ ] HSTS (HTTP Strict Transport Security)
- [ ] Remove server information

### General
- [ ] HTTPS everywhere
- [ ] Regular security updates
- [ ] Proper error handling
- [ ] Encrypt sensitive data

---

**Key Takeaway**: Security must be built into every layer. Use defense in depth and regularly audit for vulnerabilities. 
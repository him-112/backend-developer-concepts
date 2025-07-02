# Input Validation - Security Interview Guide 🛡️

## 🎯 What You'll Learn
Master input validation to protect applications from malicious attacks and data corruption!

## 🌟 Input Validation Fundamentals

### What is Input Validation?
Think of input validation like a **security checkpoint**:
- **Guard** = Validation rules
- **Visitors** = User input
- **ID Check** = Type validation
- **Metal detector** = Content scanning
- **Approved list** = Whitelist validation

No one gets through without proper verification!

### Key Interview Points ⭐
1. "First line of defense against attacks"
2. "Validate all inputs on both client and server"
3. "Use whitelist approach over blacklist"
4. "Sanitize and escape output properly"
5. "Defense in depth strategy"

## 🔒 Types of Input Validation

### 1. **Syntax Validation**
*Check format and structure*

```javascript
// Email validation
function validateEmail(email) {
  const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  
  if (!email || typeof email !== 'string') {
    return { valid: false, error: 'Email is required' };
  }
  
  if (email.length > 254) {
    return { valid: false, error: 'Email too long' };
  }
  
  if (!emailRegex.test(email)) {
    return { valid: false, error: 'Invalid email format' };
  }
  
  return { valid: true };
}

// Phone number validation
function validatePhone(phone) {
  // Remove all non-digits
  const cleaned = phone.replace(/\D/g, '');
  
  if (cleaned.length < 10 || cleaned.length > 15) {
    return { valid: false, error: 'Invalid phone number length' };
  }
  
  return { valid: true, cleaned };
}

// URL validation
function validateURL(url) {
  try {
    const parsed = new URL(url);
    
    // Only allow HTTP/HTTPS
    if (!['http:', 'https:'].includes(parsed.protocol)) {
      return { valid: false, error: 'Only HTTP/HTTPS URLs allowed' };
    }
    
    return { valid: true, parsed };
  } catch (error) {
    return { valid: false, error: 'Invalid URL format' };
  }
}
```

### 2. **Semantic Validation**
*Check meaning and business rules*

```javascript
// Age validation
function validateAge(age, context = 'general') {
  const numAge = parseInt(age);
  
  if (isNaN(numAge)) {
    return { valid: false, error: 'Age must be a number' };
  }
  
  const rules = {
    general: { min: 0, max: 150 },
    adult: { min: 18, max: 150 },
    senior: { min: 65, max: 150 },
    child: { min: 0, max: 17 }
  };
  
  const rule = rules[context] || rules.general;
  
  if (numAge < rule.min || numAge > rule.max) {
    return { 
      valid: false, 
      error: `Age must be between ${rule.min} and ${rule.max}` 
    };
  }
  
  return { valid: true, age: numAge };
}

// Date validation
function validateDate(dateStr, context = 'any') {
  const date = new Date(dateStr);
  
  if (isNaN(date.getTime())) {
    return { valid: false, error: 'Invalid date format' };
  }
  
  const now = new Date();
  
  switch(context) {
    case 'birth':
      if (date > now) {
        return { valid: false, error: 'Birth date cannot be in the future' };
      }
      if (date < new Date('1900-01-01')) {
        return { valid: false, error: 'Birth date too far in the past' };
      }
      break;
      
    case 'future':
      if (date <= now) {
        return { valid: false, error: 'Date must be in the future' };
      }
      break;
  }
  
  return { valid: true, date };
}
```

### 3. **Business Logic Validation**
*Check against business rules*

```javascript
class OrderValidator {
  static validateOrder(order) {
    const errors = [];
    
    // Validate customer
    if (!order.customerId) {
      errors.push('Customer ID is required');
    }
    
    // Validate items
    if (!order.items || order.items.length === 0) {
      errors.push('Order must contain at least one item');
    }
    
    if (order.items && order.items.length > 100) {
      errors.push('Order cannot contain more than 100 items');
    }
    
    // Validate quantities
    for (const item of order.items || []) {
      if (!Number.isInteger(item.quantity) || item.quantity <= 0) {
        errors.push(`Invalid quantity for item ${item.productId}`);
      }
      
      if (item.quantity > 999) {
        errors.push(`Quantity too high for item ${item.productId}`);
      }
    }
    
    // Validate total amount
    const calculatedTotal = this.calculateTotal(order.items);
    if (Math.abs(order.total - calculatedTotal) > 0.01) {
      errors.push('Order total does not match item prices');
    }
    
    return {
      valid: errors.length === 0,
      errors
    };
  }
  
  static calculateTotal(items) {
    return items.reduce((sum, item) => sum + (item.price * item.quantity), 0);
  }
}
```

## 🚨 Common Attack Prevention

### 1. **SQL Injection Prevention**

```javascript
// ❌ Vulnerable to SQL injection
function getUserBad(userId) {
  const query = `SELECT * FROM users WHERE id = ${userId}`;
  return db.query(query);
}

// ✅ Safe with parameterized queries
function getUserGood(userId) {
  // Validate input first
  const validation = validateInteger(userId, { min: 1, max: 999999999 });
  if (!validation.valid) {
    throw new Error(validation.error);
  }
  
  // Use parameterized query
  const query = 'SELECT * FROM users WHERE id = ?';
  return db.query(query, [validation.value]);
}

// Input sanitization for SQL
function sanitizeForSQL(input) {
  if (typeof input !== 'string') {
    return input;
  }
  
  // Remove or escape dangerous characters
  return input
    .replace(/'/g, "''")  // Escape single quotes
    .replace(/;/g, '')    // Remove semicolons
    .replace(/--/g, '')   // Remove SQL comments
    .replace(/\/\*/g, '') // Remove SQL comments
    .replace(/\*\//g, '');
}
```

### 2. **XSS Prevention**

```javascript
// HTML escaping
function escapeHTML(str) {
  const escapeMap = {
    '&': '&amp;',
    '<': '&lt;',
    '>': '&gt;',
    '"': '&quot;',
    "'": '&#x27;',
    '/': '&#x2F;'
  };
  
  return str.replace(/[&<>"'/]/g, (char) => escapeMap[char]);
}

// Comprehensive XSS prevention
class XSSProtection {
  static sanitizeInput(input) {
    if (typeof input !== 'string') {
      return input;
    }
    
    // Remove script tags
    let sanitized = input.replace(/<script\b[^<]*(?:(?!<\/script>)<[^<]*)*<\/script>/gi, '');
    
    // Remove javascript: URLs
    sanitized = sanitized.replace(/javascript:/gi, '');
    
    // Remove on* event handlers
    sanitized = sanitized.replace(/\s*on\w+\s*=\s*[^>]*/gi, '');
    
    // Escape remaining HTML
    return escapeHTML(sanitized);
  }
  
  static validateHTML(html, allowedTags = ['p', 'br', 'strong', 'em']) {
    const tagRegex = /<(\/?)([\w]+)[^>]*>/g;
    const matches = [...html.matchAll(tagRegex)];
    
    for (const match of matches) {
      const tag = match[2].toLowerCase();
      if (!allowedTags.includes(tag)) {
        return {
          valid: false,
          error: `Tag '${tag}' is not allowed`
        };
      }
    }
    
    return { valid: true };
  }
}
```

### 3. **Command Injection Prevention**

```javascript
// ❌ Vulnerable to command injection
function processFileBad(filename) {
  const command = `convert ${filename} output.jpg`;
  exec(command); // Dangerous!
}

// ✅ Safe input validation and sanitization
function processFileGood(filename) {
  // Validate filename
  const validation = validateFilename(filename);
  if (!validation.valid) {
    throw new Error(validation.error);
  }
  
  // Use safe command execution
  const command = 'convert';
  const args = [validation.sanitized, 'output.jpg'];
  execFile(command, args); // Safe with argument array
}

function validateFilename(filename) {
  if (!filename || typeof filename !== 'string') {
    return { valid: false, error: 'Filename is required' };
  }
  
  // Check length
  if (filename.length > 255) {
    return { valid: false, error: 'Filename too long' };
  }
  
  // Only allow safe characters
  const safeChars = /^[a-zA-Z0-9._-]+$/;
  if (!safeChars.test(filename)) {
    return { valid: false, error: 'Filename contains invalid characters' };
  }
  
  // Prevent directory traversal
  if (filename.includes('..') || filename.includes('/') || filename.includes('\\')) {
    return { valid: false, error: 'Directory traversal not allowed' };
  }
  
  return { valid: true, sanitized: filename };
}
```

## 🔧 Validation Frameworks

### 1. **Express.js with Joi**

```javascript
const Joi = require('joi');

const userSchema = Joi.object({
  email: Joi.string().email().required(),
  password: Joi.string().min(8).pattern(new RegExp('^(?=.*[a-z])(?=.*[A-Z])(?=.*[0-9])(?=.*[!@#\$%\^&\*])')).required(),
  age: Joi.number().integer().min(18).max(100),
  preferences: Joi.object({
    newsletter: Joi.boolean(),
    notifications: Joi.boolean()
  })
});

app.post('/users', async (req, res) => {
  try {
    const { error, value } = userSchema.validate(req.body);
    
    if (error) {
      return res.status(400).json({
        error: 'Validation failed',
        details: error.details.map(d => d.message)
      });
    }
    
    // Proceed with validated data
    const user = await createUser(value);
    res.status(201).json(user);
    
  } catch (err) {
    res.status(500).json({ error: 'Internal server error' });
  }
});
```

### 2. **Custom Validation Middleware**

```javascript
class ValidationMiddleware {
  static validate(schema) {
    return (req, res, next) => {
      const errors = [];
      
      for (const [field, rules] of Object.entries(schema)) {
        const value = req.body[field];
        const fieldErrors = this.validateField(field, value, rules);
        errors.push(...fieldErrors);
      }
      
      if (errors.length > 0) {
        return res.status(400).json({
          error: 'Validation failed',
          details: errors
        });
      }
      
      next();
    };
  }
  
  static validateField(field, value, rules) {
    const errors = [];
    
    // Required check
    if (rules.required && (value === undefined || value === null || value === '')) {
      errors.push(`${field} is required`);
      return errors; // Skip other validations if required field is missing
    }
    
    if (value === undefined || value === null) {
      return errors; // Skip validation for optional empty fields
    }
    
    // Type check
    if (rules.type && typeof value !== rules.type) {
      errors.push(`${field} must be of type ${rules.type}`);
    }
    
    // Length checks
    if (rules.minLength && value.length < rules.minLength) {
      errors.push(`${field} must be at least ${rules.minLength} characters`);
    }
    
    if (rules.maxLength && value.length > rules.maxLength) {
      errors.push(`${field} must be no more than ${rules.maxLength} characters`);
    }
    
    // Pattern check
    if (rules.pattern && !rules.pattern.test(value)) {
      errors.push(`${field} has invalid format`);
    }
    
    // Custom validator
    if (rules.custom) {
      const customResult = rules.custom(value);
      if (!customResult.valid) {
        errors.push(customResult.error);
      }
    }
    
    return errors;
  }
}

// Usage
const userValidationSchema = {
  email: {
    required: true,
    type: 'string',
    pattern: /^[^\s@]+@[^\s@]+\.[^\s@]+$/,
    maxLength: 254
  },
  password: {
    required: true,
    type: 'string',
    minLength: 8,
    custom: (value) => {
      const strongPattern = /^(?=.*[a-z])(?=.*[A-Z])(?=.*[0-9])(?=.*[!@#$%^&*])/;
      return strongPattern.test(value) 
        ? { valid: true }
        : { valid: false, error: 'Password must contain uppercase, lowercase, number, and special character' };
    }
  }
};

app.post('/users', ValidationMiddleware.validate(userValidationSchema), createUser);
```

## 💡 Common Interview Questions & Answers

### Q1: "What's the difference between validation and sanitization?"
**Answer:**
"Validation checks if input meets criteria and rejects invalid data. Sanitization modifies input to make it safe:
- **Validation**: 'Is this email format valid?' (accept/reject)
- **Sanitization**: 'Remove dangerous characters from this string' (modify)
- **Best practice**: Validate first, then sanitize if needed, and always escape output"

### Q2: "Why validate on both client and server side?"
**Answer:**
"Client-side validation provides immediate user feedback and better UX. Server-side validation provides security:
- **Client-side**: Can be bypassed, disabled, or manipulated
- **Server-side**: Cannot be bypassed, authoritative source of truth
- **Never trust client-side validation alone** for security decisions"

### Q3: "What's the difference between whitelist and blacklist validation?"
**Answer:**
"Whitelist allows only known good values, blacklist blocks known bad values:
- **Whitelist**: 'Only allow letters and numbers' (more secure)
- **Blacklist**: 'Block script tags and semicolons' (can be bypassed)
- **Whitelist is preferred** because it's impossible to anticipate all attack vectors"

### Q4: "How do you handle file upload validation?"
**Answer:**
"Multi-layered approach:
1. **File type validation** - Check MIME type and extension
2. **File size limits** - Prevent DoS attacks
3. **Content scanning** - Scan file contents, not just extension
4. **Filename sanitization** - Prevent directory traversal
5. **Virus scanning** - Use antivirus tools
6. **Storage isolation** - Store uploads outside web root"

## 🚀 Best Practices

### 1. **Comprehensive Validation Strategy**

```javascript
class InputValidator {
  static validateUserInput(data, context = 'web') {
    const result = {
      valid: true,
      errors: [],
      sanitized: {}
    };
    
    // Step 1: Type and structure validation
    const structureValidation = this.validateStructure(data);
    if (!structureValidation.valid) {
      return structureValidation;
    }
    
    // Step 2: Business rule validation
    const businessValidation = this.validateBusinessRules(data, context);
    if (!businessValidation.valid) {
      result.errors.push(...businessValidation.errors);
    }
    
    // Step 3: Security validation
    const securityValidation = this.validateSecurity(data);
    if (!securityValidation.valid) {
      result.errors.push(...securityValidation.errors);
    }
    
    // Step 4: Sanitization
    result.sanitized = this.sanitizeData(data);
    
    result.valid = result.errors.length === 0;
    return result;
  }
  
  static validateStructure(data) {
    // Check required fields, types, formats
    return { valid: true };
  }
  
  static validateBusinessRules(data, context) {
    // Check business logic constraints
    return { valid: true, errors: [] };
  }
  
  static validateSecurity(data) {
    // Check for common attack patterns
    return { valid: true, errors: [] };
  }
  
  static sanitizeData(data) {
    // Clean and escape data
    const sanitized = {};
    for (const [key, value] of Object.entries(data)) {
      if (typeof value === 'string') {
        sanitized[key] = escapeHTML(value.trim());
      } else {
        sanitized[key] = value;
      }
    }
    return sanitized;
  }
}
```

## 🔧 Quick Checklist

- [ ] Validate all inputs on server-side
- [ ] Use whitelist approach over blacklist
- [ ] Implement proper error handling
- [ ] Sanitize data before processing
- [ ] Escape output to prevent XSS
- [ ] Use parameterized queries for SQL
- [ ] Validate file uploads thoroughly

## 🎉 You're Input Validation Ready!

**Key Message**: Input validation is your **first line of defense** against attacks and data corruption!

**Interview Golden Rule**: Always emphasize that validation must happen on the server-side and explain the difference between validation, sanitization, and output escaping.

Perfect! Now you can confidently discuss input validation security! 🛡️ 
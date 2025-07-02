# Encryption & Hashing - Security Interview Guide 🔐

## 🎯 What You'll Learn
Master encryption and hashing to protect sensitive data and secure communications!

## 🌟 Cryptography Fundamentals

### What is Encryption vs Hashing?
Think of them like different **security safes**:
- **Encryption** = Safe with a key (reversible)
- **Hashing** = Paper shredder (irreversible)
- **Salt** = Adding random data to prevent rainbow table attacks
- **Digital signatures** = Wax seal for authenticity

### Key Interview Points ⭐
1. "Encryption is reversible, hashing is one-way"
2. "Use symmetric encryption for data, asymmetric for key exchange"
3. "Always salt and hash passwords"
4. "Different algorithms for different use cases"
5. "Key management is critical for security"

## 🔒 Hashing & Password Security

### 1. **Password Hashing**

```javascript
const bcrypt = require('bcrypt');
const crypto = require('crypto');

// ❌ Never do this - plaintext storage
function badPasswordStorage(password) {
  return password; // Terrible!
}

// ❌ Also bad - simple hash without salt
function badHashPassword(password) {
  return crypto.createHash('sha256').update(password).digest('hex');
}

// ✅ Proper password hashing with bcrypt
async function hashPassword(password) {
  const saltRounds = 12; // Higher = more secure but slower
  
  try {
    const hashedPassword = await bcrypt.hash(password, saltRounds);
    return hashedPassword;
  } catch (error) {
    throw new Error('Password hashing failed');
  }
}

// ✅ Password verification
async function verifyPassword(password, hashedPassword) {
  try {
    const isMatch = await bcrypt.compare(password, hashedPassword);
    return isMatch;
  } catch (error) {
    throw new Error('Password verification failed');
  }
}

// Usage example
async function registerUser(email, password) {
  // Validate password strength first
  if (!isStrongPassword(password)) {
    throw new Error('Password too weak');
  }
  
  const hashedPassword = await hashPassword(password);
  
  const user = await db.users.create({
    email,
    password: hashedPassword,
    createdAt: new Date()
  });
  
  return { id: user.id, email: user.email };
}

function isStrongPassword(password) {
  // At least 8 chars, uppercase, lowercase, number, special char
  const strongRegex = /^(?=.*[a-z])(?=.*[A-Z])(?=.*[0-9])(?=.*[!@#$%^&*])/;
  return password.length >= 8 && strongRegex.test(password);
}
```

### 2. **Custom Salt Implementation**

```javascript
// Manual salt implementation (bcrypt handles this automatically)
class PasswordHasher {
  static generateSalt(length = 16) {
    return crypto.randomBytes(length).toString('hex');
  }
  
  static hashWithSalt(password, salt) {
    return crypto.pbkdf2Sync(password, salt, 100000, 64, 'sha256').toString('hex');
  }
  
  static hashPassword(password) {
    const salt = this.generateSalt();
    const hash = this.hashWithSalt(password, salt);
    
    // Store salt with hash (separated by $)
    return `${salt}$${hash}`;
  }
  
  static verifyPassword(password, storedHash) {
    const [salt, hash] = storedHash.split('$');
    const newHash = this.hashWithSalt(password, salt);
    
    // Use constant-time comparison to prevent timing attacks
    return this.constantTimeCompare(hash, newHash);
  }
  
  static constantTimeCompare(a, b) {
    if (a.length !== b.length) {
      return false;
    }
    
    let result = 0;
    for (let i = 0; i < a.length; i++) {
      result |= a.charCodeAt(i) ^ b.charCodeAt(i);
    }
    
    return result === 0;
  }
}
```

## 🔐 Symmetric Encryption

### AES Encryption Implementation

```javascript
const crypto = require('crypto');

class AESEncryption {
  constructor() {
    this.algorithm = 'aes-256-gcm';
    this.keyLength = 32; // 256 bits
    this.ivLength = 16;  // 128 bits
  }
  
  // Generate a random encryption key
  generateKey() {
    return crypto.randomBytes(this.keyLength);
  }
  
  // Encrypt data
  encrypt(plaintext, key) {
    const iv = crypto.randomBytes(this.ivLength);
    const cipher = crypto.createCipher(this.algorithm, key, iv);
    
    let encrypted = cipher.update(plaintext, 'utf8', 'hex');
    encrypted += cipher.final('hex');
    
    // Get authentication tag (for GCM mode)
    const authTag = cipher.getAuthTag();
    
    // Return IV + authTag + encrypted data
    return {
      iv: iv.toString('hex'),
      authTag: authTag.toString('hex'),
      encrypted: encrypted
    };
  }
  
  // Decrypt data
  decrypt(encryptedData, key) {
    const { iv, authTag, encrypted } = encryptedData;
    
    const decipher = crypto.createDecipher(
      this.algorithm, 
      key, 
      Buffer.from(iv, 'hex')
    );
    
    decipher.setAuthTag(Buffer.from(authTag, 'hex'));
    
    let decrypted = decipher.update(encrypted, 'hex', 'utf8');
    decrypted += decipher.final('utf8');
    
    return decrypted;
  }
  
  // Encrypt sensitive user data
  static async encryptUserData(userData, masterKey) {
    const aes = new AESEncryption();
    
    const sensitiveFields = ['ssn', 'creditCard', 'bankAccount'];
    const encrypted = { ...userData };
    
    for (const field of sensitiveFields) {
      if (userData[field]) {
        encrypted[field] = aes.encrypt(userData[field], masterKey);
      }
    }
    
    return encrypted;
  }
}

// Usage for database encryption
class SecureUserService {
  constructor(masterKey) {
    this.masterKey = masterKey;
    this.aes = new AESEncryption();
  }
  
  async saveUser(userData) {
    // Encrypt sensitive data before storing
    const encryptedData = await AESEncryption.encryptUserData(
      userData, 
      this.masterKey
    );
    
    return await db.users.create(encryptedData);
  }
  
  async getUser(userId) {
    const user = await db.users.findById(userId);
    
    // Decrypt sensitive data after retrieval
    const sensitiveFields = ['ssn', 'creditCard', 'bankAccount'];
    
    for (const field of sensitiveFields) {
      if (user[field] && typeof user[field] === 'object') {
        user[field] = this.aes.decrypt(user[field], this.masterKey);
      }
    }
    
    return user;
  }
}
```

## 🔑 Asymmetric Encryption (RSA)

```javascript
const crypto = require('crypto');

class RSAEncryption {
  // Generate RSA key pair
  static generateKeyPair() {
    return crypto.generateKeyPairSync('rsa', {
      modulusLength: 2048,
      publicKeyEncoding: {
        type: 'spki',
        format: 'pem'
      },
      privateKeyEncoding: {
        type: 'pkcs8',
        format: 'pem',
        cipher: 'aes-256-cbc',
        passphrase: process.env.RSA_PASSPHRASE
      }
    });
  }
  
  // Encrypt with public key (for others to send you data)
  static encryptWithPublicKey(data, publicKey) {
    const buffer = Buffer.from(data, 'utf8');
    const encrypted = crypto.publicEncrypt(publicKey, buffer);
    return encrypted.toString('base64');
  }
  
  // Decrypt with private key (to read data sent to you)
  static decryptWithPrivateKey(encryptedData, privateKey, passphrase) {
    const buffer = Buffer.from(encryptedData, 'base64');
    const decrypted = crypto.privateDecrypt({
      key: privateKey,
      passphrase: passphrase
    }, buffer);
    return decrypted.toString('utf8');
  }
  
  // Sign data with private key (to prove authenticity)
  static signData(data, privateKey, passphrase) {
    const sign = crypto.createSign('RSA-SHA256');
    sign.update(data);
    const signature = sign.sign({
      key: privateKey,
      passphrase: passphrase
    }, 'base64');
    return signature;
  }
  
  // Verify signature with public key
  static verifySignature(data, signature, publicKey) {
    const verify = crypto.createVerify('RSA-SHA256');
    verify.update(data);
    return verify.verify(publicKey, signature, 'base64');
  }
}

// Example: Secure API communication
class SecureAPIClient {
  constructor(serverPublicKey, clientPrivateKey, passphrase) {
    this.serverPublicKey = serverPublicKey;
    this.clientPrivateKey = clientPrivateKey;
    this.passphrase = passphrase;
  }
  
  async sendSecureData(data) {
    // 1. Encrypt data with server's public key
    const encryptedData = RSAEncryption.encryptWithPublicKey(
      JSON.stringify(data), 
      this.serverPublicKey
    );
    
    // 2. Sign the encrypted data with our private key
    const signature = RSAEncryption.signData(
      encryptedData, 
      this.clientPrivateKey, 
      this.passphrase
    );
    
    // 3. Send both encrypted data and signature
    return {
      data: encryptedData,
      signature: signature
    };
  }
}
```

## 🛡️ JWT & Token Security

```javascript
const jwt = require('jsonwebtoken');

class JWTSecurity {
  constructor(secretKey) {
    this.secretKey = secretKey;
    this.algorithm = 'HS256';
  }
  
  // Create JWT token
  createToken(payload, expiresIn = '1h') {
    const options = {
      algorithm: this.algorithm,
      expiresIn: expiresIn,
      issuer: 'your-app',
      audience: 'your-users'
    };
    
    return jwt.sign(payload, this.secretKey, options);
  }
  
  // Verify JWT token
  verifyToken(token) {
    try {
      const decoded = jwt.verify(token, this.secretKey, {
        algorithm: this.algorithm,
        issuer: 'your-app',
        audience: 'your-users'
      });
      
      return { valid: true, payload: decoded };
    } catch (error) {
      return { valid: false, error: error.message };
    }
  }
  
  // Refresh token mechanism
  static createRefreshToken() {
    return crypto.randomBytes(64).toString('hex');
  }
  
  // Token blacklist for logout
  static blacklistedTokens = new Set();
  
  static blacklistToken(tokenId) {
    this.blacklistedTokens.add(tokenId);
  }
  
  static isTokenBlacklisted(tokenId) {
    return this.blacklistedTokens.has(tokenId);
  }
}

// Secure authentication service
class AuthService {
  constructor() {
    this.jwtSecurity = new JWTSecurity(process.env.JWT_SECRET);
  }
  
  async login(email, password) {
    // 1. Verify user credentials
    const user = await db.users.findOne({ email });
    if (!user || !await verifyPassword(password, user.password)) {
      throw new Error('Invalid credentials');
    }
    
    // 2. Create tokens
    const tokenId = crypto.randomUUID();
    const accessToken = this.jwtSecurity.createToken({
      userId: user.id,
      email: user.email,
      jti: tokenId // JWT ID for blacklisting
    }, '15m');
    
    const refreshToken = JWTSecurity.createRefreshToken();
    
    // 3. Store refresh token in database
    await db.refreshTokens.create({
      token: refreshToken,
      userId: user.id,
      expiresAt: new Date(Date.now() + 7 * 24 * 60 * 60 * 1000) // 7 days
    });
    
    return {
      accessToken,
      refreshToken,
      expiresIn: 900 // 15 minutes
    };
  }
  
  async logout(token) {
    const verification = this.jwtSecurity.verifyToken(token);
    if (verification.valid) {
      // Blacklist the token
      JWTSecurity.blacklistToken(verification.payload.jti);
    }
  }
}
```

## 🔍 Cryptographic Best Practices

### 1. **Key Management**

```javascript
class KeyManager {
  constructor() {
    this.keys = new Map();
  }
  
  // Key rotation
  async rotateKey(keyId) {
    const oldKey = this.keys.get(keyId);
    const newKey = crypto.randomBytes(32);
    
    // Store both keys temporarily during rotation
    this.keys.set(`${keyId}_old`, oldKey);
    this.keys.set(keyId, newKey);
    
    // Re-encrypt data with new key (in background)
    this.scheduleReEncryption(keyId);
    
    return newKey;
  }
  
  // Secure key derivation
  static deriveKey(password, salt, iterations = 100000) {
    return crypto.pbkdf2Sync(password, salt, iterations, 32, 'sha256');
  }
  
  // Key splitting for enhanced security
  static splitKey(key, shares = 3, threshold = 2) {
    // Shamir's Secret Sharing implementation
    // This is a simplified version
    const shares = [];
    
    for (let i = 1; i <= shares; i++) {
      const share = {
        id: i,
        value: crypto.randomBytes(32) // Simplified
      };
      shares.push(share);
    }
    
    return shares;
  }
}
```

## 💡 Common Interview Questions & Answers

### Q1: "What's the difference between encryption and hashing?"
**Answer:**
"Encryption is reversible with the right key, hashing is one-way:
- **Encryption**: Protects data in transit/storage, can be decrypted
- **Hashing**: Verifies data integrity, cannot be reversed
- **Use encryption for**: Sensitive data storage, secure communication
- **Use hashing for**: Passwords, data integrity verification, digital signatures"

### Q2: "Why do we salt passwords?"
**Answer:**
"Salt prevents rainbow table attacks and makes identical passwords have different hashes:
- **Without salt**: Same password = same hash (vulnerable to rainbow tables)
- **With salt**: Same password + unique salt = different hash
- **Additional benefit**: Prevents timing attacks on password comparison
- **Best practice**: Use cryptographically secure random salt for each password"

### Q3: "When would you use symmetric vs asymmetric encryption?"
**Answer:**
"Choose based on use case:
- **Symmetric (AES)**: Fast, same key for encrypt/decrypt. Use for bulk data encryption
- **Asymmetric (RSA)**: Slower, public/private key pair. Use for key exchange, digital signatures
- **Hybrid approach**: Use RSA to exchange AES keys, then AES for actual data
- **Real world**: HTTPS uses both - RSA for handshake, AES for data transfer"

### Q4: "How do you secure JWT tokens?"
**Answer:**
"Multiple layers of JWT security:
1. **Strong secret**: Use cryptographically secure random key
2. **Short expiration**: 15-30 minutes for access tokens
3. **Secure storage**: HttpOnly cookies, not localStorage
4. **Token blacklisting**: Track revoked tokens
5. **Refresh tokens**: Separate long-lived tokens for renewal
6. **HTTPS only**: Never send tokens over HTTP"

## 🚀 Advanced Security Patterns

### 1. **Envelope Encryption**

```javascript
class EnvelopeEncryption {
  constructor(kmsClient) {
    this.kms = kmsClient;
  }
  
  async encryptData(plaintext) {
    // 1. Generate data encryption key (DEK)
    const dek = crypto.randomBytes(32);
    
    // 2. Encrypt data with DEK
    const aes = new AESEncryption();
    const encryptedData = aes.encrypt(plaintext, dek);
    
    // 3. Encrypt DEK with key encryption key (KEK) from KMS
    const encryptedDEK = await this.kms.encrypt(dek);
    
    return {
      encryptedData,
      encryptedDEK
    };
  }
  
  async decryptData(encryptedPackage) {
    // 1. Decrypt DEK using KMS
    const dek = await this.kms.decrypt(encryptedPackage.encryptedDEK);
    
    // 2. Decrypt data using DEK
    const aes = new AESEncryption();
    const plaintext = aes.decrypt(encryptedPackage.encryptedData, dek);
    
    return plaintext;
  }
}
```

## 🔧 Quick Checklist

- [ ] Use bcrypt or similar for password hashing
- [ ] Always salt passwords with unique salts
- [ ] Use AES-256 for symmetric encryption
- [ ] Implement proper key management and rotation
- [ ] Use HTTPS for all sensitive communications
- [ ] Secure JWT tokens with short expiration
- [ ] Implement proper random number generation

## 🎉 You're Encryption & Hashing Ready!

**Key Message**: Proper cryptography is **essential for protecting sensitive data** and securing communications!

**Interview Golden Rule**: Always discuss the trade-offs between security, performance, and usability when explaining cryptographic choices.

Perfect! Now you can confidently discuss encryption and hashing! 🔐

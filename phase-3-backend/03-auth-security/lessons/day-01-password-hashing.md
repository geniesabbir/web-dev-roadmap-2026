# Day 1: Password Hashing & Security

## Introduction

Storing passwords securely is one of the most critical aspects of web application security. A single data breach exposing plain-text passwords can destroy user trust and expose them to identity theft across multiple services. Today, you'll learn why password hashing is essential and how to implement it properly using bcrypt.

## Learning Objectives

By the end of this lesson, you will be able to:
- Understand why plain-text password storage is dangerous
- Explain how cryptographic hashing works
- Implement secure password hashing with bcrypt
- Compare passwords securely during authentication
- Apply password security best practices

---

## Why Password Hashing Matters

### The Problem with Plain-Text Passwords

```javascript
// NEVER DO THIS - Storing plain-text passwords
const users = [
  { email: 'user@example.com', password: 'MySecret123' }
];

// If your database is compromised, attackers get:
// 1. Access to all user accounts on your platform
// 2. Access to users' accounts on OTHER platforms (password reuse)
// 3. Insight into password patterns for targeted attacks
```

### Real-World Breaches

| Company | Year | Passwords Exposed | Storage Method |
|---------|------|-------------------|----------------|
| Adobe | 2013 | 153 million | Encrypted (not hashed) |
| LinkedIn | 2012 | 117 million | Unsalted SHA-1 |
| RockYou | 2009 | 32 million | Plain text |

---

## Understanding Hashing

### What is Hashing?

Hashing is a **one-way** mathematical function that converts input into a fixed-length string:

```
Input: "password123"
Hash:  "ef92b778bafe771e89245b89ecbc08a44a4e166c06659911881f383d4473e94f"

Input: "password124" (one character different)
Hash:  "b5af0cb21d9f16c3a0c2e8f447bb8d41c9c69f0d35a7f01b1e2d1c3b4a5d6e7f"
```

### Key Properties of Cryptographic Hashes

```javascript
// 1. Deterministic - Same input always produces same output
hash('password') === hash('password')  // true

// 2. One-way - Cannot reverse the hash to get the original
unhash('ef92b778...') // Impossible!

// 3. Avalanche effect - Small changes create completely different hashes
hash('password')  // "5e884898da28..."
hash('Password')  // "8c6976e5b541..."  // Completely different!

// 4. Collision resistant - Extremely hard to find two inputs with same hash
```

---

## Why Simple Hashing Isn't Enough

### The Rainbow Table Attack

Attackers pre-compute hashes for common passwords:

```javascript
// Attacker's rainbow table (pre-computed)
const rainbowTable = {
  '5e884898da28047151d0e56f8dc6292773603d0d6aabbdd62a11ef721d1542d8': 'password',
  'ef92b778bafe771e89245b89ecbc08a44a4e166c06659911881f383d4473e94f': 'password123',
  '6ca13d52ca70c883e0f0bb101e425a89e8624de51db2d2392593af6a84118090': 'abc123',
  // ... millions more entries
};

// Attacker steals your database
const stolenHash = 'ef92b778bafe...';
console.log(rainbowTable[stolenHash]); // "password123" - Cracked instantly!
```

### The Solution: Salting

A **salt** is random data added to the password before hashing:

```javascript
// Without salt - Same passwords = Same hashes
hash('password123') === hash('password123')  // true

// With salt - Same passwords = Different hashes
const salt1 = 'x7Hk2pQr';
const salt2 = 'mN9bLwYz';

hash('password123' + salt1)  // "a3f8d2e1..."
hash('password123' + salt2)  // "7c4b9f6a..."  // Different!
```

---

## Introducing bcrypt

bcrypt is the industry standard for password hashing. It combines:
- **Salting** - Automatic random salt generation
- **Key stretching** - Intentionally slow to resist brute force
- **Cost factor** - Adjustable difficulty as hardware improves

### Installing bcrypt

```bash
npm install bcrypt
```

### How bcrypt Works

```javascript
const bcrypt = require('bcrypt');

// bcrypt output format:
// $2b$10$N9qo8uLOickgx2ZMRZoMye.IjqQBrkHx7J9G2q.W5yz3dz7VqP9Vy
// $2b = bcrypt version
// $10 = cost factor (2^10 = 1024 iterations)
// $N9qo8uLOickgx2ZMRZoMye = 22-character salt
// .IjqQBrkHx7J9G2q.W5yz3dz7VqP9Vy = 31-character hash
```

---

## Implementing Password Hashing

### Hashing a Password

```javascript
const bcrypt = require('bcrypt');

// Recommended cost factor (adjustable)
const SALT_ROUNDS = 12;

// Method 1: Async with callback
bcrypt.hash('userPassword123', SALT_ROUNDS, (err, hash) => {
  if (err) throw err;
  console.log('Hash:', hash);
  // Store hash in database
});

// Method 2: Async with Promise (recommended)
async function hashPassword(plainPassword) {
  const hash = await bcrypt.hash(plainPassword, SALT_ROUNDS);
  return hash;
}

// Method 3: Generate salt separately (for more control)
async function hashPasswordWithSalt(plainPassword) {
  const salt = await bcrypt.genSalt(SALT_ROUNDS);
  const hash = await bcrypt.hash(plainPassword, salt);
  return hash;
}
```

### Verifying a Password

```javascript
async function verifyPassword(plainPassword, storedHash) {
  const isMatch = await bcrypt.compare(plainPassword, storedHash);
  return isMatch;
}

// Usage
const storedHash = '$2b$12$LQv3c1yqBWVHxkd0LHAkCOYz6TtxMQJqhN8/X4u.sC.5f9C.Qm9G2';

const isValid = await verifyPassword('userPassword123', storedHash);
console.log('Password valid:', isValid);  // true or false
```

---

## Complete User Registration & Login

### User Model with Password Hashing

```javascript
// models/User.js
const bcrypt = require('bcrypt');
const { prisma } = require('../lib/prisma');

const SALT_ROUNDS = 12;

const UserService = {
  // Create user with hashed password
  async createUser(email, password, name) {
    // Hash password before storing
    const hashedPassword = await bcrypt.hash(password, SALT_ROUNDS);

    const user = await prisma.user.create({
      data: {
        email,
        password: hashedPassword,
        name
      }
    });

    // Never return password in response
    const { password: _, ...userWithoutPassword } = user;
    return userWithoutPassword;
  },

  // Verify user credentials
  async verifyCredentials(email, password) {
    const user = await prisma.user.findUnique({
      where: { email }
    });

    if (!user) {
      return null;  // User not found
    }

    const isValid = await bcrypt.compare(password, user.password);

    if (!isValid) {
      return null;  // Invalid password
    }

    const { password: _, ...userWithoutPassword } = user;
    return userWithoutPassword;
  },

  // Update password
  async updatePassword(userId, currentPassword, newPassword) {
    const user = await prisma.user.findUnique({
      where: { id: userId }
    });

    if (!user) {
      throw new Error('User not found');
    }

    // Verify current password
    const isValid = await bcrypt.compare(currentPassword, user.password);
    if (!isValid) {
      throw new Error('Current password is incorrect');
    }

    // Hash and update new password
    const hashedPassword = await bcrypt.hash(newPassword, SALT_ROUNDS);

    await prisma.user.update({
      where: { id: userId },
      data: { password: hashedPassword }
    });

    return true;
  }
};

module.exports = UserService;
```

### Authentication Controller

```javascript
// controllers/authController.js
const UserService = require('../models/User');
const { validationResult } = require('express-validator');

const authController = {
  // Register new user
  async register(req, res) {
    try {
      const errors = validationResult(req);
      if (!errors.isEmpty()) {
        return res.status(400).json({ errors: errors.array() });
      }

      const { email, password, name } = req.body;

      // Check if user exists
      const existingUser = await prisma.user.findUnique({
        where: { email }
      });

      if (existingUser) {
        return res.status(409).json({
          error: 'User with this email already exists'
        });
      }

      const user = await UserService.createUser(email, password, name);

      res.status(201).json({
        message: 'User registered successfully',
        user
      });
    } catch (error) {
      console.error('Registration error:', error);
      res.status(500).json({ error: 'Registration failed' });
    }
  },

  // Login user
  async login(req, res) {
    try {
      const { email, password } = req.body;

      const user = await UserService.verifyCredentials(email, password);

      if (!user) {
        // Use generic message to prevent user enumeration
        return res.status(401).json({
          error: 'Invalid email or password'
        });
      }

      // TODO: Generate JWT token (next lesson)
      res.json({
        message: 'Login successful',
        user
      });
    } catch (error) {
      console.error('Login error:', error);
      res.status(500).json({ error: 'Login failed' });
    }
  },

  // Change password
  async changePassword(req, res) {
    try {
      const { currentPassword, newPassword } = req.body;
      const userId = req.user.id;  // From auth middleware

      await UserService.updatePassword(userId, currentPassword, newPassword);

      res.json({ message: 'Password updated successfully' });
    } catch (error) {
      if (error.message === 'Current password is incorrect') {
        return res.status(400).json({ error: error.message });
      }
      res.status(500).json({ error: 'Failed to update password' });
    }
  }
};

module.exports = authController;
```

### Input Validation

```javascript
// middleware/validators.js
const { body } = require('express-validator');

const registerValidation = [
  body('email')
    .isEmail()
    .normalizeEmail()
    .withMessage('Please provide a valid email'),

  body('password')
    .isLength({ min: 8 })
    .withMessage('Password must be at least 8 characters')
    .matches(/[A-Z]/)
    .withMessage('Password must contain an uppercase letter')
    .matches(/[a-z]/)
    .withMessage('Password must contain a lowercase letter')
    .matches(/[0-9]/)
    .withMessage('Password must contain a number')
    .matches(/[!@#$%^&*]/)
    .withMessage('Password must contain a special character'),

  body('name')
    .trim()
    .isLength({ min: 2, max: 100 })
    .withMessage('Name must be 2-100 characters')
];

const loginValidation = [
  body('email')
    .isEmail()
    .normalizeEmail(),
  body('password')
    .notEmpty()
];

const changePasswordValidation = [
  body('currentPassword')
    .notEmpty()
    .withMessage('Current password is required'),
  body('newPassword')
    .isLength({ min: 8 })
    .withMessage('New password must be at least 8 characters')
    .custom((value, { req }) => {
      if (value === req.body.currentPassword) {
        throw new Error('New password must be different');
      }
      return true;
    })
];

module.exports = {
  registerValidation,
  loginValidation,
  changePasswordValidation
};
```

### Routes

```javascript
// routes/auth.js
const express = require('express');
const router = express.Router();
const authController = require('../controllers/authController');
const {
  registerValidation,
  loginValidation,
  changePasswordValidation
} = require('../middleware/validators');
const { authenticate } = require('../middleware/auth');

router.post('/register', registerValidation, authController.register);
router.post('/login', loginValidation, authController.login);
router.post(
  '/change-password',
  authenticate,
  changePasswordValidation,
  authController.changePassword
);

module.exports = router;
```

---

## Choosing the Right Cost Factor

### Benchmarking bcrypt Performance

```javascript
const bcrypt = require('bcrypt');

async function benchmarkBcrypt() {
  const password = 'testPassword123';
  const costFactors = [8, 10, 12, 14, 16];

  for (const cost of costFactors) {
    const start = Date.now();
    await bcrypt.hash(password, cost);
    const duration = Date.now() - start;

    console.log(`Cost factor ${cost}: ${duration}ms`);
  }
}

benchmarkBcrypt();

// Example output:
// Cost factor 8: 12ms
// Cost factor 10: 48ms
// Cost factor 12: 192ms
// Cost factor 14: 768ms
// Cost factor 16: 3072ms
```

### Recommendations

```javascript
// Cost factor guidelines:
// - Minimum: 10 (for development)
// - Recommended: 12 (good balance for most apps)
// - High security: 14+ (banking, healthcare)

// Rule of thumb: Hash should take 250-500ms on your server
// Increase cost factor as hardware improves

const SALT_ROUNDS = process.env.NODE_ENV === 'production' ? 12 : 10;
```

---

## Security Best Practices

### 1. Timing Attack Prevention

```javascript
// BAD: Vulnerable to timing attacks
async function unsafeLogin(email, password) {
  const user = await findUser(email);
  if (!user) return false;  // Fast return reveals user exists
  return bcrypt.compare(password, user.password);
}

// GOOD: Constant-time comparison
async function safeLogin(email, password) {
  const user = await findUser(email);

  // Always perform hash comparison, even for non-existent users
  const hashToCompare = user?.password || '$2b$12$invalidhashforconstanttime';
  const isValid = await bcrypt.compare(password, hashToCompare);

  return user && isValid ? user : null;
}
```

### 2. Password Policy Enforcement

```javascript
// Password strength checker
function checkPasswordStrength(password) {
  const checks = {
    length: password.length >= 12,
    uppercase: /[A-Z]/.test(password),
    lowercase: /[a-z]/.test(password),
    numbers: /[0-9]/.test(password),
    special: /[!@#$%^&*(),.?":{}|<>]/.test(password),
    noCommon: !isCommonPassword(password),
    noSequential: !/(.)\1{2,}|012|123|234|345|456|567|678|789|abc|bcd/i.test(password)
  };

  const score = Object.values(checks).filter(Boolean).length;
  const passed = Object.values(checks).every(Boolean);

  return {
    score,
    maxScore: Object.keys(checks).length,
    passed,
    checks
  };
}

// Common password check (simplified)
const commonPasswords = new Set([
  'password', 'password123', '123456', 'qwerty',
  'letmein', 'welcome', 'monkey', 'dragon'
]);

function isCommonPassword(password) {
  return commonPasswords.has(password.toLowerCase());
}
```

### 3. Secure Password Reset Flow

```javascript
const crypto = require('crypto');

// Generate secure reset token
function generateResetToken() {
  return crypto.randomBytes(32).toString('hex');
}

// Store hashed token (don't store plain token!)
async function createPasswordReset(email) {
  const user = await findUserByEmail(email);
  if (!user) return;  // Silent fail to prevent enumeration

  const token = generateResetToken();
  const hashedToken = crypto
    .createHash('sha256')
    .update(token)
    .digest('hex');

  await prisma.passwordReset.create({
    data: {
      userId: user.id,
      token: hashedToken,
      expiresAt: new Date(Date.now() + 3600000)  // 1 hour
    }
  });

  // Send email with plain token
  await sendResetEmail(email, token);
}

// Verify reset token
async function verifyResetToken(token) {
  const hashedToken = crypto
    .createHash('sha256')
    .update(token)
    .digest('hex');

  const reset = await prisma.passwordReset.findFirst({
    where: {
      token: hashedToken,
      expiresAt: { gt: new Date() },
      usedAt: null
    }
  });

  return reset;
}
```

---

## Common Mistakes to Avoid

```javascript
// MISTAKE 1: Using weak hashing algorithms
const crypto = require('crypto');
// DON'T USE for passwords:
crypto.createHash('md5').update(password).digest('hex');     // Weak
crypto.createHash('sha1').update(password).digest('hex');    // Weak
crypto.createHash('sha256').update(password).digest('hex');  // No salt/stretching

// MISTAKE 2: Using encryption instead of hashing
// Encryption is reversible - if key is stolen, all passwords exposed

// MISTAKE 3: Client-side hashing
// DON'T hash passwords on the client - attacker can send hash directly

// MISTAKE 4: Hardcoded salt
const salt = 'my-secret-salt';  // Same for all users = rainbow table vulnerable

// MISTAKE 5: Logging passwords
console.log(`User ${email} registered with password ${password}`);  // NEVER!
```

---

## Practice Exercise

Build a complete authentication system:

1. **Create Prisma schema for users**:
```prisma
model User {
  id        String   @id @default(uuid())
  email     String   @unique
  password  String
  name      String
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
}

model PasswordReset {
  id        String    @id @default(uuid())
  userId    String
  token     String
  expiresAt DateTime
  usedAt    DateTime?
  createdAt DateTime  @default(now())
  user      User      @relation(fields: [userId], references: [id])
}
```

2. **Implement**:
   - User registration with validation
   - Login with secure password verification
   - Password change functionality
   - Password reset flow (token-based)

3. **Test scenarios**:
   - Register with weak password (should fail)
   - Login with wrong password (should fail gracefully)
   - Change password with incorrect current password
   - Verify same password produces different hashes

---

## Key Takeaways

1. **Never store plain-text passwords** - Always hash with bcrypt
2. **bcrypt includes salt automatically** - No need for separate salt management
3. **Use appropriate cost factor** - Balance security and performance (12 recommended)
4. **Prevent timing attacks** - Always perform comparison, even for invalid users
5. **Validate password strength** - Enforce minimum requirements
6. **Generic error messages** - Don't reveal whether email exists
7. **Hash reset tokens** - Store hashed version, send plain to user

---

## What's Next?

Tomorrow, we'll learn about **JSON Web Tokens (JWT)** - how to create secure tokens for maintaining user sessions without server-side storage.

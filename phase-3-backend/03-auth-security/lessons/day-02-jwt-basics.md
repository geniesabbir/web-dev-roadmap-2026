# Day 2: JWT Basics - Understanding JSON Web Tokens

## Introduction

JSON Web Tokens (JWT) provide a stateless, scalable way to handle authentication in modern web applications. Unlike session-based authentication that requires server-side storage, JWTs are self-contained tokens that carry user information securely. Today, you'll learn how JWTs work and why they're ideal for modern APIs.

## Learning Objectives

By the end of this lesson, you will be able to:
- Understand the structure and components of a JWT
- Explain how JWT-based authentication works
- Differentiate between JWT and session-based auth
- Identify appropriate use cases for JWTs
- Recognize common JWT security considerations

---

## What is a JWT?

A JSON Web Token is a compact, URL-safe means of representing claims between two parties. It's commonly used to:

- Authenticate users after login
- Authorize access to protected resources
- Exchange information securely between services

### JWT Structure

A JWT consists of three parts separated by dots:

```
xxxxx.yyyyy.zzzzz
Header.Payload.Signature
```

**Example JWT:**
```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
```

---

## JWT Components Explained

### 1. Header

The header typically contains two parts:
- **alg**: The signing algorithm (e.g., HS256, RS256)
- **typ**: The token type (JWT)

```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

**Base64Url encoded:**
```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9
```

### 2. Payload

The payload contains claims - statements about the user and additional data:

```json
{
  "sub": "user123",
  "name": "John Doe",
  "email": "john@example.com",
  "role": "admin",
  "iat": 1516239022,
  "exp": 1516242622
}
```

**Types of Claims:**

| Claim Type | Description | Examples |
|------------|-------------|----------|
| Registered | Predefined, recommended claims | iss, sub, aud, exp, iat |
| Public | Custom claims registered in IANA | name, email, role |
| Private | Custom claims for your app | userId, permissions |

**Common Registered Claims:**

```javascript
const payload = {
  // Registered claims
  iss: 'https://myapp.com',      // Issuer
  sub: 'user123',                 // Subject (user ID)
  aud: 'https://api.myapp.com',   // Audience
  exp: 1516242622,                // Expiration time (Unix timestamp)
  nbf: 1516239022,                // Not valid before
  iat: 1516239022,                // Issued at
  jti: 'unique-token-id',         // JWT ID (for token revocation)

  // Custom claims
  name: 'John Doe',
  role: 'admin'
};
```

### 3. Signature

The signature ensures the token hasn't been tampered with:

```javascript
// For HMAC SHA256 (HS256):
HMACSHA256(
  base64UrlEncode(header) + "." + base64UrlEncode(payload),
  secret
)

// The signature is created by:
// 1. Combining encoded header and payload
// 2. Signing with the secret key
// 3. Base64Url encoding the result
```

---

## How JWT Authentication Works

```
┌─────────┐                                    ┌─────────┐
│  Client │                                    │  Server │
└────┬────┘                                    └────┬────┘
     │                                              │
     │  1. POST /login {email, password}            │
     │─────────────────────────────────────────────>│
     │                                              │
     │                    2. Validate credentials   │
     │                    3. Generate JWT           │
     │                                              │
     │  4. Response {token: "eyJhbGciOi..."}        │
     │<─────────────────────────────────────────────│
     │                                              │
     │  5. Store token (localStorage/cookie)        │
     │                                              │
     │  6. GET /api/profile                         │
     │  Authorization: Bearer eyJhbGciOi...         │
     │─────────────────────────────────────────────>│
     │                                              │
     │                    7. Verify signature       │
     │                    8. Check expiration       │
     │                    9. Extract user data      │
     │                                              │
     │  10. Response {user: {...}}                  │
     │<─────────────────────────────────────────────│
     │                                              │
```

### Authentication Flow

```javascript
// 1. User logs in
const response = await fetch('/api/login', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({ email, password })
});

const { token } = await response.json();

// 2. Store token
localStorage.setItem('token', token);

// 3. Use token for authenticated requests
const profileResponse = await fetch('/api/profile', {
  headers: {
    'Authorization': `Bearer ${token}`
  }
});

// 4. Server verifies token on each request
// - No database lookup needed!
// - User info extracted from token payload
```

---

## JWT vs Session-Based Authentication

### Session-Based Authentication

```
┌─────────┐         ┌─────────┐         ┌───────────┐
│  Client │         │  Server │         │  Session  │
│         │         │         │         │   Store   │
└────┬────┘         └────┬────┘         └─────┬─────┘
     │                   │                    │
     │  Login            │                    │
     │──────────────────>│                    │
     │                   │  Create session    │
     │                   │───────────────────>│
     │                   │                    │
     │  Set-Cookie:      │<───────────────────│
     │  sessionId=abc123 │                    │
     │<──────────────────│                    │
     │                   │                    │
     │  Request +        │                    │
     │  Cookie           │                    │
     │──────────────────>│                    │
     │                   │  Lookup session    │
     │                   │───────────────────>│
     │                   │                    │
     │  Response         │<───────────────────│
     │<──────────────────│                    │
```

### Comparison Table

| Aspect | Session-Based | JWT |
|--------|---------------|-----|
| **Storage** | Server (Redis/DB) | Client (token) |
| **Scalability** | Requires shared session store | Stateless, easy horizontal scaling |
| **Database lookup** | Every request | None (token is self-contained) |
| **Revocation** | Easy (delete session) | Complex (need blocklist) |
| **Mobile apps** | Requires cookies | Natural fit (header-based) |
| **Cross-domain** | Complex (CORS cookies) | Simple (header works everywhere) |
| **Size** | Small session ID | Larger token (~1KB+) |
| **Security** | Session fixation risks | Token theft risks |

### When to Use What

**Use Sessions When:**
- Traditional web apps with server-side rendering
- Need instant logout/revocation capability
- Token size is a concern
- Single-domain application

**Use JWT When:**
- Building REST/GraphQL APIs
- Mobile applications
- Microservices architecture
- Cross-domain authentication
- Need stateless authentication

---

## JWT Signing Algorithms

### Symmetric Algorithms (HMAC)

Uses the same secret key for signing and verification:

```javascript
// HS256 - HMAC with SHA-256
// Same secret used to sign and verify
const secret = 'my-super-secret-key';

// Server signs
const token = jwt.sign(payload, secret);

// Server verifies
const decoded = jwt.verify(token, secret);
```

**Pros:**
- Faster performance
- Simpler setup

**Cons:**
- Secret must be shared if multiple services verify
- Key compromise = all tokens invalid

### Asymmetric Algorithms (RSA/ECDSA)

Uses public/private key pairs:

```javascript
// RS256 - RSA with SHA-256
// Private key signs, public key verifies

// Auth server signs with private key
const token = jwt.sign(payload, privateKey, { algorithm: 'RS256' });

// Any service verifies with public key
const decoded = jwt.verify(token, publicKey, { algorithms: ['RS256'] });
```

**Pros:**
- Public key can be shared freely
- Multiple services can verify without secret
- Better for microservices

**Cons:**
- Larger token size
- Slower than HMAC

### Algorithm Recommendations

```javascript
// For single-server applications:
// Use HS256 with strong secret (256+ bits)
const secret = crypto.randomBytes(32).toString('hex');

// For microservices/distributed systems:
// Use RS256 or ES256 with key pairs
const { publicKey, privateKey } = crypto.generateKeyPairSync('rsa', {
  modulusLength: 2048
});
```

---

## Token Expiration Strategy

### Access Token + Refresh Token Pattern

```javascript
// Short-lived access token (15 min - 1 hour)
const accessToken = jwt.sign(
  { userId: user.id, type: 'access' },
  ACCESS_SECRET,
  { expiresIn: '15m' }
);

// Long-lived refresh token (7-30 days)
const refreshToken = jwt.sign(
  { userId: user.id, type: 'refresh' },
  REFRESH_SECRET,
  { expiresIn: '7d' }
);

// Refresh flow
async function refreshAccessToken(refreshToken) {
  try {
    const decoded = jwt.verify(refreshToken, REFRESH_SECRET);

    if (decoded.type !== 'refresh') {
      throw new Error('Invalid token type');
    }

    // Issue new access token
    const newAccessToken = jwt.sign(
      { userId: decoded.userId, type: 'access' },
      ACCESS_SECRET,
      { expiresIn: '15m' }
    );

    return { accessToken: newAccessToken };
  } catch (error) {
    throw new Error('Invalid refresh token');
  }
}
```

### Token Flow Diagram

```
┌─────────────────────────────────────────────────────────────┐
│                        Token Lifecycle                       │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  Login ──> Access Token (15m) + Refresh Token (7d)          │
│                     │                                        │
│                     ▼                                        │
│           ┌─────────────────┐                                │
│           │ Use Access Token │◄───────────────────┐          │
│           │ for API requests │                    │          │
│           └────────┬────────┘                     │          │
│                    │                              │          │
│                    ▼                              │          │
│           Access Token Expired?                   │          │
│                /        \                         │          │
│              No          Yes                      │          │
│              │            │                       │          │
│              │            ▼                       │          │
│              │   Use Refresh Token ───> New Access Token     │
│              │            │                       │          │
│              │            ▼                       │          │
│              │   Refresh Token Expired?           │          │
│              │        /        \                  │          │
│              │      No          Yes               │          │
│              │      │            │                │          │
│              │      │            ▼                │          │
│              │      │      Force Re-login         │          │
│              │      └────────────┘                │          │
│              └────────────────────────────────────┘          │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

---

## Security Considerations

### 1. Never Store Sensitive Data in Payload

```javascript
// BAD - Sensitive data exposed
const badPayload = {
  userId: 123,
  password: 'secret123',     // NEVER!
  creditCard: '4111...',     // NEVER!
  ssn: '123-45-6789'         // NEVER!
};

// GOOD - Only include necessary claims
const goodPayload = {
  userId: 123,
  email: 'user@example.com',
  role: 'user'
};
```

### 2. Token Storage Best Practices

```javascript
// Browser Storage Options:

// 1. localStorage
// + Easy to use
// - Vulnerable to XSS attacks
localStorage.setItem('token', token);

// 2. sessionStorage
// + Cleared on tab close
// - Still vulnerable to XSS
sessionStorage.setItem('token', token);

// 3. HttpOnly Cookie (Recommended for web apps)
// + Not accessible via JavaScript
// + Protected from XSS
// - Requires CSRF protection
res.cookie('token', token, {
  httpOnly: true,
  secure: true,       // HTTPS only
  sameSite: 'strict', // CSRF protection
  maxAge: 900000      // 15 minutes
});

// 4. Memory (React state) + Refresh token in HttpOnly cookie
// + Most secure for SPAs
// - Token lost on page refresh
```

### 3. Common Vulnerabilities

```javascript
// VULNERABILITY 1: Algorithm confusion
// Never allow "none" algorithm
const decoded = jwt.verify(token, secret, {
  algorithms: ['HS256']  // Explicitly specify allowed algorithms
});

// VULNERABILITY 2: Weak secrets
// BAD
const secret = 'password123';

// GOOD - Use strong, random secrets
const secret = crypto.randomBytes(64).toString('hex');

// VULNERABILITY 3: Missing expiration
// ALWAYS set expiration
const token = jwt.sign(payload, secret, {
  expiresIn: '1h'  // Always include!
});

// VULNERABILITY 4: Not validating all claims
const decoded = jwt.verify(token, secret, {
  algorithms: ['HS256'],
  issuer: 'https://myapp.com',
  audience: 'https://api.myapp.com'
});
```

---

## JWT Token Inspection

### Debugging JWTs

```javascript
// Decode without verification (for debugging only!)
function decodeToken(token) {
  const parts = token.split('.');

  return {
    header: JSON.parse(Buffer.from(parts[0], 'base64url').toString()),
    payload: JSON.parse(Buffer.from(parts[1], 'base64url').toString()),
    signature: parts[2]
  };
}

// Usage
const token = 'eyJhbGci...';
const decoded = decodeToken(token);
console.log(decoded);

// Online tool: jwt.io (for development only!)
```

### Check Token Expiration

```javascript
function isTokenExpired(token) {
  try {
    const { payload } = decodeToken(token);

    if (!payload.exp) {
      return false;  // No expiration set
    }

    // exp is in seconds, Date.now() is in milliseconds
    return payload.exp * 1000 < Date.now();
  } catch {
    return true;  // Invalid token = treat as expired
  }
}

// Client-side usage
if (isTokenExpired(localStorage.getItem('token'))) {
  // Redirect to login or refresh token
  refreshAccessToken();
}
```

---

## JWT Best Practices Summary

### Do's ✅

```javascript
// 1. Use strong secrets (256+ bits)
const secret = crypto.randomBytes(32).toString('hex');

// 2. Always set expiration
jwt.sign(payload, secret, { expiresIn: '15m' });

// 3. Validate issuer and audience
jwt.verify(token, secret, {
  issuer: 'https://myapp.com',
  audience: 'https://api.myapp.com'
});

// 4. Use HTTPS only
res.cookie('token', token, { secure: true });

// 5. Keep payloads small
const payload = { userId: 123, role: 'user' };

// 6. Use appropriate algorithm
// HS256 for single server, RS256 for distributed
```

### Don'ts ❌

```javascript
// 1. Don't store sensitive data in payload
// 2. Don't use weak secrets
// 3. Don't accept "none" algorithm
// 4. Don't store tokens in localStorage for high-security apps
// 5. Don't use long-lived access tokens without refresh
// 6. Don't trust tokens without verification
```

---

## Practice Exercise

1. **Decode a JWT manually**:
   - Take a sample JWT from jwt.io
   - Split by dots
   - Base64 decode each part
   - Examine the structure

2. **Answer these questions**:
   - What happens if you modify the payload without re-signing?
   - Why can't you recover the secret from a JWT?
   - When would RS256 be preferred over HS256?

3. **Design a token strategy**:
   - For a banking app (high security)
   - For a blog platform (low risk)
   - What expiration times would you use?

---

## Key Takeaways

1. **JWTs are self-contained** - Carry user info without database lookup
2. **Three parts** - Header, Payload, Signature (separated by dots)
3. **Base64 encoded, not encrypted** - Anyone can read the payload
4. **Signature ensures integrity** - Tampering is detectable
5. **Stateless = scalable** - Perfect for distributed systems
6. **Use access + refresh tokens** - Balance security and UX
7. **Choose storage carefully** - HttpOnly cookies preferred for web

---

## What's Next?

Tomorrow, we'll implement JWT authentication in a Node.js application - creating login endpoints, protecting routes, and handling token refresh.

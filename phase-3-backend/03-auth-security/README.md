# Authentication & Security

**Duration:** 2 weeks

## Learning Objectives

By the end of this section, you will:
- Implement secure user authentication
- Use JWT and session-based auth appropriately
- Protect routes and resources
- Implement OAuth 2.0
- Understand and prevent common vulnerabilities

---

## Week 1: Authentication

### Day 1: Password Security
**Topics:**
- Why plain passwords are dangerous
- Hashing vs encryption
- bcrypt algorithm
- Salt and work factor
- Password validation rules

**Implementation:**
```typescript
// src/utils/password.ts
import bcrypt from 'bcrypt'

const SALT_ROUNDS = 12

export async function hashPassword(password: string): Promise<string> {
  return bcrypt.hash(password, SALT_ROUNDS)
}

export async function verifyPassword(
  password: string,
  hash: string
): Promise<boolean> {
  return bcrypt.compare(password, hash)
}

// Password validation with Zod
import { z } from 'zod'

export const passwordSchema = z
  .string()
  .min(8, 'Password must be at least 8 characters')
  .regex(/[A-Z]/, 'Password must contain an uppercase letter')
  .regex(/[a-z]/, 'Password must contain a lowercase letter')
  .regex(/[0-9]/, 'Password must contain a number')
  .regex(/[^A-Za-z0-9]/, 'Password must contain a special character')
```

### Day 2: JWT Authentication
**Topics:**
- What is JWT?
- JWT structure (header, payload, signature)
- Access tokens vs refresh tokens
- Token storage
- Token expiration

**Implementation:**
```typescript
// src/utils/jwt.ts
import jwt from 'jsonwebtoken'

const ACCESS_SECRET = process.env.JWT_ACCESS_SECRET!
const REFRESH_SECRET = process.env.JWT_REFRESH_SECRET!

interface TokenPayload {
  userId: string
  email: string
}

export function generateAccessToken(payload: TokenPayload): string {
  return jwt.sign(payload, ACCESS_SECRET, { expiresIn: '15m' })
}

export function generateRefreshToken(payload: TokenPayload): string {
  return jwt.sign(payload, REFRESH_SECRET, { expiresIn: '7d' })
}

export function verifyAccessToken(token: string): TokenPayload {
  return jwt.verify(token, ACCESS_SECRET) as TokenPayload
}

export function verifyRefreshToken(token: string): TokenPayload {
  return jwt.verify(token, REFRESH_SECRET) as TokenPayload
}

// src/services/authService.ts
export const authService = {
  async login(email: string, password: string) {
    const user = await prisma.user.findUnique({ where: { email } })

    if (!user || !(await verifyPassword(password, user.password))) {
      throw new AppError('Invalid credentials', 401)
    }

    const payload = { userId: user.id, email: user.email }
    const accessToken = generateAccessToken(payload)
    const refreshToken = generateRefreshToken(payload)

    // Store refresh token in database
    await prisma.refreshToken.create({
      data: {
        token: refreshToken,
        userId: user.id,
        expiresAt: new Date(Date.now() + 7 * 24 * 60 * 60 * 1000),
      },
    })

    return { accessToken, refreshToken, user: { id: user.id, email: user.email, name: user.name } }
  },

  async refresh(refreshToken: string) {
    const payload = verifyRefreshToken(refreshToken)

    const storedToken = await prisma.refreshToken.findUnique({
      where: { token: refreshToken },
    })

    if (!storedToken || storedToken.expiresAt < new Date()) {
      throw new AppError('Invalid refresh token', 401)
    }

    const newAccessToken = generateAccessToken({
      userId: payload.userId,
      email: payload.email,
    })

    return { accessToken: newAccessToken }
  },

  async logout(refreshToken: string) {
    await prisma.refreshToken.delete({ where: { token: refreshToken } })
  },
}
```

### Day 3: Auth Middleware
**Topics:**
- Protecting routes
- Extracting tokens from headers
- Attaching user to request
- Role-based authorization

**Implementation:**
```typescript
// src/middleware/auth.ts
import { Request, Response, NextFunction } from 'express'
import { verifyAccessToken } from '../utils/jwt'
import { AppError } from '../utils/AppError'

// Extend Express Request type
declare global {
  namespace Express {
    interface Request {
      user?: {
        userId: string
        email: string
      }
    }
  }
}

export function authenticate(req: Request, res: Response, next: NextFunction) {
  const authHeader = req.headers.authorization

  if (!authHeader?.startsWith('Bearer ')) {
    throw new AppError('No token provided', 401)
  }

  const token = authHeader.split(' ')[1]

  try {
    const payload = verifyAccessToken(token)
    req.user = payload
    next()
  } catch (error) {
    throw new AppError('Invalid token', 401)
  }
}

export function authorize(...roles: string[]) {
  return async (req: Request, res: Response, next: NextFunction) => {
    const user = await prisma.user.findUnique({
      where: { id: req.user!.userId },
    })

    if (!user || !roles.includes(user.role)) {
      throw new AppError('Not authorized', 403)
    }

    next()
  }
}

// Usage in routes
router.get('/admin/users', authenticate, authorize('ADMIN'), getUsers)
router.get('/profile', authenticate, getProfile)
```

### Day 4: Registration Flow
**Topics:**
- User registration
- Email verification
- Account activation
- Welcome emails

**Implementation:**
```typescript
// src/services/authService.ts
import crypto from 'crypto'

export const authService = {
  async register(data: { email: string; password: string; name: string }) {
    const existingUser = await prisma.user.findUnique({
      where: { email: data.email },
    })

    if (existingUser) {
      throw new AppError('Email already registered', 400)
    }

    const hashedPassword = await hashPassword(data.password)
    const verificationToken = crypto.randomBytes(32).toString('hex')

    const user = await prisma.user.create({
      data: {
        email: data.email,
        password: hashedPassword,
        name: data.name,
        emailVerificationToken: verificationToken,
        emailVerificationExpires: new Date(Date.now() + 24 * 60 * 60 * 1000),
      },
    })

    // Send verification email
    await sendVerificationEmail(user.email, verificationToken)

    return { message: 'Registration successful. Please verify your email.' }
  },

  async verifyEmail(token: string) {
    const user = await prisma.user.findFirst({
      where: {
        emailVerificationToken: token,
        emailVerificationExpires: { gt: new Date() },
      },
    })

    if (!user) {
      throw new AppError('Invalid or expired verification token', 400)
    }

    await prisma.user.update({
      where: { id: user.id },
      data: {
        emailVerified: true,
        emailVerificationToken: null,
        emailVerificationExpires: null,
      },
    })

    return { message: 'Email verified successfully' }
  },
}
```

### Day 5: Password Reset
**Topics:**
- Forgot password flow
- Secure reset tokens
- Token expiration
- Password change endpoint

**Implementation:**
```typescript
export const authService = {
  async forgotPassword(email: string) {
    const user = await prisma.user.findUnique({ where: { email } })

    // Don't reveal if user exists
    if (!user) {
      return { message: 'If that email exists, we sent a reset link' }
    }

    const resetToken = crypto.randomBytes(32).toString('hex')
    const resetTokenHash = crypto
      .createHash('sha256')
      .update(resetToken)
      .digest('hex')

    await prisma.user.update({
      where: { id: user.id },
      data: {
        passwordResetToken: resetTokenHash,
        passwordResetExpires: new Date(Date.now() + 60 * 60 * 1000), // 1 hour
      },
    })

    await sendPasswordResetEmail(email, resetToken)

    return { message: 'If that email exists, we sent a reset link' }
  },

  async resetPassword(token: string, newPassword: string) {
    const resetTokenHash = crypto
      .createHash('sha256')
      .update(token)
      .digest('hex')

    const user = await prisma.user.findFirst({
      where: {
        passwordResetToken: resetTokenHash,
        passwordResetExpires: { gt: new Date() },
      },
    })

    if (!user) {
      throw new AppError('Invalid or expired reset token', 400)
    }

    const hashedPassword = await hashPassword(newPassword)

    await prisma.user.update({
      where: { id: user.id },
      data: {
        password: hashedPassword,
        passwordResetToken: null,
        passwordResetExpires: null,
      },
    })

    // Invalidate all existing sessions
    await prisma.refreshToken.deleteMany({ where: { userId: user.id } })

    return { message: 'Password reset successful' }
  },
}
```

---

## Week 2: Security & OAuth

### Day 1: OAuth 2.0
**Topics:**
- OAuth 2.0 flow
- Authorization code grant
- Setting up providers (Google, GitHub)
- Passport.js

**Implementation with Passport:**
```typescript
// src/config/passport.ts
import passport from 'passport'
import { Strategy as GoogleStrategy } from 'passport-google-oauth20'

passport.use(
  new GoogleStrategy(
    {
      clientID: process.env.GOOGLE_CLIENT_ID!,
      clientSecret: process.env.GOOGLE_CLIENT_SECRET!,
      callbackURL: '/api/auth/google/callback',
    },
    async (accessToken, refreshToken, profile, done) => {
      try {
        let user = await prisma.user.findUnique({
          where: { email: profile.emails![0].value },
        })

        if (!user) {
          user = await prisma.user.create({
            data: {
              email: profile.emails![0].value,
              name: profile.displayName,
              password: '', // No password for OAuth users
              emailVerified: true,
              oauthProvider: 'google',
              oauthId: profile.id,
            },
          })
        }

        done(null, user)
      } catch (error) {
        done(error as Error)
      }
    }
  )
)

// Routes
router.get('/auth/google', passport.authenticate('google', { scope: ['profile', 'email'] }))

router.get(
  '/auth/google/callback',
  passport.authenticate('google', { session: false }),
  (req, res) => {
    const user = req.user as User
    const tokens = generateTokens(user)
    res.redirect(`${CLIENT_URL}/auth/callback?token=${tokens.accessToken}`)
  }
)
```

### Day 2: OWASP Top 10
**Topics:**
- Injection attacks (SQL, NoSQL, Command)
- Broken authentication
- Sensitive data exposure
- XSS (Cross-Site Scripting)
- CSRF (Cross-Site Request Forgery)
- Security misconfiguration

### Day 3: Input Validation & Sanitization
**Topics:**
- Validation vs sanitization
- Preventing injection
- Content Security Policy
- Security headers

**Implementation:**
```typescript
// Helmet for security headers
import helmet from 'helmet'

app.use(helmet())
app.use(
  helmet.contentSecurityPolicy({
    directives: {
      defaultSrc: ["'self'"],
      scriptSrc: ["'self'"],
      styleSrc: ["'self'", "'unsafe-inline'"],
      imgSrc: ["'self'", 'data:', 'https:'],
    },
  })
)

// Input sanitization
import { z } from 'zod'
import xss from 'xss'

const commentSchema = z.object({
  content: z
    .string()
    .min(1)
    .max(1000)
    .transform((val) => xss(val)), // Sanitize HTML
})

// Parameterized queries (Prisma does this automatically)
// NEVER do: `SELECT * FROM users WHERE id = ${userId}`
// Prisma: prisma.user.findUnique({ where: { id: userId } })
```

### Day 4: Rate Limiting & Protection
**Topics:**
- Rate limiting strategies
- Brute force protection
- Account lockout
- CAPTCHA integration

**Implementation:**
```typescript
import rateLimit from 'express-rate-limit'
import slowDown from 'express-slow-down'

// General rate limit
const apiLimiter = rateLimit({
  windowMs: 15 * 60 * 1000,
  max: 100,
  message: { error: 'Too many requests' },
})

// Strict limit for auth endpoints
const authLimiter = rateLimit({
  windowMs: 60 * 60 * 1000, // 1 hour
  max: 5, // 5 attempts
  message: { error: 'Too many login attempts. Try again later.' },
})

// Slow down repeated requests
const speedLimiter = slowDown({
  windowMs: 15 * 60 * 1000,
  delayAfter: 50,
  delayMs: 500,
})

app.use('/api/', apiLimiter)
app.use('/api/', speedLimiter)
app.use('/api/auth/login', authLimiter)
app.use('/api/auth/register', authLimiter)

// Account lockout
const MAX_LOGIN_ATTEMPTS = 5
const LOCK_TIME = 30 * 60 * 1000 // 30 minutes

async function handleFailedLogin(userId: string) {
  const user = await prisma.user.update({
    where: { id: userId },
    data: {
      loginAttempts: { increment: 1 },
      lastFailedLogin: new Date(),
    },
  })

  if (user.loginAttempts >= MAX_LOGIN_ATTEMPTS) {
    await prisma.user.update({
      where: { id: userId },
      data: {
        lockedUntil: new Date(Date.now() + LOCK_TIME),
      },
    })
  }
}
```

### Day 5: Security Audit
**Project:** Audit and secure your existing APIs
- Add rate limiting
- Implement account lockout
- Add security headers
- Review all inputs for validation
- Add CORS configuration
- Document security measures

---

## Security Checklist

### Authentication
- [ ] Passwords hashed with bcrypt (12+ rounds)
- [ ] Secure password requirements
- [ ] Rate limited auth endpoints
- [ ] Account lockout after failed attempts
- [ ] Secure token storage
- [ ] Token expiration and refresh

### Authorization
- [ ] Route protection middleware
- [ ] Role-based access control
- [ ] Resource ownership verification
- [ ] Proper 401 vs 403 responses

### Data Protection
- [ ] Input validation on all endpoints
- [ ] Output encoding/escaping
- [ ] Parameterized queries (ORM)
- [ ] Sensitive data encryption
- [ ] No sensitive data in logs

### Headers & Configuration
- [ ] Security headers (Helmet)
- [ ] CORS properly configured
- [ ] HTTPS only in production
- [ ] Secure cookie settings
- [ ] No sensitive info in error messages

---

## Resources

### Documentation
- [OWASP Top 10](https://owasp.org/www-project-top-ten/)
- [JWT.io](https://jwt.io/)
- [Passport.js](https://www.passportjs.org/)

### Tools
- [OWASP ZAP](https://www.zaproxy.org/) - Security scanner
- [npm audit](https://docs.npmjs.com/cli/audit) - Dependency vulnerabilities

---

## Checklist Before Moving On

- [ ] Can implement secure password handling
- [ ] Can create JWT auth system
- [ ] Understand refresh token rotation
- [ ] Can implement OAuth with providers
- [ ] Know the OWASP Top 10
- [ ] Can protect against common attacks
- [ ] Implemented security measures in projects

---

**Next:** [API Design](../04-api-design/README.md)

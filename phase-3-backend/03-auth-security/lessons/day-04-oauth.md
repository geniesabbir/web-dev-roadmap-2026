# Day 4: OAuth 2.0 - Social Login Implementation

## Introduction

OAuth 2.0 allows users to authenticate using their existing accounts from providers like Google, GitHub, and Facebook. This improves user experience by eliminating the need to create yet another password while leveraging the security infrastructure of major tech companies. Today, you'll learn how OAuth works and implement social login in your application.

## Learning Objectives

By the end of this lesson, you will be able to:
- Understand the OAuth 2.0 authorization flow
- Set up OAuth applications with Google and GitHub
- Implement social login using Passport.js
- Link social accounts to existing users
- Handle OAuth security considerations

---

## Understanding OAuth 2.0

### What is OAuth?

OAuth 2.0 is an **authorization framework** that enables applications to obtain limited access to user accounts on third-party services. It works by delegating user authentication to the service that hosts the user account.

### Key Concepts

```
┌──────────────────────────────────────────────────────────────┐
│                     OAuth 2.0 Roles                          │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Resource Owner    - The user who owns the data              │
│  Client           - Your application                         │
│  Authorization    - The OAuth provider (Google, GitHub)      │
│    Server                                                    │
│  Resource Server  - API that holds user data                 │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### OAuth 2.0 Authorization Code Flow

```
┌─────────┐                                    ┌─────────────┐
│  User   │                                    │ Your Server │
└────┬────┘                                    └──────┬──────┘
     │                                                │
     │  1. Click "Login with Google"                  │
     │───────────────────────────────────────────────>│
     │                                                │
     │  2. Redirect to Google with client_id,         │
     │     redirect_uri, scope, state                 │
     │<───────────────────────────────────────────────│
     │                                                │
     │                    ┌───────────────────────┐   │
     │  3. Login to       │      Google           │   │
     │     Google         │   Authorization       │   │
     │───────────────────>│      Server           │   │
     │                    └───────────┬───────────┘   │
     │                                │               │
     │  4. "Allow access?"            │               │
     │<───────────────────────────────│               │
     │                                │               │
     │  5. User grants permission     │               │
     │───────────────────────────────>│               │
     │                                │               │
     │  6. Redirect to your app       │               │
     │     with authorization code    │               │
     │<───────────────────────────────│               │
     │                                │               │
     │───────────────────────────────────────────────>│
     │                                                │
     │                    7. Exchange code for tokens │
     │                       (server-to-server)       │
     │                    ┌───────────────────────┐   │
     │                    │      Google           │   │
     │                    │    Token Server       │<──│
     │                    └───────────┬───────────┘   │
     │                                │               │
     │                    8. Access token +           │
     │                       Refresh token            │
     │                    ────────────────────────────>
     │                                                │
     │                    9. Fetch user profile       │
     │                       with access token        │
     │                                                │
     │  10. Create session, redirect to app           │
     │<───────────────────────────────────────────────│
     │                                                │
```

---

## Setting Up OAuth Providers

### Google OAuth Setup

1. Go to [Google Cloud Console](https://console.cloud.google.com/)
2. Create a new project
3. Navigate to "APIs & Services" > "Credentials"
4. Click "Create Credentials" > "OAuth client ID"
5. Configure OAuth consent screen
6. Select "Web application" as application type
7. Add authorized redirect URIs:
   - Development: `http://localhost:3000/api/auth/google/callback`
   - Production: `https://yourdomain.com/api/auth/google/callback`

```env
# .env
GOOGLE_CLIENT_ID=your-google-client-id.apps.googleusercontent.com
GOOGLE_CLIENT_SECRET=your-google-client-secret
GOOGLE_CALLBACK_URL=http://localhost:3000/api/auth/google/callback
```

### GitHub OAuth Setup

1. Go to [GitHub Developer Settings](https://github.com/settings/developers)
2. Click "New OAuth App"
3. Fill in application details:
   - Application name: Your App Name
   - Homepage URL: `http://localhost:3000`
   - Authorization callback URL: `http://localhost:3000/api/auth/github/callback`

```env
# .env (add to existing)
GITHUB_CLIENT_ID=your-github-client-id
GITHUB_CLIENT_SECRET=your-github-client-secret
GITHUB_CALLBACK_URL=http://localhost:3000/api/auth/github/callback
```

---

## Project Setup with Passport.js

### Install Dependencies

```bash
npm install passport passport-google-oauth20 passport-github2
```

### Prisma Schema Update

```prisma
// prisma/schema.prisma
model User {
  id            String         @id @default(uuid())
  email         String         @unique
  password      String?        // Optional for OAuth-only users
  name          String
  avatar        String?
  role          Role           @default(USER)
  isActive      Boolean        @default(true)
  emailVerified Boolean        @default(false)
  createdAt     DateTime       @default(now())
  updatedAt     DateTime       @updatedAt
  accounts      Account[]
  refreshTokens RefreshToken[]
}

model Account {
  id                String   @id @default(uuid())
  userId            String
  provider          String   // "google", "github", etc.
  providerAccountId String   // ID from the provider
  accessToken       String?  @db.Text
  refreshToken      String?  @db.Text
  expiresAt         DateTime?
  createdAt         DateTime @default(now())
  updatedAt         DateTime @updatedAt
  user              User     @relation(fields: [userId], references: [id], onDelete: Cascade)

  @@unique([provider, providerAccountId])
  @@index([userId])
}
```

Run migration:

```bash
npx prisma migrate dev --name add-oauth-accounts
```

---

## Passport Configuration

### Main Passport Setup

```javascript
// config/passport.js
const passport = require('passport');
const GoogleStrategy = require('passport-google-oauth20').Strategy;
const GitHubStrategy = require('passport-github2').Strategy;
const { prisma } = require('../lib/prisma');

// Serialize user for session (not used with JWT, but required)
passport.serializeUser((user, done) => {
  done(null, user.id);
});

passport.deserializeUser(async (id, done) => {
  try {
    const user = await prisma.user.findUnique({ where: { id } });
    done(null, user);
  } catch (error) {
    done(error, null);
  }
});

// Google Strategy
passport.use(new GoogleStrategy({
    clientID: process.env.GOOGLE_CLIENT_ID,
    clientSecret: process.env.GOOGLE_CLIENT_SECRET,
    callbackURL: process.env.GOOGLE_CALLBACK_URL,
    scope: ['profile', 'email']
  },
  async (accessToken, refreshToken, profile, done) => {
    try {
      const result = await handleOAuthLogin({
        provider: 'google',
        providerAccountId: profile.id,
        email: profile.emails[0].value,
        name: profile.displayName,
        avatar: profile.photos[0]?.value,
        accessToken,
        refreshToken
      });

      done(null, result);
    } catch (error) {
      done(error, null);
    }
  }
));

// GitHub Strategy
passport.use(new GitHubStrategy({
    clientID: process.env.GITHUB_CLIENT_ID,
    clientSecret: process.env.GITHUB_CLIENT_SECRET,
    callbackURL: process.env.GITHUB_CALLBACK_URL,
    scope: ['user:email']
  },
  async (accessToken, refreshToken, profile, done) => {
    try {
      // GitHub may not return email in profile, need to fetch separately
      const email = profile.emails?.[0]?.value || await fetchGitHubEmail(accessToken);

      const result = await handleOAuthLogin({
        provider: 'github',
        providerAccountId: profile.id.toString(),
        email,
        name: profile.displayName || profile.username,
        avatar: profile.photos[0]?.value,
        accessToken,
        refreshToken
      });

      done(null, result);
    } catch (error) {
      done(error, null);
    }
  }
));

// Fetch GitHub email if not in profile
async function fetchGitHubEmail(accessToken) {
  const response = await fetch('https://api.github.com/user/emails', {
    headers: {
      'Authorization': `Bearer ${accessToken}`,
      'Accept': 'application/vnd.github+json'
    }
  });

  const emails = await response.json();
  const primaryEmail = emails.find(e => e.primary) || emails[0];
  return primaryEmail?.email;
}

// Handle OAuth login/registration
async function handleOAuthLogin({ provider, providerAccountId, email, name, avatar, accessToken, refreshToken }) {
  // Check if account already linked
  let account = await prisma.account.findUnique({
    where: {
      provider_providerAccountId: {
        provider,
        providerAccountId
      }
    },
    include: { user: true }
  });

  if (account) {
    // Update tokens
    await prisma.account.update({
      where: { id: account.id },
      data: {
        accessToken,
        refreshToken,
        expiresAt: new Date(Date.now() + 3600000) // 1 hour
      }
    });

    return account.user;
  }

  // Check if user exists with this email
  let user = await prisma.user.findUnique({
    where: { email }
  });

  if (user) {
    // Link new provider to existing user
    await prisma.account.create({
      data: {
        userId: user.id,
        provider,
        providerAccountId,
        accessToken,
        refreshToken,
        expiresAt: new Date(Date.now() + 3600000)
      }
    });

    // Update avatar if not set
    if (!user.avatar && avatar) {
      await prisma.user.update({
        where: { id: user.id },
        data: { avatar }
      });
    }
  } else {
    // Create new user with linked account
    user = await prisma.user.create({
      data: {
        email,
        name,
        avatar,
        emailVerified: true, // OAuth emails are verified
        accounts: {
          create: {
            provider,
            providerAccountId,
            accessToken,
            refreshToken,
            expiresAt: new Date(Date.now() + 3600000)
          }
        }
      }
    });
  }

  return user;
}

module.exports = passport;
```

---

## OAuth Routes

```javascript
// routes/oauth.js
const express = require('express');
const router = express.Router();
const passport = require('../config/passport');
const TokenService = require('../services/tokenService');
const config = require('../config/jwt');

const CLIENT_URL = process.env.CLIENT_URL || 'http://localhost:3000';

// Google OAuth
router.get('/google',
  passport.authenticate('google', {
    scope: ['profile', 'email'],
    session: false
  })
);

router.get('/google/callback',
  passport.authenticate('google', {
    session: false,
    failureRedirect: `${CLIENT_URL}/login?error=google_auth_failed`
  }),
  handleOAuthCallback
);

// GitHub OAuth
router.get('/github',
  passport.authenticate('github', {
    scope: ['user:email'],
    session: false
  })
);

router.get('/github/callback',
  passport.authenticate('github', {
    session: false,
    failureRedirect: `${CLIENT_URL}/login?error=github_auth_failed`
  }),
  handleOAuthCallback
);

// Common callback handler
async function handleOAuthCallback(req, res) {
  try {
    const user = req.user;

    // Generate tokens
    const tokens = TokenService.generateTokenPair(user.id, user.role);

    // Store refresh token
    await TokenService.storeRefreshToken(
      user.id,
      tokens.refreshToken,
      req.headers['user-agent'],
      req.ip
    );

    // Set refresh token cookie
    res.cookie('refreshToken', tokens.refreshToken, config.cookie);

    // Redirect to client with access token
    // Option 1: URL parameter (less secure, but simple)
    res.redirect(`${CLIENT_URL}/auth/callback?token=${tokens.accessToken}`);

    // Option 2: Set in cookie and redirect (more secure)
    // res.cookie('accessToken', tokens.accessToken, { ...config.cookie, httpOnly: false });
    // res.redirect(`${CLIENT_URL}/auth/callback`);
  } catch (error) {
    console.error('OAuth callback error:', error);
    res.redirect(`${CLIENT_URL}/login?error=auth_failed`);
  }
}

// Link additional OAuth provider to existing account
router.get('/link/google',
  authenticateJWT,
  (req, res, next) => {
    // Store user ID in session for linking
    req.session.linkUserId = req.user.userId;
    next();
  },
  passport.authenticate('google', { scope: ['profile', 'email'] })
);

router.get('/link/github',
  authenticateJWT,
  (req, res, next) => {
    req.session.linkUserId = req.user.userId;
    next();
  },
  passport.authenticate('github', { scope: ['user:email'] })
);

// Unlink OAuth provider
router.delete('/unlink/:provider',
  authenticateJWT,
  async (req, res) => {
    try {
      const { provider } = req.params;
      const userId = req.user.userId;

      // Check if user has password or other providers
      const user = await prisma.user.findUnique({
        where: { id: userId },
        include: { accounts: true }
      });

      if (!user.password && user.accounts.length <= 1) {
        return res.status(400).json({
          error: 'Cannot unlink the only login method. Set a password first.'
        });
      }

      await prisma.account.deleteMany({
        where: {
          userId,
          provider
        }
      });

      res.json({ message: `${provider} account unlinked` });
    } catch (error) {
      console.error('Unlink error:', error);
      res.status(500).json({ error: 'Failed to unlink account' });
    }
  }
);

// Get linked accounts
router.get('/accounts',
  authenticateJWT,
  async (req, res) => {
    const accounts = await prisma.account.findMany({
      where: { userId: req.user.userId },
      select: {
        provider: true,
        createdAt: true
      }
    });

    res.json({ accounts });
  }
);

// Helper middleware
function authenticateJWT(req, res, next) {
  const token = req.headers.authorization?.split(' ')[1];
  const decoded = TokenService.verifyAccessToken(token);

  if (!decoded) {
    return res.status(401).json({ error: 'Unauthorized' });
  }

  req.user = { userId: decoded.userId };
  next();
}

module.exports = router;
```

---

## App Integration

```javascript
// app.js
const express = require('express');
const session = require('express-session');
const cookieParser = require('cookie-parser');
const cors = require('cors');
const helmet = require('helmet');
const passport = require('./config/passport');
const authRoutes = require('./routes/auth');
const oauthRoutes = require('./routes/oauth');

const app = express();

// Security
app.use(helmet());
app.use(cors({
  origin: process.env.CLIENT_URL,
  credentials: true
}));

// Parsing
app.use(express.json());
app.use(cookieParser());

// Session (needed for OAuth state)
app.use(session({
  secret: process.env.SESSION_SECRET,
  resave: false,
  saveUninitialized: false,
  cookie: {
    secure: process.env.NODE_ENV === 'production',
    maxAge: 10 * 60 * 1000 // 10 minutes
  }
}));

// Passport
app.use(passport.initialize());

// Routes
app.use('/api/auth', authRoutes);
app.use('/api/auth', oauthRoutes);

module.exports = app;
```

---

## Client-Side Integration

### React OAuth Component

```jsx
// components/SocialLogin.jsx
import { useState } from 'react';

const API_URL = process.env.REACT_APP_API_URL || 'http://localhost:3000';

export function SocialLogin({ onSuccess, onError }) {
  const [loading, setLoading] = useState(null);

  const handleOAuthLogin = (provider) => {
    setLoading(provider);
    // Redirect to OAuth endpoint
    window.location.href = `${API_URL}/api/auth/${provider}`;
  };

  return (
    <div className="social-login">
      <button
        onClick={() => handleOAuthLogin('google')}
        disabled={loading}
        className="oauth-btn google"
      >
        {loading === 'google' ? 'Connecting...' : 'Continue with Google'}
      </button>

      <button
        onClick={() => handleOAuthLogin('github')}
        disabled={loading}
        className="oauth-btn github"
      >
        {loading === 'github' ? 'Connecting...' : 'Continue with GitHub'}
      </button>
    </div>
  );
}
```

### OAuth Callback Handler

```jsx
// pages/AuthCallback.jsx
import { useEffect } from 'react';
import { useNavigate, useSearchParams } from 'react-router-dom';
import { setAccessToken } from '../auth';

export function AuthCallback() {
  const navigate = useNavigate();
  const [searchParams] = useSearchParams();

  useEffect(() => {
    const token = searchParams.get('token');
    const error = searchParams.get('error');

    if (error) {
      navigate(`/login?error=${error}`);
      return;
    }

    if (token) {
      setAccessToken(token);
      navigate('/dashboard');
    } else {
      navigate('/login?error=no_token');
    }
  }, [searchParams, navigate]);

  return (
    <div className="auth-callback">
      <p>Completing authentication...</p>
    </div>
  );
}
```

### Login Page with Social Options

```jsx
// pages/Login.jsx
import { useState } from 'react';
import { useNavigate, useSearchParams } from 'react-router-dom';
import { SocialLogin } from '../components/SocialLogin';
import { login } from '../auth';

export function Login() {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const [error, setError] = useState('');
  const navigate = useNavigate();
  const [searchParams] = useSearchParams();

  // Show OAuth errors
  const oauthError = searchParams.get('error');

  const handleSubmit = async (e) => {
    e.preventDefault();
    try {
      await login(email, password);
      navigate('/dashboard');
    } catch (err) {
      setError('Invalid email or password');
    }
  };

  return (
    <div className="login-page">
      <h1>Login</h1>

      {(error || oauthError) && (
        <div className="error">
          {error || `OAuth error: ${oauthError}`}
        </div>
      )}

      {/* Social Login */}
      <SocialLogin />

      <div className="divider">or</div>

      {/* Email/Password Login */}
      <form onSubmit={handleSubmit}>
        <input
          type="email"
          value={email}
          onChange={(e) => setEmail(e.target.value)}
          placeholder="Email"
          required
        />
        <input
          type="password"
          value={password}
          onChange={(e) => setPassword(e.target.value)}
          placeholder="Password"
          required
        />
        <button type="submit">Login</button>
      </form>
    </div>
  );
}
```

---

## Handling Edge Cases

### Account Linking Conflicts

```javascript
// Handle case where OAuth email already exists with different provider
async function handleOAuthLogin({ provider, providerAccountId, email, name, avatar, accessToken, refreshToken }) {
  // Check if this provider account is already linked
  const existingAccount = await prisma.account.findUnique({
    where: {
      provider_providerAccountId: { provider, providerAccountId }
    },
    include: { user: true }
  });

  if (existingAccount) {
    return { user: existingAccount.user, isNewUser: false };
  }

  // Check if email already exists
  const existingUser = await prisma.user.findUnique({
    where: { email },
    include: { accounts: true }
  });

  if (existingUser) {
    // User exists - link this provider
    const account = await prisma.account.create({
      data: {
        userId: existingUser.id,
        provider,
        providerAccountId,
        accessToken,
        refreshToken
      }
    });

    return { user: existingUser, isNewUser: false, linked: true };
  }

  // Create new user
  const newUser = await prisma.user.create({
    data: {
      email,
      name,
      avatar,
      emailVerified: true,
      accounts: {
        create: {
          provider,
          providerAccountId,
          accessToken,
          refreshToken
        }
      }
    }
  });

  return { user: newUser, isNewUser: true };
}
```

### Requiring Password for OAuth Users

```javascript
// Allow OAuth users to set a password later
router.post('/set-password',
  authenticate,
  async (req, res) => {
    try {
      const { password } = req.body;
      const userId = req.user.userId;

      const user = await prisma.user.findUnique({
        where: { id: userId }
      });

      if (user.password) {
        return res.status(400).json({
          error: 'Password already set. Use change-password instead.'
        });
      }

      const hashedPassword = await bcrypt.hash(password, 12);

      await prisma.user.update({
        where: { id: userId },
        data: { password: hashedPassword }
      });

      res.json({ message: 'Password set successfully' });
    } catch (error) {
      res.status(500).json({ error: 'Failed to set password' });
    }
  }
);
```

---

## Security Best Practices

### 1. State Parameter (CSRF Protection)

```javascript
// Passport handles this automatically, but for manual implementation:
const crypto = require('crypto');

// Generate state
app.get('/auth/google', (req, res) => {
  const state = crypto.randomBytes(16).toString('hex');
  req.session.oauthState = state;

  const params = new URLSearchParams({
    client_id: process.env.GOOGLE_CLIENT_ID,
    redirect_uri: process.env.GOOGLE_CALLBACK_URL,
    response_type: 'code',
    scope: 'profile email',
    state
  });

  res.redirect(`https://accounts.google.com/o/oauth2/v2/auth?${params}`);
});

// Verify state in callback
app.get('/auth/google/callback', (req, res) => {
  if (req.query.state !== req.session.oauthState) {
    return res.status(400).json({ error: 'Invalid state parameter' });
  }

  // Continue with token exchange...
});
```

### 2. Secure Token Exchange

```javascript
// Always exchange code on server side, never client side
async function exchangeCodeForTokens(code) {
  const response = await fetch('https://oauth2.googleapis.com/token', {
    method: 'POST',
    headers: { 'Content-Type': 'application/x-www-form-urlencoded' },
    body: new URLSearchParams({
      code,
      client_id: process.env.GOOGLE_CLIENT_ID,
      client_secret: process.env.GOOGLE_CLIENT_SECRET,  // Never expose to client
      redirect_uri: process.env.GOOGLE_CALLBACK_URL,
      grant_type: 'authorization_code'
    })
  });

  return response.json();
}
```

### 3. Validate OAuth Tokens

```javascript
// Verify Google ID token
const { OAuth2Client } = require('google-auth-library');
const client = new OAuth2Client(process.env.GOOGLE_CLIENT_ID);

async function verifyGoogleToken(idToken) {
  const ticket = await client.verifyIdToken({
    idToken,
    audience: process.env.GOOGLE_CLIENT_ID
  });

  const payload = ticket.getPayload();
  return {
    sub: payload.sub,
    email: payload.email,
    name: payload.name,
    picture: payload.picture,
    emailVerified: payload.email_verified
  };
}
```

---

## Adding More Providers

### Discord OAuth

```javascript
// npm install passport-discord
const DiscordStrategy = require('passport-discord').Strategy;

passport.use(new DiscordStrategy({
    clientID: process.env.DISCORD_CLIENT_ID,
    clientSecret: process.env.DISCORD_CLIENT_SECRET,
    callbackURL: process.env.DISCORD_CALLBACK_URL,
    scope: ['identify', 'email']
  },
  async (accessToken, refreshToken, profile, done) => {
    try {
      const result = await handleOAuthLogin({
        provider: 'discord',
        providerAccountId: profile.id,
        email: profile.email,
        name: profile.username,
        avatar: profile.avatar
          ? `https://cdn.discordapp.com/avatars/${profile.id}/${profile.avatar}.png`
          : null,
        accessToken,
        refreshToken
      });

      done(null, result);
    } catch (error) {
      done(error, null);
    }
  }
));
```

### Twitter OAuth 2.0

```javascript
// npm install passport-twitter-oauth2
const TwitterStrategy = require('passport-twitter-oauth2').Strategy;

passport.use(new TwitterStrategy({
    clientID: process.env.TWITTER_CLIENT_ID,
    clientSecret: process.env.TWITTER_CLIENT_SECRET,
    callbackURL: process.env.TWITTER_CALLBACK_URL,
    scope: ['tweet.read', 'users.read', 'offline.access']
  },
  async (accessToken, refreshToken, profile, done) => {
    // Handle Twitter OAuth...
  }
));
```

---

## Practice Exercise

Build a complete OAuth authentication system:

1. **Set up OAuth providers**:
   - Create Google OAuth credentials
   - Create GitHub OAuth credentials

2. **Implement Passport strategies**:
   - Configure Google and GitHub strategies
   - Handle user creation/linking

3. **Create OAuth routes**:
   - Initiate OAuth flow
   - Handle callbacks
   - Link/unlink accounts

4. **Build client UI**:
   - Social login buttons
   - OAuth callback handler
   - Account linking page

5. **Test scenarios**:
   - New user signs up with Google
   - Existing user links GitHub
   - User unlinking OAuth account

---

## Key Takeaways

1. **OAuth delegates authentication** - Users login with existing accounts
2. **Authorization code flow** - Most secure for server-side apps
3. **Store provider account IDs** - Link OAuth to your users
4. **Handle account conflicts** - Same email, different providers
5. **Secure the flow** - Use state parameter, exchange on server
6. **Allow account linking** - Users may want multiple providers

---

## What's Next?

Tomorrow, we'll learn about **Security Headers** - configuring HTTP headers to protect your application from common web vulnerabilities.

# Days 6-7: Authentication in Next.js

## Introduction

Authentication is crucial for any application that needs to protect resources or personalize user experiences. This lesson covers implementing authentication in Next.js using Auth.js (NextAuth.js v5), including OAuth providers, credentials authentication, session management, and protecting routes.

## Learning Objectives

By the end of this lesson, you will:
- Set up Auth.js in a Next.js application
- Implement OAuth authentication (Google, GitHub)
- Build credentials-based authentication
- Protect routes and API endpoints
- Manage user sessions
- Handle authorization and roles

---

## Setting Up Auth.js

### Installation

```bash
npm install next-auth@beta
```

### Generate Auth Secret

```bash
npx auth secret
```

Add to `.env.local`:

```
AUTH_SECRET=your-generated-secret
```

### Basic Configuration

```tsx
// auth.ts (at project root)
import NextAuth from 'next-auth';
import GitHub from 'next-auth/providers/github';
import Google from 'next-auth/providers/google';

export const { handlers, signIn, signOut, auth } = NextAuth({
  providers: [
    GitHub({
      clientId: process.env.GITHUB_ID,
      clientSecret: process.env.GITHUB_SECRET,
    }),
    Google({
      clientId: process.env.GOOGLE_ID,
      clientSecret: process.env.GOOGLE_SECRET,
    }),
  ],
});
```

### Route Handler

```tsx
// app/api/auth/[...nextauth]/route.ts
import { handlers } from '@/auth';

export const { GET, POST } = handlers;
```

---

## Environment Variables

```
# .env.local

# Auth.js
AUTH_SECRET=your-secret-key

# GitHub OAuth
GITHUB_ID=your-github-client-id
GITHUB_SECRET=your-github-client-secret

# Google OAuth
GOOGLE_ID=your-google-client-id
GOOGLE_SECRET=your-google-client-secret

# Database
DATABASE_URL=your-database-url
```

---

## OAuth Providers

### GitHub Setup

1. Go to GitHub Settings > Developer settings > OAuth Apps
2. Create new OAuth App
3. Set Homepage URL: `http://localhost:3000`
4. Set Callback URL: `http://localhost:3000/api/auth/callback/github`

### Google Setup

1. Go to Google Cloud Console
2. Create new project
3. Enable Google+ API
4. Create OAuth credentials
5. Set Authorized redirect URIs: `http://localhost:3000/api/auth/callback/google`

---

## Sign In / Sign Out

### Server-Side

```tsx
// app/actions.ts
'use server';

import { signIn, signOut } from '@/auth';

export async function handleGitHubSignIn() {
  await signIn('github', { redirectTo: '/dashboard' });
}

export async function handleGoogleSignIn() {
  await signIn('google', { redirectTo: '/dashboard' });
}

export async function handleSignOut() {
  await signOut({ redirectTo: '/' });
}
```

### Sign In Button Component

```tsx
// components/SignInButton.tsx
import { handleGitHubSignIn, handleGoogleSignIn } from '@/app/actions';

export function SignInButtons() {
  return (
    <div className="space-y-4">
      <form action={handleGitHubSignIn}>
        <button type="submit" className="btn btn-github">
          Sign in with GitHub
        </button>
      </form>

      <form action={handleGoogleSignIn}>
        <button type="submit" className="btn btn-google">
          Sign in with Google
        </button>
      </form>
    </div>
  );
}
```

### Sign Out Button

```tsx
// components/SignOutButton.tsx
import { handleSignOut } from '@/app/actions';

export function SignOutButton() {
  return (
    <form action={handleSignOut}>
      <button type="submit">Sign Out</button>
    </form>
  );
}
```

---

## Getting Session Data

### Server Component

```tsx
// app/dashboard/page.tsx
import { auth } from '@/auth';
import { redirect } from 'next/navigation';

export default async function DashboardPage() {
  const session = await auth();

  if (!session) {
    redirect('/login');
  }

  return (
    <div>
      <h1>Dashboard</h1>
      <p>Welcome, {session.user?.name}!</p>
      <img src={session.user?.image} alt="Profile" />
    </div>
  );
}
```

### Client Component

```tsx
// components/UserInfo.tsx
'use client';

import { useSession } from 'next-auth/react';

export function UserInfo() {
  const { data: session, status } = useSession();

  if (status === 'loading') {
    return <div>Loading...</div>;
  }

  if (status === 'unauthenticated') {
    return <div>Not signed in</div>;
  }

  return (
    <div>
      <img src={session?.user?.image} alt="Profile" />
      <p>{session?.user?.name}</p>
      <p>{session?.user?.email}</p>
    </div>
  );
}
```

### Session Provider Setup

```tsx
// components/Providers.tsx
'use client';

import { SessionProvider } from 'next-auth/react';

export function Providers({ children }: { children: React.ReactNode }) {
  return <SessionProvider>{children}</SessionProvider>;
}

// app/layout.tsx
import { Providers } from '@/components/Providers';

export default function RootLayout({ children }) {
  return (
    <html>
      <body>
        <Providers>{children}</Providers>
      </body>
    </html>
  );
}
```

---

## Credentials Authentication

### Setup with Database

```tsx
// auth.ts
import NextAuth from 'next-auth';
import Credentials from 'next-auth/providers/credentials';
import { PrismaAdapter } from '@auth/prisma-adapter';
import { prisma } from '@/lib/prisma';
import bcrypt from 'bcryptjs';
import { z } from 'zod';

const credentialsSchema = z.object({
  email: z.string().email(),
  password: z.string().min(8),
});

export const { handlers, signIn, signOut, auth } = NextAuth({
  adapter: PrismaAdapter(prisma),
  session: { strategy: 'jwt' },
  providers: [
    Credentials({
      credentials: {
        email: { label: 'Email', type: 'email' },
        password: { label: 'Password', type: 'password' },
      },
      async authorize(credentials) {
        const parsed = credentialsSchema.safeParse(credentials);

        if (!parsed.success) {
          return null;
        }

        const { email, password } = parsed.data;

        const user = await prisma.user.findUnique({
          where: { email },
        });

        if (!user || !user.password) {
          return null;
        }

        const passwordMatch = await bcrypt.compare(password, user.password);

        if (!passwordMatch) {
          return null;
        }

        return {
          id: user.id,
          email: user.email,
          name: user.name,
          image: user.image,
        };
      },
    }),
  ],
  pages: {
    signIn: '/login',
    error: '/login',
  },
});
```

### Prisma Schema

```prisma
// prisma/schema.prisma
model User {
  id            String    @id @default(cuid())
  name          String?
  email         String    @unique
  emailVerified DateTime?
  image         String?
  password      String?
  accounts      Account[]
  sessions      Session[]
  createdAt     DateTime  @default(now())
  updatedAt     DateTime  @updatedAt
}

model Account {
  id                String  @id @default(cuid())
  userId            String
  type              String
  provider          String
  providerAccountId String
  refresh_token     String? @db.Text
  access_token      String? @db.Text
  expires_at        Int?
  token_type        String?
  scope             String?
  id_token          String? @db.Text
  session_state     String?
  user              User    @relation(fields: [userId], references: [id], onDelete: Cascade)

  @@unique([provider, providerAccountId])
}

model Session {
  id           String   @id @default(cuid())
  sessionToken String   @unique
  userId       String
  expires      DateTime
  user         User     @relation(fields: [userId], references: [id], onDelete: Cascade)
}
```

### Login Page

```tsx
// app/login/page.tsx
'use client';

import { useState } from 'react';
import { useFormState, useFormStatus } from 'react-dom';
import { authenticate } from '@/app/actions';
import Link from 'next/link';

function SubmitButton() {
  const { pending } = useFormStatus();
  return (
    <button type="submit" disabled={pending} className="btn btn-primary w-full">
      {pending ? 'Signing in...' : 'Sign In'}
    </button>
  );
}

export default function LoginPage() {
  const [state, formAction] = useFormState(authenticate, null);

  return (
    <div className="max-w-md mx-auto mt-20 p-6">
      <h1 className="text-2xl font-bold mb-6">Sign In</h1>

      <form action={formAction} className="space-y-4">
        <div>
          <label htmlFor="email">Email</label>
          <input
            id="email"
            name="email"
            type="email"
            required
            className="input"
          />
        </div>

        <div>
          <label htmlFor="password">Password</label>
          <input
            id="password"
            name="password"
            type="password"
            required
            className="input"
          />
        </div>

        {state?.error && (
          <p className="text-red-500">{state.error}</p>
        )}

        <SubmitButton />
      </form>

      <p className="mt-4 text-center">
        Don't have an account? <Link href="/register">Register</Link>
      </p>
    </div>
  );
}
```

### Authentication Action

```tsx
// app/actions.ts
'use server';

import { signIn } from '@/auth';
import { AuthError } from 'next-auth';

export async function authenticate(prevState: any, formData: FormData) {
  try {
    await signIn('credentials', {
      email: formData.get('email'),
      password: formData.get('password'),
      redirectTo: '/dashboard',
    });
  } catch (error) {
    if (error instanceof AuthError) {
      switch (error.type) {
        case 'CredentialsSignin':
          return { error: 'Invalid email or password' };
        default:
          return { error: 'Something went wrong' };
      }
    }
    throw error;
  }
}
```

### Registration

```tsx
// app/actions.ts
'use server';

import { z } from 'zod';
import bcrypt from 'bcryptjs';
import { prisma } from '@/lib/prisma';
import { redirect } from 'next/navigation';

const RegisterSchema = z.object({
  name: z.string().min(2),
  email: z.string().email(),
  password: z.string().min(8),
});

export async function register(prevState: any, formData: FormData) {
  const validatedFields = RegisterSchema.safeParse({
    name: formData.get('name'),
    email: formData.get('email'),
    password: formData.get('password'),
  });

  if (!validatedFields.success) {
    return {
      errors: validatedFields.error.flatten().fieldErrors,
    };
  }

  const { name, email, password } = validatedFields.data;

  // Check if user exists
  const existingUser = await prisma.user.findUnique({
    where: { email },
  });

  if (existingUser) {
    return { error: 'Email already in use' };
  }

  // Hash password
  const hashedPassword = await bcrypt.hash(password, 10);

  // Create user
  await prisma.user.create({
    data: {
      name,
      email,
      password: hashedPassword,
    },
  });

  redirect('/login?registered=true');
}
```

---

## Protecting Routes

### Middleware

```tsx
// middleware.ts
import { auth } from '@/auth';
import { NextResponse } from 'next/server';

export default auth((req) => {
  const { nextUrl } = req;
  const isLoggedIn = !!req.auth;

  const isAuthPage = nextUrl.pathname.startsWith('/login') ||
                     nextUrl.pathname.startsWith('/register');

  const isProtectedPage = nextUrl.pathname.startsWith('/dashboard') ||
                          nextUrl.pathname.startsWith('/settings');

  const isApiAuthRoute = nextUrl.pathname.startsWith('/api/auth');

  // Allow auth API routes
  if (isApiAuthRoute) {
    return NextResponse.next();
  }

  // Redirect logged-in users away from auth pages
  if (isAuthPage && isLoggedIn) {
    return NextResponse.redirect(new URL('/dashboard', nextUrl));
  }

  // Redirect unauthenticated users to login
  if (isProtectedPage && !isLoggedIn) {
    const callbackUrl = encodeURIComponent(nextUrl.pathname);
    return NextResponse.redirect(
      new URL(`/login?callbackUrl=${callbackUrl}`, nextUrl)
    );
  }

  return NextResponse.next();
});

export const config = {
  matcher: ['/((?!_next/static|_next/image|favicon.ico).*)'],
};
```

### Page-Level Protection

```tsx
// app/dashboard/page.tsx
import { auth } from '@/auth';
import { redirect } from 'next/navigation';

export default async function DashboardPage() {
  const session = await auth();

  if (!session) {
    redirect('/login');
  }

  return <div>Protected Dashboard</div>;
}
```

### API Route Protection

```tsx
// app/api/protected/route.ts
import { auth } from '@/auth';
import { NextResponse } from 'next/server';

export async function GET() {
  const session = await auth();

  if (!session) {
    return NextResponse.json(
      { error: 'Unauthorized' },
      { status: 401 }
    );
  }

  return NextResponse.json({ data: 'Protected data' });
}
```

---

## Role-Based Access Control

### Extended User Type

```tsx
// types/next-auth.d.ts
import { DefaultSession, DefaultUser } from 'next-auth';

declare module 'next-auth' {
  interface Session {
    user: {
      id: string;
      role: 'USER' | 'ADMIN';
    } & DefaultSession['user'];
  }

  interface User extends DefaultUser {
    role: 'USER' | 'ADMIN';
  }
}
```

### Auth Config with Callbacks

```tsx
// auth.ts
export const { handlers, signIn, signOut, auth } = NextAuth({
  // ... providers

  callbacks: {
    async jwt({ token, user }) {
      if (user) {
        token.role = user.role;
        token.id = user.id;
      }
      return token;
    },
    async session({ session, token }) {
      if (session.user) {
        session.user.role = token.role as 'USER' | 'ADMIN';
        session.user.id = token.id as string;
      }
      return session;
    },
  },
});
```

### Admin Route Protection

```tsx
// app/admin/page.tsx
import { auth } from '@/auth';
import { redirect } from 'next/navigation';

export default async function AdminPage() {
  const session = await auth();

  if (!session || session.user.role !== 'ADMIN') {
    redirect('/unauthorized');
  }

  return <div>Admin Dashboard</div>;
}
```

### Role-Based Component

```tsx
// components/RoleGate.tsx
import { auth } from '@/auth';

interface RoleGateProps {
  children: React.ReactNode;
  allowedRoles: ('USER' | 'ADMIN')[];
  fallback?: React.ReactNode;
}

export async function RoleGate({
  children,
  allowedRoles,
  fallback = null,
}: RoleGateProps) {
  const session = await auth();

  if (!session || !allowedRoles.includes(session.user.role)) {
    return fallback;
  }

  return children;
}

// Usage
<RoleGate allowedRoles={['ADMIN']} fallback={<p>Access denied</p>}>
  <AdminPanel />
</RoleGate>
```

---

## Session Management

### Custom Session Data

```tsx
// auth.ts
callbacks: {
  async jwt({ token, user, account, profile }) {
    // First sign in
    if (user) {
      token.id = user.id;
      token.role = user.role;
    }

    // Access token refresh for OAuth
    if (account) {
      token.accessToken = account.access_token;
      token.refreshToken = account.refresh_token;
      token.expiresAt = account.expires_at;
    }

    return token;
  },

  async session({ session, token }) {
    session.user.id = token.id;
    session.user.role = token.role;
    session.accessToken = token.accessToken;
    return session;
  },
}
```

### Session Expiry

```tsx
// auth.ts
export const { handlers, signIn, signOut, auth } = NextAuth({
  // ... other config

  session: {
    strategy: 'jwt',
    maxAge: 30 * 24 * 60 * 60, // 30 days
    updateAge: 24 * 60 * 60, // 24 hours
  },

  jwt: {
    maxAge: 30 * 24 * 60 * 60, // 30 days
  },
});
```

---

## Complete Example: Auth System

### File Structure

```
app/
├── (auth)/
│   ├── login/
│   │   └── page.tsx
│   ├── register/
│   │   └── page.tsx
│   └── layout.tsx
├── (protected)/
│   ├── dashboard/
│   │   └── page.tsx
│   ├── settings/
│   │   └── page.tsx
│   └── layout.tsx
├── api/
│   └── auth/
│       └── [...nextauth]/
│           └── route.ts
├── actions.ts
└── layout.tsx
auth.ts
middleware.ts
```

### Auth Layout

```tsx
// app/(auth)/layout.tsx
import { auth } from '@/auth';
import { redirect } from 'next/navigation';

export default async function AuthLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  const session = await auth();

  if (session) {
    redirect('/dashboard');
  }

  return (
    <div className="min-h-screen flex items-center justify-center">
      {children}
    </div>
  );
}
```

### Protected Layout

```tsx
// app/(protected)/layout.tsx
import { auth } from '@/auth';
import { redirect } from 'next/navigation';
import { Sidebar } from '@/components/Sidebar';
import { Header } from '@/components/Header';

export default async function ProtectedLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  const session = await auth();

  if (!session) {
    redirect('/login');
  }

  return (
    <div className="flex h-screen">
      <Sidebar />
      <div className="flex-1 flex flex-col">
        <Header user={session.user} />
        <main className="flex-1 p-6">{children}</main>
      </div>
    </div>
  );
}
```

### Header with User Menu

```tsx
// components/Header.tsx
import { SignOutButton } from './SignOutButton';

interface HeaderProps {
  user: {
    name?: string | null;
    email?: string | null;
    image?: string | null;
  };
}

export function Header({ user }: HeaderProps) {
  return (
    <header className="bg-white border-b px-6 py-4 flex justify-between">
      <h1 className="text-xl font-semibold">Dashboard</h1>

      <div className="flex items-center gap-4">
        <div className="text-right">
          <p className="font-medium">{user.name}</p>
          <p className="text-sm text-gray-500">{user.email}</p>
        </div>
        {user.image && (
          <img
            src={user.image}
            alt="Profile"
            className="w-10 h-10 rounded-full"
          />
        )}
        <SignOutButton />
      </div>
    </header>
  );
}
```

---

## Key Takeaways

1. **Auth.js (NextAuth.js v5)** is the standard for Next.js authentication
2. **OAuth providers** are easy to set up (GitHub, Google, etc.)
3. **Credentials authentication** requires password hashing and validation
4. **Middleware** provides centralized route protection
5. **Session callbacks** customize token and session data
6. **Role-based access** enables fine-grained permissions
7. **Prisma Adapter** integrates with databases seamlessly

---

## What's Next?

Tomorrow we'll cover **Deployment** - deploying your Next.js application to Vercel and other platforms!

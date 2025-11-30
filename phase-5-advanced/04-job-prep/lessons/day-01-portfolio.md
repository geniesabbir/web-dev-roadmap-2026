# Day 1: Building a Standout Developer Portfolio

## Introduction

Your portfolio is often the first impression you make on potential employers. A well-crafted portfolio showcases your skills, demonstrates your ability to ship projects, and tells your story as a developer. Today, we'll build a portfolio that stands out and gets you interviews.

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    Portfolio Success Formula                            │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│     ┌─────────────┐     ┌─────────────┐     ┌─────────────┐            │
│     │   Strong    │     │  Polished   │     │   Clear     │            │
│     │  Projects   │  +  │   Design    │  +  │   Story     │  = HIRED   │
│     └─────────────┘     └─────────────┘     └─────────────┘            │
│                                                                         │
│  Quality > Quantity    Professional UX    Your unique journey          │
│  Real-world impact     Fast, accessible   Career narrative             │
│  Technical depth       Mobile-friendly    Personality shine            │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

## Learning Objectives

By the end of this lesson, you will:
- Understand what makes portfolios effective
- Choose and present projects strategically
- Build a modern, performant portfolio site
- Write compelling project descriptions
- Optimize for recruiter and hiring manager review

---

## 1. Portfolio Strategy

### What Hiring Managers Look For

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    Hiring Manager Priorities                            │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  FIRST 10 SECONDS (Quick Scan)                                          │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │  • Does it load fast?                                            │   │
│  │  • Is it visually professional?                                  │   │
│  │  • Can I quickly understand who this person is?                  │   │
│  │  • Are there real, working projects?                             │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                                                         │
│  NEXT 60 SECONDS (Project Review)                                       │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │  • Do projects demonstrate relevant skills?                      │   │
│  │  • Is there evidence of problem-solving?                         │   │
│  │  • Can I see the code? Is it clean?                              │   │
│  │  • Are projects deployed and working?                            │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                                                         │
│  DEEP DIVE (If Interested)                                              │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │  • Technical decision-making ability                             │   │
│  │  • Code quality and architecture                                 │   │
│  │  • Documentation and communication                               │   │
│  │  • Growth trajectory and learning                                │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### Portfolio Anti-Patterns to Avoid

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         What NOT to Do                                  │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  ❌ TOO MANY TUTORIAL PROJECTS                                          │
│     "Todo App", "Calculator", "Weather App" with no unique spin         │
│     → Shows you can follow instructions, not build things               │
│                                                                         │
│  ❌ BROKEN LINKS AND DEMOS                                              │
│     Links that 404, demos that crash on load                            │
│     → Signals lack of attention to detail                               │
│                                                                         │
│  ❌ NO LIVE DEMOS                                                       │
│     Just GitHub links, no deployed versions                             │
│     → Makes it hard for non-technical reviewers                         │
│                                                                         │
│  ❌ WALLS OF TEXT                                                       │
│     Long paragraphs explaining every decision                           │
│     → Busy people won't read it all                                     │
│                                                                         │
│  ❌ OUTDATED TECH STACK                                                 │
│     jQuery, Bootstrap 3, no TypeScript, class components                │
│     → Suggests you're not keeping up with the industry                  │
│                                                                         │
│  ❌ NO PERSONALITY                                                      │
│     Generic template with no personal touches                           │
│     → Forgettable among hundreds of applicants                          │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 2. Selecting Projects

### The Ideal Portfolio Project Mix

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    Recommended Project Mix (3-5 Projects)               │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  1. FLAGSHIP PROJECT (Your Best Work)                                   │
│     ┌─────────────────────────────────────────────────────────────┐    │
│     │  • Full-stack application                                    │    │
│     │  • Demonstrates end-to-end skills                            │    │
│     │  • Polished UI/UX                                            │    │
│     │  • Has real users or solves real problem                     │    │
│     │  • Well-documented with README                               │    │
│     └─────────────────────────────────────────────────────────────┘    │
│                                                                         │
│  2. TECHNICAL DEPTH PROJECT                                             │
│     ┌─────────────────────────────────────────────────────────────┐    │
│     │  • Shows mastery of specific technology                      │    │
│     │  • Complex problem-solving                                   │    │
│     │  • Performance optimization                                  │    │
│     │  • Examples: Real-time app, AI integration, complex API      │    │
│     └─────────────────────────────────────────────────────────────┘    │
│                                                                         │
│  3. OPEN SOURCE OR COLLABORATION                                        │
│     ┌─────────────────────────────────────────────────────────────┐    │
│     │  • Shows you can work with others                            │    │
│     │  • Contribution to popular library                           │    │
│     │  • Team project with clear your contribution                 │    │
│     │  • Demonstrates communication skills                         │    │
│     └─────────────────────────────────────────────────────────────┘    │
│                                                                         │
│  4. PASSION/CREATIVE PROJECT (Optional)                                 │
│     ┌─────────────────────────────────────────────────────────────┐    │
│     │  • Something unique to you                                   │    │
│     │  • Shows personality and creativity                          │    │
│     │  • Conversation starter in interviews                        │    │
│     │  • Examples: Game, art tool, niche community tool            │    │
│     └─────────────────────────────────────────────────────────────┘    │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### Project Ideas That Stand Out

```typescript
// Examples of portfolio-worthy projects

// 1. SaaS Application (Full-Stack)
// - Multi-tenant architecture
// - Stripe integration for payments
// - User authentication and authorization
// - Dashboard with analytics
// Example: Project management tool, Invoice generator, Booking system

// 2. Real-Time Application
// - WebSocket or Server-Sent Events
// - Live collaboration features
// - Optimistic updates
// Example: Collaborative whiteboard, Live code editor, Chat application

// 3. AI-Powered Application
// - Integration with LLM APIs
// - RAG implementation
// - Streaming responses
// Example: Document Q&A, AI writing assistant, Code reviewer

// 4. Developer Tool
// - CLI application
// - VS Code extension
// - GitHub Action
// Example: Code generator, Documentation tool, Linting tool

// 5. E-Commerce Platform
// - Product catalog with search/filter
// - Shopping cart and checkout
// - Order management
// Example: Niche marketplace, Digital product store
```

---

## 3. Building Your Portfolio Site

### Modern Portfolio Architecture

```typescript
// Recommended tech stack for portfolio site
// Next.js + TypeScript + Tailwind CSS + MDX

// Project structure
/*
portfolio/
├── app/
│   ├── page.tsx           # Homepage
│   ├── projects/
│   │   ├── page.tsx       # Project list
│   │   └── [slug]/
│   │       └── page.tsx   # Individual project
│   ├── about/
│   │   └── page.tsx       # About page
│   └── blog/              # Optional blog
│       ├── page.tsx
│       └── [slug]/
│           └── page.tsx
├── components/
│   ├── Hero.tsx
│   ├── ProjectCard.tsx
│   ├── Skills.tsx
│   └── Contact.tsx
├── content/
│   └── projects/          # MDX project descriptions
│       ├── project-1.mdx
│       └── project-2.mdx
└── lib/
    └── projects.ts        # Project data utilities
*/
```

### Hero Section Component

```tsx
// components/Hero.tsx
import Link from 'next/link';
import { ArrowRight, Github, Linkedin, Mail } from 'lucide-react';

export function Hero() {
  return (
    <section className="min-h-[80vh] flex items-center">
      <div className="max-w-4xl">
        {/* Quick identifier */}
        <p className="text-blue-600 font-medium mb-4">
          Full-Stack Developer
        </p>

        {/* Name and tagline - Most important */}
        <h1 className="text-5xl md:text-6xl font-bold text-gray-900 mb-6">
          Hi, I'm <span className="text-blue-600">Your Name</span>
        </h1>

        {/* Value proposition - Keep it concise */}
        <p className="text-xl text-gray-600 mb-8 max-w-2xl">
          I build exceptional web applications with React and Node.js.
          Currently focused on creating AI-powered tools that help
          developers work smarter.
        </p>

        {/* Clear CTAs */}
        <div className="flex flex-wrap gap-4 mb-12">
          <Link
            href="/projects"
            className="inline-flex items-center gap-2 px-6 py-3 bg-blue-600 text-white rounded-lg hover:bg-blue-700 transition-colors"
          >
            View My Work
            <ArrowRight className="w-4 h-4" />
          </Link>
          <Link
            href="/contact"
            className="inline-flex items-center gap-2 px-6 py-3 border border-gray-300 rounded-lg hover:bg-gray-50 transition-colors"
          >
            Get in Touch
          </Link>
        </div>

        {/* Social links */}
        <div className="flex gap-4">
          <a
            href="https://github.com/yourusername"
            target="_blank"
            rel="noopener noreferrer"
            className="p-2 text-gray-600 hover:text-gray-900 transition-colors"
            aria-label="GitHub"
          >
            <Github className="w-6 h-6" />
          </a>
          <a
            href="https://linkedin.com/in/yourusername"
            target="_blank"
            rel="noopener noreferrer"
            className="p-2 text-gray-600 hover:text-gray-900 transition-colors"
            aria-label="LinkedIn"
          >
            <Linkedin className="w-6 h-6" />
          </a>
          <a
            href="mailto:you@email.com"
            className="p-2 text-gray-600 hover:text-gray-900 transition-colors"
            aria-label="Email"
          >
            <Mail className="w-6 h-6" />
          </a>
        </div>
      </div>
    </section>
  );
}
```

### Project Card Component

```tsx
// components/ProjectCard.tsx
import Image from 'next/image';
import Link from 'next/link';
import { ExternalLink, Github } from 'lucide-react';

interface Project {
  slug: string;
  title: string;
  description: string;
  image: string;
  tags: string[];
  liveUrl?: string;
  githubUrl?: string;
  featured?: boolean;
}

export function ProjectCard({ project }: { project: Project }) {
  return (
    <article
      className={`group bg-white rounded-xl border border-gray-200 overflow-hidden hover:shadow-lg transition-shadow ${
        project.featured ? 'md:col-span-2' : ''
      }`}
    >
      {/* Project image */}
      <Link href={`/projects/${project.slug}`}>
        <div className="relative aspect-video overflow-hidden bg-gray-100">
          <Image
            src={project.image}
            alt={project.title}
            fill
            className="object-cover group-hover:scale-105 transition-transform duration-300"
          />
        </div>
      </Link>

      <div className="p-6">
        {/* Title */}
        <Link href={`/projects/${project.slug}`}>
          <h3 className="text-xl font-semibold text-gray-900 mb-2 group-hover:text-blue-600 transition-colors">
            {project.title}
          </h3>
        </Link>

        {/* Description - Keep it short */}
        <p className="text-gray-600 mb-4 line-clamp-2">
          {project.description}
        </p>

        {/* Tech stack tags */}
        <div className="flex flex-wrap gap-2 mb-4">
          {project.tags.slice(0, 4).map((tag) => (
            <span
              key={tag}
              className="px-2 py-1 text-xs font-medium bg-gray-100 text-gray-600 rounded"
            >
              {tag}
            </span>
          ))}
        </div>

        {/* Links */}
        <div className="flex gap-4">
          {project.liveUrl && (
            <a
              href={project.liveUrl}
              target="_blank"
              rel="noopener noreferrer"
              className="inline-flex items-center gap-1 text-sm text-blue-600 hover:text-blue-700"
            >
              <ExternalLink className="w-4 h-4" />
              Live Demo
            </a>
          )}
          {project.githubUrl && (
            <a
              href={project.githubUrl}
              target="_blank"
              rel="noopener noreferrer"
              className="inline-flex items-center gap-1 text-sm text-gray-600 hover:text-gray-900"
            >
              <Github className="w-4 h-4" />
              Source Code
            </a>
          )}
        </div>
      </div>
    </article>
  );
}
```

### Individual Project Page

```tsx
// app/projects/[slug]/page.tsx
import { getProjectBySlug, getAllProjects } from '@/lib/projects';
import Image from 'next/image';
import Link from 'next/link';
import { ArrowLeft, ExternalLink, Github } from 'lucide-react';
import { notFound } from 'next/navigation';

export async function generateStaticParams() {
  const projects = await getAllProjects();
  return projects.map((project) => ({ slug: project.slug }));
}

export default async function ProjectPage({
  params,
}: {
  params: { slug: string };
}) {
  const project = await getProjectBySlug(params.slug);

  if (!project) {
    notFound();
  }

  return (
    <main className="max-w-4xl mx-auto px-4 py-16">
      {/* Back link */}
      <Link
        href="/projects"
        className="inline-flex items-center gap-2 text-gray-600 hover:text-gray-900 mb-8"
      >
        <ArrowLeft className="w-4 h-4" />
        Back to Projects
      </Link>

      {/* Header */}
      <header className="mb-12">
        <h1 className="text-4xl font-bold text-gray-900 mb-4">
          {project.title}
        </h1>
        <p className="text-xl text-gray-600 mb-6">{project.description}</p>

        {/* Links */}
        <div className="flex gap-4">
          {project.liveUrl && (
            <a
              href={project.liveUrl}
              target="_blank"
              rel="noopener noreferrer"
              className="inline-flex items-center gap-2 px-4 py-2 bg-blue-600 text-white rounded-lg hover:bg-blue-700"
            >
              <ExternalLink className="w-4 h-4" />
              View Live
            </a>
          )}
          {project.githubUrl && (
            <a
              href={project.githubUrl}
              target="_blank"
              rel="noopener noreferrer"
              className="inline-flex items-center gap-2 px-4 py-2 border border-gray-300 rounded-lg hover:bg-gray-50"
            >
              <Github className="w-4 h-4" />
              Source Code
            </a>
          )}
        </div>
      </header>

      {/* Hero image */}
      <div className="relative aspect-video rounded-xl overflow-hidden mb-12">
        <Image
          src={project.image}
          alt={project.title}
          fill
          className="object-cover"
          priority
        />
      </div>

      {/* Project details */}
      <div className="grid md:grid-cols-3 gap-8 mb-12">
        <div>
          <h3 className="font-semibold text-gray-900 mb-2">Tech Stack</h3>
          <div className="flex flex-wrap gap-2">
            {project.tags.map((tag) => (
              <span
                key={tag}
                className="px-2 py-1 text-sm bg-gray-100 text-gray-700 rounded"
              >
                {tag}
              </span>
            ))}
          </div>
        </div>
        <div>
          <h3 className="font-semibold text-gray-900 mb-2">Timeline</h3>
          <p className="text-gray-600">{project.timeline}</p>
        </div>
        <div>
          <h3 className="font-semibold text-gray-900 mb-2">Role</h3>
          <p className="text-gray-600">{project.role}</p>
        </div>
      </div>

      {/* Main content - Problem, Solution, Results */}
      <article className="prose prose-lg max-w-none">
        <h2>The Problem</h2>
        <p>{project.problem}</p>

        <h2>The Solution</h2>
        <p>{project.solution}</p>

        {/* Key features with screenshots */}
        <h2>Key Features</h2>
        {project.features.map((feature, index) => (
          <div key={index} className="mb-8">
            <h3>{feature.title}</h3>
            <p>{feature.description}</p>
            {feature.image && (
              <Image
                src={feature.image}
                alt={feature.title}
                width={800}
                height={450}
                className="rounded-lg"
              />
            )}
          </div>
        ))}

        {/* Technical highlights */}
        <h2>Technical Highlights</h2>
        <ul>
          {project.technicalHighlights.map((highlight, index) => (
            <li key={index}>{highlight}</li>
          ))}
        </ul>

        {/* Results/Impact */}
        {project.results && (
          <>
            <h2>Results</h2>
            <p>{project.results}</p>
          </>
        )}

        {/* Lessons learned */}
        <h2>What I Learned</h2>
        <p>{project.lessons}</p>
      </article>
    </main>
  );
}
```

---

## 4. Writing Compelling Project Descriptions

### The STAR Framework for Projects

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    STAR Framework for Projects                          │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  S - SITUATION                                                          │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │  What problem existed? Who was affected?                         │   │
│  │  Example: "Small businesses struggle to manage invoices          │   │
│  │  efficiently, leading to cash flow problems."                    │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                                                         │
│  T - TASK                                                               │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │  What was your goal? What did you set out to build?              │   │
│  │  Example: "I built an invoice management platform that           │   │
│  │  automates billing and payment reminders."                       │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                                                         │
│  A - ACTION                                                             │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │  What technologies did you use? What challenges did you solve?   │   │
│  │  Example: "Used Next.js for the frontend, Prisma + PostgreSQL    │   │
│  │  for data, and implemented Stripe for payments."                 │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                                                         │
│  R - RESULT                                                             │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │  What was the outcome? Any metrics or user feedback?             │   │
│  │  Example: "Platform now has 50+ active users processing          │   │
│  │  $10k+ monthly in invoices."                                     │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### Example Project Writeup

```markdown
# InvoiceFlow - Invoice Management Platform

## Overview
InvoiceFlow is a modern invoicing platform designed for freelancers and small
businesses. It streamlines the entire billing workflow from creating invoices
to tracking payments and sending automated reminders.

## The Problem
Many freelancers and small business owners spend hours each week on invoicing
tasks. Manual processes lead to late payments, missed follow-ups, and poor
cash flow visibility.

## The Solution
I built InvoiceFlow to automate the invoicing workflow:
- Create professional invoices in seconds with customizable templates
- Automatic payment reminders sent at configurable intervals
- Real-time dashboard showing outstanding amounts and payment history
- Stripe integration for instant online payments

## Technical Highlights

### Architecture
- **Frontend**: Next.js 14 with App Router, TypeScript, Tailwind CSS
- **Backend**: Next.js API routes with tRPC for type-safe APIs
- **Database**: PostgreSQL with Prisma ORM
- **Payments**: Stripe Connect for multi-party payments
- **Email**: Resend for transactional emails

### Key Technical Decisions

**Why tRPC over REST?**
Type-safety across the full stack eliminated an entire class of bugs and
improved development speed significantly.

**Why Stripe Connect?**
Allows users to receive payments directly while the platform takes a small
fee, enabling a sustainable business model.

### Performance Optimizations
- Implemented optimistic updates for instant UI feedback
- Used React Query for intelligent caching and background refetching
- Server Components reduce client-side JavaScript by 40%

## Results
- 50+ active users within 3 months of launch
- $10,000+ in invoices processed monthly
- Average invoice payment time reduced from 14 days to 5 days

## What I Learned
This project taught me the importance of understanding the business domain
deeply. Initially, I over-engineered features that users didn't need.
After conducting user interviews, I simplified the UI and focused on the
core workflow, which dramatically improved adoption.
```

---

## 5. Portfolio Performance & SEO

### Technical Optimization

```typescript
// next.config.js
/** @type {import('next').NextConfig} */
const nextConfig = {
  images: {
    formats: ['image/avif', 'image/webp'],
    deviceSizes: [640, 750, 828, 1080, 1200],
  },
  experimental: {
    optimizeCss: true,
  },
};

module.exports = nextConfig;

// app/layout.tsx - SEO metadata
import type { Metadata } from 'next';

export const metadata: Metadata = {
  metadataBase: new URL('https://yourname.dev'),
  title: {
    default: 'Your Name - Full-Stack Developer',
    template: '%s | Your Name',
  },
  description:
    'Full-stack developer specializing in React, Node.js, and TypeScript. Building modern web applications and AI-powered tools.',
  keywords: [
    'Full-Stack Developer',
    'React Developer',
    'Node.js',
    'TypeScript',
    'Web Developer',
    'Your City', // Local SEO
  ],
  authors: [{ name: 'Your Name' }],
  creator: 'Your Name',
  openGraph: {
    type: 'website',
    locale: 'en_US',
    url: 'https://yourname.dev',
    siteName: 'Your Name - Developer Portfolio',
    title: 'Your Name - Full-Stack Developer',
    description: 'Building exceptional web applications',
    images: [
      {
        url: '/og-image.png',
        width: 1200,
        height: 630,
        alt: 'Your Name - Full-Stack Developer',
      },
    ],
  },
  twitter: {
    card: 'summary_large_image',
    title: 'Your Name - Full-Stack Developer',
    description: 'Building exceptional web applications',
    creator: '@yourusername',
    images: ['/og-image.png'],
  },
  robots: {
    index: true,
    follow: true,
    googleBot: {
      index: true,
      follow: true,
      'max-video-preview': -1,
      'max-image-preview': 'large',
      'max-snippet': -1,
    },
  },
};
```

### Performance Checklist

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    Portfolio Performance Checklist                      │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  LOADING SPEED                                                          │
│  □ First Contentful Paint < 1.5s                                        │
│  □ Largest Contentful Paint < 2.5s                                      │
│  □ Time to Interactive < 3.5s                                           │
│  □ Total bundle size < 200KB (gzipped)                                  │
│                                                                         │
│  IMAGES                                                                 │
│  □ Use next/image for automatic optimization                            │
│  □ Serve WebP/AVIF formats                                              │
│  □ Implement lazy loading for below-fold images                         │
│  □ Add blur placeholders for better UX                                  │
│                                                                         │
│  ACCESSIBILITY                                                          │
│  □ WCAG 2.1 AA compliance                                               │
│  □ Keyboard navigation works                                            │
│  □ Screen reader friendly                                               │
│  □ Color contrast passes                                                │
│  □ Alt text on all images                                               │
│                                                                         │
│  MOBILE                                                                 │
│  □ Responsive at all breakpoints                                        │
│  □ Touch targets 44px minimum                                           │
│  □ No horizontal scroll                                                 │
│  □ Text readable without zooming                                        │
│                                                                         │
│  SEO                                                                    │
│  □ Proper meta tags                                                     │
│  □ Open Graph images                                                    │
│  □ Sitemap.xml                                                          │
│  □ robots.txt                                                           │
│  □ Structured data (JSON-LD)                                            │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### Structured Data for Portfolio

```tsx
// components/StructuredData.tsx
export function PersonStructuredData() {
  const structuredData = {
    '@context': 'https://schema.org',
    '@type': 'Person',
    name: 'Your Name',
    url: 'https://yourname.dev',
    image: 'https://yourname.dev/headshot.jpg',
    jobTitle: 'Full-Stack Developer',
    worksFor: {
      '@type': 'Organization',
      name: 'Freelance',
    },
    sameAs: [
      'https://github.com/yourusername',
      'https://linkedin.com/in/yourusername',
      'https://twitter.com/yourusername',
    ],
    knowsAbout: [
      'React',
      'Node.js',
      'TypeScript',
      'Next.js',
      'PostgreSQL',
    ],
  };

  return (
    <script
      type="application/ld+json"
      dangerouslySetInnerHTML={{ __html: JSON.stringify(structuredData) }}
    />
  );
}
```

---

## 6. Portfolio Deployment

### Recommended Hosting Options

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    Hosting Options for Portfolios                       │
├──────────────────┬──────────────────────────────────────────────────────┤
│ Vercel           │ Best for Next.js sites                              │
│                  │ Free tier generous for portfolios                    │
│                  │ Automatic HTTPS, global CDN                          │
│                  │ Preview deployments for branches                     │
├──────────────────┼──────────────────────────────────────────────────────┤
│ Netlify          │ Great alternative to Vercel                         │
│                  │ Good for static sites                                │
│                  │ Form handling built-in                               │
├──────────────────┼──────────────────────────────────────────────────────┤
│ Cloudflare Pages │ Fastest global performance                          │
│                  │ Generous free tier                                   │
│                  │ Workers for serverless functions                     │
├──────────────────┼──────────────────────────────────────────────────────┤
│ GitHub Pages     │ Free for open source                                │
│                  │ Limited to static sites                              │
│                  │ Good for simple portfolios                           │
└──────────────────┴──────────────────────────────────────────────────────┘
```

### Domain Name Tips

```
Best practices for portfolio domains:

1. USE YOUR NAME
   yourname.dev
   yourname.io
   yourname.com

2. KEEP IT SIMPLE
   ✓ johndoe.dev
   ✗ john-doe-developer-portfolio-2024.com

3. PROFESSIONAL TLDs
   Recommended: .dev, .io, .com, .co
   Avoid: .xyz, .info, .ninja

4. SETUP
   - Purchase from Namecheap, Google Domains, or Cloudflare
   - Point nameservers to your host
   - Enable HTTPS (automatic on Vercel/Netlify)
   - Setup email forwarding (you@yourname.dev)
```

---

## Practice Exercises

### Exercise 1: Portfolio Audit
Review your current portfolio (or a peer's) using the checklist above. Identify 3 areas for improvement.

### Exercise 2: Project Description
Write a project description for your best project using the STAR framework. Include:
- Problem statement (2-3 sentences)
- Technical solution (include stack)
- 3 key technical highlights
- Results/metrics if available

### Exercise 3: Portfolio Speedrun
Build a minimal portfolio in 2 hours using:
- Next.js + Tailwind CSS
- 3 project cards
- Hero section with your info
- Deploy to Vercel

---

## Key Takeaways

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    Portfolio Success Checklist                          │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  ✓ 3-5 quality projects (not 10 mediocre ones)                         │
│  ✓ All projects have live demos that work                              │
│  ✓ Clear, concise descriptions using STAR framework                    │
│  ✓ Modern tech stack (React/Next.js, TypeScript)                       │
│  ✓ Fast loading, mobile-friendly, accessible                           │
│  ✓ Professional domain (yourname.dev)                                  │
│  ✓ Easy to find contact information                                    │
│  ✓ Links to GitHub, LinkedIn                                           │
│  ✓ Some personality - stand out from templates                         │
│  ✓ Regular updates - not abandoned                                     │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## Additional Resources

- [Vercel Templates](https://vercel.com/templates) - Portfolio starters
- [Dribbble](https://dribbble.com/search/developer-portfolio) - Design inspiration
- [Awwwards](https://www.awwwards.com/) - Award-winning designs
- [Josh Comeau's Blog](https://www.joshwcomeau.com/) - Great portfolio example
- [Lighthouse](https://developers.google.com/web/tools/lighthouse) - Performance auditing

---

Tomorrow, we'll create a **resume that gets past ATS systems** and catches the attention of hiring managers.

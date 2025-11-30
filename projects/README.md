# Projects

Build these projects throughout your learning journey to solidify your skills and create a portfolio.

## Project Progression

| # | Project | Phase | Difficulty | Tech Stack |
|---|---------|-------|------------|------------|
| 1 | Portfolio Website | 1 | ⭐ | HTML, CSS, JS |
| 2 | Task Manager | 2 | ⭐⭐ | React, TypeScript, Tailwind |
| 3 | Real-time Chat | 2-3 | ⭐⭐⭐ | Next.js, WebSockets, PostgreSQL |
| 4 | E-commerce Platform | 3 | ⭐⭐⭐⭐ | Full Stack + Stripe |
| 5 | SaaS Capstone | 5 | ⭐⭐⭐⭐⭐ | Everything |

---

## Project 1: Portfolio Website
**Phase:** 1 | **Duration:** 1-2 weeks

Build your personal developer portfolio.

### Requirements
- [ ] Responsive design (mobile-first)
- [ ] Home/Hero section
- [ ] About section
- [ ] Projects showcase
- [ ] Skills section
- [ ] Contact form
- [ ] Smooth scrolling
- [ ] Animations/transitions

### Tech Stack
- HTML5
- CSS3 (Flexbox, Grid)
- Vanilla JavaScript
- GitHub Pages deployment

### Stretch Goals
- Dark mode toggle
- Blog section
- Project filtering

---

## Project 2: Task Manager
**Phase:** 2 | **Duration:** 2-3 weeks

A full-featured todo application.

### Requirements
- [ ] Create, read, update, delete tasks
- [ ] Mark tasks complete
- [ ] Filter by status (all, active, completed)
- [ ] Sort by date/priority
- [ ] Persist to localStorage
- [ ] Responsive design

### Tech Stack
- React 18+
- TypeScript
- Tailwind CSS
- Zustand (state)
- Zod (validation)

### Stretch Goals
- Drag and drop reordering
- Due dates with calendar
- Categories/tags
- Search functionality
- Dark mode

---

## Project 3: Real-time Chat
**Phase:** 2-3 | **Duration:** 3-4 weeks

Build a real-time chat application.

### Requirements
- [ ] User authentication
- [ ] Create/join chat rooms
- [ ] Real-time messaging
- [ ] Online status indicators
- [ ] Message history
- [ ] Typing indicators

### Tech Stack
- Next.js 14+
- TypeScript
- Prisma + PostgreSQL
- Socket.io or Pusher
- NextAuth.js
- Tailwind CSS

### Stretch Goals
- Direct messages
- File/image sharing
- Message reactions
- Read receipts
- Push notifications

---

## Project 4: E-commerce Platform
**Phase:** 3 | **Duration:** 4-6 weeks

Build a complete e-commerce store.

### Requirements

**Frontend**
- [ ] Product listing with filtering
- [ ] Product detail pages
- [ ] Shopping cart
- [ ] Checkout flow
- [ ] User authentication
- [ ] Order history

**Backend**
- [ ] Product CRUD API
- [ ] Cart management
- [ ] Order processing
- [ ] Stripe integration
- [ ] Email notifications
- [ ] Admin dashboard

### Tech Stack
- Next.js 14+
- TypeScript
- Prisma + PostgreSQL
- Stripe
- React Query
- Tailwind CSS
- SendGrid (emails)

### Stretch Goals
- Product reviews
- Wishlist
- Inventory tracking
- Discount codes
- Multiple payment methods
- Analytics dashboard

---

## Project 5: SaaS Capstone
**Phase:** 5 | **Duration:** 6-8 weeks

Build a complete SaaS application.

### Choose Your SaaS Idea
- Project management tool
- Invoice/billing software
- Appointment booking system
- Survey/form builder
- Analytics dashboard
- Content management system

### Requirements

**Core Features**
- [ ] User registration/login
- [ ] Email verification
- [ ] Password reset
- [ ] User dashboard
- [ ] Team/organization support
- [ ] Role-based access

**Billing**
- [ ] Stripe subscription integration
- [ ] Multiple pricing tiers
- [ ] Usage-based billing (optional)
- [ ] Invoice generation
- [ ] Cancel/upgrade flows

**Admin**
- [ ] Admin dashboard
- [ ] User management
- [ ] Analytics/metrics
- [ ] Feature flags

**Infrastructure**
- [ ] CI/CD pipeline
- [ ] Automated testing
- [ ] Error tracking (Sentry)
- [ ] Monitoring
- [ ] Documentation

### Tech Stack
- Next.js 14+ (App Router)
- TypeScript
- Prisma + PostgreSQL
- Redis (caching, sessions)
- Stripe
- NextAuth.js
- React Query
- Tailwind CSS
- Vercel/Railway
- GitHub Actions

### Project Structure
```
saas-app/
├── apps/
│   ├── web/           # Next.js frontend
│   └── api/           # Optional separate API
├── packages/
│   ├── ui/            # Shared components
│   ├── db/            # Prisma schema
│   └── config/        # Shared config
├── docker-compose.yml
└── turbo.json
```

---

## Project Tips

### Before Starting
1. Plan your features (MVP first)
2. Design your database schema
3. Create wireframes/mockups
4. Set up project structure
5. Initialize Git repository

### During Development
1. Work in small, focused commits
2. Write tests as you go
3. Document your decisions
4. Take breaks when stuck
5. Ask for code reviews

### After Completing
1. Write a good README
2. Add screenshots/demo video
3. Deploy to production
4. Add to portfolio
5. Share on social media

---

## README Template

Use this template for your project READMEs:

```markdown
# Project Name

Brief description of what the project does.

## Demo

🔗 [Live Demo](https://your-demo-url.com)

## Screenshots

![Screenshot](./screenshots/home.png)

## Features

- Feature 1
- Feature 2
- Feature 3

## Tech Stack

- Frontend: React, TypeScript, Tailwind CSS
- Backend: Node.js, Express, PostgreSQL
- Deployment: Vercel, Railway

## Getting Started

### Prerequisites

- Node.js 18+
- PostgreSQL

### Installation

```bash
git clone https://github.com/username/project.git
cd project
npm install
cp .env.example .env
npm run dev
```

## Environment Variables

```
DATABASE_URL=
NEXTAUTH_SECRET=
```

## License

MIT
```

---

Good luck with your projects! 🚀

# 04 — Platform Development Plan

## Overview

We will build a **multi-platform system** comprising:
1. **Website** (Web app — PWA capable)
2. **Android App** (Native or cross-platform)
3. **iOS App** (Native or cross-platform)
4. **Admin Dashboard** (Web-based)
5. **F-commerce Seller Portal** (Web + Mobile)
6. **Backend API** (Headless, powers all frontends)

## Development Philosophy

- **"Vibe coding" with Claude Code** — Build fast, iterate quickly
- **Mobile-first** — Bangladesh is mobile-heavy
- **Automation-first** — AI for support, marketing, operations
- **Start lean, scale fast** — MVP in 60 days, iterate based on real usage

---

## PHASE 1: MVP (Weeks 1–8)

### What MVP Does
- Users can paste Taobao/1688 product links to get quotes
- Admins manage orders manually (semi-automated)
- Payment via bKash/Nagad (manual verification initially)
- Basic order tracking
- Public website with info + quote form

### MVP Tech Stack

| Layer | Technology | Why |
|-------|------------|-----|
| **Frontend (Web)** | Next.js (React) | Fast, SEO-friendly, PWA ready |
| **Mobile** | React Native (Expo) | One codebase for iOS + Android |
| **Backend API** | Node.js (Express/Nest.js) | Monir's comfort zone |
| **Database** | PostgreSQL | Reliable, scalable, good with ORM |
| **ORM** | Prisma / Drizzle | Type-safe, great DX |
| **Auth** | NextAuth.js / Clerk | Quick setup, social login |
| **Payment** | SSLCommerz + bKash API | Standard BD payment gateways |
| **Cloud** | Vercel + Supabase or Railway | Quick deployment |
| **File Storage** | Cloudflare R2 / AWS S3 | Product images, QC photos |

### MVP Feature List

#### User-Facing
- [ ] Home page with service explanation
- [ ] "Paste Link" form (submit Taobao/1688/Alibaba URL)
- [ ] Manual quote request system
- [ ] Basic curated product catalog (20-50 products)
- [ ] User registration/login
- [ ] Order placement & payment (bKash, Nagad)
- [ ] Order status tracking (view-only)
- [ ] Bengali + English language toggle
- [ ] Cost estimation calculator

#### Admin Dashboard
- [ ] Order management dashboard
- [ ] Manual quote generation
- [ ] User management
- [ ] Payment verification
- [ ] Order status updates
- [ ] Customer notifications

#### Mobile App (MVP)
- [ ] Same as web — React Native code sharing
- [ ] Push notifications for order updates
- [ ] bKash payment integration

### MVP Development Milestones

| Week | Deliverable |
|------|-------------|
| 1 | Project setup, design system, wireframes |
| 2 | Backend API — auth, products, orders |
| 3 | Web frontend — homepage, product search, link submission |
| 4 | Admin dashboard — order management |
| 5 | Payment integration (SSLCommerz + bKash) |
| 6 | Mobile app — core screens (React Native) |
| 7 | Mobile payment + push notifications |
| 8 | Testing, bug fixes, MVP launch |

---

## PHASE 2: Automation & API Integration (Weeks 9–16)

### Key Features
- **Taobao/1688 API Integration** — Auto-fetch product details from links
- **Price Comparison Engine** — Show Taobao vs 1688 price for same product
- **AI Customer Support Chatbot** — Answer queries in Bengali + English
- **Automated Quote System** — Real-time pricing from link input
- **Tracking Auto-Update** — Real-time logistics tracking
- **F-commerce Portal** — Dedicated tools for Facebook sellers
- **Curated Catalog Expansion** — 500+ products

### API Integration Options

#### Option A: Official APIs (Recommended for Phase 2)
| Platform | API Availability | Notes |
|----------|-----------------|-------|
| **1688.com** | Limited (B2B) | Official API requires Chinese business registration |
| **Taobao** | Taobao Open Platform | Requires app key, Chinese entity |
| **Alibaba.com** | Alibaba Open Platform | Best documented, international-friendly |

#### Option B: Third-Party Aggregators (Faster, for Phase 1.5)
| Service | Features | Cost |
|---------|----------|------|
| **Onebound** | Taobao/Tmall/1688/京东 product data API | Pay-per-request |
| **SimplyBatch** | Product search, detail, pricing APIs | Subscription |
| **Diffreight** | China sourcing API with logistics | Custom pricing |
| **Accio** | 1688 sourcing agent marketplace | Commission-based |

#### Option C: Scraping + Human Fallback (For MVP)
- Use Puppeteer/Playwright to scrape product pages (for MVP speed)
- Human verification for unclear cases
- Migrate to API in Phase 2

### AI & Automation Features

```
┌─────────────────────────────────────────────┐
│           AI AUTOMATION LAYER                │
├─────────────────────────────────────────────┤
│ Chatbot (WhatsApp/Telegram/Web)             │
│   • Answer "How does it work?" queries       │
│   • Accept product links via chat            │
│   • Provide instant quotes                   │
│   • Track orders via chat                    │
│   • Escalate to human when needed            │
├─────────────────────────────────────────────┤
│ Social Media Automation                      │
│   • Auto-post product catalogs on FB/IG     │
│   • Respond to comments & DMs                │
│   • Schedule promotional content             │
├─────────────────────────────────────────────┤
│ Sales Automation                             │
│   • Follow-up abandoned quotes               │
│   • Send reminders for pending payments      │
│   • Re-engagement campaigns                  │
├─────────────────────────────────────────────┤
│ Operations Automation                        │
│   • Auto-categorize orders by urgency        │
│   • Auto-generate customs docs               │
│   • Auto-notify China warehouse team         │
└─────────────────────────────────────────────┘
```

---

## PHASE 3: Scale & Optimization (Weeks 17–24)

### Features
- **Inventory Management System** — Track stock in China warehouse
- **B2B Wholesale Portal** — Bulk ordering, container shipping
- **Seller Dashboard** — For F-commerce sellers to manage their store
- **Analytics Dashboard** — Sales trends, shipping performance, customer insights
- **Affiliate/Referral Program** — Users earn commission referring others
- **Advanced Search** — Image-based search, voice search
- **Multi-Currency** — USD, CNY, BDT display
- **Supplier Rating System** — Rate Chinese suppliers

### Performance Optimization
- CDN for static assets (Cloudflare)
- Image optimization (WebP, lazy loading)
- Database query optimization
- App performance profiling
- Load testing

---

## PHASE 4: Expansion (Month 6+)

- **API for Partners** — Let other BD e-commerce sites integrate our sourcing
- **Dropshipping Automation** — Auto-route orders from seller → China → BD
- **Warehouse Management Module** — Real-time inventory, barcode scanning
- **BD Warehouse** — Local warehouse for faster delivery on popular items
- **Subscription Plans** — Monthly packages for frequent buyers
- **Logistics Optimization** — Route optimization, carrier comparison

## Design Requirements

### UI/UX Principles
- **Mobile-first responsive design**
- **Bilingual (Bengali + English)** with easy toggle
- **Simple, 3-step process** prominently displayed
- **Trust signals** everywhere: office address, trade license, customer reviews
- **Transparent pricing** — show cost breakdown before checkout
- **Fast loading** — optimize for slower mobile connections in BD

### Brand Identity (TBD)
- Brand name, logo, color scheme
- Need team input on naming

## Testing Strategy

| Type | Tools | Frequency |
|------|-------|-----------|
| Unit Testing | Jest / Vitest | Every PR |
| Integration | Playwright | Daily |
| Mobile (iOS) | Xcode simulator | Weekly |
| Mobile (Android) | Android Studio + physical device | Weekly |
| Payment Flow | SSLCommerz sandbox + bKash test | Every payment change |
| Load Testing | k6 / Artillery | Monthly |
| Security Audit | OWASP checklist | Quarterly |

# 05 — Technical Architecture

[← Home](./) | [Document Index](./INDEX)
🌐 [বাংলা](./bn/05-Technical-Architecture)

## System Architecture Overview

```
┌─────────────────────────────────────────────────────────────────────┐
│                        CLIENT LAYER                                  │
├─────────────────┬──────────────────┬───────────────────┬────────────┤
│   Next.js Web   │  React Native    │  React Native     │  Telegram   │
│   App (PWA)     │  Android App     │  iOS App          │  Bot        │
├────────┴────────┴──────────┴───────┴──────────┴────────┴─────┴──────┤
│                        CDN / Load Balancer                           │
│                      (Cloudflare / Vercel)                           │
├─────────────────────────────────────────────────────────────────────┤
│                        API GATEWAY                                   │
│                   (Next.js API Routes / Express)                     │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│   ┌────────────┐  ┌──────────┐  ┌───────────┐  ┌────────────────┐  │
│   │ Auth       │  │ Product  │  │ Order     │  │ Payment        │  │
│   │ Service    │  │ Service  │  │ Service   │  │ Service        │  │
│   └────────────┘  └──────────┘  └───────────┘  └────────────────┘  │
│   ┌────────────┐  ┌──────────┐  ┌───────────┐  ┌────────────────┐  │
│   │ Tracking   │  │ Shipping│  │ Admin     │  │ Notification   │  │
│   │ Service    │  │ Calc    │  │ Service   │  │ Service        │  │
│   └────────────┘  └──────────┘  └───────────┘  └────────────────┘  │
│   ┌────────────┐  ┌──────────┐  ┌───────────┐                       │
│   │ AI/Chatbot │  │ Analytics│  │ Content   │                       │
│   │ Service    │  │ Service  │  │ Service   │                       │
│   └────────────┘  └──────────┘  └───────────┘                       │
│                                                                      │
├─────────────────────────────────────────────────────────────────────┤
│                        DATA LAYER                                    │
├───────────────────┬────────────────────┬────────────────────────────┤
│   PostgreSQL      │    Redis           │    Object Storage          │
│   (Primary DB)    │    (Cache/Session) │    (Product Images/QC)     │
│                   │                    │    S3 / R2                 │
├───────────────────┴────────────────────┴────────────────────────────┤
│                        EXTERNAL INTEGRATIONS                         │
├───────────────────┬────────────────────┬────────────────────────────┤
│   Taobao API      │  1688 API         │   Alibaba API              │
│   SSLCommerz      │  bKash API        │   Nagad API                │
│   Logistics APIs  │  SMS Gateway      │   Facebook/IG API          │
│   (Air/Sea)       │  (BulkSMS)        │   (Social Auto)            │
└───────────────────┴────────────────────┴────────────────────────────┘
```

## Database Schema (Core Tables)

### Users
| Column | Type | Notes |
|--------|------|-------|
| id | UUID | Primary key |
| email | String | Unique |
| phone | String | Unique, used for payment |
| password_hash | String | bcrypt |
| full_name | String | |
| address | JSON | Multiple addresses |
| role | Enum | customer, admin, seller |
| language | Enum | bn, en |
| created_at | Timestamp | |

### Products
| Column | Type | Notes |
|--------|------|-------|
| id | UUID | |
| type | Enum | curated, link-based, manual |
| source_platform | Enum | taobao, alibaba_1688, alibaba_global, manual |
| source_url | String | Original Taobao/1688 URL |
| source_product_id | String | ID on the original platform |
| title_en | String | |
| title_bn | String | |
| description | Text | |
| images | JSON[] | URLs |
| price_cny | Decimal | In RMB |
| price_bdt | Decimal | Converted |
| weight_kg | Decimal | |
| category_id | UUID | FK to categories |
| supplier_info | JSON | Supplier details |
| created_at | Timestamp | |

### Orders
| Column | Type | Notes |
|--------|------|-------|
| id | UUID | |
| user_id | UUID | FK |
| order_type | Enum | single, consolidated, bulk |
| source_platform | Enum | curated, link, manual |
| status | Enum | quoted, pending_payment, paid, purchasing, in_warehouse, qc_done, shipping, in_transit, customs, in_bd, delivered, cancelled |
| product_id | UUID | FK |
| product_detail | JSON | Snapshot of product at order time |
| quantity | Integer | |
| total_product_cost_cny | Decimal | |
| service_fee_bdt | Decimal | |
| shipping_fee_bdt | Decimal | |
| customs_duty_bdt | Decimal | |
| grand_total_bdt | Decimal | |
| shipping_method | Enum | air, sea, express |
| tracking_number | String | |
| notes | Text | |
| created_at | Timestamp | |
| updated_at | Timestamp | |

### Payments
| Column | Type | Notes |
|--------|------|-------|
| id | UUID | |
| order_id | UUID | FK |
| user_id | UUID | FK |
| amount_bdt | Decimal | |
| payment_method | Enum | bKash, Nagad, Rocket, card, bank |
| transaction_id | String | From payment gateway |
| status | Enum | pending, completed, failed, refunded |
| verified_by | UUID | Admin who verified |
| created_at | Timestamp | |

### Shipments
| Column | Type | Notes |
|--------|------|-------|
| id | UUID | |
| order_id | UUID | FK |
| tracking_number | String | |
| carrier | String | DHL, EMS, private |
| origin_warehouse | String | China warehouse location |
| destination_city | String | In Bangladesh |
| estimated_delivery | Date | |
| actual_delivery | Date | |
| weight_kg | Decimal | |
| shipping_cost_cny | Decimal | |
| customs_docs | JSON[] | Uploaded docs |
| status_updates | JSON[] | Timeline of tracking events |

## API Endpoints (Core)

### Public
```
GET  /api/products              — List curated products
GET  /api/products/:id          — Product detail
POST /api/quote                 — Get quote from link
GET  /api/shipping-cost         — Calculate shipping estimate
POST /api/auth/register         — Register
POST /api/auth/login            — Login
GET  /api/categories            — Product categories
```

### Authenticated
```
GET  /api/orders                — My orders
GET  /api/orders/:id            — Order detail
POST /api/orders                — Place order
POST /api/orders/:id/pay        — Payment
GET  /api/tracking/:id          — Order tracking
PUT  /api/profile               — Update profile
```

### Admin
```
GET  /api/admin/orders          — All orders
GET  /api/admin/orders/:id      — Order detail
PUT  /api/admin/orders/:id      — Update order status
POST /api/admin/products        — Add curated product
PUT  /api/admin/products/:id    — Update product
GET  /api/admin/users           — User list
POST /api/admin/quotes          — Generate manual quote
GET  /api/admin/analytics       — Dashboard stats
```

## Security Measures

| Area | Implementation |
|------|---------------|
| **Authentication** | JWT with refresh tokens |
| **Payment Security** | SSL/TLS, SSLCommerz integration (no card storage) |
| **Data Encryption** | At rest (AES-256), in transit (TLS 1.3) |
| **API Security** | Rate limiting, CORS, API keys for admin |
| **XSS/CSRF** | Input sanitization, CSRF tokens |
| **SQL Injection** | Parameterized queries via Prisma |
| **DDoS** | Cloudflare WAF |
| **Backups** | Daily automated DB backups |
| **Monitoring** | Sentry for errors, Uptime monitoring |

## Mobile App Architecture

### Cross-Platform Approach: React Native (Expo)
- **One codebase** for iOS + Android
- **Expo SDK** for camera, push notifications, payments
- **Native modules** if needed for bKash SDK integration
- **Code sharing** with web where possible (API layer)
- **OTA Updates** via EAS (no app store delay for minor fixes)

### App Screens (MVP)
1. **Home** — Services overview, how it works, CTA
2. **Product Search/Catalog** — Browse + search curated products
3. **Link Submit Form** — Paste Taobao/1688 URL
4. **Quote Result** — See estimated cost breakdown
5. **Checkout** — Address, payment method
6. **Order Tracking** — Real-time status
7. **Profile** — Orders, address, settings
8. **Notifications** — Push notification list
-e 
---
[← Back to Home](./) | [Full Document Index](./INDEX)
🌐 [বাংলা](./bn/05-Technical-Architecture)


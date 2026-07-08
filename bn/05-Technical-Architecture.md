# ০৫ — প্রযুক্তিগত আর্কিটেকচার

🌐 [English](../05-Technical-Architecture)

[← হোম](../) | [ডকুমেন্ট সূচী](../INDEX)

## সিস্টেম আর্কিটেকচার ওভারভিউ

```
┌─────────────────────────────────────────────────────────────────────┐
│                        ক্লায়েন্ট লেয়ার                              │
├─────────────────┬──────────────────┬───────────────────┬────────────┤
│   Next.js ওয়েব │  React Native    │  React Native     │  Telegram  │
│   অ্যাপ (PWA)   │  Android অ্যাপ   │  iOS অ্যাপ        │  বট        │
├────────┴────────┴──────────┴───────┴──────────┴────────┴─────┴──────┤
│                        CDN / লোড ব্যালান্সার                        │
│                      (Cloudflare / Vercel)                           │
├─────────────────────────────────────────────────────────────────────┤
│                        API গেটওয়ে                                   │
│                   (Next.js API Routes / Express)                     │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│   ┌────────────┐  ┌──────────┐  ┌───────────┐  ┌────────────────┐  │
│   │ Auth       │  │ Product  │  │ Order     │  │ Payment        │  │
│   │ সার্ভিস   │  │ সার্ভিস  │  │ সার্ভিস   │  │ সার্ভিস       │  │
│   └────────────┘  └──────────┘  └───────────┘  └────────────────┘  │
│   ┌────────────┐  ┌──────────┐  ┌───────────┐  ┌────────────────┐  │
│   │ Tracking   │  │ Shipping │  │ Admin     │  │ Notification   │  │
│   │ সার্ভিস   │  │ ক্যালক   │  │ সার্ভিস   │  │ সার্ভিস       │  │
│   └────────────┘  └──────────┘  └───────────┘  └────────────────┘  │
│   ┌────────────┐  ┌──────────┐  ┌───────────┐                       │
│   │ AI/চ্যাটবট │  │ Analytics│  │ Content   │                       │
│   │ সার্ভিস   │  │ সার্ভিস  │  │ সার্ভিস   │                       │
│   └────────────┘  └──────────┘  └───────────┘                       │
│                                                                      │
├─────────────────────────────────────────────────────────────────────┤
│                        ডেটা লেয়ার                                   │
├───────────────────┬────────────────────┬────────────────────────────┤
│   PostgreSQL      │    Redis           │    অবজেক্ট স্টোরেজ         │
│   (প্রাথমিক DB)   │    (ক্যাশ/সেশন)   │    (পণ্যের ছবি/QC)         │
│                   │                    │    S3 / R2                 │
├───────────────────┴────────────────────┴────────────────────────────┤
│                        বাহ্যিক ইন্টিগ্রেশন                           │
├───────────────────┬────────────────────┬────────────────────────────┤
│   Taobao API      │  1688 API         │   Alibaba API              │
│   SSLCommerz      │  bKash API        │   Nagad API                │
│   লজিস্টিকস API   │  SMS গেটওয়ে      │   Facebook/IG API          │
│   (এয়ার/সি)      │  (বাল্কএসএমএস)    │   (সোশ্যাল অটো)            │
└───────────────────┴────────────────────┴────────────────────────────┘
```

## ডেটাবেস স্কিমা (মূল টেবিল)

### ব্যবহারকারী
| কলাম | ধরন | নোট |
|-------|------|------|
| id | UUID | প্রাথমিক কী |
| email | String | ইউনিক |
| phone | String | ইউনিক, পেমেন্টের জন্য ব্যবহৃত |
| password_hash | String | bcrypt |
| full_name | String | |
| address | JSON | একাধিক ঠিকানা |
| role | Enum | customer, admin, seller |
| language | Enum | bn, en |
| created_at | Timestamp | |

### পণ্য
| কলাম | ধরন | নোট |
|-------|------|------|
| id | UUID | |
| type | Enum | curated, link-based, manual |
| source_platform | Enum | taobao, alibaba_1688, alibaba_global, manual |
| source_url | String | মূল Taobao/1688 URL |
| source_product_id | String | মূল প্ল্যাটফর্মে আইডি |
| title_en | String | |
| title_bn | String | |
| description | Text | |
| images | JSON[] | URLs |
| price_cny | Decimal | RMB-তে |
| price_bdt | Decimal | রূপান্তরিত |
| weight_kg | Decimal | |
| category_id | UUID | FK to categories |
| supplier_info | JSON | সাপ্লায়ার বিস্তারিত |
| created_at | Timestamp | |

### অর্ডার
| কলাম | ধরন | নোট |
|-------|------|------|
| id | UUID | |
| user_id | UUID | FK |
| order_type | Enum | single, consolidated, bulk |
| source_platform | Enum | curated, link, manual |
| status | Enum | quoted, pending_payment, paid, purchasing, in_warehouse, qc_done, shipping, in_transit, customs, in_bd, delivered, cancelled |
| product_id | UUID | FK |
| product_detail | JSON | অর্ডার সময় পণ্যের স্ন্যাপশট |
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

### পেমেন্ট
| কলাম | ধরন | নোট |
|-------|------|------|
| id | UUID | |
| order_id | UUID | FK |
| user_id | UUID | FK |
| amount_bdt | Decimal | |
| payment_method | Enum | bKash, Nagad, Rocket, card, bank |
| transaction_id | String | পেমেন্ট গেটওয়ে থেকে |
| status | Enum | pending, completed, failed, refunded |
| verified_by | UUID | যিনি ভেরিফাই করেছেন |
| created_at | Timestamp | |

### শিপমেন্ট
| কলাম | ধরন | নোট |
|-------|------|------|
| id | UUID | |
| order_id | UUID | FK |
| tracking_number | String | |
| carrier | String | DHL, EMS, private |
| origin_warehouse | String | চায়না গুদামের অবস্থান |
| destination_city | String | বাংলাদেশে |
| estimated_delivery | Date | |
| actual_delivery | Date | |
| weight_kg | Decimal | |
| shipping_cost_cny | Decimal | |
| customs_docs | JSON[] | আপলোড করা ডক্স |
| status_updates | JSON[] | ট্র্যাকিং ইভেন্টের টাইমলাইন |

## API এন্ডপয়েন্ট (মূল)

### পাবলিক
```
GET  /api/products              — কিউরেটেড পণ্য তালিকা
GET  /api/products/:id          — পণ্যের বিস্তারিত
POST /api/quote                 — লিঙ্ক থেকে কোট পান
GET  /api/shipping-cost         — শিপিং অনুমান গণনা
POST /api/auth/register         — নিবন্ধন
POST /api/auth/login            — লগইন
GET  /api/categories            — পণ্য বিভাগ
```

### প্রমাণীকৃত
```
GET  /api/orders                — আমার অর্ডার
GET  /api/orders/:id            — অর্ডারের বিস্তারিত
POST /api/orders                — অর্ডার দিন
POST /api/orders/:id/pay        — পেমেন্ট
GET  /api/tracking/:id          — অর্ডার ট্র্যাকিং
PUT  /api/profile               — প্রোফাইল আপডেট
```

### অ্যাডমিন
```
GET  /api/admin/orders          — সব অর্ডার
GET  /api/admin/orders/:id      — অর্ডারের বিস্তারিত
PUT  /api/admin/orders/:id      — অর্ডার স্ট্যাটাস আপডেট
POST /api/admin/products        — কিউরেটেড পণ্য যোগ
PUT  /api/admin/products/:id    — পণ্য আপডেট
GET  /api/admin/users           — ব্যবহারকারী তালিকা
POST /api/admin/quotes          — ম্যানুয়াল কোট তৈরি
GET  /api/admin/analytics       — ড্যাশবোর্ড পরিসংখ্যান
```

## নিরাপত্তা ব্যবস্থা

| ক্ষেত্র | বাস্তবায়ন |
|---------|------------|
| **প্রমাণীকরণ** | রিফ্রেশ টোকেনসহ JWT |
| **পেমেন্ট নিরাপত্তা** | SSL/TLS, SSLCommerz ইন্টিগ্রেশন (কার্ড সংরক্ষণ নেই) |
| **ডেটা এনক্রিপশন** | বিশ্রামে (AES-256), ট্রানজিটে (TLS 1.3) |
| **API নিরাপত্তা** | রেট লিমিটিং, CORS, অ্যাডমিনের জন্য API কী |
| **XSS/CSRF** | ইনপুট স্যানিটাইজেশন, CSRF টোকেন |
| **SQL ইনজেকশন** | Prisma-এর মাধ্যমে প্যারামিটারাইজড কোয়েরি |
| **DDoS** | Cloudflare WAF |
| **ব্যাকআপ** | দৈনিক স্বয়ংক্রিয় DB ব্যাকআপ |
| **মনিটরিং** | ত্রুটির জন্য Sentry, আপটাইম মনিটরিং |

## মোবাইল অ্যাপ আর্কিটেকচার

### ক্রস-প্ল্যাটফর্ম অ্যাপ্রোচ: React Native (Expo)
- **এক কোডবেস** iOS + Android-এর জন্য
- **Expo SDK** ক্যামেরা, পুশ নোটিফিকেশন, পেমেন্টের জন্য
- **নেটিভ মডিউল** প্রয়োজনে bKash SDK ইন্টিগ্রেশনের জন্য
- **কোড শেয়ারিং** ওয়েবের সাথে যেখানে সম্ভব (API লেয়ার)
- **OTA আপডেট** EAS-এর মাধ্যমে (ছোট ফিক্সের জন্য অ্যাপ স্টোর বিলম্ব নেই)

### অ্যাপ স্ক্রিন (এমভিপি)
১. **হোম** — সেবার ওভারভিউ, এটি কীভাবে কাজ করে, CTA
২. **পণ্য অনুসন্ধান/ক্যাটালগ** — কিউরেটেড পণ্য ব্রাউজ + অনুসন্ধান
৩. **লিঙ্ক জমা ফর্ম** — Taobao/1688 URL পেস্ট করুন
৪. **কোট ফলাফল** — আনুমানিক খরচ ভাঙ্গন দেখুন
৫. **চেকআউট** — ঠিকানা, পেমেন্ট পদ্ধতি
৬. **অর্ডার ট্র্যাকিং** — রিয়েল-টাইম স্ট্যাটাস
৭. **প্রোফাইল** — অর্ডার, ঠিকানা, সেটিংস
৮. **নোটিফিকেশন** — পুশ নোটিফিকেশন তালিকা

---
[← হোমে ফিরে যান](../) | [সম্পূর্ণ ডকুমেন্ট সূচী](../INDEX)
🌐 [English](../05-Technical-Architecture)

# SYSTEM ANALYSIS - PropertyHub Indonesia

## 1. EXECUTIVE SUMMARY

PropertyHub Indonesia adalah **Enterprise Marketplace Properti Terintegrasi** yang menghubungkan 7 user personas dalam ekosistem property nasional dengan standar production-ready, scalable, dan maintainable.

**Key Differentiators:**
- ✅ Dashboard khusus: Developer, Notaris, Marketing, Asosiasi
- ✅ Sistem Komisi Otomatis & Referral
- ✅ Digital Document Management
- ✅ Heatmap Analytics & AI Recommendation
- ✅ Mobile-First Architecture
- ✅ Multi-language & Multi-currency ready
- ✅ Enterprise Security & Compliance

---

## 2. BUSINESS OBJECTIVES

### 2.1 Primary Goals
1. **Market Aggregation**: Kumpulkan seluruh properti nasional dalam satu platform
2. **Transaction Enablement**: Fasilitasi transaksi end-to-end dari listing hingga ownership transfer
3. **Stakeholder Ecosystem**: Koneksikan semua pihak (pembeli, penjual, developer, notaris, marketing)
4. **Revenue Generation**: Komisi, referral, premium listing
5. **Data Intelligence**: Insights analytics untuk stakeholder

### 2.2 Success Metrics
- **User Growth**: 100K users dalam 6 bulan
- **GMV**: Target 50 miliar dalam tahun pertama
- **Conversion Rate**: 5-8%
- **Platform Efficiency**: 95+ Lighthouse score
- **Uptime**: 99.9%

---

## 3. USER PERSONAS & FLOWS

### 3.1 User Roles

```
┌─────────────────────────────────────────────────────────┐
│                    USER HIERARCHY                        │
├─────────────────────────────────────────────────────────┤
│ SUPER ADMIN    │ Full system control, audit, reports     │
├─────────────────────────────────────────────────────────┤
│ ADMIN          │ User management, approval, moderation   │
├─────────────────────────────────────────────────────────┤
│ ASSOCIATION    │ Developer verification, suspension      │
├─────────────────────────────────────────────────────────┤
│ DEVELOPER      │ Project & property management           │
├─────────────────────────────────────────────────────────┤
│ MARKETING      │ Referral & commission management        │
├─────────────────────────────────────────────────────────┤
│ NOTARY         │ Legal verification & document upload    │
├─────────────────────────────────────────────────────────┤
│ SELLER         │ Individual property seller              │
├─────────────────────────────────────────────────────────┤
│ BUYER          │ Property search & purchase              │
├─────────────────────────────────────────────────────────┤
│ GUEST          │ Read-only access                        │
└─────────────────────────────────────────────────────────┘
```

### 3.2 Core User Flows

**BOOKING FLOW (Buyer → Developer)**
```
Guest Login → Search Property → View Detail → Booking
  ↓
Upload Identity (KTP) → Pay Booking Fee
  ↓
Developer Validation → Notary Verification
  ↓
Generate PPJB/AJB → Sign Digitally
  ↓
Sold Status
```

**REFERRAL FLOW (Marketing → Commission)**
```
Marketing Create Campaign → Generate Referral Link
  ↓
Share via Social/Email → Buyer Click Link
  ↓
Buyer Booking Property → Commission Auto-created
  ↓
Payment Verified → Withdraw Request
  ↓
Admin Approval → Transfer to Bank
```

**DEVELOPER PROJECT FLOW**
```
Developer Register & Verify → Create Project
  ↓
Upload Properties → Set Pricing & Status
  ↓
Monitor Sales → Manage Inventory
  ↓
View Analytics & Reports
```

---

## 4. SYSTEM ARCHITECTURE

### 4.1 Architecture Pattern: Modular Monolith

```
┌──────────────────────────────────────────────────────────┐
│                    API GATEWAY LAYER                      │
│              (Rate Limit, JWT, CORS, Logging)             │
└──────────────────────────────────────────────────────────┘
                           ↓
┌──────────────────────────────────────────────────────────┐
│                   APPLICATION LAYER                       │
├──────────────────────────────────────────────────────────┤
│ Modules (Independent, Loosely Coupled)                   │
├──────────────────────────────────────────────────────────┤
│ ┌─────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐       │
│ │  Auth   │ │  Users   │ │ Properties│ │Developers│      │
│ └─────────┘ └──────────┘ └──────────┘ └──────────┘       │
│ ┌─────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐       │
│ │Marketing│ │  Booking │ │Transactions│ │Payments│       │
│ └─────────┘ └──────────┘ └──────────┘ └──────────┘       │
│ ┌─────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐       │
│ │ Notary  │ │Commission│ │Documents │ │Dashboard│       │
│ └─────────┘ └──────────┘ └──────────┘ └──────────┘       │
│ ┌─────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐       │
│ │Analytics│ │Notification│ │Chat    │ │Settings │       │
│ └─────────┘ └──────────┘ └──────────┘ └──────────┘       │
└──────────────────────────────────────────────────────────┘
                           ↓
┌──────────────────────────────────────────────────────────┐
│               CROSS-CUTTING CONCERNS                      │
├──────────────────────────────────────────────────────────┤
│ Events │ Queue │ Cache │ Logger │ Audit │ Security      │
└──────────────────────────────────────────────────────────┘
                           ↓
┌──────────────────────────────────────────────────────────┐
│                 DATA & STORAGE LAYER                      │
├──────────────────────────────────────────────────────────┤
│ MySQL 8 │ Redis │ R2/S3 │ Elasticsearch (future)         │
└──────────────────────────────────────────────────────────┘
```

### 4.2 Technology Stack Rationale

| Layer | Technology | Reason |
|-------|-----------|--------|
| **Backend** | Laravel 12 + PHP 8.4 | Enterprise ecosystem, packages, performance |
| **Frontend** | Next.js 14 + React 19 | SSR, SSG, Server Components, performance |
| **Mobile** | Flutter | Cross-platform, performance, maintainability |
| **Database** | MySQL 8 | ACID, complex queries, established ecosystem |
| **Cache** | Redis | In-memory, queue, real-time, sessioning |
| **Storage** | Cloudflare R2 + S3 | Global CDN, durability, cost-effective |
| **Realtime** | Laravel Reverb | Native WebSocket, Laravel integration |
| **Payment** | Midtrans/Xendit | Indonesia-focused, multi-method, secure |
| **Maps** | Google Maps + Leaflet | Comprehensive coverage, fallback option |
| **CI/CD** | Docker + GitHub Actions | Containerization, automation, scalability |

---

## 5. MODULE BREAKDOWN

### 5.1 Core Modules (Priority Order)

```
PHASE 1: Foundation (Weeks 1-4)
├── Auth Module (Login, Register, JWT, OAuth2, 2FA)
├── Users Module (Profile, RBAC, Permissions)
└── Settings Module (Global configuration)

PHASE 2: Core Business (Weeks 5-12)
├── Properties Module (CRUD, categorization, status)
├── Developers Module (Project management, verification)
├── Marketing Module (Referral, campaigns)
└── Booking Module (Reservation, status tracking)

PHASE 3: Financial & Legal (Weeks 13-18)
├── Transactions Module (Order management, lifecycle)
├── Payments Module (Payment gateway integration)
├── Notary Module (Document verification, signing)
└── Commission Module (Auto-calculation, distribution)

PHASE 4: Supporting Services (Weeks 19-24)
├── Documents Module (PDF generation, management)
├── Notification Module (Email, SMS, WhatsApp, Push)
├── Chat Module (Realtime messaging)
├── Analytics Module (Dashboards, reports)

PHASE 5: Enhancement & DevOps (Weeks 25-26)
├── Dashboard Module (Analytics, reporting)
├── CMS Module (Content management)
├── Audit Log Module (Compliance, tracking)
├── Docker & CI/CD
└── Testing & Deployment
```

### 5.2 Module Interdependencies

```
Auth
  ↓
Users → RBAC → Permissions
  ↓
Properties ← Developers
  ├─ Marketing (Referral)
  ├─ Booking
  │   ├─ Transactions
  │   ├─ Payments
  │   ├─ Notary (Documents)
  │   └─ Commission
  ├─ Chat
  ├─ Notification
  └─ Analytics
```

---

## 6. DATABASE DESIGN PRINCIPLES

### 6.1 Key Conventions
- **UUID**: Primary key untuk semua table (UUIDv7 untuk sequential writes)
- **Soft Delete**: Semua table memiliki `deleted_at` column
- **Timestamps**: `created_at`, `updated_at`, `deleted_at`
- **Audit Trail**: Log semua perubahan data di `audit_logs` table
- **Status Tracking**: Setiap entity memiliki status field yang jelas
- **Relationships**: Foreign key dengan on_delete policy

### 6.2 Schema Organization

```
Core Tables
├── users
├── roles
├── permissions
└── role_permissions

Property Tables
├── properties
├── property_categories
├── property_images
├── property_videos
└── property_nearby_facilities

Transaction Tables
├── bookings
├── transactions
├── payments
└── payment_gateways

Financial Tables
├── commissions
├── referrals
├── withdrawals
└── invoice

Legal Tables
├── documents
├── digital_signatures
├── notary_verifications
└── legal_statuses

User Tables
├── developers
├── marketing
├── notaries
├── associations
└── seller_profiles

Meta Tables
├── audit_logs
├── activities
├── notifications
└── system_settings
```

---

## 7. API STRATEGY

### 7.1 API Versioning

```
/api/v1/
  /auth
    /login (POST)
    /register (POST)
    /refresh (POST)
    /logout (POST)
    /verify-otp (POST)
  /users
    / (GET, POST)
    /{id} (GET, PUT, DELETE)
  /properties
    / (GET, POST)
    /{id} (GET, PUT, DELETE)
    /{id}/images (POST)
    /{id}/booking (POST)
  ... (other endpoints)
```

### 7.2 API Standards
- **REST Principles**: Resource-oriented, HTTP methods, status codes
- **Response Format**: JSON, consistent structure
- **Pagination**: cursor-based & offset-based
- **Filtering**: Query parameters
- **Sorting**: Multi-column support
- **Rate Limiting**: Per user, per IP
- **Authentication**: JWT Bearer token
- **Documentation**: OpenAPI 3.0 (Swagger)

---

## 8. SECURITY ARCHITECTURE

### 8.1 Security Layers

```
┌──────────────────────────────��─────────┐
│        Cloudflare WAF & DDoS            │
├────────────────────────────────────────┤
│    NGINX (Rate Limit, SSL/TLS)          │
├────────────────────────────────────────┤
│  API Gateway (Auth, CORS, Validation)   │
├────────────────────────────────────────┤
│  RBAC (Role-Based Access Control)       │
├────────────────────────────────────────┤
│  Encryption (At-rest, In-transit)       │
├────────────────────────────────────────┤
│  Audit Log (All changes tracked)        │
├────────────────────────────────────────┤
│  Data Validation (Input, Output)        │
└────────────────────────────────────────┘
```

### 8.2 Security Implementations
- ✅ JWT with HS256/RS256 signing
- ✅ OAuth2 for social login
- ✅ 2FA (TOTP, SMS)
- ✅ CSRF protection (CSRF token)
- ✅ SQL Injection prevention (Parameterized queries)
- ✅ XSS protection (Content Security Policy)
- ✅ Rate limiting (per user, per IP, per endpoint)
- ✅ Encryption (AES-256 for sensitive data)
- ✅ HTTPS/TLS everywhere
- ✅ Audit logging (all user actions)

---

## 9. PERFORMANCE TARGETS

### 9.1 Lighthouse Metrics

| Metric | Target | Strategy |
|--------|--------|----------|
| **Performance** | 95+ | Code splitting, lazy loading, optimization |
| **LCP** | <2.5s | Critical CSS, image optimization |
| **CLS** | <0.1 | Reserve space, avoid layout shifts |
| **FID** | <100ms | Request chunks, optimized JavaScript |
| **Page Load** | <2s | CDN, caching, compression |

### 9.2 Caching Strategy

```
CDN Cache (Cloudflare R2)
├── Static Assets (CSS, JS, Images) → 30 days
└── API Responses → Smart cache headers

Browser Cache
├── Static → 1 year
├── Dynamic → 0-5 minutes
└── API → Conditional (ETag)

Server-Side Cache (Redis)
├── User Session → 24 hours
├── Query Cache → 1-24 hours
├── Rate Limit Counter → Real-time
└── WebSocket Connection → Session-based
```

---

## 10. DEPLOYMENT ARCHITECTURE

### 10.1 Infrastructure Stack

```
┌──────────────────────────────────────────┐
│        Cloudflare (CDN, WAF, DDoS)       │
└──────────────────────────────────────────┘
                    ↓
┌──────────────────────────────────────────┐
│    Docker Container Orchestration         │
│    (Dev, Staging, Production)             │
└──────────────────────────────────────────┘
                    ↓
┌────────────────────┬──────────────────────┐
│  NGINX (Proxy)     │  Application Server  │
│  (Load Balancer)   │  (Laravel + PHP-FPM) │
├────────────────────┼──────────────────────┤
│  Logging           │  Queue Worker        │
│  (ELK Stack)       │  (Redis/Supervisor)  │
└────────────────────┴──────────────────────┘
                    ↓
┌──────────────────────────────────────────┐
│    MySQL 8 (Master-Slave Replication)    │
│    Redis (Cluster)                        │
│    Cloudflare R2 / S3                     │
└──────────────────────────────────────────┘
```

### 10.2 CI/CD Pipeline

```
Git Push
  ↓
GitHub Actions Trigger
  ↓
├─ Lint & Style (ESLint, Prettier, PHPStan)
├─ Unit Tests
├─ Integration Tests
├─ Security Scan (SAST, Dependency Check)
└─ Build Docker Image
                    ↓
        Docker Push to Registry
                    ↓
        Deploy to Staging
                    ↓
        Smoke Tests
                    ↓
        Approval Required
                    ↓
        Deploy to Production
                    ↓
        Health Checks & Rollback Ready
```

---

## 11. DEVELOPMENT ROADMAP

### Phase 1: Foundation (Weeks 1-4)
- [ ] Project setup (Laravel, Next.js, Flutter structure)
- [ ] Database design & migration
- [ ] Auth module (JWT, OAuth2, 2FA)
- [ ] User management & RBAC
- [ ] API documentation (OpenAPI)

### Phase 2: Core Features (Weeks 5-12)
- [ ] Properties module (CRUD, images, categorization)
- [ ] Developers module (registration, verification)
- [ ] Marketing referral system
- [ ] Booking system
- [ ] Basic frontend (Next.js)

### Phase 3: Financial (Weeks 13-18)
- [ ] Transactions module
- [ ] Payment gateway integration (Midtrans, Xendit)
- [ ] Notary verification system
- [ ] Digital documents (PDF generation)
- [ ] Commission auto-calculation

### Phase 4: Support Services (Weeks 19-24)
- [ ] Real-time chat (WebSocket)
- [ ] Notification system (Email, SMS, Push)
- [ ] Analytics dashboard
- [ ] CMS module
- [ ] Mobile app (Flutter)

### Phase 5: Deployment (Weeks 25-26)
- [ ] Docker containerization
- [ ] CI/CD pipeline
- [ ] Performance optimization
- [ ] Security hardening
- [ ] Production deployment

---

## 12. KEY RISKS & MITIGATION

| Risk | Impact | Mitigation |
|------|--------|-----------|
| **Database Performance** | High | Indexing, partitioning, read replicas |
| **Concurrent Transactions** | High | Lock strategy, versioning, queue system |
| **Payment Integration** | High | Comprehensive testing, webhook handling |
| **Real-time Sync** | Medium | Event-driven architecture, Redis queue |
| **Scalability** | High | Microservices-ready modular monolith |
| **Security Breach** | Critical | WAF, encryption, audit logging, 2FA |
| **Data Loss** | Critical | MySQL replication, daily backups, R2 redundancy |

---

## 13. SUCCESS CRITERIA

### Technical Success
- ✅ 99.9% uptime
- ✅ <2s page load time
- ✅ 95+ Lighthouse score
- ✅ Zero critical security vulnerabilities
- ✅ Automated testing coverage >80%

### Business Success
- ✅ 100K active users in 6 months
- ✅ 5-8% booking conversion rate
- ✅ 50 miliar GMV in year 1
- ✅ 4.5+ star rating
- ✅ 95% uptime SLA maintained

---

## NEXT STEPS

1. ✅ **System Analysis** (This document)
2. 📋 **Software Requirement Specification**
3. 📋 **Business Process Documentation**
4. 📋 **Use Case Diagrams**
5. 📋 **Activity Diagrams**
6. 📋 **Sequence Diagrams**
7. 📋 **Entity-Relationship Diagram (ERD)**
8. 📋 **Database Implementation**
9. 📋 **Folder Structure & Setup**
10. 📋 **Backend Implementation**

---

**Created**: 2026-07-04  
**Version**: 1.0  
**Status**: APPROVED FOR DEVELOPMENT

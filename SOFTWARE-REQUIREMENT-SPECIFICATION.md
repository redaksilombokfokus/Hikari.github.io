# SOFTWARE REQUIREMENT SPECIFICATION (SRS)
# PropertyHub Indonesia - Enterprise Marketplace Properti

**Document Version**: 1.0  
**Last Updated**: 2026-07-04  
**Status**: APPROVED  
**Author**: Principal Software Architect  
**Classification**: Internal

---

## TABLE OF CONTENTS

1. [Introduction](#introduction)
2. [Functional Requirements](#functional-requirements)
3. [Non-Functional Requirements](#non-functional-requirements)
4. [System Requirements](#system-requirements)
5. [User Requirements](#user-requirements)
6. [Module Specifications](#module-specifications)
7. [API Specifications](#api-specifications)
8. [Data Requirements](#data-requirements)
9. [Security Requirements](#security-requirements)
10. [Performance Requirements](#performance-requirements)

---

## INTRODUCTION

### 1.1 Purpose

Dokumen ini mendefinisikan semua kebutuhan fungsional dan non-fungsional untuk **PropertyHub Indonesia**, sebuah marketplace properti enterprise-grade yang menghubungkan pembeli, penjual, developer, notaris, marketing, dan asosiasi dalam satu ekosistem terintegrasi.

### 1.2 Scope

**In Scope:**
- REST API (v1)
- GraphQL API (future)
- Website (Next.js)
- Mobile Apps (Android & iOS via Flutter)
- Admin Dashboard
- Role-based access control
- Payment gateway integration
- Real-time features (chat, notifications)
- Analytics & reporting
- Digital document management

**Out of Scope:**
- Multi-tenant architecture (Phase 2)
- AI/ML features (Phase 2)
- Blockchain integration
- Legacy system migration

### 1.3 Definitions & Abbreviations

| Term | Definition |
|------|-----------|
| **PropertyHub** | Platform PropertyHub Indonesia |
| **Buyer** | End-user yang mencari/membeli properti |
| **Seller** | Individual yang menjual properti |
| **Developer** | Perusahaan developer yang menjual properti |
| **Marketing** | Pihak yang melakukan promosi dengan referral |
| **Notary** | Pihak legal yang melakukan verifikasi dokumen |
| **Association** | Asosiasi Developer yang memverifikasi developer |
| **RBAC** | Role-Based Access Control |
| **JWT** | JSON Web Token |
| **SLA** | Service Level Agreement |
| **GMV** | Gross Merchandise Value |

---

## FUNCTIONAL REQUIREMENTS

### 2.1 AUTHENTICATION & AUTHORIZATION MODULE

#### 2.1.1 User Registration

**FR-AUTH-001: Buyer Registration**
- Sistem harus memungkinkan calon pembeli untuk mendaftar
- Input: Email, Password, Nama Lengkap, Nomor Telepon
- Validasi email (unique, format)
- Password requirement: min 8 karakter, uppercase, lowercase, number, special char
- Output: Account created, verification email sent
- Acceptance Criteria:
  - ✅ Email verification email dikirim dalam < 2 detik
  - ✅ Email unique check working
  - ✅ Password validation working
  - ✅ Account status = "pending_verification"

**FR-AUTH-002: Developer Registration**
- Developer harus register dengan informasi perusahaan
- Input: Company Name, Tax ID (NPWP), Address, Contact Person, Email
- Requires Association verification before activation
- Status flow: pending_verification → verified → active
- Acceptance Criteria:
  - ✅ Developer can only list properties after verified
  - ✅ Association can see pending developers
  - ✅ Verification audit log created

**FR-AUTH-003: Email Verification**
- Token-based email verification
- Token expiry: 24 hours
- Resend option dengan rate limit (max 3x per jam)
- Acceptance Criteria:
  - ✅ Token validates correctly
  - ✅ Token expires after 24 hours
  - ✅ Rate limiting enforced

**FR-AUTH-004: Login**
- Email + Password authentication
- JWT token generation (access + refresh)
- Session tracking
- Failed login attempt tracking
- Lock account after 5 failed attempts (15 menit)
- Acceptance Criteria:
  - ✅ JWT token contains proper claims
  - ✅ Account lockout working
  - ✅ Failed attempts tracked in audit log

**FR-AUTH-005: Password Reset**
- Forgot password flow
- Reset link valid untuk 1 jam
- Old password tidak bisa dipakai kembali
- Acceptance Criteria:
  - ✅ Reset token generated
  - ✅ Email sent dengan reset link
  - ✅ Token expires correctly

**FR-AUTH-006: Two-Factor Authentication (2FA)**
- Support TOTP (Google Authenticator, Authy)
- Support SMS (opsional)
- Backup codes generation
- Acceptance Criteria:
  - ✅ TOTP QR code generated
  - ✅ Verification code validated
  - ✅ Backup codes working

**FR-AUTH-007: OAuth2 Integration**
- Support login via Google
- Support login via Facebook
- Support login via WhatsApp Business
- Auto-link existing account jika email sama
- Acceptance Criteria:
  - ✅ Social login callback handled correctly
  - ✅ User data mapped properly
  - ✅ Account linking working

#### 2.1.2 Authorization & Permissions

**FR-AUTH-008: Role-Based Access Control (RBAC)**
- 9 roles: Super Admin, Admin, Association, Developer, Marketing, Notary, Seller, Buyer, Guest
- Each role has specific permissions
- Dynamic permission assignment
- Permission inheritance
- Acceptance Criteria:
  - ✅ User can only access permitted resources
  - ✅ Permission validation on every API call
  - ✅ Audit log records permission checks

**FR-AUTH-009: Permission Management**
- Super Admin can assign/revoke permissions
- Multiple permissions per role
- Permission caching (Redis) untuk performa
- Acceptance Criteria:
  - ✅ Permission changes applied immediately
  - ✅ Cache invalidation working
  - ✅ Audit log created

---

### 2.2 USER MANAGEMENT MODULE

**FR-USER-001: User Profile Management**
- View/Edit own profile
- Upload profile picture
- Update personal info (name, email, phone, address)
- Acceptance Criteria:
  - ✅ Profile updated correctly
  - ✅ Image upload working (max 5MB)
  - ✅ Email uniqueness maintained

**FR-USER-002: User Verification**
- Manual identity verification (KTP for Buyer)
- Admin dapat approve/reject
- Status: unverified → pending_review → verified → rejected
- Acceptance Criteria:
  - ✅ Document upload working
  - ✅ Admin review interface functional
  - ✅ Status change tracked

**FR-USER-003: User Deactivation/Deletion**
- Soft delete implementation
- User dapat meminta deactivation
- Admin dapat deactivate/ban
- Retention period: 30 hari sebelum hard delete
- Acceptance Criteria:
  - ✅ Soft delete working
  - ✅ User data still retrievable by admin
  - ✅ Audit trail maintained

**FR-USER-004: User Activity Tracking**
- Track login attempts
- Track profile changes
- Track booking/transaction history
- Acceptance Criteria:
  - ✅ Activity log comprehensive
  - ✅ Can query activity by user/date
  - ✅ Old logs archived

**FR-USER-005: Notification Preferences**
- User dapat set notification preferences
- Channel: Email, SMS, Push Notification, WhatsApp
- Frequency: Immediate, Daily, Weekly
- Acceptance Criteria:
  - ✅ Preferences saved correctly
  - ✅ Notifications respect user preference
  - ✅ Can opt-out completely (except critical)

---

### 2.3 PROPERTIES MODULE

#### 2.3.1 Property Management

**FR-PROP-001: Property Listing Creation**
- Support 12 property types:
  - Rumah Subsidi, Rumah Komersial, Rumah Second, Apartemen
  - Tanah, Villa, Gudang, Kos, Ruko, Kavling, Take Over, Syariah
- Mandatory fields: Name, Description, Price, Location, Category, Status
- Optional fields: Video, 360 View, Virtual Tour
- Status: draft → active → inactive → sold → archived
- Acceptance Criteria:
  - ✅ All property types supported
  - ✅ Status workflow enforced
  - ✅ Mandatory field validation

**FR-PROP-002: Property Details**
- Location data (province, city, district, address, coordinates)
- Building specs (bedrooms, bathrooms, building area, land area)
- Amenities (list of amenities)
- Legal status (SHM, SHGB, HGB, Hak Milik, etc.)
- Completion status (ready, under construction, off-plan)
- Acceptance Criteria:
  - ✅ All details can be queried
  - ✅ Filtering by specs working
  - ✅ Location validation (must be valid coordinates)

**FR-PROP-003: Property Media Management**
- Support photo gallery (multiple images)
- Support video (YouTube, Vimeo URLs)
- Support 360 view
- Support street view
- Max file size: 10MB per image
- Max 100 images per property
- Acceptance Criteria:
  - ✅ Images uploaded to R2/S3
  - ✅ CDN serving images
  - ✅ File size validation
  - ✅ Batch upload supported

**FR-PROP-004: Nearby Facilities**
- Auto-detect nearby facilities (Schools, Hospitals, Mosques, Airports, Malls, Gas Stations, Beaches, Terminals)
- Google Places API integration
- Manual add/edit facilities
- Distance calculation (< 5 km)
- Acceptance Criteria:
  - ✅ Nearby facilities auto-populated
  - ✅ Distance calculated correctly
  - ✅ Can manually override

**FR-PROP-005: Property Status Management**
- Available: Property dapat dipesan
- Booking: Sudah ada booking/pre-order
- Sold: Properti sudah terjual
- Take Over: Properti dengan skema take over
- Acceptance Criteria:
  - ✅ Status change tracked in audit log
  - ✅ Status change triggers notification
  - ✅ Only authorized users dapat change status

**FR-PROP-006: Property Search & Filter**
- Search by keyword (name, location)
- Filter by:
  - Property type
  - Price range (min-max)
  - Location (province, city, district)
  - Bedrooms (1-5+)
  - Bathrooms (1-4+)
  - Land area range
  - Building area range
  - Status
  - Legal status
- Radius search (dari coordinates, max 50km)
- Sorting: newest, price asc/desc, name asc/desc
- Pagination (limit 20-100 items)
- Acceptance Criteria:
  - ✅ Search performance < 500ms
  - ✅ Filters work in combination
  - ✅ Radius search accurate
  - ✅ Pagination working

**FR-PROP-007: Property Favorites/Wishlist**
- Buyer dapat save properti ke wishlist
- Can add/remove from wishlist
- Can view wishlist
- Can share wishlist
- Acceptance Criteria:
  - ✅ Wishlist saved to DB
  - ✅ Wishlist private to user
  - ✅ Can filter/search wishlist

**FR-PROP-008: Property Inquiry**
- Buyer dapat inquiry tentang properti
- Inquiry dikirim ke seller/developer
- Status: new → viewed → responded → closed
- Acceptance Criteria:
  - ✅ Inquiry created & notification sent
  - ✅ Seller can respond
  - ✅ Inquiry history tracked

---

### 2.4 BOOKING MODULE

**FR-BOOK-001: Booking Creation**
- Buyer dapat membuat booking
- Workflow: search → detail → booking
- Input: property_id, buyer_id, booking_date
- Generate booking reference number (unique)
- Status: pending → confirmed → cancelled
- Acceptance Criteria:
  - ✅ Booking created with unique ref
  - ✅ Booking fee calculated
  - ✅ Notification sent to developer

**FR-BOOK-002: Booking Identity Verification**
- Buyer harus upload KTP/ID card
- Support: KTP, Paspor, SIM
- Verify document readability
- Acceptance Criteria:
  - ✅ Document upload working
  - ✅ File size validation
  - ✅ Format validation (jpg, png, pdf)

**FR-BOOK-003: Booking Fee Payment**
- Booking fee calculation: typically 10-20% dari property price
- Support pembayaran via:
  - Virtual Account
  - QRIS
  - Credit Card (via Midtrans/Xendit)
  - E-wallet
- Webhook integration untuk payment confirmation
- Acceptance Criteria:
  - ✅ Payment gateway integrated
  - ✅ Webhook processed correctly
  - ✅ Payment status updated

**FR-BOOK-004: Developer Validation**
- Developer harus validate booking
- Can approve / reject / request more info
- Acceptance Criteria:
  - ✅ Developer receive notification
  - ✅ Can provide feedback
  - ✅ Status change tracked

**FR-BOOK-005: Notary Validation**
- Notary harus verify booking & identity
- Generate legal documents (PPJB, etc)
- Digital signature required
- Acceptance Criteria:
  - ✅ Notary interface functional
  - ✅ Document generation working
  - ✅ Signature validation working

**FR-BOOK-006: Booking Cancellation**
- Buyer dapat cancel dalam periode tertentu
- Seller dapat reject
- Refund policy implementation
- Acceptance Criteria:
  - ✅ Cancellation processed
  - ✅ Refund calculated correctly
  - ✅ Audit trail maintained

---

### 2.5 TRANSACTION & PAYMENT MODULE

**FR-TRANS-001: Transaction Lifecycle**
- Status: pending → payment_pending → payment_confirmed → processing → completed → failed
- Track transaction progression
- Acceptance Criteria:
  - ✅ Status flow enforced
  - ✅ Status change triggers notifications
  - ✅ Transaction history maintained

**FR-TRANS-002: Payment Methods**
- Virtual Account (Bank Transfer)
- QRIS (QR Code Indonesia Standard)
- Credit Card (processed via Midtrans)
- E-wallet (OVO, GoPay, Dana)
- Manual Transfer (admin verification)
- Acceptance Criteria:
  - ✅ All payment methods working
  - ✅ Payment gateway callbacks handled
  - ✅ Payment confirmation received

**FR-TRANS-003: Webhook Handling**
- Midtrans/Xendit webhook integration
- Verify webhook signature
- Idempotency handling (prevent duplicate processing)
- Retry mechanism untuk failed webhooks
- Acceptance Criteria:
  - ✅ Webhook signature verified
  - ✅ Duplicate webhooks handled
  - ✅ Failed webhooks retried

**FR-TRANS-004: Invoice Generation**
- Generate invoice PDF
- Include booking details, payment terms, legal notes
- Send invoice to buyer/seller
- Acceptance Criteria:
  - ✅ Invoice generated correctly
  - ✅ PDF with proper formatting
  - ✅ Invoice emailed to parties

**FR-TRANS-005: Refund Processing**
- Process refund ke original payment method
- Partial refund support
- Refund tracking & status
- Acceptance Criteria:
  - ✅ Refund processed correctly
  - ✅ Refund status tracked
  - ✅ Notification sent

---

### 2.6 COMMISSION & REFERRAL MODULE

**FR-COMM-001: Referral System**
- Marketing dapat generate referral link
- Referral link contains unique code
- Buyer click referral link → auto-attributed
- Commission triggered when booking confirmed
- Status: pending → approved → paid → failed
- Acceptance Criteria:
  - ✅ Referral link tracking working
  - ✅ Attribution correct
  - ✅ Commission calculation accurate

**FR-COMM-002: Commission Calculation**
- Auto-calculate commission based on:
  - Transaction amount
  - Commission rate (configurable per marketing/developer)
  - Bonus/incentive rules
- Formula: Commission = (Transaction Amount * Rate) + Bonuses
- Acceptance Criteria:
  - ✅ Calculation formula correct
  - ✅ Rate configurable by admin
  - ✅ Bonuses applied correctly

**FR-COMM-003: Commission Distribution**
- Generate commission records
- Track pending → approved → disbursed
- Support scheduled disbursement (weekly/monthly)
- Acceptance Criteria:
  - ✅ Commission records created
  - ✅ Status tracking working
  - ✅ Batch disbursement possible

**FR-COMM-004: Withdraw Request**
- Marketing dapat request withdraw
- Minimum withdraw amount (e.g., Rp 100,000)
- Support bank transfer
- Status: pending → approved → transferred
- Acceptance Criteria:
  - ✅ Withdraw request created
  - ✅ Bank account validation working
  - ✅ Transfer confirmation tracked

**FR-COMM-005: Commission Leaderboard**
- Ranking marketing by commission
- Period: Daily, Weekly, Monthly, All-time
- Top performers highlighted
- Acceptance Criteria:
  - ✅ Leaderboard calculated correctly
  - ✅ Rankings updating in real-time
  - ✅ Period filtering working

---

### 2.7 NOTARY & LEGAL DOCUMENTS MODULE

**FR-NOTARY-001: Document Management**
- Support document types:
  - PPJB (Perjanjian Pengikatan Jual Beli)
  - AJB (Akta Jual Beli)
  - SHM (Sertifikat Hak Milik)
  - SHGB (Sertifikat Hak Guna Bangunan)
  - Siteplan
  - PBG (Persetujuan Bangunan)
  - IMB (Izin Mendirikan Bangunan)
  - Certificate
- Upload, store, retrieve documents
- Version control for documents
- Acceptance Criteria:
  - ✅ All document types supported
  - ✅ Document storage secure
  - ✅ Version tracking working

**FR-NOTARY-002: Digital Signature**
- Support digital signature (e-sign)
- Integration dengan penyedia e-sign (e.g., Privy, Stacksign)
- Signature verification
- Timestamp recording
- Acceptance Criteria:
  - ✅ E-sign integration working
  - ✅ Signatures verified
  - ✅ Legally valid

**FR-NOTARY-003: Notary Schedule**
- Notary dapat set available time slots
- Buyer/Seller dapat book signing appointment
- Calendar integration
- Acceptance Criteria:
  - ✅ Schedule CRUD working
  - ✅ Booking prevents double-booking
  - ✅ Reminder sent before appointment

**FR-NOTARY-004: Payment Verification**
- Notary verify payment received
- Status: pending_verification → verified → rejected
- Acceptance Criteria:
  - ✅ Notary interface functional
  - ✅ Payment details visible
  - ✅ Verification recorded

**FR-NOTARY-005: Legal Status Management**
- Track legal status of property
- Status: SHM (Sertifikat Hak Milik), SHGB, HGB, etc
- Verify legal status authenticity
- Acceptance Criteria:
  - ✅ Status can be queried
  - ✅ Status history tracked
  - ✅ Verification possible

---

### 2.8 NOTIFICATION MODULE

**FR-NOTIF-001: Multi-Channel Notifications**
- Email notifications
- SMS notifications (via Twilio/Nexmo)
- WhatsApp notifications (via WhatsApp Business API)
- Push notifications (mobile app)
- In-app notifications
- Acceptance Criteria:
  - ✅ All channels working
  - ✅ Notifications delivered
  - ✅ Delivery tracking

**FR-NOTIF-002: Notification Templates**
- Customizable notification templates
- Variable substitution
- Multi-language support
- Acceptance Criteria:
  - ✅ Templates stored in DB
  - ✅ Variables replaced correctly
  - ✅ Multiple languages supported

**FR-NOTIF-003: Notification Queue**
- Queue-based notification processing
- Retry mechanism untuk failed notifications
- Rate limiting untuk prevent spam
- Acceptance Criteria:
  - ✅ Queue processing working
  - ✅ Failed notifications retried
  - ✅ Rate limiting enforced

**FR-NOTIF-004: Notification Preferences**
- User dapat opt-in/opt-out per channel
- Critical notifications dapat tidak di-opt-out
- Frequency preferences
- Acceptance Criteria:
  - ✅ Preferences respected
  - ✅ Critical notifications always sent
  - ✅ Unsubscribe working

---

### 2.9 CHAT & MESSAGING MODULE

**FR-CHAT-001: Real-time Chat**
- 1-to-1 messaging
- WebSocket-based real-time communication
- Message delivery confirmation
- Typing indicator
- Acceptance Criteria:
  - ✅ Messages delivered in real-time
  - ✅ Delivery status tracked
  - ✅ Typing indicator working

**FR-CHAT-002: Chat Participants**
- Support chat between:
  - Buyer ↔ Seller/Developer
  - Buyer ↔ Marketing
  - Buyer ↔ Notary
  - Admin ↔ Any user
- Access control untuk chat
- Acceptance Criteria:
  - ✅ Only authorized users dapat chat
  - ✅ Chat history accessible
  - ✅ Participant validation

**FR-CHAT-003: Message History**
- Retrieve message history
- Pagination support
- Search messages
- Acceptance Criteria:
  - ✅ Message history retrievable
  - ✅ Search functional
  - ✅ Performance acceptable

**FR-CHAT-004: Chat Notifications**
- Notify user of new messages
- Notification only if not active in chat
- Sound notification option
- Acceptance Criteria:
  - ✅ Notifications sent correctly
  - ✅ Not sent if user active
  - ✅ Notification preferences respected

---

### 2.10 DASHBOARD & ANALYTICS MODULE

**FR-DASH-001: Buyer Dashboard**
- Overview: Active bookings, wishlist count, total transactions
- Bookings list: status, property, date
- Transaction history: dates, amounts, status
- Wishlist: properties saved
- Favorite properties: recently viewed
- Notifications: recent notifications
- Chat: recent conversations
- Profile section: edit info, change password
- Acceptance Criteria:
  - ✅ Dashboard loads in < 2s
  - ✅ Data is current
  - ✅ All sections functional

**FR-DASH-002: Seller Dashboard**
- Listings: active, inactive, sold
- Listing stats: views, inquiries, bookings
- Inquiries: list, respond
- Chat: buyer conversations
- Documents: download/upload
- Analytics: simple stats
- Acceptance Criteria:
  - ✅ Real-time stats
  - ✅ Inquiry management working
  - ✅ Document management functional

**FR-DASH-003: Developer Dashboard**
- Projects: list, create, edit, stats
- Properties: manage, add, edit, status
- Construction progress: upload images/videos
- Brochures: upload, manage
- Sales management: bookings, transactions
- Inventory: available units, sold units
- Reports: sales by period
- Acceptance Criteria:
  - ✅ Project management working
  - ✅ Property upload functional
  - ✅ Sales tracking accurate

**FR-DASH-004: Marketing Dashboard**
- Referral link: generate, view metrics
- Campaign: create, manage
- QR codes: generate for campaigns
- Commission: current, history
- Withdraw: pending, paid history
- Performance: conversion rate, commission trend
- Leaderboard: ranking
- Acceptance Criteria:
  - ✅ Referral link generation working
  - ✅ QR code generation functional
  - ✅ Metrics calculating correctly

**FR-DASH-005: Notary Dashboard**
- Pending bookings: list, action required
- Payments: verify status
- Signing schedule: appointments, reminders
- Documents: upload, manage
- Certificates: issued, archived
- Legal status: verification history
- Acceptance Criteria:
  - ✅ Booking queue visible
  - ✅ Document upload working
  - ✅ Schedule management functional

**FR-DASH-006: Association Dashboard**
- Developer applications: pending, approved, rejected
- Developer list: active, suspended
- Verification: approve/reject, feedback
- Membership: status, fees
- Reports: member activity, status
- Acceptance Criteria:
  - ✅ Application review interface working
  - ✅ Verification process functional
  - ✅ Reports generating correctly

**FR-DASH-007: Admin Dashboard**
- Master data management: users, properties, developers
- Approvals: pending items requiring admin action
- Audit logs: activity tracking
- Analytics: system-wide stats
- Settings: configuration management
- Reports: comprehensive reports
- Acceptance Criteria:
  - ✅ Master data CRUD working
  - ✅ Approval workflows functional
  - ✅ Reports comprehensive

**FR-DASH-008: Super Admin Dashboard**
- All admin features
- System health: uptime, errors, performance
- User management: all roles
- Permission management
- Backup/restore access
- System logs
- Acceptance Criteria:
  - ✅ System health visible
  - ✅ User management complete
  - ✅ Logs comprehensive

**FR-DASH-009: Analytics & Reporting**
- Sales analytics: volume, revenue by period
- Revenue analytics: by property type, location
- Commission analytics: paid, pending
- Heatmap: popular properties by location
- Popular properties: top 10 by views/bookings
- Popular developers: top developers
- Marketing performance: top marketers
- User growth: trend over time
- Acceptance Criteria:
  - ✅ All metrics calculated correctly
  - ✅ Charts rendering properly
  - ✅ Data export (CSV) working

---

### 2.11 CMS MODULE

**FR-CMS-001: Content Management**
- Manage landing page content
- Blog post management: create, edit, publish
- News management
- FAQ management
- Acceptance Criteria:
  - ✅ CRUD operations working
  - ✅ Publishing workflow functional
  - ✅ Content versioning

**FR-CMS-002: Slider/Banner Management**
- Create/edit sliders
- Manage banner placement
- Schedule content (start/end date)
- A/B testing support
- Acceptance Criteria:
  - ✅ Slider management working
  - ✅ Scheduling functional
  - ✅ Content displays correctly

**FR-CMS-003: SEO Management**
- Meta tags management
- Dynamic sitemap generation
- Schema.org markup
- OpenGraph tags
- Twitter Cards
- Canonical URLs
- Breadcrumb navigation
- Acceptance Criteria:
  - ✅ Meta tags applied
  - ✅ Sitemap generated & updated
  - ✅ Schema validation passing

---

### 2.12 AUDIT LOG & COMPLIANCE MODULE

**FR-AUDIT-001: Audit Logging**
- Log all user actions:
  - Login/logout
  - Create/update/delete
  - Permission changes
  - Payment transactions
  - Admin actions
- Include: user_id, action, resource, timestamp, ip_address, user_agent
- Acceptance Criteria:
  - ✅ All actions logged
  - ✅ Logs immutable
  - ✅ Retention policy enforced

**FR-AUDIT-002: Audit Log Retrieval**
- Query audit logs by user, action, resource, date range
- Filter & search functionality
- Export to CSV
- Acceptance Criteria:
  - ✅ Queries performant
  - ✅ Filters working
  - ✅ Export functional

**FR-AUDIT-003: Compliance Tracking**
- Track compliance-related events
- Regulatory requirements documentation
- Acceptance Criteria:
  - ✅ Compliance events tracked
  - ✅ Can generate compliance reports

---

## NON-FUNCTIONAL REQUIREMENTS

### 3.1 PERFORMANCE REQUIREMENTS

**NFR-PERF-001: Response Time**
- API response time: < 200ms (p95)
- Page load time: < 2 seconds
- Search query: < 500ms
- Acceptance Criteria:
  - ✅ Meet response time targets
  - ✅ Monitor with APM tools

**NFR-PERF-002: Throughput**
- Support 10,000 concurrent users
- API: 1000 req/sec
- Database: 500 connections
- Acceptance Criteria:
  - ✅ Load testing validates

**NFR-PERF-003: Scalability**
- Horizontal scaling for app servers
- Database read replicas
- Cache layer (Redis Cluster)
- CDN for static assets
- Acceptance Criteria:
  - ✅ Can scale dynamically

**NFR-PERF-004: Caching**
- Redis cache layer
- Browser caching (1 year for static)
- CDN caching
- Cache invalidation strategy
- Acceptance Criteria:
  - ✅ Cache hit ratio > 80%
  - ✅ Cache invalidation working

---

### 3.2 AVAILABILITY & RELIABILITY

**NFR-AVAIL-001: Uptime SLA**
- Target: 99.9% uptime (43 minutes downtime/month)
- Measured monthly
- Excludes scheduled maintenance
- Acceptance Criteria:
  - ✅ Infrastructure redundancy
  - ✅ Automated failover
  - ✅ Health checks configured

**NFR-AVAIL-002: Disaster Recovery**
- RTO (Recovery Time Objective): < 1 hour
- RPO (Recovery Point Objective): < 15 minutes
- Daily backups
- Backup replication to different region
- Acceptance Criteria:
  - ✅ Backup & restore tested
  - ✅ DR plan documented

**NFR-AVAIL-003: Monitoring & Alerting**
- Real-time monitoring
- Alert thresholds for critical metrics
- Incident response procedures
- Acceptance Criteria:
  - ✅ Monitoring tools configured
  - ✅ Alert channels working

---

### 3.3 SECURITY REQUIREMENTS

**NFR-SEC-001: Authentication**
- JWT token-based authentication
- Token expiry: 1 hour (access), 7 days (refresh)
- HTTPS/TLS 1.2+
- Acceptance Criteria:
  - ✅ All endpoints secured
  - ✅ Token validation working

**NFR-SEC-002: Authorization**
- RBAC enforcement
- Permission checks on every API call
- Attribute-based access control (ABAC) for complex rules
- Acceptance Criteria:
  - ✅ Unauthorized requests blocked
  - ✅ Permission logs generated

**NFR-SEC-003: Data Encryption**
- At-rest: AES-256
- In-transit: TLS 1.2+
- PII fields encrypted (credit card, KTP number)
- Acceptance Criteria:
  - ✅ Encryption implemented
  - ✅ Key rotation automated

**NFR-SEC-004: API Security**
- Rate limiting: 1000 req/min per user
- CORS: whitelist allowed origins
- Input validation & sanitization
- SQL injection prevention (parameterized queries)
- XSS prevention (output encoding)
- CSRF token validation
- Acceptance Criteria:
  - ✅ All protections implemented
  - ✅ Penetration testing passed

**NFR-SEC-005: WAF & DDoS Protection**
- Cloudflare WAF enabled
- DDoS protection active
- Rate limiting rules
- Acceptance Criteria:
  - ✅ WAF rules configured
  - ✅ DDoS mitigation tested

**NFR-SEC-006: Audit Logging**
- Log all sensitive operations
- Log retention: 2 years
- Logs cannot be deleted/modified by regular users
- Acceptance Criteria:
  - ✅ Audit logs complete
  - ✅ Retention enforced

**NFR-SEC-007: Vulnerability Management**
- Regular security audits (quarterly)
- Dependency scanning (automated)
- SAST (Static Application Security Testing)
- Penetration testing (annual)
- Acceptance Criteria:
  - ✅ No critical vulnerabilities
  - ✅ Dependency updates current

---

### 3.4 SCALABILITY & MAINTAINABILITY

**NFR-SCALE-001: Architecture**
- Modular monolith design
- Event-driven architecture
- Queue system for async tasks
- Microservices-ready (future migration path)
- Acceptance Criteria:
  - ✅ Modules independent
  - ✅ Can extract to microservices

**NFR-SCALE-002: Code Quality**
- Code coverage: > 80%
- SOLID principles adherence
- Clean code practices (naming, functions, classes)
- Code review before merge
- Acceptance Criteria:
  - ✅ Tests passing
  - ✅ Code review approved

**NFR-SCALE-003: Documentation**
- API documentation (OpenAPI/Swagger)
- Code documentation (PHPDoc, JSDoc)
- Architecture documentation
- Deployment documentation
- Acceptance Criteria:
  - ✅ Docs complete & current
  - ✅ Developer onboarding easy

**NFR-SCALE-004: Deployment**
- CI/CD pipeline (GitHub Actions)
- Blue-green deployment
- Rollback capability
- Staging environment
- Acceptance Criteria:
  - ✅ Automated deployments
  - ✅ Zero-downtime deploys

---

### 3.5 USABILITY REQUIREMENTS

**NFR-USE-001: Accessibility**
- WCAG 2.1 AA compliance
- Screen reader support
- Keyboard navigation
- Color contrast ratio > 4.5:1
- Acceptance Criteria:
  - ✅ Accessibility audit passed

**NFR-USE-002: Responsiveness**
- Mobile-first design
- Support: Mobile (375px), Tablet (768px), Desktop (1440px+)
- Touch-friendly UI (min 44px touch target)
- Acceptance Criteria:
  - ✅ Works on all screen sizes
  - ✅ Touch targets appropriate

**NFR-USE-003: User Experience**
- Intuitive navigation
- Clear error messages
- Loading indicators
- Progressive enhancement
- Dark mode support
- Acceptance Criteria:
  - ✅ Usability testing passed
  - ✅ User satisfaction > 4/5

**NFR-USE-004: Performance (UX)**
- Lighthouse score > 95
- First Contentful Paint (FCP) < 1.8s
- Largest Contentful Paint (LCP) < 2.5s
- Cumulative Layout Shift (CLS) < 0.1
- Acceptance Criteria:
  - ✅ Lighthouse scores achieved
  - ✅ Core Web Vitals passed

---

## SYSTEM REQUIREMENTS

### 4.1 INFRASTRUCTURE REQUIREMENTS

**SYS-INFRA-001: Server Requirements**
- Backend: Laravel 12 + PHP 8.4
- Web Server: NGINX 1.24+
- Application Server: PHP-FPM 8.4
- Load Balancer: NGINX/HAProxy
- Acceptance Criteria:
  - ✅ Versions specified & tested

**SYS-INFRA-002: Database Requirements**
- MySQL 8.0+
- Master-Slave replication
- Connection pooling
- Backup strategy: daily incremental, weekly full
- Acceptance Criteria:
  - ✅ Database configured
  - ✅ Replication working
  - ✅ Backups scheduled

**SYS-INFRA-003: Cache Requirements**
- Redis 7.0+
- Cluster mode for HA
- Data persistence (RDB + AOF)
- Acceptance Criteria:
  - ✅ Redis cluster operational
  - ✅ Persistence configured

**SYS-INFRA-004: Queue Requirements**
- Redis-based queue
- Job processing with Supervisor
- Retry mechanism
- Failed job tracking
- Acceptance Criteria:
  - ✅ Queue processing working
  - ✅ Failed jobs handled

**SYS-INFRA-005: Storage Requirements**
- Cloudflare R2 or AWS S3
- CDN integration
- Backup replication
- Acceptance Criteria:
  - ✅ Files stored securely
  - ✅ CDN serving correctly
  - ✅ Backups replicated

---

### 4.2 CONTAINER & ORCHESTRATION

**SYS-CONT-001: Docker**
- Dockerfile per service
- Docker Compose for dev environment
- Multi-stage builds for optimization
- .dockerignore configured
- Acceptance Criteria:
  - ✅ Images build successfully
  - ✅ Dev environment works

**SYS-CONT-002: Container Registry**
- Private Docker registry (GitHub Container Registry)
- Image scanning for vulnerabilities
- Versioning/tagging strategy
- Acceptance Criteria:
  - ✅ Images pushed & available
  - ✅ Security scans passing

**SYS-CONT-003: Orchestration**
- Consider Docker Swarm or Kubernetes (future)
- For MVP: Docker Compose + managed containers
- Acceptance Criteria:
  - ✅ Container management working

---

## USER REQUIREMENTS

### 5.1 BUYER REQUIREMENTS

**USR-BUYER-001:**
- Dapat search & filter properti dengan mudah
- Dapat save favorit & wishlist
- Dapat melakukan booking dengan proses jelas
- Menerima notifikasi status booking
- Dapat download dokumen & invoice
- Dapat berkomunikasi dengan seller/developer/notary

**USR-BUYER-002:**
- Interface mudah digunakan (intuitif)
- Fast loading time
- Mobile-friendly
- Aman (verifikasi 2FA, enkripsi data)

---

### 5.2 SELLER REQUIREMENTS

**USR-SELLER-001:**
- Dapat easily upload & manage listing
- Dapat melihat inquiries dari buyers
- Dapat respond inquiries & chat dengan buyers
- Dapat track bookings & transactions
- Dapat melihat analytics (views, inquiries, bookings)

---

### 5.3 DEVELOPER REQUIREMENTS

**USR-DEV-001:**
- Project management dashboard
- Property inventory management
- Construction progress tracking
- Sales performance monitoring
- Commission & withdraw management
- Document management

---

### 5.4 MARKETING REQUIREMENTS

**USR-MARKETING-001:**
- Generate referral links & campaigns
- QR code generation
- Real-time commission tracking
- Leaderboard ranking
- Withdraw request submission
- Performance analytics

---

### 5.5 NOTARY REQUIREMENTS

**USR-NOTARY-001:**
- Booking verification queue
- Payment verification
- Signing schedule management
- Document upload & management
- Digital signature capability
- Legal status tracking

---

### 5.6 ASSOCIATION REQUIREMENTS

**USR-ASSOC-001:**
- Developer application review
- Verification approval/rejection
- Member management
- Activity monitoring
- Reports generation

---

### 5.7 ADMIN REQUIREMENTS

**USR-ADMIN-001:**
- Complete system control
- User management
- Property moderation
- Approval workflows
- Audit log review
- System configuration

---

### 5.8 SUPER ADMIN REQUIREMENTS

**USR-SUPER-ADMIN-001:**
- All admin capabilities
- Permission management
- System health monitoring
- Comprehensive reporting
- Backup/restore access

---

## MODULE SPECIFICATIONS

*(Detailed specifications for each module - referenced from Module Breakdown section)*

Each module shall implement:
1. Database schema with migrations
2. Models with relationships
3. Repository pattern for data access
4. Service layer for business logic
5. API endpoints (REST)
6. Input validation (Form Requests)
7. Authorization policies
8. Event listeners & queue jobs
9. Unit & integration tests
10. API documentation (OpenAPI)
11. Frontend components (React/Next.js)
12. Mobile screens (Flutter)

---

## API SPECIFICATIONS

### 7.1 REST API Standards

**Base URL**: `https://api.propertyhub.id/api/v1`

**API Format**:
```json
{
  "status": "success|error",
  "code": 200,
  "message": "string",
  "data": {},
  "meta": {
    "pagination": {
      "current_page": 1,
      "per_page": 20,
      "total": 1000,
      "last_page": 50
    }
  }
}
```

**HTTP Status Codes**:
- 200: OK
- 201: Created
- 204: No Content
- 400: Bad Request
- 401: Unauthorized
- 403: Forbidden
- 404: Not Found
- 422: Unprocessable Entity
- 429: Too Many Requests
- 500: Internal Server Error

**Authentication**: Bearer Token (JWT)
```
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

**Rate Limiting**:
- Headers: `X-RateLimit-Limit`, `X-RateLimit-Remaining`, `X-RateLimit-Reset`
- Limit: 1000 requests per minute per user

**Pagination**:
- Query params: `page=1&per_page=20`
- Support cursor-based: `cursor=abc123&limit=20`

**Filtering**:
- Query params: `filter[status]=active&filter[price_min]=1000000`

**Sorting**:
- Query params: `sort=-created_at,name` (- for DESC)

---

### 7.2 GraphQL API (Future)

GraphQL support planned for Phase 2. Specifications TBD.

---

## DATA REQUIREMENTS

### 8.1 Data Storage

**Primary**: MySQL 8.0+ (OLTP)
**Cache**: Redis (sessions, query cache)
**Document Store**: Potentially MongoDB (future for analytics)
**Data Warehouse**: Planned for Phase 2 (analytics aggregation)

### 8.2 Data Retention

| Data Type | Retention | Notes |
|-----------|-----------|-------|
| User data | Until deletion | Soft delete with 30-day grace period |
| Transaction data | 7 years | Legal requirement |
| Audit logs | 2 years | Compliance |
| Chat messages | 1 year | Retention after booking completion |
| Analytics data | 2 years | Aggregated after 1 year |
| Temporary files | 30 days | Auto-cleanup |

---

### 8.3 Data Backup

- **Frequency**: Daily (incremental), Weekly (full)
- **Retention**: 30 days
- **Location**: Primary + Secondary region (geo-redundancy)
- **Testing**: Monthly restore test

---

## SECURITY REQUIREMENTS

*(Detailed in Section 3.3 above)*

Key implementations:
- JWT authentication with RS256
- RBAC with permission matrix
- Input validation & sanitization
- SQL injection prevention
- XSS protection
- CSRF tokens
- Rate limiting
- WAF (Cloudflare)
- DDoS protection
- Encryption (AES-256 at-rest, TLS 1.2+ in-transit)
- Audit logging
- Regular security audits & penetration testing

---

## PERFORMANCE REQUIREMENTS

*(Detailed in Section 3.1 above)*

Key metrics:
- API response < 200ms (p95)
- Page load < 2s
- Lighthouse > 95
- LCP < 2.5s
- CLS < 0.1
- 10,000 concurrent users
- 99.9% uptime

---

## ACCEPTANCE & SIGN-OFF

| Role | Name | Signature | Date |
|------|------|-----------|------|
| Product Manager | _____ | _____ | _____ |
| Tech Lead | _____ | _____ | _____ |
| QA Lead | _____ | _____ | _____ |
| Security Lead | _____ | _____ | _____ |

---

## REVISION HISTORY

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2026-07-04 | Principal Architect | Initial SRS |

---

**Document Status**: READY FOR DEVELOPMENT  
**Next Step**: Business Process Diagrams & Use Case Analysis

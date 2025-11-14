

# **Ticketing Web Application – Project Overview**

## **1. Project Summary**

This project is a scalable, modern ticketing platform that allows **promoters** to publish events, **customers** to purchase tickets, and **promoters/admins** to validate tickets using QR-code scanning in real time.

The platform must support:

* Secure online payments with automatic revenue split
* Fraud-proof tickets
* Real-time scanning at venue doors
* Promoter dashboards
* Admin oversight
* Event search & filtering
* Email notifications & ticket delivery

The long-term goal is to build a reliable alternative to Ticketmaster for small venues, starting with a low-cost but scalable MVP.

---

## **2. User Roles & Permissions**

### **Customer**

* Browse and filter events
* Purchase tickets via Stripe Checkout
* Access personal dashboard
* View tickets (QR code), download PDF
* Receive email confirmation & reminders
* Manage tickets (view history, transfer in future versions)

### **Promoter**

* Account approved by admin
* Connect Stripe account for payouts
* Create and manage events
* Configure ticket quantities and prices
* View sales analytics
* Download attendee list
* Issue refunds (when allowed for event)
* Access in-app **QR code scanner** for validating tickets in real time

### **Admin**

* Full control over all events, users, promoters, orders
* Approve promoter accounts
* Manage refunds and disputes
* Edit any event or ticket type

---

## **3. Core Features**

### **Event Management**

* Promoters create events
* Add date, venue, description, cover photo
* Define multiple ticket types (GA, VIP, Free, etc.)

### **Ticketing System**

* Each purchased ticket = **unique, non-forgeable QR code**
* Tickets linked to orders and scanned via promoter app section
* Detailed anti-fraud system:

  * Row-level locking in the database
  * Real-time scan validation
  * Race-condition safe

### **Payments System**

* Fully implemented with **Stripe Checkout + Stripe Connect**
* Customer pays the ticket price + platform fee
* Automatic revenue split:

  * Promoter receives their share
  * Platform receives its fee
* Support for refunds (unless event is marked “no refunds”)
* Stripe webhooks ensure payment reliability and anti-fraud

### **QR Scanning**

* In-browser scanning using smartphone/desktop camera
* Multiple staff can scan simultaneously
* Real-time updates & sync
* Backend prevents duplicates through DB transactions

### **Email & Notifications**

* Purchase confirmation email
* Ticket PDF/QR delivered by email
* Event reminder email 24h before event
* Transactional email system integrated
  (**Resend**)

### **Search & Filtering**

* Search events by:

  * Date
  * Location
  * Venue
  * Price (≤, ≥)
  * Free events
  * Categories (later addition)

---

## **4. Final Tech Stack**

This is the definitive chosen stack for the entire project.

### **Frontend + Backend Framework**

### → **Next.js (App Router) + TypeScript**

Reasons:

* Server and client in one codebase
* Built-in API routes / server actions
* SEO-friendly for public event pages
* Fast deployment & low cost (Vercel)
* Perfect for scalable MVPs

### **Styling**

### → **Tailwind CSS**

Reasons:

* Fast UI development
* Perfect with Next.js
* Consistent with modern SaaS UI patterns

### **Database**

### → **PostgreSQL (hosted on Neon or Supabase)**

Reasons:

* ACID transactions (critical for ticket scanning)
* Scalable and reliable
* SQL indexing perfect for searching/filtering

### **ORM**

### → **Prisma**

Reasons:

* Strong typing
* Great developer experience
* Clean schema management
* Perfect for TypeScript systems

### **Authentication**

### → **Auth.js (NextAuth)**

Reasons:

* Native to Next.js
* Secure sessions
* Supports email/password + Google/Apple later
* Role-based auth easily added

### **Payments**

### → **Stripe + Stripe Checkout + Stripe Connect**

Reasons:

* Easiest path to launch payments
* Checkout handles all compliance
* Connect enables **automatic revenue split**
* Full support for refunds

### **Email**

### → **Resend**

Reasons:

* Modern, reliable
* Simple API
* Works beautifully with Next.js
* Free tier perfect for MVP

### **QR Code Generation**

### → **qrcode (npm library)** for generating images

### → **html5-qrcode** for scanning

Reasons:

* Both widely used and production-safe
* Work perfectly in browser-based scanner

### **Hosting**

### → **Vercel** (for the entire webapp)

### → **Neon / Supabase** (for DB)

Reasons:

* Free/cheap
* Auto-scaling
* CI/CD built in
* Ideal for MVP → production growth

---

## **5. High-Level Architecture**

```
┌───────────────┐        ┌───────────────────┐
│   Frontend     │        │   Backend (API)   │
│ Next.js (UI)   │ <────> │ Next.js Routes    │
└───────────────┘        └───────────────────┘
           │                          │
           │                          │
           ▼                          ▼
   ┌───────────────────┐      ┌────────────────────┐
   │  Auth.js Sessions  │      │  PostgreSQL (Neon) │
   │  (Cookies/JWT)     │      │  Prisma ORM        │
   └───────────────────┘      └────────────────────┘
           │                          │
           ▼                          ▼
        ┌─────────────────────────────────┐
        │              Stripe              │
        │   Checkout + Connect + Webhooks  │
        └─────────────────────────────────┘
           │                          
           ▼                          
   ┌───────────────────┐
   │      Resend        │
   │ (Transactional Email)
   └───────────────────┘
```

---

## **6. Database Entities (MVP)**

### **User**

* id
* email
* password_hash
* role: `customer | promoter | admin`

### **PromoterProfile**

* user_id
* stripe_connect_id
* legal info

### **Event**

* promoter_id
* title
* date
* venue_name
* location
* description
* banner_image_url

### **TicketType**

* event_id
* name
* price
* quantity_total
* quantity_sold

### **Order**

* user_id
* event_id
* total_amount
* stripe_payment_intent_id
* status

### **Ticket**

* order_id
* ticket_type_id
* unique_token
* status: valid | used | refunded
* scanned_at

### **ScanLog**

* ticket_id
* scanned_by_user_id
* scanned_at

---

## **7. Real-Time QR Scan Validation Logic**

### **Scan request flow:**

1. Browser reads QR → sends token to API
2. Backend starts **DB transaction**
3. Finds ticket with `FOR UPDATE` lock
4. If ticket is:

   * nonexistent → invalid
   * refunded → reject
   * already used → reject
   * belongs to wrong promoter/event → reject
5. Otherwise:

   * mark as `used`
   * save ScanLog
   * commit transaction
6. Return **“VALID”** to UI

This guarantees **no duplicates**, even if 5 bouncers scan the same ticket simultaneously.

---

## **8. Development Roadmap (Phases)**

### **Phase 1 — Foundations**

* Setup Next.js project
* Configure Tailwind
* Setup Auth.js
* Connect Postgres via Prisma
* Add roles & protected routes

### **Phase 2 — Event & Ticket Models**

* Implement CRUD for events
* Implement TicketType creation & editing
* Public event pages

### **Phase 3 — Payments**

* Stripe integration
* Stripe Checkout
* Stripe Connect onboard flow for promoters
* Webhooks setup
* Order + ticket creation logic

### **Phase 4 — Ticketing**

* QR code generation
* Ticket viewer
* Email delivery with Resend

### **Phase 5 — Scanning System**

* In-browser scanner
* Real-time validation
* Anti-fraud DB locks

### **Phase 6 — Promoter Dashboard**

* Analytics
* Ticket list
* Refund requests

### **Phase 7 — Search System**

* Filtering by date, location, free, price

### **Phase 8 — Admin Panel**

* Full oversight UI
* Promoter approvals

---

## **9. Scalability Plan**

* Vercel auto-scaling for frontend/serverless functions
* PostgreSQL vertical & horizontal scaling via Neon
* Add Redis cache later for:

  * Search speed
  * Event load optimization
* CDN for images (Vercel Blob / R2 Storage)
* Logging with Sentry + Vercel Analytics

This architecture can easily support **tens of thousands of users**.

---

## **10. Long-Term Vision**

* Competitive alternative for small-to-mid venues
* Low platform fees compared to Ticketmaster
* Mobile apps integrated with the same API
* Advanced promoter analytics
* Ticket transfer market
* Dynamic pricing
* Multi-language & multi-currency support

---



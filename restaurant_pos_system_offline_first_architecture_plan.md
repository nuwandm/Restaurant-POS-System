# 🍽️ Restaurant POS System — COMPLETE PRODUCT PLAN (PRODUCTION-READY)

> **Strategy:** Simple → Sellable → Scalable
> **Architecture:** Offline-First → Cloud-Ready → SaaS-Enabled
> **Stack:** Electron + React + SQLite → Node.js + MongoDB

---

# 🧠 1. PROJECT VISION

Build a **restaurant POS system** that is:

- Sold as a **standalone offline desktop application** (immediate revenue)
- Upgraded to a **cloud-connected system on demand** (recurring revenue)
- Scalable to a **multi-branch SaaS platform** (maximum revenue)

## Core Business Goal

| Stage | Goal | Revenue Model |
|---|---|---|
| Phase 1 | Offline POS — fast to market | One-time license fee |
| Phase 2–3 | Add business modules | Upsell to higher tier |
| Phase 4–5 | Cloud + Sync | Monthly SaaS subscription |
| Phase 6 | Ecosystem + Franchise | Enterprise contracts |

## Why This Will Succeed

- **Sri Lanka market gap**: Most existing POS systems are expensive, foreign, or poorly localized
- **Offline-first is king**: Power cuts and poor internet are real — competitors fail here
- **Grow with the customer**: Start free/cheap, monetize as they grow
- **Own the hardware relationship**: Bundle hardware for turnkey deals

---

# 🎯 2. TARGET USERS & PERSONAS

## Persona 1 — Small Restaurant (Offline Only)
- 1–2 staff, limited budget
- Needs: Fast order entry, bill printing, daily totals
- Pain: Current system is pen & paper or old software
- Budget: One-time LKR 15,000–40,000

## Persona 2 — Medium Restaurant (Business Owner)
- 5–15 staff, wants reports and inventory
- Needs: Stock control, customer data, month-end reports
- Pain: No visibility into what's profitable
- Budget: LKR 5,000–10,000/month SaaS

## Persona 3 — Large / Multi-Branch (Entrepreneur)
- 3–10 branches, hires managers
- Needs: Central dashboard, branch comparison, remote access
- Pain: No single view across all branches
- Budget: LKR 15,000–30,000/month enterprise

---

# 🏗️ 3. TECHNOLOGY ARCHITECTURE

## 3.1 Tech Stack (Full Decision)

| Layer | Technology | Why |
|---|---|---|
| **Desktop App** | Electron 28+ | Cross-platform, web tech, hardware access |
| **Frontend UI** | React 18 + TypeScript | Component reuse, type safety |
| **UI Library** | shadcn/ui + Tailwind CSS | Fast, modern, fully customizable |
| **State Management** | Zustand | Lightweight, no boilerplate |
| **Local Database** | SQLite via better-sqlite3 | Embedded, zero-config, fast queries |
| **Local ORM** | Drizzle ORM | Type-safe SQL for SQLite local DB |
| **Cloud API** | Node.js + Express / Fastify | Fast, low overhead REST/WebSocket server |
| **Cloud Database** | MongoDB Atlas | Flexible documents, JSON-native, free tier |
| **Cloud ODM** | Mongoose 8+ | Schema validation + middleware for MongoDB |
| **Auth** | JWT + refresh tokens | Stateless, works offline-first |
| **Sync Engine** | Custom queue + timestamps | Conflict resolution for offline edits |
| **Print** | node-escpos + USB/Network | ESC/POS thermal printer protocol |
| **Build** | Vite + electron-builder | Fast builds, auto-update support |
| **Mobile App** | React Native + Expo | Code share with web components |
| **CI/CD** | GitHub Actions | Automated builds + releases |
| **Hosting** | Render / Railway + MongoDB Atlas | Atlas free tier → paid as you scale |

## 3.2 Offline-First Architecture

```
┌─────────────────────────────────────────────┐
│              Electron Desktop App            │
│  ┌──────────┐  ┌──────────┐  ┌───────────┐  │
│  │  React   │  │ Zustand  │  │  Drizzle  │  │
│  │   UI     │◄─┤  Store   │◄─┤ ORM/SQLite│  │
│  └──────────┘  └──────────┘  └─────┬─────┘  │
│                                     │        │
│                              ┌──────▼──────┐ │
│                              │   SQLite    │ │
│                              │  (Local DB) │ │
│                              └──────┬──────┘ │
└─────────────────────────────────────┼────────┘
                                      │
                              ┌───────▼────────┐
                              │  Sync Engine   │
                              │  (Queue-based) │
                              └───────┬────────┘
                                      │ (when online)
                              ┌───────▼────────┐
                              │  Cloud API     │
                              │  (Express)     │
                              └───────┬────────┘
                                      │
                              ┌───────▼────────┐
                              │  MongoDB Atlas │
                              │  (Cloud DB)    │
                              └────────────────┘
```

## 3.3 Database Design

### Local (SQLite — Offline)
Normalized relational tables for fast local querying:
```
restaurants, branches, staff, roles, permissions
categories, menu_items, item_variants, item_modifiers, combos
orders, order_items, order_status_history, kot_tickets
floor_plans, tables, table_reservations
bills, payments, payment_splits, discounts, refunds
ingredients, recipes, stock_levels, stock_movements, suppliers, purchase_orders
customers, loyalty_accounts, loyalty_transactions, gift_cards
staff_shifts, attendance_logs
sync_queue, sync_log, device_registry
```

### Cloud (MongoDB Atlas — Collections)
Document model — embed related data to reduce joins, use refs for large/shared entities:

```js
// restaurants collection
{ _id, name, branches: [...], settings: { tax, currency, timezone }, plan }

// menuItems collection
{
  _id, branchId, categoryId, name, description, price,
  variants: [{ name, priceAdjustment }],
  modifierGroups: [{ name, required, options: [{ name, price }] }],
  ingredients: [{ ingredientId, qty, unit }],
  tags: ['vegan', 'gluten-free', 'spicy'],
  isAvailable: true, imageUrl, costPrice
}

// orders collection
{
  _id, branchId, orderType, tableId, staffId, customerId,
  status: 'pending|preparing|ready|served|billed|cancelled',
  items: [{ menuItemId, name, qty, price, modifiers, notes }],
  statusHistory: [{ status, changedBy, changedAt }],
  createdAt, updatedAt, syncId, deviceId
}

// bills collection
{
  _id, orderId, branchId, items: [...],
  subtotal, taxAmount, serviceCharge, discount, total,
  payments: [{ method, amount, reference }],
  status: 'paid|partial|refunded', createdAt, staffId
}

// inventory collection
{
  _id, branchId, ingredientId, name, unit,
  currentStock, reorderLevel, costPerUnit,
  movements: [{ type, qty, reason, date, staffId }]
}

// customers collection
{
  _id, name, phone, email, birthday,
  loyaltyPoints, tier, totalSpend, visitCount,
  preferences: { allergies, diet }, createdAt
}

// staff collection
{
  _id, branchId, name, phone, role, pin,
  permissions: [...], shifts: [...], attendance: [...]
}

// syncQueue collection (cloud-side log)
{
  _id, deviceId, branchId, entity, entityId,
  operation: 'create|update|delete',
  payload, syncedAt, status: 'pending|done|conflict'
}
```

### Why MongoDB for Cloud?
- **JSON-native** — Electron app works with JS objects end to end, no SQL mapping
- **Flexible schema** — menu items, modifiers, combos vary wildly per restaurant
- **Embedded documents** — order items embed modifiers (no 4-table join per order)
- **Atlas free tier** — start at $0, scale to M10/M20 as needed
- **Change Streams** — real-time sync push to connected devices via WebSocket
- **Atlas Search** — full-text menu search without extra infrastructure

## 3.4 Sync Strategy

- Every local record has: `syncId` (UUID), `deviceId`, `syncedAt`, `deletedAt` (soft delete), `version`
- **Local → Cloud**: All mutations added to `sync_queue` table → flushed in batches when online
- **Cloud → Local**: Poll or WebSocket push for changes newer than `syncedAt`
- **Conflict resolution**: Server timestamp wins; conflicted records logged in `syncQueue` for review
- **Idempotent API**: All sync endpoints keyed on `syncId` — safe to replay, no duplicates
- **MongoDB Change Streams**: Cloud pushes real-time updates to all connected branch devices

---

# 📦 4. FEATURE MODULES — FULL SPECIFICATION

---

## MODULE 1: POS SCREEN (CORE)

### Must-Have (Phase 1)
- [ ] Item grid with category tabs (horizontal scroll)
- [ ] Fast item search (fuzzy search, instant results)
- [ ] Add to cart — quantity control, swipe to remove
- [ ] Item notes field per line item ("no onion", "extra spicy")
- [ ] Item modifiers & variants (size, add-ons) with price delta
- [ ] Combo deals — auto-group items, show savings
- [ ] Hold order — park order, start new one, resume later
- [ ] Order type selector — Dine-in / Takeaway / Delivery

### Value-Added (Phase 1+)
- [ ] **Quick Keys panel** — pin top 8 most-sold items
- [ ] **Barcode scan to add item** — scan packaged items instantly
- [ ] **Recent orders sidebar** — reorder last 5 orders with one tap
- [ ] **Waiter name on order** — track who took which order
- [ ] **Color-coded order timer** — green→yellow→red based on wait time
- [ ] **Kitchen notes** — separate field for kitchen-only instructions

---

## MODULE 2: TABLE MANAGEMENT

### Must-Have (Phase 1)
- [ ] Visual floor plan — drag & drop table layout editor
- [ ] Table statuses: Available (green), Occupied (red), Reserved (yellow), Cleaning (blue)
- [ ] Assign order to table — one-click
- [ ] Transfer order to another table
- [ ] Merge tables — combine bills

### Value-Added
- [ ] **Multiple floor levels** — Ground floor, 1st floor, Outdoor
- [ ] **Table timer** — show how long table has been occupied
- [ ] **Cover count** — number of guests at each table
- [ ] **Server assignment** — assign waiter to table zone
- [ ] **Reservation calendar** — book tables with date/time/name/phone
- [ ] **Table turnover heatmap** — see which tables earn most per day

---

## MODULE 3: KITCHEN DISPLAY SYSTEM (KDS)

### Must-Have (Phase 1)
- [ ] Live order ticket display — auto-refreshes
- [ ] Order status: Pending → Preparing → Ready → Served
- [ ] Bump to next status (touch or bump bar)
- [ ] Sound alert for new tickets
- [ ] Priority flag (mark as VIP/urgent)

### Value-Added
- [ ] **Color urgency system** — green (<5min), yellow (5–10min), red (>10min)
- [ ] **Station routing** — hot food to kitchen screen, cold/drinks to bar screen
- [ ] **Item recall** — mark accidentally bumped items back to pending
- [ ] **Order consolidation** — same item across orders shown as batch
- [ ] **KOT printing** — print kitchen order ticket to dedicated printer
- [ ] **Prep time tracker** — average time per dish for performance review
- [ ] **Rush hour mode** — simplified view with larger text during peak

---

## MODULE 4: BILLING & PAYMENT

### Must-Have (Phase 1)
- [ ] Auto-generate bill from order
- [ ] Itemized bill with quantity, unit price, total
- [ ] Tax calculation — VAT 8%, Service Charge (configurable %)
- [ ] Cash payment with change calculator
- [ ] Card payment (manual entry amount)
- [ ] QR/Online payment (mark as paid after confirmation)
- [ ] Discount — flat amount or percentage, with reason
- [ ] Print thermal receipt — 58mm or 80mm
- [ ] Refund & cancellation with reason log

### Value-Added
- [ ] **Split bill by item** — each person pays for their items
- [ ] **Split bill equally** — divide total by N people
- [ ] **Partial payment** — pay part now, rest later (tab)
- [ ] **Tip collection** — add tip amount, track per staff
- [ ] **Digital receipt** — send via WhatsApp or SMS
- [ ] **Custom receipt footer** — "Thank you", Wi-Fi password, social handles
- [ ] **Bill on hold** — print preliminary bill while customer decides
- [ ] **Complimentary items** — mark as comp with manager approval
- [ ] **Day close reconciliation** — cash drawer count vs system total

---

## MODULE 5: INVENTORY MANAGEMENT

### Must-Have (Phase 2)
- [ ] Menu item stock on/off toggle — mark item as unavailable
- [ ] Manual stock level update
- [ ] Low stock notification (configurable threshold)
- [ ] Basic stock history log

### Advanced (Phase 2+)
- [ ] **Ingredient-level tracking** — recipe-based auto-deduction
- [ ] **Recipe builder** — define ingredients per dish with quantities
- [ ] **Cost per dish** — auto-calculate from ingredient costs
- [ ] **Gross margin per item** — selling price vs cost price %
- [ ] **Waste log** — record spoiled/expired/dropped ingredients
- [ ] **Supplier directory** — contacts, lead times, pricing
- [ ] **Purchase orders** — create PO, receive stock, auto-update levels
- [ ] **Stock taking** — periodic count with variance report
- [ ] **Expiry tracking** — FIFO logic, expiry date alerts
- [ ] **Auto-reorder** — trigger PO when stock hits reorder level

---

## MODULE 6: CUSTOMER MANAGEMENT

### Must-Have (Phase 2)
- [ ] Customer profile — name, phone, email, birthday
- [ ] Order history per customer
- [ ] Search customer at billing

### Value-Added
- [ ] **Loyalty points** — earn points per LKR spent, redeem at checkout
- [ ] **Customer tiers** — Bronze/Silver/Gold based on spend
- [ ] **Birthday alert** — notify staff when loyal customer visits on birthday
- [ ] **Dietary preferences** — store customer's allergies/preferences
- [ ] **Tab/credit account** — regular customers pay at end of month
- [ ] **Blacklist flag** — mark problem customers with note
- [ ] **Customer feedback** — post-meal rating via QR link
- [ ] **Visit frequency tracking** — "This customer hasn't visited in 30 days"
- [ ] **SMS/WhatsApp marketing** — send offers to opted-in customers
- [ ] **Gift cards** — sell and redeem digital gift cards

---

## MODULE 7: STAFF MANAGEMENT

### Must-Have (Phase 1)
- [ ] Staff profiles — name, role, PIN login
- [ ] Roles: Admin, Manager, Cashier, Waiter, Kitchen
- [ ] Permission control per role (which screens/actions allowed)
- [ ] Shift assignment

### Value-Added
- [ ] **Activity log** — every action logged with staff ID + timestamp
- [ ] **Performance dashboard** — orders served, revenue generated, upsells
- [ ] **Gamification leaderboard** — top waiter of the week
- [ ] **Tip distribution** — allocate pooled tips by hours worked
- [ ] **Manager override system** — require manager PIN for voids/discounts
- [ ] **Staff sales targets** — set daily/weekly revenue goals per staff

---

## MODULE 8: ATTENDANCE SYSTEM

### Must-Have (Phase 3)
- [ ] Clock-in / Clock-out via PIN or fingerprint
- [ ] Daily attendance log
- [ ] Overtime tracking

### Value-Added
- [ ] **Fingerprint device integration** — ZKTeco/similar USB device
- [ ] **Geofence check-in** (cloud phase) — prevent remote check-ins
- [ ] **Late arrival alerts** — notify manager when staff is late
- [ ] **Payroll summary** — hours × rate = gross pay export
- [ ] **Leave management** — request and approve leave within app
- [ ] **Shift swap** — staff can swap shifts with manager approval

---

## MODULE 9: REPORTING & ANALYTICS

### Must-Have (Phase 2)
- [ ] Daily sales summary — total revenue, orders, avg order value
- [ ] Item-wise sales report — top sellers ranked
- [ ] Payment method breakdown — cash/card/QR split
- [ ] Hourly sales trend chart
- [ ] Staff performance report

### Advanced Reports (Phase 2–4)
- [ ] **Profit & Loss report** — revenue vs cost of goods vs expenses
- [ ] **Monthly trend comparison** — this month vs last month
- [ ] **Category performance** — which menu categories drive most revenue
- [ ] **Table efficiency report** — revenue per table per hour
- [ ] **Customer lifetime value** — top spenders ranked
- [ ] **Inventory consumption report** — what's used vs wasted
- [ ] **Void/discount/refund report** — fraud detection view
- [ ] **Tax report** — VAT collected, service charge, ready for filing
- [ ] **Peak hours heatmap** — busiest days/hours visualization
- [ ] **Forecast report** (AI Phase) — predicted demand for next week

### Export Options
- [ ] PDF (formatted, branded)
- [ ] Excel/CSV (for accountant)
- [ ] Automated email delivery (daily/weekly/monthly)
- [ ] WhatsApp summary to owner

---

## MODULE 10: HARDWARE SUPPORT

### Phase 3 — Hardware Integration

| Device | Protocol | Library |
|---|---|---|
| Thermal Printer 58/80mm | ESC/POS over USB/Network | node-escpos |
| Cash Drawer | Triggered via printer port | node-escpos |
| Barcode Scanner | USB HID (keyboard mode) | Native input |
| Fingerprint Machine | ZKTeco SDK / USB | node-zklib |
| Customer Display | VFD / Second monitor | Electron second window |
| Label Printer | ESC/POS or ZPL | node-escpos / zpl-lib |
| Bump Bar | USB HID mapping | Native input |

### Fail-Safe Rules
- Printer offline → orders still saved, reprint when reconnected
- Cash drawer fail → manual override with log entry
- Fingerprint fail → fallback to PIN login

---

## MODULE 11: QR MENU & SELF-ORDER (Phase 3)

- [ ] Each table gets unique QR code
- [ ] Customer scans → PWA opens (no app install)
- [ ] Browse menu with photos, descriptions, allergens
- [ ] Place order → goes directly to KDS
- [ ] View order status (Preparing / Ready)
- [ ] Request the bill from table

### Value for Restaurant
- Reduce waiter calls by 40%
- Increase upsell (customers browse more when self-ordering)
- Works on any smartphone, no download required

---

## MODULE 12: CLOUD & OWNER DASHBOARD (Phase 4)

- [ ] Real-time sales dashboard (live numbers)
- [ ] Remote menu management — update prices/items/availability
- [ ] Multi-device sync — 2+ POS terminals, KDS, waiter tablets
- [ ] Remote staff management — view attendance from anywhere
- [ ] Push alerts — low stock, high void rate, sales milestone
- [ ] Owner mobile app (React Native)
- [ ] Automated daily summary report (email + WhatsApp at midnight)
- [ ] Cloud backup with point-in-time restore (keep 90 days)
- [ ] API access for integrations (3rd party delivery platforms)

---

## MODULE 13: MULTI-BRANCH MANAGEMENT (Phase 6)

- [ ] Central menu management — push menu changes to all branches
- [ ] Branch performance comparison dashboard
- [ ] Consolidated P&L across all branches
- [ ] Cross-branch inventory transfer
- [ ] Central customer database — customer recognized at any branch
- [ ] Branch manager vs owner permission levels
- [ ] Franchise mode — white-label branding per branch

---

# 🆕 5. MODERN VALUE-ADDED FEATURES

These differentiate this product from every generic POS on the market:

## 5.1 AI & Smart Features (Phase 5–6)

| Feature | How It Works | Value |
|---|---|---|
| **Demand Forecasting** | ML on historical sales data | Predict busy days, reduce waste |
| **Smart Reorder** | Auto-trigger PO when stock predicted to run out | Never run out of key ingredients |
| **Menu Optimization** | Identify low-margin, low-popularity items to cut | Increase profitability |
| **Upsell Suggestions** | "Customers who ordered X also ordered Y" | Increase average order value |
| **Anomaly Detection** | Flag unusual void/discount patterns | Detect staff theft |
| **Price Sensitivity** | Test price changes and measure impact | Optimize pricing |

## 5.2 Customer Experience Features

| Feature | Description |
|---|---|
| **Digital Menu with Photos** | Beautiful menu display on customer-facing screen or QR |
| **Allergen & Diet Filters** | Mark items: Vegan, Gluten-Free, Nut-Free, Halal, Spicy level |
| **Calorie Information** | Display nutritional info per item |
| **Post-Meal QR Feedback** | Scan receipt QR → rate experience → goes to owner dashboard |
| **WhatsApp Bill** | Send digital receipt directly to customer WhatsApp |
| **Loyalty App** | Customer-facing app to check points, view offers |

## 5.3 Operational Efficiency Features

| Feature | Description |
|---|---|
| **Rush Hour Mode** | Simplified POS UI during peak — big buttons, one-tap items |
| **Prep Time Standards** | Set expected prep time per dish, alert if exceeded |
| **Opening Checklist** | Digital checklist staff must complete before POS unlocks |
| **Closing Checklist** | End-of-day tasks with photo evidence upload |
| **Maintenance Log** | Track equipment issues and repairs |
| **Multi-Language UI** | Toggle between Sinhala / Tamil / English |

## 5.4 Financial & Compliance Features

| Feature | Description |
|---|---|
| **VAT Report** | Auto-calculate VAT collected, ready for IRD submission |
| **Expense Tracking** | Log non-inventory expenses (rent, utilities) |
| **Petty Cash Log** | Track small cash-out from drawer |
| **Auditor View** | Read-only role for accountant with full transaction history |
| **CCTV Timestamp Link** | Record POS event timestamps alongside camera footage |

---

# 🗂️ 6. DATABASE DESIGN PRINCIPLES

## 6.1 Key Design Decisions

### Local SQLite Rules
- **UUID primary keys** — safe for multi-device, no collision on merge
- **Soft deletes only** — `deleted_at` timestamp, never hard delete operational data
- **Audit columns on all tables** — `created_at`, `updated_at`, `created_by`, `updated_by`
- **Immutable financial records** — bills and payments never edited, only reversed with new record
- **Branch isolation** — all records scoped to `branch_id` from day one
- **Event sourcing for orders** — `order_status_history` table records every state change

### Cloud MongoDB Rules
- **Use `_id` as UUID string** — match local SQLite `sync_id` for seamless mapping
- **Embed for reads, reference for writes** — embed order items inside order document; reference menu items by ID
- **Soft deletes via `deletedAt` field** — never hard delete, query `{ deletedAt: null }`
- **Mongoose schemas with strict mode** — no unvalidated fields enter the database
- **Index strategy**: `branchId + createdAt` compound index on orders, bills; `phone` unique index on customers
- **Change Streams on orders collection** — real-time KDS and dashboard updates

## 6.2 Offline Sync Fields

Every syncable SQLite table has:
```sql
sync_id        TEXT DEFAULT (lower(hex(randomblob(16))))  -- UUID
device_id      TEXT
synced_at      INTEGER  -- Unix timestamp ms
sync_version   INTEGER DEFAULT 0
deleted_at     INTEGER  -- soft delete
```

Every MongoDB document receives these on sync ingest:
```js
{ syncId, deviceId, syncedAt, syncVersion, branchId, deletedAt }
```

---

# 🔐 7. SECURITY DESIGN

| Area | Implementation |
|---|---|
| **Authentication** | PIN (offline) + JWT (cloud), auto-lock after inactivity |
| **Authorization** | Role-based, permission checked server-side AND client-side |
| **Data encryption** | SQLite database encrypted at rest (SQLCipher) |
| **Audit trail** | Every mutation logged: who, what, when, from which device |
| **Void/discount approval** | Requires manager PIN above configurable threshold |
| **Export restriction** | Only Admin/Manager can export reports |
| **API security** | Rate limiting, HTTPS only, JWT refresh rotation |

---

# 💻 8. UI/UX DESIGN PRINCIPLES

## 8.1 POS Screen Rules (Critical)
- **0-click to most common action** — cashier should never need to scroll for top items
- **Large touch targets** — minimum 48×48px buttons (works on touchscreen)
- **No confirmation dialogs for add-to-cart** — kill flow for speed
- **Confirmation for destructive actions only** — void, cancel, refund
- **Color is status** — green=good, red=action needed, yellow=warning
- **Works on 15.6" touchscreen** — primary hardware target

## 8.2 Screen Layout

```
┌─────────────────────────────────────────────────────────┐
│  [Logo] [Branch]        [Staff: John]  [Time] [Settings]│
├────────────────────────────┬────────────────────────────┤
│  CATEGORY TABS             │  ORDER PANEL               │
│  [All][Rice][Curry][Drinks]│  Table 5 | Dine-in          │
├────────────────────────────┤  ─────────────────────     │
│  ITEM GRID (4 cols)        │  2x Fried Rice    LKR 600  │
│  [Item][Item][Item][Item]  │  1x Fish Curry    LKR 450  │
│  [Item][Item][Item][Item]  │  ─────────────────────     │
│  [Item][Item][Item][Item]  │  Subtotal:      LKR 1,050  │
│                            │  Tax (8%):        LKR  84  │
│  [🔍 Search...]            │  Total:         LKR 1,134  │
│                            │  ─────────────────────     │
│  QUICK KEYS                │  [HOLD] [KOT] [BILL/PAY]  │
│  [Item][Item][Item][Item]  │                            │
└────────────────────────────┴────────────────────────────┘
```

---

# 🧪 9. DEVELOPMENT PHASE PLAN

---

## Phase 1 — MVP Offline POS (Weeks 1–4)

**Goal: First paying customer by end of week 4**

### Week 1 — Foundation
- [ ] Electron + React + TypeScript project setup
- [ ] SQLite + Drizzle ORM setup with local schema
- [ ] Basic routing (POS, Tables, Bills, Settings)
- [ ] Authentication — PIN-based login, role system
- [ ] Menu management CRUD (categories + items)

### Week 2 — POS Core
- [ ] POS screen — item grid, cart, search
- [ ] Item modifiers & variants
- [ ] Order creation (dine-in / takeaway / delivery)
- [ ] Hold order functionality
- [ ] Table management — floor plan, status

### Week 3 — Billing & Print
- [ ] Bill generation from order
- [ ] Tax + discount + service charge calculation
- [ ] Cash/card/QR payment recording
- [ ] Thermal receipt printing (58mm + 80mm)
- [ ] Cash drawer trigger
- [ ] Refund & void with reason log

### Week 4 — Polish & Deploy
- [ ] Day close / reconciliation flow
- [ ] Basic daily sales summary screen
- [ ] KDS screen (standalone window)
- [ ] Settings — restaurant name, tax rates, receipt footer
- [ ] Electron installer build (Windows)
- [ ] Test on actual hardware

**Phase 1 Deliverable:** Working offline POS ready to sell

---

## Phase 2 — Business Features (Weeks 5–8)

### Add:
- [ ] Inventory module — items, stock levels, adjustments
- [ ] Recipe builder — link ingredients to menu items
- [ ] Cost per dish calculation
- [ ] Customer profiles + order history
- [ ] Loyalty points engine
- [ ] Full reporting suite — daily, monthly, item-wise
- [ ] PDF + Excel export
- [ ] Staff performance reports
- [ ] Supplier management
- [ ] Purchase order management

---

## Phase 3 — Hardware & QR (Weeks 9–12)

### Add:
- [ ] Fingerprint clock-in/out (ZKTeco integration)
- [ ] Barcode scanner for inventory receiving
- [ ] Customer display screen (second Electron window)
- [ ] Label printer support
- [ ] QR table menu (PWA, works offline on local network)
- [ ] Attendance reports + payroll summary export

---

## Phase 4 — Cloud System (Weeks 13–18)

### Add:
- [ ] Cloud API setup (Express + MongoDB Atlas + Mongoose)
- [ ] User accounts + JWT authentication
- [ ] Online owner dashboard (React web app)
- [ ] Real-time dashboard (WebSocket or Server-Sent Events)
- [ ] Remote menu management
- [ ] Cloud backup system
- [ ] Automated daily report (email/WhatsApp)
- [ ] Push notifications for alerts

---

## Phase 5 — Sync & Hybrid (Weeks 19–22)

### Add:
- [ ] Sync engine — queue-based offline → cloud push
- [ ] Conflict resolution logic
- [ ] Multi-device support (2nd POS terminal, waiter tablet)
- [ ] Sync status indicator in UI
- [ ] Data migration tool (upgrade offline to cloud)

---

## Phase 6 — Ecosystem (Months 6–12)

### Add:
- [ ] React Native owner mobile app
- [ ] Multi-branch management
- [ ] Franchise dashboard
- [ ] Online ordering integration (PickMe Food, etc.)
- [ ] AI demand forecasting module
- [ ] WhatsApp marketing integration
- [ ] Gift card system
- [ ] White-label / custom branding

---

# 💰 10. MONETIZATION STRATEGY

## Pricing Tiers

### Tier 1 — Starter (Offline Only)
- **Price:** LKR 25,000 one-time
- **Includes:** POS, tables, billing, print, basic reports
- **Support:** 3 months included
- **Target:** Small cafes, food stalls

### Tier 2 — Business (+ Inventory & Cloud Backup)
- **Price:** LKR 6,000/month
- **Includes:** All Tier 1 + inventory, customers, full reports, cloud backup
- **Target:** Medium restaurants

### Tier 3 — Pro (Full Cloud + Multi-Device)
- **Price:** LKR 12,000/month
- **Includes:** All Tier 2 + cloud dashboard, owner app, 3 devices, QR menu
- **Target:** Busy single-branch restaurants

### Tier 4 — Enterprise (Multi-Branch)
- **Price:** LKR 25,000+/month (per negotiation)
- **Includes:** All Tier 3 + multi-branch, franchise tools, API access, SLA
- **Target:** Chains and franchises

## Add-On Revenue
- Hardware bundle (printer + drawer + display): LKR 35,000–60,000
- Setup & training fee: LKR 10,000–20,000
- Custom report development: LKR 5,000–15,000 per report
- Priority support plan: LKR 2,000/month

## Revenue Projections (Conservative)

| Month | Customers | MRR (LKR) |
|---|---|---|
| Month 3 | 3 offline sales | 75,000 (one-time) |
| Month 6 | 5 SaaS + 5 offline | 60,000 |
| Month 12 | 20 SaaS + 15 offline | 240,000 |
| Month 24 | 60 SaaS | 720,000 |

---

# ⚠️ 11. RISKS & MITIGATION

| Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|
| Feature creep delays launch | High | High | Strict phase gates — no Phase 2 before Phase 1 is sold |
| Hardware incompatibility | Medium | Medium | Test with 3 specific models, document supported hardware |
| Data loss on crash | Medium | Critical | Auto-save every action, WAL mode SQLite, nightly backup |
| Staff resistance to new system | High | High | Simple UI, 1-hour training, cheat sheet printout |
| Competitor undercuts price | Low | Medium | Win on local support + Sinhala language + hardware bundle |
| Power cut during transaction | High | High | UPS recommended, last transaction recovery on restart |
| Internet dependency for cloud | High | High | Everything works 100% offline first — cloud is bonus |

---

# 📋 12. QUALITY & TESTING STRATEGY

## Testing Levels

| Level | What | Tool |
|---|---|---|
| Unit tests | Business logic (tax calc, split bill, loyalty points) | Vitest |
| Integration tests | Database operations, sync logic | Vitest + test SQLite |
| E2E tests | Critical user flows (create order → pay → print) | Playwright |
| Manual QA | Hardware testing, edge cases | Checklist per phase |
| Beta testing | 2 real restaurants before public launch | Free extended trial |

## Performance Targets
- POS screen load: < 1 second
- Item search response: < 100ms
- Bill generation: < 500ms
- Report generation (daily): < 3 seconds
- Sync (100 orders): < 30 seconds on 4G

---

# 🚀 13. GO-TO-MARKET STRATEGY

## Phase 1 Launch (Month 1–2)
1. Install free at 2 pilot restaurants (friends/family)
2. Collect real feedback, fix critical issues
3. Create 2-minute demo video
4. WhatsApp broadcast to restaurant owners in network
5. Offer first 5 paying customers 50% discount

## Phase 2 Launch (Month 3–4)
1. Facebook/Instagram ads targeting restaurant owners in Sri Lanka
2. Partner with hardware suppliers — they refer software + earn commission
3. List on local software marketplaces
4. Create Sinhala YouTube tutorials

## Phase 3+ Growth
1. Restaurant associations partnerships
2. Accountant referral program (they recommend to clients)
3. Case studies from pilot restaurants
4. Hotel & bakery vertical expansion

---

# ✅ 14. DEFINITION OF DONE — PHASE 1

A feature is **done** when:
- [ ] It works completely offline
- [ ] It handles the error case (printer offline, invalid input)
- [ ] It has been tested on a touchscreen
- [ ] An untrained user can figure it out in 30 seconds
- [ ] The relevant data is correctly stored in SQLite
- [ ] A non-technical manager can view and understand the result

---

# 🧭 15. FINAL EXECUTION PLAN

```
Week 1–4:   Build Phase 1 MVP (offline POS + billing + print)
Week 5:     Install at 2 pilot restaurants FREE
Week 6:     Fix issues from real-world feedback
Week 7:     First paid customer (Tier 1 — LKR 25,000)
Week 8–12:  Build Phase 2 (inventory + reports + customers)
Week 12:    Launch Tier 2 SaaS pricing
Month 4–5:  Hardware integration + QR menu
Month 5–6:  Cloud system + owner dashboard
Month 6:    Launch Tier 3 Pro plan
Month 8–12: Multi-branch + mobile app + AI features
```

---

# 📌 PRODUCT PRINCIPLES (NEVER VIOLATE)

1. **Offline first, always** — power cut cannot stop a restaurant
2. **Speed over features** — a slow POS kills staff and customers
3. **Never lose a transaction** — auto-save before any network call
4. **Every phase must be independently sellable** — don't wait for perfection
5. **Simple beats clever** — if a 50-year-old kitchen staff can't use it, redesign it
6. **Own the support relationship** — fast local support is the biggest moat
7. **Design for cloud from day one** — don't paint yourself into a corner with schema

---

*Built for Sri Lankan restaurants. Designed to scale globally.*

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
| **Online Ordering PWA** | React + Vite (deployed web) | Customer-facing ordering portal, no app install |
| **Payment Gateway** | PayHere / Stripe (configurable) | Card, QR, and COD support for online orders |
| **Realtime Push** | WebSocket (Socket.io) | Instant online order delivery to POS + customer tracking |
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
online_orders, online_order_items, delivery_zones, delivery_drivers, delivery_assignments
sync_queue, sync_log, device_registry
```

### Cloud (MongoDB Atlas — Collections)
Document model — embed related data to reduce joins, use refs for large/shared entities:

```js
// restaurants collection (one per owner/organization)
{
  _id, ownerId, name, plan,
  settings: { currency, timezone, defaultTaxRate },
  createdAt
}

// branches collection (one per physical location)
{
  _id, restaurantId, name, address, phone,
  timezone, taxRate, operatingHours: { mon: { open, close }, ... },
  status: 'active|closed|inactive',
  receiptHeader, receiptFooter,
  isOnlineOrderingEnabled, onlineOrderingUrl,
  createdAt
}

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

> **Designed for:** A single owner running 2–10+ restaurant locations. One login. Full visibility. Each branch operates independently offline — owner oversees everything from the cloud.

---

### 13.1 Owner Account & Branch Access

- [ ] **Single owner account** — one login, access to all branches under one organization
- [ ] **Branch switcher** — owner switches active branch view from dashboard or POS without re-login
- [ ] **"All Branches" consolidated view** — aggregated dashboard showing all locations simultaneously
- [ ] **Per-branch POS login** — staff at each branch log in with their own PIN, only see their branch data
- [ ] **Branch access control** — staff are locked to their assigned branch(es); owner is unrestricted
- [ ] **Branch manager role** — manages their own branch only; cannot see other branches
- [ ] **Add new branch** — owner provisions a new branch from cloud dashboard (name, address, timezone, tax rate)

---

### 13.2 Menu Management Across Branches

- [ ] **Shared master menu** — owner maintains a central menu; all branches inherit it
- [ ] **Push menu update to all branches** — price change or new item deployed everywhere in one action
- [ ] **Push to selected branches** — update only specific locations (e.g. new item only at city branch)
- [ ] **Branch-specific price override** — individual branch can charge a different price for any item
- [ ] **Branch-exclusive items** — items that only appear at one branch (seasonal, location-specific)
- [ ] **Item availability per branch** — mark items unavailable at a branch without deleting them
- [ ] **Category visibility per branch** — hide entire categories at a specific branch

---

### 13.3 Consolidated Reporting & Analytics

- [ ] **Consolidated daily sales** — total revenue across all branches with per-branch breakdown
- [ ] **Branch comparison dashboard** — side-by-side revenue, order count, avg order value
- [ ] **Top-performing branch ranking** — daily / weekly / monthly
- [ ] **Consolidated P&L** — combined income vs cost of goods across all branches
- [ ] **Branch-wise item performance** — which items sell better at which location
- [ ] **Unified tax report** — VAT collected per branch and combined total (for filing)
- [ ] **Custom date range + branch filter** — slice any report by branch and period
- [ ] **Export consolidated report** — PDF/Excel covering all branches

---

### 13.4 Inventory & Supplier Management

- [ ] **Per-branch stock levels** — each branch tracks inventory independently (offline-safe)
- [ ] **Cross-branch stock transfer** — request and approve stock movement between branches
- [ ] **Transfer log** — full audit trail of all inter-branch stock movements
- [ ] **Centralized supplier directory** — shared supplier list, no duplication across branches
- [ ] **Branch-specific purchase orders** — each branch orders independently from shared supplier list
- [ ] **Consolidated purchase overview** — owner sees all open POs across all branches

---

### 13.5 Customer & Loyalty (Cross-Branch)

- [ ] **Unified customer database** — customer recognized at any branch by phone number
- [ ] **Cross-branch loyalty points** — points earned at branch A redeemable at branch B
- [ ] **Customer visit history across branches** — owner sees which branches a customer visits
- [ ] **Centralized gift cards** — sold at one branch, redeemable at all branches
- [ ] **Cross-branch marketing** — send promotions to all customers regardless of visited branch

---

### 13.6 Branch Settings & Configuration

- [ ] **Branch-specific tax rates** — different regions may have different VAT rules
- [ ] **Branch-specific operating hours** — each branch sets its own open/close schedule
- [ ] **Branch-specific receipt header/footer** — each branch prints its own address and contact
- [ ] **Branch-specific delivery zones** — online ordering configured independently per branch
- [ ] **Branch status control** — owner marks branch as Active / Temporarily Closed / Inactive

---

### 13.7 Multi-Branch Online Ordering

- [ ] **Per-branch ordering page** — each branch has its own online ordering URL
- [ ] **Branch selector on main ordering page** — customer picks nearest branch, sees that branch's menu
- [ ] **Branch-specific delivery zones and fees** — each branch serves its own radius
- [ ] **Centralized online order view** — owner sees all online orders across all branches in one place
- [ ] **Branch-specific promo codes** — run location-specific promotions

---

### 13.8 Franchise Mode (Optional)

- [ ] **White-label per branch** — each franchisee gets their own branded receipts and ordering page
- [ ] **Royalty tracking** — % of branch revenue tracked for franchise fee calculation
- [ ] **Franchisee owner role** — franchisee sees only their branch; master franchisor sees all
- [ ] **Template menu** — franchisor pushes standard menu; franchisee adds local items with approval

---

## MODULE 14: CUSTOMER ONLINE ORDERING & DELIVERY (Phase 4–5)

> Customers place delivery orders from their phone or browser — orders flow directly into the POS KDS with no manual entry.

### Must-Have (Phase 4 — Cloud Required)

- [ ] **Public ordering web app (PWA)** — branded page for the restaurant, accessible via URL or QR code
- [ ] Browse menu with photos, descriptions, allergen tags, and availability status
- [ ] Add to cart — quantity, item notes, modifier/variant selection
- [ ] Customer registration / guest checkout (name, phone, delivery address)
- [ ] Select delivery or pickup at checkout
- [ ] Online payment gateway — card / QR / cash-on-delivery option
- [ ] Order confirmation screen + SMS/WhatsApp notification to customer
- [ ] Order injected directly into POS as a new order (type: `online-delivery`)
- [ ] KDS shows online orders with a distinct badge — staff can't miss them
- [ ] Order status updates pushed to customer in real-time: `Received → Preparing → Out for Delivery → Delivered`
- [ ] Customer order tracking page — shareable link

### Value-Added (Phase 5)

- [ ] **Delivery zone management** — set serviceable areas by radius or pin zones on map
- [ ] **Delivery fee calculator** — flat fee, or by zone/distance
- [ ] **Estimated delivery time** — configurable per zone, shown at checkout
- [ ] **Driver assignment** — assign an in-house delivery driver to an order
- [ ] **Driver app (PWA)** — driver sees assigned orders, marks as picked up / delivered
- [ ] **Live driver tracking** — customer sees driver on map (Google Maps embed)
- [ ] **Minimum order amount** — configurable per branch
- [ ] **Scheduled orders** — customer selects a future delivery time slot
- [ ] **Promo codes & online-only discounts** — attract first-time online customers
- [ ] **Repeat order** — customer reorders from their order history with one tap
- [ ] **Customer account portal** — view order history, saved addresses, loyalty points
- [ ] **3rd-party platform integration** — accept orders from PickMe Food / Uber Eats via API and inject into POS

### Online Order Flow (System)

```
Customer (PWA)                     POS System (Cloud)
───────────────────────            ─────────────────────────────────
1. Browse menu
2. Add to cart
3. Enter address + payment          4. Payment gateway processes
                                    5. Order saved to cloud DB
                                    6. WebSocket push → Electron POS
                                    7. New online order appears in KDS
                                    8. Staff accepts / starts preparing
                                    9. Status update → WebSocket → Customer PWA
10. Customer sees "Preparing..."
                                    11. Driver assigned → status = Out for Delivery
12. Customer sees driver on map
                                    13. Driver marks Delivered
14. Customer sees "Delivered" ✅
                                    15. Auto-closes order → triggers loyalty points
```

### Online Order Data (SQLite — local cache for offline resilience)

```sql
online_orders           -- id, branch_id, customer_id, order_type, status, payment_status,
                        -- subtotal, delivery_fee, discount, total, delivery_address,
                        -- estimated_delivery_at, driver_id, sync_id, created_at
online_order_items      -- id, online_order_id, menu_item_id, qty, unit_price, modifiers, notes
delivery_zones          -- id, branch_id, name, fee, min_order, est_minutes, polygon_coords
delivery_drivers        -- id, branch_id, staff_id, name, phone, status (available/busy/offline)
delivery_assignments    -- id, online_order_id, driver_id, assigned_at, picked_up_at, delivered_at
```

### Online Order Data (MongoDB — Cloud)

```js
// onlineOrders collection
{
  _id, branchId, customerId, orderType: 'online-delivery' | 'online-pickup',
  status: 'pending|confirmed|preparing|out-for-delivery|delivered|cancelled',
  paymentStatus: 'pending|paid|refunded',
  items: [{ menuItemId, name, qty, price, modifiers, notes }],
  deliveryAddress: { line1, city, lat, lng },
  deliveryFee, promoCode, discount, subtotal, total,
  driverId, estimatedDeliveryAt,
  statusHistory: [{ status, changedBy, changedAt }],
  customerNote, createdAt, updatedAt, syncId
}

// deliveryZones collection
{
  _id, branchId, name, feeAmount, minOrderAmount,
  estimatedMinutes, polygon: [{ lat, lng }], isActive
}

// deliveryDrivers collection
{
  _id, branchId, staffId, name, phone,
  status: 'available|busy|offline',
  currentOrderId, lastLocationAt
}
```

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

# 🔑 6. LICENSE KEY SYSTEM

> The software is locked until activated. The owner calls the vendor, gets a key, enters it once. Works 100% offline.

## 6.1 How It Works (Flow)

```
CUSTOMER SIDE                         VENDOR SIDE (You)
─────────────────                     ──────────────────────────
1. Install POS on laptop
2. Open app → "Not Activated" screen
3. App shows Device ID:
   e.g.  A3F2-BC91-44DE-77FA          4. Customer calls/WhatsApps you
                                       5. You run:
                                          node generate-key.js A3F2-BC91-44DE-77FA business 2027-12-31
                                       6. Tool outputs Activation Key:
                                          B84A2-C19FE-7734A-A2891-DD320
                                       7. You send key to customer
8. Customer enters key in app
9. App verifies offline (no internet)
10. Software unlocks for that tier
```

## 6.2 Device ID Generation

- Built from: **MAC address + hostname + OS platform** → SHA-256 hash
- Format: `XXXX-XXXX-XXXX-XXXX` (16 hex chars, grouped)
- Same machine always produces the same Device ID
- Changing the network card changes the Device ID → customer calls you for new key

## 6.3 Activation Key Generation (Vendor Tool)

- **Algorithm**: `HMAC-SHA256(deviceId + "|" + licenseType + "|" + expiryDate, VENDOR_SECRET)`
- Take first 25 chars of result → format as `XXXXX-XXXXX-XXXXX-XXXXX-XXXXX`
- `VENDOR_SECRET` is kept only on your machine — never shipped to customers
- Verification in the app: re-computes the HMAC and compares → no server needed

## 6.4 Vendor Keygen Tool

A simple Node.js CLI script (kept private by developer):

```
Usage:
  node tools/keygen/generate-key.js <deviceId> <licenseType> <expiryDate>

Examples:
  node generate-key.js A3F2-BC91-44DE-77FA starter 2026-12-31
  node generate-key.js A3F2-BC91-44DE-77FA business 2027-06-30
  node generate-key.js A3F2-BC91-44DE-77FA pro 2099-12-31

License Types: starter | business | pro | enterprise
Expiry: use 2099-12-31 for lifetime licenses
```

## 6.5 What the License Stores (locally on customer machine)

```json
{
  "deviceId": "A3F2-BC91-44DE-77FA",
  "licenseType": "business",
  "expiryDate": "2027-12-31",
  "activationKey": "B84A2-C19FE-7734A-A2891-DD320",
  "activatedAt": "2026-04-08T10:30:00.000Z"
}
```

## 6.6 Feature Gating by License Tier

| Feature | Starter | Business | Pro | Enterprise |
|---|:---:|:---:|:---:|:---:|
| POS, Tables, Billing, KDS | ✅ | ✅ | ✅ | ✅ |
| Inventory, Customers, Reports | ❌ | ✅ | ✅ | ✅ |
| Loyalty, Supplier Management | ❌ | ✅ | ✅ | ✅ |
| Cloud Sync, Owner Dashboard, QR Menu | ❌ | ❌ | ✅ | ✅ |
| **Online Ordering PWA, Delivery Management** | ❌ | ❌ | ✅ | ✅ |
| **Driver App, Live Tracking, Promo Codes** | ❌ | ❌ | ✅ | ✅ |
| Multi-Branch, API Access, White-Label | ❌ | ❌ | ❌ | ✅ |
| **3rd-Party Delivery Platform Integration** | ❌ | ❌ | ❌ | ✅ |

## 6.7 Security Rules

- License file is re-verified on every app launch (prevents manual file editing)
- Key is re-derived and compared — if tampered with, app shows "Invalid License"
- Key is bound to Device ID — copying `license.json` to another machine fails
- If customer changes their laptop → call vendor → get new key for new Device ID

---

# 🔐 7. PASSWORD SYSTEM

## 7.1 Owner Password Recovery (Vendor-Assisted)

When an owner forgets their password they call you (the vendor). You generate a **one-time reset key** valid only for today.

### Flow

```
OWNER                                 VENDOR (You)
─────────────────────                 ──────────────────────
1. Login screen → "Forgot Password"
2. App shows Device ID
3. Owner calls/WhatsApps you          4. You run:
                                         node generate-password-reset.js A3F2-BC91-44DE-77FA
                                      5. Tool outputs Reset Key:
                                         A3F2-BC91-44DE-77FA-9C21
                                      6. You send key to owner
7. Owner enters Reset Key
8. App verifies (offline, date-bound)
9. "Set New Password" screen appears
10. Owner sets new password
11. Reset key is marked used (can't reuse)
```

### Reset Key Security Design

- **Bound to**: Device ID + today's calendar date (`YYYY-MM-DD`)
- **Expires**: At midnight of the day it was issued — useless next day
- **One-time use**: App marks the key used after successful reset
- **Algorithm**: `HMAC-SHA256("RESET|" + deviceId + "|" + date, VENDOR_SECRET)` → first 20 chars → `XXXX-XXXX-XXXX-XXXX-XXXX`
- No internet needed — everything verified locally

### Vendor Tool

```
Usage:
  node tools/keygen/generate-password-reset.js <deviceId>
  node tools/keygen/generate-password-reset.js <deviceId> --date 2026-04-08

Example output:
  KEY: A3F2-BC91-44DE-77FA-9C21
  Valid for today only. Expires at midnight.
```

## 7.2 Staff Password Reset (By Owner)

Owner can reset any staff member's password without calling the vendor.

### Flow

```
Owner → Staff Management → Select Staff → "Reset Password"
  ↓
App generates a 6-digit temporary PIN (e.g. 483920)
  ↓
Owner tells staff the temporary PIN verbally
  ↓
Staff logs in with temp PIN
  ↓
"You must set a new password to continue" screen
  ↓
Staff sets new password → saved as hash, temp PIN invalidated
```

### Key Rules

- Owner sees the temp PIN once — on screen — must verbally tell staff
- Temp PIN is stored as a hash in DB with `mustChangePassword = true` flag
- Staff cannot use the app until they set a new password after a reset
- Owner cannot see a staff member's current password (one-way hash)
- All password changes are logged in the audit trail (who reset whose password, when)

## 7.3 Password Storage (Security)

- Passwords are **never stored in plain text**
- Storage: `SHA-256(salt + password)` where `salt` is a random 16-byte hex string
- Both `hash` and `salt` stored in the `staff` table
- Staff table columns: `password_hash TEXT`, `password_salt TEXT`, `must_change_password INTEGER DEFAULT 0`

---

# 💾 8. BACKUP & DATA RECOVERY SYSTEM

> If the laptop breaks, gets stolen, or crashes — no data is lost.

## 8.1 Backup File Format

A `.posbackup` file is a **ZIP archive** containing:

```
pos-backup-2026-04-08.posbackup
  ├── pos.db          → Full SQLite database (all data)
  └── metadata.json   → Backup info for validation
```

### metadata.json contents

```json
{
  "version": "1.0.0",
  "backupDate": "2026-04-08T22:00:00.000Z",
  "restaurantName": "Green Leaf Restaurant",
  "branchName": "Main Branch",
  "dbSizeBytes": 2457600,
  "totalOrders": 14523,
  "checksum": "sha256-hash-of-pos.db"
}
```

## 8.2 Manual Export (Owner-Initiated)

- Settings → Backup → **Export Backup**
- File save dialog opens → owner chooses location (USB drive, Documents folder, etc.)
- Saves `pos-backup-RestaurantName-2026-04-08.posbackup` file
- **Recommended**: Save to a USB drive stored off-site

## 8.3 Manual Import / Restore

- Settings → Backup → **Restore from Backup**
- File picker opens → owner selects `.posbackup` file
- App validates:
  - ✅ ZIP contains `pos.db` + `metadata.json`
  - ✅ SHA-256 checksum of embedded database matches metadata
  - ✅ Metadata version is compatible
- Before overwriting: **auto-saves a pre-restore backup** to `AppData/backups/auto/pre-restore-*.db`
- Replaces current database
- App prompts restart

## 8.4 Auto-Backup (Scheduled)

- Runs automatically **on day-close** (when cashier closes the day)
- Also runs on a **daily timer** (e.g. 11:00 PM every night, configurable)
- Saved to: `AppData/Restaurant-POS/backups/auto/auto-2026-04-08T23-00-00.posbackup`
- Keeps the **last 30 auto-backups**, deletes older ones automatically

## 8.5 Backup to External Location (Phase 4 — Cloud)

- When cloud is enabled: auto-backup also uploads to **MongoDB Atlas** or **Google Drive**
- Owner can access backups from the cloud dashboard
- Restore can be triggered remotely if the machine is online

## 8.6 Disaster Recovery Steps (If Laptop Breaks)

```
1. Get a new laptop
2. Install the POS software
3. Plug in the USB drive with the backup file
4. Settings → Restore from Backup → select the .posbackup file
5. App restores all data (orders, menu, customers, inventory, staff)
6. Call vendor for a new Activation Key (new Device ID on new machine)
7. Enter new key → software unlocks → fully operational
```

## 8.7 Backup UI in Settings Screen

```
┌─────────────────────────────────────────────────┐
│  💾 BACKUP & RESTORE                            │
├─────────────────────────────────────────────────┤
│  Last auto-backup: Today at 11:00 PM  ✅        │
│                                                  │
│  [📤 Export Backup Now]                         │
│  [📥 Restore from Backup File]                  │
│                                                  │
│  Auto-Backup Schedule:  [Daily at] [11:00 PM ▼] │
│  Keep last: [30 ▼] backups                      │
│                                                  │
│  📁 Backup folder: C:\AppData\...\backups\auto  │
│     [Open Folder]                               │
│                                                  │
│  Recent Auto-Backups:                           │
│  • auto-2026-04-08.posbackup  (2.4 MB)         │
│  • auto-2026-04-07.posbackup  (2.3 MB)         │
│  • auto-2026-04-06.posbackup  (2.3 MB)         │
└─────────────────────────────────────────────────┘
```

---

# 🗂️ 9. DATABASE DESIGN PRINCIPLES

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

# 🔐 10. SECURITY DESIGN

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

# 💻 11. UI/UX DESIGN PRINCIPLES

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

# 🧪 12. DEVELOPMENT PHASE PLAN

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
- [ ] **Customer online ordering PWA** — branded web app per restaurant
- [ ] **Online order API** — receive, validate, push to POS via WebSocket
- [ ] **Payment gateway integration** — card / QR / COD at checkout
- [ ] **Online order KDS badge** — distinguish online orders from walk-in orders
- [ ] **Customer order tracking page** — real-time status via WebSocket
- [ ] **Delivery zone management** — admin sets zones, fees, ETAs
- [ ] **SMS/WhatsApp order confirmation** to customer on order received

---

## Phase 5 — Sync & Hybrid (Weeks 19–22)

### Add:
- [ ] Sync engine — queue-based offline → cloud push
- [ ] Conflict resolution logic
- [ ] Multi-device support (2nd POS terminal, waiter tablet)
- [ ] Sync status indicator in UI
- [ ] Data migration tool (upgrade offline to cloud)
- [ ] **Delivery driver PWA** — driver app: view assigned orders, update pickup/delivery status
- [ ] **Driver live location** — driver shares GPS, customer sees on map
- [ ] **Scheduled/future delivery orders** — customer books a delivery time slot
- [ ] **Promo codes & online-only discounts engine**
- [ ] **Customer order history portal** — account login, past orders, saved addresses
- [ ] **3rd-party delivery platform API** — PickMe Food / Uber Eats order ingestion

---

## Phase 6 — Ecosystem (Months 6–12)

### Add:
- [ ] React Native owner mobile app
- [ ] **Multi-branch management (Module 13 — full build)**
  - [ ] Single owner account → all branches under one login
  - [ ] Branch provisioning wizard (add new location from dashboard)
  - [ ] Central menu with per-branch price/availability overrides
  - [ ] Push menu changes to all or selected branches
  - [ ] Consolidated sales dashboard across all branches
  - [ ] Branch comparison report (revenue, orders, top items)
  - [ ] Cross-branch stock transfer with approval flow
  - [ ] Unified customer + loyalty database across branches
  - [ ] Branch manager role — scoped access per location
  - [ ] Per-branch settings (tax rate, hours, receipt, delivery zones)
  - [ ] Multi-branch online ordering with branch selector
- [ ] Franchise dashboard + royalty tracking
- [ ] Online ordering integration (PickMe Food, etc.)
- [ ] AI demand forecasting module
- [ ] WhatsApp marketing integration
- [ ] Gift card system (cross-branch redemption)
- [ ] White-label / custom branding per branch

---

# 💰 13. MONETIZATION STRATEGY

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

# ⚠️ 14. RISKS & MITIGATION

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

# 📋 15. QUALITY & TESTING STRATEGY

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

# 🚀 16. GO-TO-MARKET STRATEGY

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

# ✅ 17. DEFINITION OF DONE — PHASE 1

A feature is **done** when:
- [ ] It works completely offline
- [ ] It handles the error case (printer offline, invalid input)
- [ ] It has been tested on a touchscreen
- [ ] An untrained user can figure it out in 30 seconds
- [ ] The relevant data is correctly stored in SQLite
- [ ] A non-technical manager can view and understand the result

---

# 🧭 18. FINAL EXECUTION PLAN

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

# 🍽️ Restaurant POS System (Offline-First + Hybrid Cloud)

## 🧠 Project Vision
Build a **hybrid POS system** that works in three modes:

1. **Offline Desktop Only** (Standalone)
2. **Offline + Sync (Hybrid)**
3. **Full Online SaaS (MERN Stack)**

---

# 🏗️ Architecture Overview

## Desktop (Offline First)
- Electron.js (Desktop App)
- React (UI)
- SQLite (Local Database)
- Embedded Express Server

## Cloud Backend
- Node.js + Express
- MongoDB
- JWT Authentication

## Sync Engine
- Background sync queue
- Conflict resolution (last-write-wins or versioning)

---

# 🌐 System Design

```
[ POS Desktop App ]
   ↓ (SQLite)
[ Local Database ]
   ↓ (Sync Engine)
[ Cloud API ]
   ↓
[ MongoDB ]
```

---

# 🔌 Offline Communication (POS ↔ Kitchen Tablet)

## ✅ Recommended: Local LAN Communication

### Setup:
- All devices connected to same WiFi/router
- POS acts as **local server**
- Kitchen tablet connects via IP

### Example:
```
POS Server: http://192.168.1.5:5000
Kitchen Tablet: http://192.168.1.5:5000/kitchen
```

---

## 🔄 Communication Methods

### 1. REST API (Simple)
- Polling every few seconds

### 2. WebSockets (Recommended)
- Real-time updates using Socket.io

### Flow:
1. POS creates order
2. Emit event: `new-order`
3. Kitchen receives instantly
4. Kitchen updates status → sends back

---

# 📦 Core Modules

## 🧾 Order Management
- Dine-in
- Takeaway
- Delivery
- Split bills
- Merge tables
- Hold orders

## 🪑 Table Management
- Table layout UI
- Status tracking (Available, Occupied, Reserved)

## 🍳 Kitchen Display System (KDS)
- Live orders
- Status: Pending, Preparing, Ready

## 💳 Billing & Payments
- Bill printing
- Payment types (Cash, Card, QR)
- Discounts & taxes
- Refunds

## 🖨️ Hardware Integration
- Thermal printer (ESC/POS)
- Cash drawer
- Barcode scanner
- Fingerprint device

## 📦 Inventory Management
- Stock tracking
- Low stock alerts
- Supplier management

## 👥 Customer Management
- Profiles
- Loyalty system
- Order history

## 👨‍💼 Staff Management
- Roles & permissions
- Shift tracking

## ⏱️ Attendance System
- Fingerprint login
- Clock-in / clock-out

## 📊 Reporting
- Sales reports
- Item analytics
- Profit tracking
- Export (PDF/Excel)

---

# 🗂️ Database Design

## Local (SQLite)
- orders
- order_items
- tables
- products
- staff
- sync_queue

## Cloud (MongoDB)
- users
- restaurants
- branches
- orders
- inventory

---

# 🔄 Sync Strategy

## Local-first writes
- All operations saved locally first

## Sync Queue
- Store unsynced data

## Conflict Handling
- Last-write-wins OR version-based merge

---

# 🧪 Development Phases

## Phase 1 — MVP (Offline POS)
- Orders
- Table management
- Billing

## Phase 2 — Inventory + Reports

## Phase 3 — Hardware Integration

## Phase 4 — Cloud Backend (MERN)

## Phase 5 — Sync Engine

## Phase 6 — Advanced Features
- Mobile dashboard
- Loyalty system
- Multi-branch

---

# 💰 Monetization

## Offline Version
- One-time payment

## Cloud Version
- Monthly subscription

---

# ⚠️ Challenges
- Network instability
- Sync conflicts
- Hardware compatibility
- Data corruption handling

---

# 💡 Advanced Features
- Sinhala/Tamil language support
- QR menu ordering
- WhatsApp bill sharing
- AI-based insights (future)

---

# 🧭 Best Implementation Approach

1. Build **offline POS first**
2. Test with real restaurants
3. Improve based on feedback
4. Add cloud sync later

---

# 🚀 MVP Setup Example

```
POS: http://192.168.1.5:5000
Kitchen: http://192.168.1.5:5000/kitchen
```

---

# ✅ Summary

This system ensures:
- Works without internet
- Real-time kitchen sync via LAN
- Scalable to cloud SaaS

---

**End of Document**


# 🛒 ShopOS — "Fixology"

### An offline-first, multi-device Point-of-Sale & Repair-Shop Operating System

> **Note:** The source code for this application/software is closed-source and maintained in a private repository to protect client intellectual property. This repository serves as an architectural case study demonstrating the interface design, state management, and business logic implemented.

**Built with Flutter + Supabase + Drift (SQLite) — one codebase, three platforms.**

> *"Parts • Accessories • Repairs — one counter."*

ShopOS is a real-world retail operating system running daily at **Fixology**, a mobile parts & repair shop. It handles billing, inventory, repair job tracking, credit (udhaar) ledger, owner analytics and multi-device sync — **with or without internet**.

🎬 **[Watch the full working demo →](docs/demo.mp4)**
💬 **[Client review & field test →](docs/client-review.mp4)**

---

## ✨ Why this project exists

Generic billing software fails at a repair counter: no job-tracking, no "fits which models" search, no offline mode when the shop's WiFi dies, no honest profit math after discounts. ShopOS was built **from the counter up**, feature by feature, against real daily usage — every feature here solved a real problem that happened behind a real counter.

---

## 🚀 Feature Highlights

### ⚡ Spotlight Search (command-palette billing)
- Type → a floating results card drops over the category grid, ranked: *name starts-with → name contains → brand → fits-models / shelf / category*
- Matched letters glow blue in **name, brand and fits-models**; expandable `+N more` fits list
- Category filter pills with live match counts (`All 12 • Tempered 7 • Folder/Combo 1`)
- Keyboard-first on desktop: `Ctrl+F` focus, `↑↓` navigate, `Enter` add, `Esc` close
- **Frequently Billed** chips (computed from invoice history) for 2-second repeat billing

### 🧾 Billing that respects reality
- Bill types: **Sale • Repair • Credit • Own** (owner-purchase at cost → zero phantom profit)
- **Labour-only repair bills** (no parts, just your fee) — a real repair-shop case
- Discount field: digits-only, hard-capped at bill total, never negative
- Stock enforcement at *every* step: add-button, cart `+`, and a final guard at save
- Partial payments, udhaar (credit) tracking, dues collection screen
- Parked / saved carts for "customer will come back in 10 minutes"
- Invoice PDF generation + WhatsApp-style share; void = archive + auto stock-restore

### 🔧 Repair Jobs workflow
- Minimal job cards: customer name, phone, note, parts — status `IN WORK → DONE`
- Status **auto-completes when its bill is saved**; note appears on the bill screen the moment you type the customer's phone
- Open-jobs picker inside the Repair bill panel; date + status filters; full history

### 📊 Money dashboard with honest accounting
- True profit = **what you actually charged (after discounts) − cost of goods gone**
- KPIs: today's sales/bills, udhaar due, low stock, stock value at cost, month sales, parked bills, received, profit
- IN/OUT/NET ledger, top items, category-wise sales, CSV exports (invoices + ledger)

### 🔐 Roles & multi-device
- Admin + staff accounts with per-module permissions (billing / inventory / jobs / settings / ledger)
- PIN login, security-question admin recovery
- Phone + laptop + shop PC converge through Supabase — last-write-safe, offline-tolerant

---

## 🏗️ Architecture

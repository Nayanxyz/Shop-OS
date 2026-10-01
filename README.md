# 🛒 ShopOS — "Fixology"

### An offline-first, multi-device Point-of-Sale & Repair-Shop Operating System

> **Note:** The source code for this application/software is closed-source and maintained in a private repository to protect client intellectual property. This repository serves as an architectural case study demonstrating the interface design, state management, and business logic implemented.

**Built with Flutter + Supabase + Drift (SQLite) — one codebase, three platforms.**

> *"Parts • Accessories • Repairs — one counter."*

ShopOS is a real-world retail operating system running daily at **Fixology**, a mobile parts & repair shop. It handles billing, inventory, repair job tracking, credit (udhaar) ledger, owner analytics and multi-device sync — **with or without internet**.

🎬 **[Watch the full working demo →](fixology_demo.mp4)**
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

```text
┌─────────────────────────── Flutter App ───────────────────────────┐
│  Windows (.exe)      Android (.apk)      Web (PWA-ready)          │
│                                                                   │
│  UI Layer: Spotlight • Billing • Jobs • Money • Inventory • Admin │
│                        │                                          │
│  Drift (SQLite) ◄──────┤  local-first reads/writes                │
│  schema v16, live migrations on every installed device            │
│                        │                                          │
│  OUTBOX TABLE ◄────────┘  every mutation queued with op+payload   │
└───────────────┬───────────────────────────────────────────────────┘
                │  push (when online) / pull (pending-guarded)
                ▼
        ┌───────────────┐
        │   Supabase    │  Postgres + Row-Level Security
        │  (cloud sync) │  source of truth across devices
        └───────────────┘
```


### The sync strategy (the heart of ShopOS)
1. **Local-first:** every read/write hits SQLite — the shop never stops for WiFi.
2. **Outbox pattern:** mutations are queued locally with `op` + JSON payload, pushed when connectivity returns.
3. **Pending-guarded pulls:** a table with unpushed local changes is *never* overwritten by a cloud pull → no lost offline work.
4. **Connectivity gate + auto-heal:** a 4-second probe gates all network calls (silent offline, zero error spam); a 45-second watcher detects reconnect and syncs automatically with a single "Internet back — synced ✔" confirmation.
5. **Poison-entry handling:** duplicate-key (23505) and stale-FK (23503) outbox entries are detected and dropped safely.

---

## 🧰 Tech Stack

| Layer | Technology |
|---|---|
| Framework | Flutter (Windows / Android / Web from one codebase) |
| Local DB | Drift (SQLite) with versioned migrations (v1 → v16) |
| Cloud | Supabase (Postgres, RLS, anon-key auth) |
| State | Streams (`watch()`) — reactive UI straight from the DB |
| PDF | `printing` + `share_plus` |
| Fonts/UI | Google Fonts, custom design system (`product_ui.dart`) |
| CI of life | Real daily shop usage as the test suite |

---

## 🧠 Hard problems & how they were solved

| Problem | Solution |
|---|---|
| **Offline billing without losing data** | Outbox queue + pending-guarded pulls + connectivity gate + auto-heal watcher |
| **Schema upgrades on already-installed devices** | Drift `MigrationStrategy` with per-version steps (v1→v16), zero data loss across phone/laptop/PC |
| **"Profit" lying after discounts & freebies** | Redefined profit as *charged − cost*; free giveaways now show as honest losses |
| **Owner taking stock personally** | `Own` bill type at cost price — real invoice, zero profit impact, clean audit trail (instead of polluting sales with 100% discounts) |
| **Overselling (billing 11 of 10 in stock)** | Stock caps enforced at add, cart-increment and a final transaction guard |
| **Negative / over-total discounts** | `digitsOnly` input formatters + live clamp with user feedback |
| **Search that finds "c21" inside fits-models** | Multi-field haystack + 4-tier ranking + in-text match highlighting |
| **Double confirmation dialogs** | Traced nested `showDialog` gates; collapsed to a single source of truth |
| **Flutter Gradle resetting `minSdk` on wrapper upgrades** | Pre-build checklist + pinned wrapper; documented gotcha |
| **Two devices editing the same invoice** | Server-side archive table (`deleted_invoices`) as void source of truth; retry-queue on pull |

---

## 📸 Screenshots

| Home + Spotlight | Billing | Inventory | Money | Admin Menu |
|---|---|---|---|---|
| <img width="1905" height="966" alt="Screenshot 2026-09-29 104853" src="https://github.com/user-attachments/assets/36756b46-94a0-4386-b4be-68d229f0e452" /> | <img width="1914" height="951" alt="Screenshot 2026-09-29 105046" src="https://github.com/user-attachments/assets/970f620f-a95c-4efc-b6aa-6ba86d010b36" /> | <img width="1919" height="971" alt="Screenshot 2026-09-29 105250" src="https://github.com/user-attachments/assets/83f039ee-76aa-4f98-b120-a59f5e891441" /> | <img width="1913" height="972" alt="Screenshot 2026-09-29 105344" src="https://github.com/user-attachments/assets/511bceb7-89a2-4e15-a67b-6b76fc0a5bc9" /> | <img width="1897" height="930" alt="Screenshot 2026-09-29 110012" src="https://github.com/user-attachments/assets/aa8aba39-c216-4165-9b6e-f031b671bedc" /> |


---

## 🎬 Demos & Field Feedback

- **Full product walkthrough:** [docs/demo.mp4](docs/demo.mp4)
- **Client (shop owner) review after live deployment:** [docs/client-review.mp4](docs/client-review.mp4)

> *"…client quote about the software going here…"*
> — **Fixology Shop Owner**, daily user since [month/year]

---

## 🗂️ Repository Layout (private production repo)

```text
lib/
├── core/          # db.dart (Drift schema+migrations), sync.dart (outbox engine),
│                  # auth, bill_pdf, invoice_detail, product_ui (design system)
├── features/
│   ├── home/      # Spotlight search, category grid, frequent chips
│   ├── cart/      # billing engine (sale/repair/credit/own), saved carts
│   ├── jobs/      # repair job workflow
│   ├── ops/       # money dashboard, inventory, stock sheet, dues, ledger
│   ├── admin/     # roles, categories, employees, security
│   └── history/   # invoice archive, void flow, customers
└── main.dart      # bootstrap: env → Supabase → Drift → app
```

## 📄 License & Contact
Built for and deployed at Fixology — Shop and Beyond.

Showcase repository; production code lives in a private repo.

Contact: choudharyji527@gmail.com 
         
         linkedin.com/in/nayanxyz
         
         nayanxyz-portfolio.vercel.app

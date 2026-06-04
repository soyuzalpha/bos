# Phase 8: Feature Implementation Roadmap

## Phase 1: Foundation
**Complexity:** ⬛⬛⬛⬛⬛⬜⬜ (5/7)
**Duration:** 4–6 weeks

| Feature | Deliverables |
|---------|-------------|
| Project scaffolding | NestJS project, Prisma setup, Redis connection, BullMQ setup, Docker Compose |
| Config module | Environment validation, typed config, per-environment profiles |
| Prisma schema & migrations | Full schema from Phase 4, seed scripts for permissions and roles |
| Auth system | JWT auth, refresh tokens, password hashing, MFA (TOTP), password policies, login rate limiting |
| Multi-tenant middleware | `X-Tenant-Id` extraction, tenant validation, tenant-scoped Prisma middleware |
| RBAC system | Permissions seeding, role CRUD, permission assignment, guards |
| User management | CRUD, activation/deactivation, role assignment |
| Audit log infrastructure | Prisma middleware for auto-audit, query API |
| API response format | `ApiSuccess`/`ApiError` interceptors, global exception filters |
| Request logging | Structured logging (Pino), request ID tracing, correlation IDs |

**Success Criteria:**
- A user can register a tenant
- A user can login with email/password and receive JWT
- A user can be assigned roles and permissions
- All database queries are scoped to the correct tenant
- All mutations are captured in audit logs
- API responses follow a consistent format

---

## Phase 2: Master Data
**Complexity:** ⬛⬛⬛⬛⬜⬜⬜ (4/7)
**Duration:** 3–4 weeks

| Feature | Deliverables |
|---------|-------------|
| Categories | CRUD, tree hierarchy (parent/child), sort ordering |
| Brands | CRUD |
| Units | CRUD, unit conversions |
| Products | CRUD, product types (simple/variant/composite/service), barcode/SKU management |
| Product variants | CRUD, attribute-based variants, variant-specific pricing |
| Product pricing | Price lists, multi-tier pricing, min-qty pricing, bulk price updates |
| Product suppliers | Supplier-product mapping, preferred supplier, supplier-specific costs |
| Customer groups | CRUD, discount percentages |
| Customers | CRUD, group assignment, credit limits, purchase history tracking |
| Suppliers | CRUD, payment terms, lead time tracking |

**Success Criteria:**
- Products can be created with full catalog hierarchy
- Product search works by SKU, barcode, name
- Product pricing supports multiple price lists and tiers
- Customers and suppliers can be managed with full profile
- Data imports work via CSV/Excel

---

## Phase 3: Inventory
**Complexity:** ⬛⬛⬛⬛⬛⬛⬜ (6/7)
**Duration:** 4–5 weeks

| Feature | Deliverables |
|---------|-------------|
| Warehouses | CRUD, type classification, default warehouse per branch |
| Inventory balances | On-hand qty, reserved qty, available qty (generated), cost price tracking |
| Inventory movements | Append-only ledger, movement types, chronological ordering |
| Stock adjustments | Physical count correction, reason tracking, cost revaluation |
| Stock transfers | Inter-warehouse transfers, transfer_out/transfer_in movements |
| Batch/Lot tracking | Batch number, expiry date, FIFO allocation |
| Serial number tracking | Serial number registry, sale assignment, warranty tracking |
| Low stock alerts | Threshold-based alerts, notification generation |
| Inventory valuation | FIFO and weighted average cost calculation |
| Negative stock prevention | Validation rules, admin override with audit |

**Success Criteria:**
- Every product has accurate on-hand/available quantities per warehouse
- Stock movements are immutable and auditable
- Cost price is calculated correctly using FIFO or weighted average
- Low stock alerts trigger automatically
- Physical count adjustments reconcile discrepancies with audit trail

---

## Phase 4: Sales POS
**Complexity:** ⬛⬛⬛⬛⬛⬛⬛ (7/7)
**Duration:** 6–8 weeks

| Feature | Deliverables |
|---------|-------------|
| Shift management | Open/close shift, opening/closing balances, discrepancy reporting |
| Cart management | Add/remove/update items, hold/recall carts, cart expiry |
| POS transaction flow | Complete sale processing with atomic inventory deduction |
| Pricing engine | Price list resolution, tiered pricing, customer group discounts |
| Discount engine | Per-item discount, per-transaction discount, auto-discount by customer group |
| Tax calculation | Tax-inclusive/exclusive, per-product tax rate, multiple tax regimes |
| Payment processing | Cash, card, QRIS, e-wallet, split payments, multi-tender transactions |
| Receipt generation | Thermal receipt (ESC/POS), PDF receipt, email receipt |
| Void transaction | Void with inventory reversal, reason tracking, authorization check |
| Refund transaction | Full/partial refund, original sale reference, payment reversal |
| Customer display | Real-time sale summary, item listing, total display |
| Barcode scanning | Keyboard wedge mode, scanner integration |

**Success Criteria:**
- A complete sale can be processed end-to-end in under 2 seconds
- Inventory is deducted atomically on sale completion
- Multiple payment methods can be combined in one transaction
- Voided sales correctly reverse inventory
- Receipts are generated in thermal and PDF formats
- Shift reconciliation balances correctly

---

## Phase 5: Purchase
**Complexity:** ⬛⬛⬛⬛⬛⬜⬜ (5/7)
**Duration:** 4–5 weeks

| Feature | Deliverables |
|---------|-------------|
| Purchase order creation | Multi-line PO, supplier selection, product search |
| PO workflow | Draft → Submit → Approve → Order → Receive → Complete |
| PO approval | Configurable approval rules, role-based approvers |
| Goods receiving | Partial/full receipt, batch allocation, cost update |
| Inventory integration | Auto-create inventory movements on receipt |
| PO reconciliation | Match PO → Receive → Invoice |
| Purchase history | Supplier performance, price history, lead time tracking |

**Success Criteria:**
- A PO can be created with multiple line items and submitted for approval
- Approvers can approve or reject POs
- Goods receipt creates inventory movements and updates cost prices
- Partial receipts are supported
- PO status tracks correctly through the full lifecycle

---

## Phase 6: Reports
**Complexity:** ⬛⬛⬛⬛⬛⬜⬜ (5/7)
**Duration:** 4–5 weeks

| Feature | Deliverables |
|---------|-------------|
| Sales reports | Daily summary, by product, by category, by cashier, by customer, by payment method |
| Inventory reports | Stock valuation, stock movement, low stock, expired stock |
| Purchase reports | PO summary, supplier performance, purchase by product |
| Financial reports | Profit & Loss, gross margin, cost of goods sold |
| Tax reports | VAT summary, taxable sales, input/output tax |
| Export engine | PDF, CSV, Excel export with formatting |
| Report scheduling | Scheduled delivery via email, auto-generation |
| Dashboard | KPI cards, revenue chart, top products, low stock alerts |

**Success Criteria:**
- All reports generate in under 5 seconds for a month of data
- Reports can be filtered by date range, branch, and other dimensions
- Exports in PDF, CSV, and Excel formats are accurate
- Reports can be scheduled for automated delivery
- Dashboard loads with real-time KPI data

---

## Phase 7: Subscription
**Complexity:** ⬛⬛⬛⬛⬜⬜⬜ (4/7)
**Duration:** 3–4 weeks

| Feature | Deliverables |
|---------|-------------|
| Plan management | CRUD, pricing (monthly/yearly), feature flags, limits |
| Subscription lifecycle | Create, activate, trial, upgrade, downgrade, cancel, expire, suspend |
| Billing integration | Stripe/Paddle webhooks, invoice generation, payment collection |
| Entitlement engine | Feature flag checks, user/product/branch limit enforcement |
| Usage metering | Transaction count, storage usage, API call count |
| Self-service portal | Plan selection, payment method management, subscription management |

**Success Criteria:**
- New tenants can register and start a free trial
- Tenants are limited by plan-based constraints (max users, branches, products)
- Feature flags toggle UI elements and API access
- Billing webhooks process subscription changes correctly
- Expired subscriptions gracefully restrict access without data loss

---

## Phase 8: Multi-Branch
**Complexity:** ⬛⬛⬛⬜⬜⬜⬜ (3/7)
**Duration:** 2–3 weeks

| Feature | Deliverables |
|---------|-------------|
| Branch management | CRUD, HQ flag, location/tax data |
| Cross-branch inventory | Branch-specific warehouse management, inter-branch transfers |
| Branch-specific pricing | Per-branch price lists, per-branch discounts |
| Branch-specific reports | Sales by branch, comparative reports |
| User-branch assignment | Role scoped to branch, branch-aware queries |

**Success Criteria:**
- A tenant can have multiple active branches
- Each branch has its own warehouses, pricing, and users
- Reports can be viewed per-branch or consolidated
- Inventory can be transferred between branches

---

## Phase 9: Enterprise Features
**Complexity:** ⬛⬛⬛⬛⬛⬛⬛ (7/7)
**Duration:** 8–12 weeks

| Feature | Deliverables |
|---------|-------------|
| Approval workflows | Configurable multi-stage approval, per-approver permissions |
| Notification system | In-app, email, WhatsApp, SMS, per-user preference, unsubscribe |
| Loyalty program | Points accumulation, tiered rewards, referral system, birthday promos |
| CRM | Customer 360, communication history, segmentation, campaign management |
| Advanced pricing | Promotional pricing, time-based discounts, BOGO, bundle pricing |
| E-commerce API | Product sync, inventory availability, order creation, webhook integration |
| Business Intelligence | Custom dashboards, OLAP-style drill-down, drag-and-drop report builder |
| Forecasting | Demand forecasting, inventory replenishment recommendations |
| Full financial accounting | Double-entry ledger, AR/AP, balance sheet, income statement, cash flow |

**Success Criteria:**
- Multi-step approval workflows can be configured without code changes
- Customer loyalty points accumulate and redeem correctly
- E-commerce API enables real-time inventory sync with marketplace platforms
- Business Intelligence provides self-service analytics
- Forecasting models achieve >80% accuracy on 90-day horizon

---

## Scale Readiness Summary

| Metric | Capacity | Mechanism |
|--------|----------|-----------|
| **Tenants** | 10,000+ | Row-level tenant isolation, shared schema, connection pooling (PgBouncer) |
| **Users** | 100,000+ | Indexed lookups, cursor pagination, shard-ready |
| **Products** | 10M+ | Covering indexes, full-text search with GIN, partial indexes |
| **Transactions** | 50M+/year | BigSerial IDs for chrono ordering, table partitioning, append-only ledger |
| **Concurrent POS** | 5,000+ | Optimistic locking on inventory balance, Redis rate limiting, BullMQ queues |
| **Data retention** | Unlimited | Partition pruning, archival to cold storage, audit log cleanup jobs |
| **High Availability** | Multi-AZ DB, read replicas | Prisma read replica support, Redis Sentinel, stateless API servers |

---

## Estimated Total Timeline

| Phase | Duration | Team Size |
|-------|----------|-----------|
| P1: Foundation | 4–6 weeks | 3–4 engineers |
| P2: Master Data | 3–4 weeks | 2–3 engineers |
| P3: Inventory | 4–5 weeks | 2–3 engineers |
| P4: Sales POS | 6–8 weeks | 3–4 engineers |
| P5: Purchase | 4–5 weeks | 2 engineers |
| P6: Reports | 4–5 weeks | 2 engineers |
| P7: Subscription | 3–4 weeks | 2 engineers |
| P8: Multi-Branch | 2–3 weeks | 1–2 engineers |
| P9: Enterprise | 8–12 weeks | 3–4 engineers |
| **Total** | **38–52 weeks** | **2–4 engineers** |

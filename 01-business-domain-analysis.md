# Phase 1: Business Domain Analysis

## 1.1 Core Domains

| Domain | Why It Exists | Strategic Value |
|--------|--------------|-----------------|
| **Auth** | Gatekeeper for the entire system. Handles authentication strategies (JWT, OAuth2, MFA), session management, password policies, and identity federation. Without this, nothing else is secure. | Non-negotiable foundation. Every request flows through auth. |
| **Tenant** | Multi-tenancy is the business model. Manages tenant lifecycle (provisioning, suspension, deletion), tenant-level configuration (branding, locale, timezone, feature flags), and tenant isolation. This domain IS the SaaS product. | Revenue driver. Each tenant = a paying customer. |
| **Users** | Represents human actors within a tenant. Handles profile management, credentials, preferences, and employee lifecycle (onboarding, offboarding). | Core identity layer for all human interaction. |
| **Roles & Permissions** | Fine-grained access control. Roles group permissions. Permissions map to specific API actions/UI elements. Enables RBAC at tenant level, branch level, and feature level. | Security & compliance. Enterprise buyers require this. |
| **Products** | The catalog of sellable items. Includes SKU management, pricing (multiple price tiers), barcodes, tax configuration, product types (simple, variant, composite/bundle), and digital assets. | The product catalog IS the system's core data asset. Everything else references it. |
| **Categories / Brands / Units** | Product classification and measurement. Categories enable hierarchy (Department → Category → Subcategory). Brands group products by manufacturer. Units define measurement (pcs, kg, liter) with conversion rules. | Enables reporting, inventory valuation, and purchasing. |
| **Inventory** | Tracks stock across warehouses and branches. Manages stock quantities, valuation methods (FIFO, Weighted Average), serial numbers, batch/lot tracking, and expiry dates. | Directly impacts P&L. Inventory errors = revenue leaks. |
| **Warehouses** | Physical or virtual storage locations. Can be a backroom, main warehouse, transit hub, or even a delivery truck. Each branch has at least one default warehouse. | Physical truth of where stock lives. |
| **Sales** | The transaction engine. Handles the complete sale lifecycle: cart management, line items, discounts, taxes, tendering, receipt generation, and hold/recall. Real-time inventory deduction. | Revenue generation. This is the heart of the POS. |
| **Payments** | Payment processing abstraction. Supports multiple payment methods (cash, card, QRIS, bank transfer, credit account, installment). Integrates with payment gateways. Handles split payments, partial payments, and refunds. | Cash flow. Every sale ends in a payment. |
| **Customers** | Buyer profiles. Tracks contact info, purchase history, credit limit, loyalty points, and customer groups for pricing/tiering. Enables CRM in later phases. | Revenue growth. Known customers = repeat business. |
| **Suppliers** | Vendor management. Tracks supplier profiles, purchase history, lead times, payment terms, and product catalogs per supplier. | Supply chain foundation. |
| **Purchase Orders** | Procurement workflow. Manages PO creation, approval, receipt, and reconciliation. Tracks ordered vs received vs invoiced quantities. | Ensures stock availability. Links inventory to accounts payable. |
| **Reports** | Aggregated business intelligence. Covers sales reports, inventory reports, financial reports, tax reports, and operational KPIs. Supports export (PDF, CSV, Excel) and scheduled delivery. | Decision-making. Raw data without reports = no value. |
| **Audit Logs** | Immutable record of every state-changing operation. Who did what, when, from which IP, with old/new values. Compliance requirement for enterprises. | Legal & compliance. Non-negotiable for enterprise sales. |
| **Subscriptions** | Billing and plan management. Handles plan definitions, feature entitlements, billing cycles, usage metering, invoicing, and payment collection. Drives tenant upgrades/downgrades. | Monetization engine. This domain IS the recurring revenue. |

## 1.2 Supporting Domains

| Domain | Why It Exists |
|--------|--------------|
| **Branches** | Multi-location support within a tenant. Each branch has its own address, tax ID, currency, pricing strategy, and operational hours. Enables chain store management. |
| **Notifications** | Outbound communication. Email, SMS, WhatsApp, and in-app notifications for invoices, receipts, low-stock alerts, PO approvals, payment confirmations, and system announcements. |
| **Tax** | Tax configuration per product, per customer group, per region. Supports multiple tax regimes (PPN, PPh, VAT, Sales Tax) and tax-inclusive/exclusive pricing. |
| **Discounts** | Promotion engine. Supports percentage off, flat amount off, buy-X-get-Y, bundle discounts, volume discounts, and coupon codes with date/time/quantity limits. |
| **Cash Management** | Tracks cash in drawers, cash-in/cash-out events, beginning/ending balances, and discrepancy resolution. Essential for shift reconciliation. |
| **Shift Management** | Cashier shift lifecycle. Opening balance, closing balance, sales summary per shift, and cashier performance. |

## 1.3 Future Enterprise Domains

| Domain | Strategic Value |
|--------|----------------|
| **CRM** | Customer lifecycle management. Segmentation, campaign management, communication history, lead tracking, and customer 360 view. |
| **Marketing & Loyalty Program** | Points accumulation, tiered rewards, referral programs, birthday promotions, and targeted campaigns. Directly drives repeat purchase rate. |
| **Finance** | Full double-entry accounting. General ledger, accounts receivable, accounts payable, journal entries, trial balance, and financial statements (balance sheet, income statement, cash flow). |
| **Cash Flow** | Real-time cash position. Forecasted inflows/outflows, DSO/DPO tracking, and working capital management. |
| **Expenses** | Employee expense reporting, approval workflow, receipt OCR, and expense categorization. Links to accounts payable. |
| **HR Management** | Employee records, attendance, payroll integration, commission calculation, and performance tracking. |
| **Approval Workflow** | Configurable multi-stage approval engine. Purchase orders above X amount, discounts above Y%, credit limit increases, and expense reports. |
| **Forecasting** | Demand forecasting using historical sales data, seasonal patterns, and ML models. Drives inventory replenishment recommendations. |
| **Business Intelligence** | Custom dashboards, drag-and-drop report builder, data warehouse integration, OLAP cubes, and drill-down analytics. Enterprise upsell. |
| **E-Commerce Integration** | Omnichannel selling. API-driven product sync, inventory availability, web order fulfillment, and marketplace integration (Shopee, Tokopedia, Lazada). |

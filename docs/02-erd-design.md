# Phase 2: Complete ERD Design

## 2.1 Entity Relationship Diagram — Table Definitions

### TENANTS

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | UUID | PK, DEFAULT uuid_generate_v4() | Primary identifier |
| name | VARCHAR(255) | NOT NULL | Company/business name |
| slug | VARCHAR(100) | NOT NULL, UNIQUE | URL-friendly identifier |
| domain | VARCHAR(255) | UNIQUE | Custom domain (self-hosted) |
| logo_url | TEXT | NULLABLE | Branding |
| address | TEXT | NULLABLE | Business address |
| phone | VARCHAR(50) | NULLABLE | Contact phone |
| email | VARCHAR(255) | NULLABLE | Contact email |
| tax_id | VARCHAR(100) | NULLABLE | NPWP / Tax registration |
| currency_code | VARCHAR(3) | NOT NULL, DEFAULT 'IDR' | ISO 4217 |
| timezone | VARCHAR(50) | NOT NULL, DEFAULT 'Asia/Jakarta' | IANA timezone |
| locale | VARCHAR(10) | NOT NULL, DEFAULT 'id-ID' | Language/region |
| date_format | VARCHAR(20) | NOT NULL, DEFAULT 'DD/MM/YYYY' | Display format |
| is_active | BOOLEAN | NOT NULL, DEFAULT true | Tenant enabled/disabled |
| subscription_id | UUID | FK → subscriptions.id, NULLABLE | Current subscription |
| settings | JSONB | DEFAULT '{}' | Feature flags, config |
| created_at | TIMESTAMPTZ | NOT NULL, DEFAULT NOW() | |
| updated_at | TIMESTAMPTZ | NOT NULL, DEFAULT NOW() | |
| deleted_at | TIMESTAMPTZ | NULLABLE | Soft delete |

---

### USERS

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | UUID | PK | |
| tenant_id | UUID | FK → tenants.id, NOT NULL | Tenant scope |
| branch_id | UUID | FK → branches.id, NULLABLE | Default branch |
| email | VARCHAR(255) | NOT NULL | Login identifier |
| phone | VARCHAR(50) | NULLABLE | Secondary identifier |
| password_hash | VARCHAR(255) | NOT NULL | bcrypt hash |
| first_name | VARCHAR(100) | NOT NULL | |
| last_name | VARCHAR(100) | NULLABLE | |
| is_active | BOOLEAN | NOT NULL, DEFAULT true | |
| must_change_password | BOOLEAN | NOT NULL, DEFAULT false | Force password change |
| last_login_at | TIMESTAMPTZ | NULLABLE | |
| failed_login_attempts | INTEGER | NOT NULL, DEFAULT 0 | |
| locked_until | TIMESTAMPTZ | NULLABLE | Account lockout |
| refresh_token_hash | VARCHAR(255) | NULLABLE | |
| mfa_secret | VARCHAR(255) | NULLABLE | TOTP secret |
| mfa_enabled | BOOLEAN | NOT NULL, DEFAULT false | |
| settings | JSONB | DEFAULT '{}' | Preferences, theme, etc. |
| created_at | TIMESTAMPTZ | NOT NULL | |
| updated_at | TIMESTAMPTZ | NOT NULL | |
| deleted_at | TIMESTAMPTZ | NULLABLE | |

**Unique Constraint:** (tenant_id, email) — email unique within tenant
**Unique Constraint:** (tenant_id, phone) — phone unique within tenant

---

### ROLES

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | UUID | PK | |
| tenant_id | UUID | FK → tenants.id, NOT NULL | |
| name | VARCHAR(100) | NOT NULL | e.g., "Admin", "Cashier", "Manager" |
| slug | VARCHAR(100) | NOT NULL | e.g., "admin", "cashier", "manager" |
| description | TEXT | NULLABLE | |
| is_system | BOOLEAN | NOT NULL, DEFAULT false | System roles cannot be deleted |
| is_default | BOOLEAN | NOT NULL, DEFAULT false | Auto-assigned to new users |
| created_at | TIMESTAMPTZ | NOT NULL | |
| updated_at | TIMESTAMPTZ | NOT NULL | |
| deleted_at | TIMESTAMPTZ | NULLABLE | |

**Unique Constraint:** (tenant_id, slug)

---

### PERMISSIONS

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | UUID | PK | |
| tenant_id | UUID | FK → tenants.id, NOT NULL | |
| name | VARCHAR(100) | NOT NULL | Display name |
| slug | VARCHAR(100) | NOT NULL | e.g., "product.create", "sale.read" |
| module | VARCHAR(50) | NOT NULL | Grouping: "products", "sales", "users" |
| action | VARCHAR(50) | NOT NULL | "create", "read", "update", "delete", "approve" |
| description | TEXT | NULLABLE | |
| is_system | BOOLEAN | NOT NULL, DEFAULT true | Auto-seeded, not user-creatable |
| created_at | TIMESTAMPTZ | NOT NULL | |
| updated_at | TIMESTAMPTZ | NOT NULL | |

**Unique Constraint:** (tenant_id, slug)

---

### ROLE_PERMISSIONS

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | UUID | PK | |
| role_id | UUID | FK → roles.id, NOT NULL, ON DELETE CASCADE | |
| permission_id | UUID | FK → permissions.id, NOT NULL, ON DELETE CASCADE | |
| created_at | TIMESTAMPTZ | NOT NULL | |

**Unique Constraint:** (role_id, permission_id)

---

### USER_ROLES

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | UUID | PK | |
| user_id | UUID | FK → users.id, NOT NULL, ON DELETE CASCADE | |
| role_id | UUID | FK → roles.id, NOT NULL, ON DELETE CASCADE | |
| branch_id | UUID | FK → branches.id, NULLABLE | Scope role to branch |
| created_at | TIMESTAMPTZ | NOT NULL | |

**Unique Constraint:** (user_id, role_id, branch_id)

---

### BRANCHES

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | UUID | PK | |
| tenant_id | UUID | FK → tenants.id, NOT NULL | |
| code | VARCHAR(50) | NOT NULL | Branch code: "JKT-01", "BDG-02" |
| name | VARCHAR(255) | NOT NULL | |
| address | TEXT | NULLABLE | |
| city | VARCHAR(100) | NULLABLE | |
| province | VARCHAR(100) | NULLABLE | |
| postal_code | VARCHAR(20) | NULLABLE | |
| phone | VARCHAR(50) | NULLABLE | |
| email | VARCHAR(255) | NULLABLE | |
| tax_id | VARCHAR(100) | NULLABLE | Branch-level tax ID |
| is_active | BOOLEAN | NOT NULL, DEFAULT true | |
| is_head_office | BOOLEAN | NOT NULL, DEFAULT false | HQ flag |
| opening_time | TIME | NULLABLE | |
| closing_time | TIME | NULLABLE | |
| settings | JSONB | DEFAULT '{}' | Branch-specific config |
| created_at | TIMESTAMPTZ | NOT NULL | |
| updated_at | TIMESTAMPTZ | NOT NULL | |
| deleted_at | TIMESTAMPTZ | NULLABLE | |

**Unique Constraint:** (tenant_id, code)

---

### WAREHOUSES

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | UUID | PK | |
| tenant_id | UUID | FK → tenants.id, NOT NULL | |
| branch_id | UUID | FK → branches.id, NOT NULL | |
| code | VARCHAR(50) | NOT NULL | "WH-01", "WH-BACKROOM" |
| name | VARCHAR(255) | NOT NULL | |
| type | VARCHAR(50) | NOT NULL | Enum: "main", "backroom", "display", "damaged", "transit" |
| address | TEXT | NULLABLE | |
| is_active | BOOLEAN | NOT NULL, DEFAULT true | |
| is_default | BOOLEAN | NOT NULL, DEFAULT false | Default warehouse for branch |
| created_at | TIMESTAMPTZ | NOT NULL | |
| updated_at | TIMESTAMPTZ | NOT NULL | |
| deleted_at | TIMESTAMPTZ | NULLABLE | |

**Unique Constraint:** (tenant_id, code)

---

### CATEGORIES

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | UUID | PK | |
| tenant_id | UUID | FK → tenants.id, NOT NULL | |
| parent_id | UUID | FK → categories.id, NULLABLE | Self-referencing for hierarchy |
| name | VARCHAR(255) | NOT NULL | |
| slug | VARCHAR(255) | NOT NULL | |
| description | TEXT | NULLABLE | |
| sort_order | INTEGER | NOT NULL, DEFAULT 0 | Display ordering |
| is_active | BOOLEAN | NOT NULL, DEFAULT true | |
| created_at | TIMESTAMPTZ | NOT NULL | |
| updated_at | TIMESTAMPTZ | NOT NULL | |
| deleted_at | TIMESTAMPTZ | NULLABLE | |

**Unique Constraint:** (tenant_id, slug)

---

### BRANDS

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | UUID | PK | |
| tenant_id | UUID | FK → tenants.id, NOT NULL | |
| name | VARCHAR(255) | NOT NULL | |
| slug | VARCHAR(255) | NOT NULL | |
| description | TEXT | NULLABLE | |
| logo_url | TEXT | NULLABLE | |
| is_active | BOOLEAN | NOT NULL, DEFAULT true | |
| created_at | TIMESTAMPTZ | NOT NULL | |
| updated_at | TIMESTAMPTZ | NOT NULL | |
| deleted_at | TIMESTAMPTZ | NULLABLE | |

**Unique Constraint:** (tenant_id, slug)

---

### UNITS

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | UUID | PK | |
| tenant_id | UUID | FK → tenants.id, NOT NULL | |
| name | VARCHAR(100) | NOT NULL | "Piece", "Kilogram", "Liter" |
| short_name | VARCHAR(20) | NOT NULL | "pcs", "kg", "L" |
| category | VARCHAR(50) | NOT NULL | Enum: "unit", "weight", "volume", "length", "area", "time" |
| is_base_unit | BOOLEAN | NOT NULL, DEFAULT false | Anchor unit for conversions |
| is_active | BOOLEAN | NOT NULL, DEFAULT true | |
| created_at | TIMESTAMPTZ | NOT NULL | |
| updated_at | TIMESTAMPTZ | NOT NULL | |
| deleted_at | TIMESTAMPTZ | NULLABLE | |

---

### UNIT_CONVERSIONS

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | UUID | PK | |
| tenant_id | UUID | FK → tenants.id, NOT NULL | |
| from_unit_id | UUID | FK → units.id, NOT NULL | |
| to_unit_id | UUID | FK → units.id, NOT NULL | |
| factor | DECIMAL(20,6) | NOT NULL | Conversion factor |
| created_at | TIMESTAMPTZ | NOT NULL | |

**Unique Constraint:** (from_unit_id, to_unit_id)
**Check:** from_unit_id != to_unit_id

---

### PRODUCTS

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | UUID | PK | |
| tenant_id | UUID | FK → tenants.id, NOT NULL | |
| category_id | UUID | FK → categories.id, NULLABLE | |
| brand_id | UUID | FK → brands.id, NULLABLE | |
| unit_id | UUID | FK → units.id, NOT NULL | Default selling unit |
| purchase_unit_id | UUID | FK → units.id, NULLABLE | Purchasing unit (may differ) |
| code | VARCHAR(100) | NOT NULL | SKU |
| barcode | VARCHAR(100) | NULLABLE | EAN-13, UPC-A, etc. |
| name | VARCHAR(255) | NOT NULL | |
| description | TEXT | NULLABLE | |
| type | VARCHAR(50) | NOT NULL | Enum: "simple", "variant", "composite", "service" |
| track_serial | BOOLEAN | NOT NULL, DEFAULT false | Serial number tracking |
| track_batch | BOOLEAN | NOT NULL, DEFAULT false | Batch/lot tracking |
| has_expiry | BOOLEAN | NOT NULL, DEFAULT false | |
| is_active | BOOLEAN | NOT NULL, DEFAULT true | |
| tax_inclusive | BOOLEAN | NOT NULL, DEFAULT true | Price includes tax |
| tax_rate | DECIMAL(5,2) | NOT NULL, DEFAULT 0 | Percentage |
| min_stock | DECIMAL(20,2) | NOT NULL, DEFAULT 0 | Low stock threshold |
| max_stock | DECIMAL(20,2) | NULLABLE | Overstock threshold |
| weight | DECIMAL(12,4) | NULLABLE | In kg |
| tags | TEXT[] | DEFAULT '{}' | Search/filter tags |
| image_url | TEXT | NULLABLE | Primary product image |
| created_at | TIMESTAMPTZ | NOT NULL | |
| updated_at | TIMESTAMPTZ | NOT NULL | |
| deleted_at | TIMESTAMPTZ | NULLABLE | |

**Unique Constraint:** (tenant_id, code)
**Unique Constraint:** (tenant_id, barcode) — barcode unique per tenant

---

### PRODUCT_VARIANTS

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | UUID | PK | |
| tenant_id | UUID | FK → tenants.id, NOT NULL | |
| product_id | UUID | FK → products.id, NOT NULL, ON DELETE CASCADE | |
| code | VARCHAR(100) | NOT NULL | Variant SKU |
| barcode | VARCHAR(100) | NULLABLE | |
| name | VARCHAR(255) | NOT NULL | "Size M - Red" |
| attributes | JSONB | NOT NULL | {"size": "M", "color": "Red"} |
| price | DECIMAL(20,2) | NULLABLE | Override parent price if set |
| cost_price | DECIMAL(20,2) | NULLABLE | |
| is_active | BOOLEAN | NOT NULL, DEFAULT true | |
| sort_order | INTEGER | NOT NULL, DEFAULT 0 | |
| created_at | TIMESTAMPTZ | NOT NULL | |
| updated_at | TIMESTAMPTZ | NOT NULL | |
| deleted_at | TIMESTAMPTZ | NULLABLE | |

**Unique Constraint:** (tenant_id, code)

---

### PRODUCT_PRICES

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | UUID | PK | |
| tenant_id | UUID | FK → tenants.id, NOT NULL | |
| product_id | UUID | FK → products.id, NOT NULL, ON DELETE CASCADE | |
| variant_id | UUID | FK → product_variants.id, NULLABLE, ON DELETE CASCADE | |
| price_list_id | UUID | FK → price_lists.id, NOT NULL | |
| price | DECIMAL(20,2) | NOT NULL | |
| min_qty | INTEGER | NOT NULL, DEFAULT 1 | Min qty for this price tier |
| created_at | TIMESTAMPTZ | NOT NULL | |
| updated_at | TIMESTAMPTZ | NOT NULL | |

**Unique Constraint:** (product_id, variant_id, price_list_id, min_qty)

---

### PRICE_LISTS

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | UUID | PK | |
| tenant_id | UUID | FK → tenants.id, NOT NULL | |
| name | VARCHAR(100) | NOT NULL | "Retail", "Wholesale", "Member" |
| slug | VARCHAR(100) | NOT NULL | |
| type | VARCHAR(50) | NOT NULL | Enum: "retail", "wholesale", "special", "membership" |
| is_default | BOOLEAN | NOT NULL, DEFAULT false | |
| markup_percentage | DECIMAL(10,2) | NULLABLE | Auto-calculate from cost |
| created_at | TIMESTAMPTZ | NOT NULL | |
| updated_at | TIMESTAMPTZ | NOT NULL | |
| deleted_at | TIMESTAMPTZ | NULLABLE | |

---

### CUSTOMER_GROUPS

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | UUID | PK | |
| tenant_id | UUID | FK → tenants.id, NOT NULL | |
| name | VARCHAR(100) | NOT NULL | "Regular", "Gold", "Platinum" |
| slug | VARCHAR(100) | NOT NULL | |
| discount_percentage | DECIMAL(5,2) | NOT NULL, DEFAULT 0 | Auto-discount |
| created_at | TIMESTAMPTZ | NOT NULL | |
| updated_at | TIMESTAMPTZ | NOT NULL | |

---

### CUSTOMERS

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | UUID | PK | |
| tenant_id | UUID | FK → tenants.id, NOT NULL | |
| group_id | UUID | FK → customer_groups.id, NULLABLE | |
| branch_id | UUID | FK → branches.id, NULLABLE | Default branch |
| code | VARCHAR(50) | NOT NULL | Customer code |
| name | VARCHAR(255) | NOT NULL | |
| email | VARCHAR(255) | NULLABLE | |
| phone | VARCHAR(50) | NULLABLE | |
| whatsapp | VARCHAR(50) | NULLABLE | |
| address | TEXT | NULLABLE | |
| city | VARCHAR(100) | NULLABLE | |
| province | VARCHAR(100) | NULLABLE | |
| postal_code | VARCHAR(20) | NULLABLE | |
| tax_id | VARCHAR(100) | NULLABLE | NPWP |
| credit_limit | DECIMAL(20,2) | NOT NULL, DEFAULT 0 | |
| total_purchases | DECIMAL(20,2) | NOT NULL, DEFAULT 0 | Computed aggregate |
| loyalty_points | INTEGER | NOT NULL, DEFAULT 0 | |
| birth_date | DATE | NULLABLE | |
| notes | TEXT | NULLABLE | |
| is_active | BOOLEAN | NOT NULL, DEFAULT true | |
| created_at | TIMESTAMPTZ | NOT NULL | |
| updated_at | TIMESTAMPTZ | NOT NULL | |
| deleted_at | TIMESTAMPTZ | NULLABLE | |

**Unique Constraint:** (tenant_id, code)
**Unique Constraint:** (tenant_id, email) — if email present

---

### SUPPLIERS

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | UUID | PK | |
| tenant_id | UUID | FK → tenants.id, NOT NULL | |
| code | VARCHAR(50) | NOT NULL | |
| name | VARCHAR(255) | NOT NULL | |
| contact_person | VARCHAR(255) | NULLABLE | |
| email | VARCHAR(255) | NULLABLE | |
| phone | VARCHAR(50) | NULLABLE | |
| whatsapp | VARCHAR(50) | NULLABLE | |
| address | TEXT | NULLABLE | |
| city | VARCHAR(100) | NULLABLE | |
| province | VARCHAR(100) | NULLABLE | |
| tax_id | VARCHAR(100) | NULLABLE | |
| payment_terms | VARCHAR(100) | NULLABLE | "Net 30", "Cash on Delivery" |
| lead_time_days | INTEGER | NULLABLE | Average lead time |
| is_active | BOOLEAN | NOT NULL, DEFAULT true | |
| created_at | TIMESTAMPTZ | NOT NULL | |
| updated_at | TIMESTAMPTZ | NOT NULL | |
| deleted_at | TIMESTAMPTZ | NULLABLE | |

**Unique Constraint:** (tenant_id, code)

---

### PRODUCT_SUPPLIERS

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | UUID | PK | |
| tenant_id | UUID | FK → tenants.id, NOT NULL | |
| product_id | UUID | FK → products.id, NOT NULL, ON DELETE CASCADE | |
| supplier_id | UUID | FK → suppliers.id, NOT NULL, ON DELETE CASCADE | |
| supplier_code | VARCHAR(100) | NULLABLE | Supplier's SKU |
| cost_price | DECIMAL(20,2) | NULLABLE | Supplier-specific cost |
| is_preferred | BOOLEAN | NOT NULL, DEFAULT false | |
| min_order_qty | DECIMAL(20,2) | NULLABLE | |
| lead_time_days | INTEGER | NULLABLE | |
| created_at | TIMESTAMPTZ | NOT NULL | |

**Unique Constraint:** (product_id, supplier_id)

---

### INVENTORY_BALANCES

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | UUID | PK | |
| tenant_id | UUID | FK → tenants.id, NOT NULL | |
| warehouse_id | UUID | FK → warehouses.id, NOT NULL | |
| product_id | UUID | FK → products.id, NOT NULL | |
| variant_id | UUID | FK → product_variants.id, NULLABLE | Null = parent product |
| batch_number | VARCHAR(100) | NULLABLE | Batch tracking |
| expiry_date | DATE | NULLABLE | Expiry tracking |
| quantity | DECIMAL(20,2) | NOT NULL, DEFAULT 0 | Current on-hand |
| reserved_quantity | DECIMAL(20,2) | NOT NULL, DEFAULT 0 | In active carts/orders |
| available_quantity | DECIMAL(20,2) | GENERATED AS (quantity - reserved_quantity) STORED | Computed |
| cost_price | DECIMAL(20,2) | NULLABLE | Current moving average cost |
| last_movement_at | TIMESTAMPTZ | NULLABLE | |
| created_at | TIMESTAMPTZ | NOT NULL | |
| updated_at | TIMESTAMPTZ | NOT NULL | |

**Unique Constraint:** (warehouse_id, product_id, variant_id, batch_number, expiry_date)

---

### INVENTORY_MOVEMENTS

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | BIGSERIAL | PK | Sequential for chronological ordering |
| tenant_id | UUID | FK → tenants.id, NOT NULL | |
| movement_type | VARCHAR(50) | NOT NULL | Enum: "purchase_receipt", "sale", "return", "transfer_out", "transfer_in", "adjustment", "write_off", "opening_balance" |
| reference_type | VARCHAR(50) | NULLABLE | "sale", "purchase_order", "transfer" |
| reference_id | UUID | NULLABLE | Polymorphic reference |
| warehouse_id | UUID | FK → warehouses.id, NOT NULL | |
| product_id | UUID | FK → products.id, NOT NULL | |
| variant_id | UUID | FK → product_variants.id, NULLABLE | |
| batch_number | VARCHAR(100) | NULLABLE | |
| expiry_date | DATE | NULLABLE | |
| quantity | DECIMAL(20,2) | NOT NULL | Positive = in, Negative = out |
| cost_before | DECIMAL(20,2) | NULLABLE | Cost before movement |
| cost_after | DECIMAL(20,2) | NULLABLE | Cost after movement |
| unit_cost | DECIMAL(20,2) | NULLABLE | Cost per unit at movement time |
| notes | TEXT | NULLABLE | |
| created_by | UUID | FK → users.id, NOT NULL | |
| created_at | TIMESTAMPTZ | NOT NULL, DEFAULT NOW() | |

**Indexes:** (tenant_id, warehouse_id, product_id), (reference_type, reference_id), (created_at)

---

### SALES

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | UUID | PK | |
| tenant_id | UUID | FK → tenants.id, NOT NULL | |
| branch_id | UUID | FK → branches.id, NOT NULL | |
| warehouse_id | UUID | FK → warehouses.id, NOT NULL | Source warehouse |
| customer_id | UUID | FK → customers.id, NULLABLE | Walk-in = null |
| user_id | UUID | FK → users.id, NOT NULL | Cashier who processed |
| shift_id | UUID | FK → shifts.id, NULLABLE | |
| invoice_number | VARCHAR(50) | NOT NULL | Tenant-sequential |
| status | VARCHAR(50) | NOT NULL, DEFAULT 'completed' | Enum: "draft", "completed", "voided", "refunded" |
| subtotal | DECIMAL(20,2) | NOT NULL | Before discount |
| discount_total | DECIMAL(20,2) | NOT NULL, DEFAULT 0 | |
| tax_total | DECIMAL(20,2) | NOT NULL, DEFAULT 0 | |
| grand_total | DECIMAL(20,2) | NOT NULL | After discount + tax |
| rounding | DECIMAL(10,2) | NOT NULL, DEFAULT 0 | Rounding adjustment |
| paid_amount | DECIMAL(20,2) | NOT NULL, DEFAULT 0 | |
| change_amount | DECIMAL(20,2) | NOT NULL, DEFAULT 0 | |
| notes | TEXT | NULLABLE | |
| void_reason | TEXT | NULLABLE | |
| voided_by | UUID | FK → users.id, NULLABLE | |
| voided_at | TIMESTAMPTZ | NULLABLE | |
| created_at | TIMESTAMPTZ | NOT NULL | |
| updated_at | TIMESTAMPTZ | NOT NULL | |
| deleted_at | TIMESTAMPTZ | NULLABLE | |

**Unique Constraint:** (tenant_id, invoice_number)

---

### SALE_ITEMS

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | UUID | PK | |
| tenant_id | UUID | FK → tenants.id, NOT NULL | |
| sale_id | UUID | FK → sales.id, NOT NULL, ON DELETE CASCADE | |
| product_id | UUID | FK → products.id, NOT NULL | |
| variant_id | UUID | FK → product_variants.id, NULLABLE | |
| line_number | INTEGER | NOT NULL | Display order |
| quantity | DECIMAL(20,2) | NOT NULL | |
| unit_price | DECIMAL(20,2) | NOT NULL | Price per unit at sale time |
| discount_per_item | DECIMAL(20,2) | NOT NULL, DEFAULT 0 | |
| tax_per_item | DECIMAL(20,2) | NOT NULL, DEFAULT 0 | |
| total_price | DECIMAL(20,2) | NOT NULL | (unit_price - discount) * qty + tax |
| cost_price | DECIMAL(20,2) | NULLABLE | COGS at sale time |
| batch_number | VARCHAR(100) | NULLABLE | |
| expiry_date | DATE | NULLABLE | |
| serial_number | VARCHAR(100) | NULLABLE | |
| notes | TEXT | NULLABLE | |
| created_at | TIMESTAMPTZ | NOT NULL | |

---

### PAYMENTS

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | UUID | PK | |
| tenant_id | UUID | FK → tenants.id, NOT NULL | |
| sale_id | UUID | FK → sales.id, NOT NULL | |
| payment_method_id | UUID | FK → payment_methods.id, NOT NULL | |
| amount | DECIMAL(20,2) | NOT NULL | |
| reference_number | VARCHAR(255) | NULLABLE | Card last4, transaction ID, QR ref |
| gateway_response | JSONB | NULLABLE | Raw payment gateway response |
| status | VARCHAR(50) | NOT NULL, DEFAULT 'completed' | Enum: "pending", "completed", "failed", "refunded" |
| paid_at | TIMESTAMPTZ | NOT NULL, DEFAULT NOW() | |
| created_at | TIMESTAMPTZ | NOT NULL | |

---

### PAYMENT_METHODS

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | UUID | PK | |
| tenant_id | UUID | FK → tenants.id, NOT NULL | |
| code | VARCHAR(50) | NOT NULL | "cash", "card_visa", "gopay", "qris" |
| name | VARCHAR(100) | NOT NULL | |
| type | VARCHAR(50) | NOT NULL | Enum: "cash", "card", "e_wallet", "qris", "transfer", "credit" |
| requires_reference | BOOLEAN | NOT NULL, DEFAULT false | |
| is_active | BOOLEAN | NOT NULL, DEFAULT true | |
| sort_order | INTEGER | NOT NULL, DEFAULT 0 | |
| created_at | TIMESTAMPTZ | NOT NULL | |

**Unique Constraint:** (tenant_id, code)

---

### PURCHASE_ORDERS

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | UUID | PK | |
| tenant_id | UUID | FK → tenants.id, NOT NULL | |
| branch_id | UUID | FK → branches.id, NOT NULL | |
| warehouse_id | UUID | FK → warehouses.id, NOT NULL | Target warehouse |
| supplier_id | UUID | FK → suppliers.id, NOT NULL | |
| po_number | VARCHAR(50) | NOT NULL | Tenant-sequential |
| status | VARCHAR(50) | NOT NULL | Enum: "draft", "pending_approval", "approved", "ordered", "partially_received", "completed", "cancelled" |
| order_date | DATE | NOT NULL | |
| expected_date | DATE | NULLABLE | |
| received_date | DATE | NULLABLE | |
| subtotal | DECIMAL(20,2) | NOT NULL | |
| discount_total | DECIMAL(20,2) | NOT NULL, DEFAULT 0 | |
| tax_total | DECIMAL(20,2) | NOT NULL, DEFAULT 0 | |
| grand_total | DECIMAL(20,2) | NOT NULL | |
| notes | TEXT | NULLABLE | |
| terms | TEXT | NULLABLE | Payment/Shipping terms |
| requested_by | UUID | FK → users.id, NOT NULL | |
| approved_by | UUID | FK → users.id, NULLABLE | |
| approved_at | TIMESTAMPTZ | NULLABLE | |
| created_at | TIMESTAMPTZ | NOT NULL | |
| updated_at | TIMESTAMPTZ | NOT NULL | |
| deleted_at | TIMESTAMPTZ | NULLABLE | |

**Unique Constraint:** (tenant_id, po_number)

---

### PURCHASE_ORDER_ITEMS

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | UUID | PK | |
| tenant_id | UUID | FK → tenants.id, NOT NULL | |
| purchase_order_id | UUID | FK → purchase_orders.id, NOT NULL, ON DELETE CASCADE | |
| product_id | UUID | FK → products.id, NOT NULL | |
| variant_id | UUID | FK → product_variants.id, NULLABLE | |
| line_number | INTEGER | NOT NULL | |
| quantity_ordered | DECIMAL(20,2) | NOT NULL | |
| quantity_received | DECIMAL(20,2) | NOT NULL, DEFAULT 0 | |
| quantity_invoiced | DECIMAL(20,2) | NOT NULL, DEFAULT 0 | |
| unit_cost | DECIMAL(20,2) | NOT NULL | Cost per unit |
| discount_per_item | DECIMAL(20,2) | NOT NULL, DEFAULT 0 | |
| tax_per_item | DECIMAL(20,2) | NOT NULL, DEFAULT 0 | |
| total_cost | DECIMAL(20,2) | NOT NULL | (unit_cost - discount + tax) * qty |
| batch_number | VARCHAR(100) | NULLABLE | Pre-assigned batch |
| expiry_date | DATE | NULLABLE | |
| notes | TEXT | NULLABLE | |

---

### SUBSCRIPTIONS

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | UUID | PK | |
| tenant_id | UUID | FK → tenants.id, NOT NULL, UNIQUE | One subscription per tenant |
| plan_id | UUID | FK → plans.id, NOT NULL | |
| status | VARCHAR(50) | NOT NULL | Enum: "active", "trialing", "past_due", "canceled", "expired", "suspended" |
| trial_ends_at | TIMESTAMPTZ | NULLABLE | |
| current_period_starts_at | TIMESTAMPTZ | NOT NULL | |
| current_period_ends_at | TIMESTAMPTZ | NOT NULL | |
| canceled_at | TIMESTAMPTZ | NULLABLE | |
| canceled_at_period_end | BOOLEAN | NOT NULL, DEFAULT false | |
| stripe_subscription_id | VARCHAR(255) | NULLABLE | External billing ref |
| stripe_customer_id | VARCHAR(255) | NULLABLE | |
| metadata | JSONB | DEFAULT '{}' | |
| created_at | TIMESTAMPTZ | NOT NULL | |
| updated_at | TIMESTAMPTZ | NOT NULL | |

---

### PLANS

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | UUID | PK | |
| name | VARCHAR(100) | NOT NULL | "Starter", "Business", "Enterprise" |
| slug | VARCHAR(100) | NOT NULL, UNIQUE | |
| description | TEXT | NULLABLE | |
| price_monthly | DECIMAL(20,2) | NOT NULL | |
| price_yearly | DECIMAL(20,2) | NOT NULL | |
| max_users | INTEGER | NOT NULL | User limit |
| max_branches | INTEGER | NOT NULL | Branch limit |
| max_products | INTEGER | NOT NULL | Product catalog limit |
| features | JSONB | NOT NULL | Feature entitlement flags |
| is_active | BOOLEAN | NOT NULL, DEFAULT true | |
| sort_order | INTEGER | NOT NULL | |
| created_at | TIMESTAMPTZ | NOT NULL | |
| updated_at | TIMESTAMPTZ | NOT NULL | |

---

### AUDIT_LOGS

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | BIGSERIAL | PK | Sequential, never exposed externally |
| tenant_id | UUID | FK → tenants.id, NOT NULL | |
| user_id | UUID | FK → users.id, NULLABLE | System actions = null |
| ip_address | VARCHAR(45) | NULLABLE | IPv4 or IPv6 |
| user_agent | TEXT | NULLABLE | |
| action | VARCHAR(100) | NOT NULL | "CREATE", "UPDATE", "DELETE", "LOGIN", "EXPORT" |
| entity_type | VARCHAR(100) | NOT NULL | "product", "sale", "user" |
| entity_id | UUID | NOT NULL | The affected record |
| old_values | JSONB | NULLABLE | Previous state |
| new_values | JSONB | NULLABLE | New state |
| changed_fields | TEXT[] | NULLABLE | Array of changed field names |
| correlation_id | UUID | NULLABLE | Link related audit events |
| created_at | TIMESTAMPTZ | NOT NULL, DEFAULT NOW() | |

---

### NOTIFICATIONS

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | UUID | PK | |
| tenant_id | UUID | FK → tenants.id, NOT NULL | |
| user_id | UUID | FK → users.id, NULLABLE | Null = broadcast to role |
| role_slug | VARCHAR(100) | NULLABLE | Broadcast target |
| type | VARCHAR(50) | NOT NULL | "low_stock", "po_approved", "new_order", "system" |
| channel | VARCHAR(50) | NOT NULL | "in_app", "email", "whatsapp", "sms" |
| title | VARCHAR(255) | NOT NULL | |
| body | TEXT | NOT NULL | |
| data | JSONB | NULLABLE | Action payload |
| is_read | BOOLEAN | NOT NULL, DEFAULT false | |
| read_at | TIMESTAMPTZ | NULLABLE | |
| created_at | TIMESTAMPTZ | NOT NULL | |

---

### SHIFTS

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | UUID | PK | |
| tenant_id | UUID | FK → tenants.id, NOT NULL | |
| branch_id | UUID | FK → branches.id, NOT NULL | |
| user_id | UUID | FK → users.id, NOT NULL | Cashier |
| cashier_name | VARCHAR(255) | NOT NULL | Denormalized for reports |
| opened_at | TIMESTAMPTZ | NOT NULL | |
| closed_at | TIMESTAMPTZ | NULLABLE | |
| opening_balance | DECIMAL(20,2) | NOT NULL | |
| closing_balance | DECIMAL(20,2) | NULLABLE | Expected |
| actual_balance | DECIMAL(20,2) | NULLABLE | Counted |
| difference | DECIMAL(20,2) | GENERATED (actual - closing) | |
| total_sales | DECIMAL(20,2) | NOT NULL, DEFAULT 0 | |
| total_transactions | INTEGER | NOT NULL, DEFAULT 0 | |
| status | VARCHAR(50) | NOT NULL, DEFAULT 'open' | "open", "closed", "verified" |
| notes | TEXT | NULLABLE | |
| verified_by | UUID | FK → users.id, NULLABLE | Manager verification |
| created_at | TIMESTAMPTZ | NOT NULL | |
| updated_at | TIMESTAMPTZ | NOT NULL | |

---

## 2.2 Relationship Summary

```
tenants 1──M users
tenants 1──M branches
tenants 1──M roles
tenants 1──M permissions
tenants 1──M categories
tenants 1──M brands
tenants 1──M units
tenants 1──M products
tenants 1──M customers
tenants 1──M suppliers
tenants 1──M warehouses
tenants 1──M sales
tenants 1──M purchase_orders
tenants 1──M subscriptions
tenants 1──M audit_logs
tenants 1──M notifications
tenants 1──M payment_methods
tenants 1──M price_lists
tenants 1──M customer_groups

branches 1──M warehouses
branches 1──M sales
branches 1──M purchase_orders
branches 1──M shifts

warehouses 1──M inventory_balances
warehouses 1──M inventory_movements

categories 1──M products
categories M──1 categories (self: parent_id)

products 1──M product_variants
products 1──M product_prices
products 1──M product_suppliers
products 1──M sale_items
products 1──M purchase_order_items
products 1──M inventory_balances
products 1──M inventory_movements

users 1──M sales
users 1──M shifts
users M──M roles (via user_roles)

roles M──M permissions (via role_permissions)

sales 1──M sale_items
sales 1──M payments

purchase_orders 1──M purchase_order_items

customers 1──M sales
suppliers 1──M purchase_orders
```

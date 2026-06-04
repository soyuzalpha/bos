# Phase 3: Database Design Strategy

## 3.1 Multi-Tenant Strategy: Discriminated Row-Level with Tenant Column

**Strategy:** Every business table includes `tenant_id UUID NOT NULL` as a discriminator column. All queries are scoped by `WHERE tenant_id = :currentTenantId`.

### Why NOT schema-per-tenant

- Schema-per-tenant requires connection pooling per tenant or dynamic DDL — operationally impossible at 10,000+ tenants.
- Migration tooling becomes exponentially harder with 10,000+ schemas.
- Shared pool destroys connection count at scale.
- Row-level isolation is proven at hyperscale (Salesforce, Shopify, HubSpot).

### Why NOT database-per-tenant

- 10,000 databases × connection pool overhead = connection exhaustion.
- Backup/recovery complexity scales linearly with tenant count.
- Operational cost is multiplied by tenant count.
- Cross-tenant reporting is impossible without aggregation services.

### Row-Level Security (RLS)

Enable PostgreSQL Row-Level Security with a session variable:

```sql
ALTER TABLE products ENABLE ROW LEVEL SECURITY;

CREATE POLICY tenant_isolation_policy ON products
  USING (tenant_id = current_setting('app.current_tenant_id')::UUID);

CREATE POLICY tenant_isolation_policy ON sales
  USING (tenant_id = current_setting('app.current_tenant_id')::UUID);
-- ... every tenant-scoped table
```

Set `app.current_tenant_id` at connection pool checkout via `SET SESSION` in Prisma middleware or connection init hook.

---

## 3.2 Tenant Security

| Layer | Mechanism |
|-------|-----------|
| **API** | Extract `X-Tenant-Id` header or JWT claim. Validate tenant exists and user belongs to tenant. |
| **Database** | RLS policies on all tables (defense in depth). Bypass if `app.is_admin = true`. |
| **Prisma** | Global middleware that injects `tenant_id` into all `create` operations and scopes all `findMany`/`findFirst`/`update`/`delete`. |
| **Cache (Redis)** | Key prefix: `{tenant_id}:{entity}:{id}` |
| **Queue (BullMQ)** | Jobs include `tenantId` in job data. Queue name prefixed by tenant for isolation. |
| **File Storage** | Path prefix: `/uploads/{tenant_id}/{entity}/{id}` |

---

## 3.3 Soft Delete Strategy

| Table Type | Strategy |
|------------|----------|
| **Master/Core** (products, customers, users, suppliers, branches, warehouses, roles) | `deleted_at TIMESTAMPTZ NULLABLE`. All queries filter `WHERE deleted_at IS NULL`. |
| **Transactional** (sales, purchase_orders, inventory_movements) | `deleted_at` available but should rarely be needed. Instead, use `voided_at` / `cancelled_at` for logical reversals. Hard delete on cascade for `sale_items` / `po_items` if parent is deleted. |
| **Join Tables** (role_permissions, user_roles, product_suppliers) | No soft delete. Hard delete via CASCADE. |
| **Audit/HLL** (audit_logs, inventory_movements) | **No soft delete, no hard delete.** Append-only. Retention via partitioning. |

**Soft Delete Filter:** Create a Prisma middleware or view that automatically filters `deleted_at IS NULL` for all queries. This prevents accidentally including soft-deleted records.

---

## 3.4 Audit Strategy

### What to audit

- All `CREATE`, `UPDATE`, `DELETE` on master data (products, customers, suppliers, users, roles, prices).
- All `CREATE` and `DELETE` on transactional data (sales, purchases).
- Authentication events (LOGIN, LOGOUT, FAILED_LOGIN).
- Export events (EXPORT).
- Configuration changes (settings, pricing, tax rates).

### What NOT to audit

- Read queries (no state change).
- Join table writes that are purely structural (already covered by parent entity audit).

### Implementation

1. **Prisma Middleware** — Intercept `create`, `update`, `delete` on registered models. Capture before/after state. Run in transaction with the main operation.
2. **Batch Insert** — Audit logs use `createMany` inside the same transaction. No separate audit service call (avoids distributed consistency issues).
3. **Retention** — Partition `audit_logs` by month. Drop partitions older than 12 months (or move to cold storage).

---

## 3.5 Indexing Strategy

### General Rules

- Every FK column gets an index.
- Every `tenant_id` column gets an index (always preceding other indexed columns for multi-column).
- Every `status` column used in WHERE gets an index.
- Every `date` column used in range queries gets an index.
- Every `slug` used in URL lookups gets a unique index.
- Every `code`/`invoice_number` used in search gets an index.

---

## 3.6 Recommended PostgreSQL Indexes

```sql
-- ============ TENANT SCOPING ============
-- These are implicitly covered by the business indexes below because
-- tenant_id is always the leading column.

-- ============ USERS ============
CREATE INDEX idx_users_tenant_email ON users(tenant_id, email) WHERE deleted_at IS NULL;
CREATE INDEX idx_users_tenant_phone ON users(tenant_id, phone) WHERE deleted_at IS NULL;

-- ============ PRODUCTS ============
CREATE INDEX idx_products_tenant_code ON products(tenant_id, code) WHERE deleted_at IS NULL;
CREATE INDEX idx_products_tenant_barcode ON products(tenant_id, barcode) WHERE barcode IS NOT NULL AND deleted_at IS NULL;
CREATE INDEX idx_products_tenant_category ON products(tenant_id, category_id) WHERE deleted_at IS NULL;
CREATE INDEX idx_products_tenant_brand ON products(tenant_id, brand_id) WHERE deleted_at IS NULL;
CREATE INDEX idx_products_tenant_active ON products(tenant_id, is_active) WHERE deleted_at IS NULL;
CREATE INDEX idx_products_search ON products USING gin(to_tsvector('simple', name || ' ' || COALESCE(code, '')));
CREATE INDEX idx_products_tags ON products USING gin(tags);

-- ============ INVENTORY ============
CREATE INDEX idx_inv_balance_warehouse_product ON inventory_balances(tenant_id, warehouse_id, product_id) INCLUDE (quantity, reserved_quantity, cost_price);
CREATE INDEX idx_inv_balance_low_stock ON inventory_balances(tenant_id, warehouse_id, quantity) WHERE quantity <= 5;
CREATE INDEX idx_inv_movements_tenant_ref ON inventory_movements(tenant_id, reference_type, reference_id);
CREATE INDEX idx_inv_movements_tenant_date ON inventory_movements(tenant_id, warehouse_id, created_at DESC);

-- ============ SALES ============
CREATE INDEX idx_sales_tenant_date ON sales(tenant_id, created_at DESC);
CREATE INDEX idx_sales_tenant_branch ON sales(tenant_id, branch_id, created_at DESC);
CREATE INDEX idx_sales_tenant_customer ON sales(tenant_id, customer_id) WHERE customer_id IS NOT NULL;
CREATE INDEX idx_sales_tenant_user ON sales(tenant_id, user_id, created_at DESC);
CREATE INDEX idx_sales_tenant_status ON sales(tenant_id, status) WHERE status IN ('draft', 'completed');
CREATE INDEX idx_sales_tenant_invoice ON sales(tenant_id, invoice_number);

-- ============ SALE ITEMS ============
CREATE INDEX idx_sale_items_sale ON sale_items(sale_id);
CREATE INDEX idx_sale_items_product ON sale_items(tenant_id, product_id, created_at DESC);

-- ============ PURCHASE ORDERS ============
CREATE INDEX idx_po_tenant_status ON purchase_orders(tenant_id, status);
CREATE INDEX idx_po_tenant_supplier ON purchase_orders(tenant_id, supplier_id, status);
CREATE INDEX idx_po_tenant_date ON purchase_orders(tenant_id, order_date DESC);
CREATE INDEX idx_po_tenant_branch ON purchase_orders(tenant_id, branch_id, status);

-- ============ AUDIT LOGS ============
CREATE INDEX idx_audit_tenant_entity ON audit_logs(tenant_id, entity_type, entity_id);
CREATE INDEX idx_audit_tenant_user ON audit_logs(tenant_id, user_id);
CREATE INDEX idx_audit_tenant_date ON audit_logs(tenant_id, created_at DESC);

-- ============ NOTIFICATIONS ============
CREATE INDEX idx_notifications_user_unread ON notifications(tenant_id, user_id, is_read) WHERE is_read = false;
CREATE INDEX idx_notifications_tenant_created ON notifications(tenant_id, created_at DESC);

-- ============ PRODUCT VARIANTS ============
CREATE INDEX idx_variants_product ON product_variants(product_id);
CREATE INDEX idx_variants_code ON product_variants(tenant_id, code);

-- ============ CUSTOMERS ============
CREATE INDEX idx_customers_tenant_code ON customers(tenant_id, code) WHERE deleted_at IS NULL;
CREATE INDEX idx_customers_tenant_phone ON customers(tenant_id, phone) WHERE phone IS NOT NULL AND deleted_at IS NULL;
CREATE INDEX idx_customers_search ON customers USING gin(to_tsvector('simple', name || ' ' || COALESCE(phone, '')));
```

---

## 3.7 UUID vs Numeric ID Strategy

### UUID for ALL primary keys (recommended)

| Pro | Con |
|-----|-----|
| Universally unique across all tenants — no collision risk at 10K+ tenants | 16 bytes vs 4/8 bytes for integer |
| Safe to expose in URLs and APIs | Slower than sequential integers for B-tree insertion (random writes) |
| No centralized ID generation bottleneck | More index bloat over time |
| Merge-safe — tenants can generate offline IDs | No natural ordering |
| Distributed systems friendly | Larger foreign keys in child tables |

### When to use BIGSERIAL

- `inventory_movements.id` — sequential ordering for chronological queries and pagination. UUIDs on 100M+ rows with range queries degrade significantly.
- `audit_logs.id` — append-only, needs chronological ordering, sequential IDs enable efficient pagination.

### Strategy

- Use `gen_random_uuid()` for all PKs on transactional and master tables (PostgreSQL 13+ has built-in `gen_random_uuid()`).
- Use `BIGSERIAL` for append-only, high-volume audit/history tables.
- Use `TIMESTAMPTZ created_at` columns for ordering (don't rely on UUIDs for sort).
- Consider `uuid-ossp` extension for v7 time-ordered UUIDs if needed (PostgreSQL 16+).

---

## 3.8 Query Optimization Patterns

| Pattern | Implementation |
|---------|---------------|
| **N+1 Prevention** | Prisma `include` / `select` only needed relations. Use `@RelationLoader` for lazy in NestJS. |
| **Cursor Pagination** | For high-volume tables (sales, movements, audit logs). `WHERE created_at < :cursor ORDER BY created_at DESC LIMIT 50`. |
| **Offset Pagination** | For low-volume master data (products < 50K, customers < 100K). Safe with `LIMIT/OFFSET`. |
| **Materialized Views** | Daily sales summary, inventory valuation, customer purchase frequency. Refresh via cron/BullMQ. |
| **Covering Indexes** | Include frequently accessed columns in indexes to avoid heap lookups. |
| **Partial Indexes** | `WHERE deleted_at IS NULL` on all soft-delete tables. Drastically reduces index size. |
| **Composite Index Order** | `(tenant_id, most_selective, second_most_selective)` — always tenant_id first. |
| **Table Partitioning** | `audit_logs` by month, `inventory_movements` by quarter. Enables partition pruning and efficient purging. |

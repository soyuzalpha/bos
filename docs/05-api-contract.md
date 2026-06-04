# Phase 5: API Contract Design

## 5.1 Global Conventions

| Convention | Value |
|------------|-------|
| **Base URL** | `/api/v1` |
| **Headers** | `Authorization: Bearer <token>`, `X-Tenant-Id: <uuid>`, `X-Request-Id: <uuid>` |
| **Pagination** | `?page=1&limit=25` (offset) or `?cursor=<id>&limit=25` (cursor) |
| **Filtering** | `?filter[status]=completed&filter[createdAt][gte]=2025-01-01` |
| **Sorting** | `?sort=-createdAt` (descending) or `?sort=name` (ascending) |
| **Fields** | `?fields=id,name,code` (sparse fieldset) |
| **Includes** | `?include=items,customer` (eager load relations) |
| **Search** | `?q=keyword` (full-text search where supported) |
| **Errors** | `{ success: false, error: { code, message, details? } }` |
| **Success** | `{ success: true, data: {...}, meta?: { page, limit, total } }` |

---

## 5.2 Module Endpoints

### Auth Module

```
POST   /api/v1/auth/login               — Login with email/password
POST   /api/v1/auth/logout              — Invalidate session
POST   /api/v1/auth/refresh             — Refresh access token
POST   /api/v1/auth/forgot-password     — Send reset email
POST   /api/v1/auth/reset-password      — Reset password with token
POST   /api/v1/auth/verify-mfa          — Verify MFA code
POST   /api/v1/auth/change-password     — Change password (authenticated)
GET    /api/v1/auth/me                  — Get current user profile
```

**Login:**
```json
// POST /api/v1/auth/login
// Request
{
  "email": "admin@tokoku.com",
  "password": "secure...123!",
  "tenantSlug": "tokoku-jkt"
}
// Response
{
  "success": true,
  "data": {
    "accessToken": "eyJhbG...NiIs...",
    "refreshToken": "dGhpcyBpcyBhIHJlZnJl...",
    "expiresIn": 3600,
    "user": {
      "id": "uuid",
      "email": "admin@tokoku.com",
      "firstName": "Admin",
      "lastName": "Tokoku",
      "roles": ["admin"],
      "permissions": ["product.*", "sale.*", "user.read"]
    }
  }
}
// Error
{
  "success": false,
  "error": {
    "code": "INVALID_CREDENTIALS",
    "message": "Email or password is incorrect",
    "statusCode": 401
  }
}
```

**Validation Rules (Login):**
| Field | Rule |
|-------|------|
| email | Required, valid email format, max 255 chars |
| password | Required, min 8 chars |
| tenantSlug | Required, min 2 chars, alphanumeric + hyphens |

---

### Tenant Module

```
GET    /api/v1/tenants               — List tenants (superadmin)
POST   /api/v1/tenants               — Register new tenant
GET    /api/v1/tenants/:id           — Get tenant details
PATCH  /api/v1/tenants/:id           — Update tenant settings
DELETE /api/v1/tenants/:id           — Soft delete tenant
```

**Create Tenant:**
```json
// POST /api/v1/tenants
// Request
{
  "name": "Toko ABC",
  "slug": "toko-abc",
  "email": "admin@tokoabc.com",
  "phone": "+6281234567890",
  "currencyCode": "IDR",
  "timezone": "Asia/Jakarta"
}
// Response
{
  "success": true,
  "data": {
    "id": "uuid",
    "name": "Toko ABC",
    "slug": "toko-abc",
    "currencyCode": "IDR",
    "timezone": "Asia/Jakarta",
    "isActive": true,
    "createdAt": "2025-06-04T00:00:00Z"
  }
}
```

---

### Users Module

```
GET    /api/v1/users                      — List users (paginated, filterable)
POST   /api/v1/users                      — Create user
GET    /api/v1/users/:id                  — Get user details
PATCH  /api/v1/users/:id                  — Update user
DELETE /api/v1/users/:id                  — Soft delete user
PATCH  /api/v1/users/:id/activate         — Activate/deactivate user
POST   /api/v1/users/:id/roles            — Assign roles to user
DELETE /api/v1/users/:id/roles/:roleId    — Remove role from user
```

**Create User:**
```json
// POST /api/v1/users
// Request
{
  "email": "kasir1@tokoku.com",
  "phone": "+6281234567891",
  "firstName": "Budi",
  "lastName": "Santoso",
  "roleIds": ["uuid-role-cashier"],
  "branchId": "uuid-branch-jkt"
}
// Paginated Response
{
  "success": true,
  "data": [
    {
      "id": "uuid",
      "email": "admin@tokoku.com",
      "firstName": "Admin",
      "lastName": "Tokoku",
      "isActive": true,
      "roles": [
        { "id": "uuid", "name": "Admin", "slug": "admin" }
      ],
      "branch": { "id": "uuid", "name": "Jakarta Pusat", "code": "JKT-01" },
      "lastLoginAt": "2025-06-03T14:30:00Z",
      "createdAt": "2025-01-01T00:00:00Z"
    }
  ],
  "meta": { "page": 1, "limit": 25, "total": 15, "totalPages": 1 }
}
```

**Validation Rules (User):**
| Field | Rule |
|-------|------|
| email | Required, unique within tenant, valid email |
| phone | Optional, unique within tenant |
| firstName | Required, max 100 chars |
| roleIds | Array of valid UUIDs, at least 1 |

---

### Roles & Permissions Module

```
GET    /api/v1/roles                     — List roles
POST   /api/v1/roles                     — Create role
GET    /api/v1/roles/:id                 — Get role with permissions
PATCH  /api/v1/roles/:id                 — Update role
DELETE /api/v1/roles/:id                 — Delete role (not system)
GET    /api/v1/permissions               — List all permissions
PATCH  /api/v1/roles/:id/permissions     — Sync permissions for role
```

```json
// PATCH /api/v1/roles/:id/permissions
// Request
{
  "permissionIds": ["uuid-p1", "uuid-p2", "uuid-p3"]
}
```

---

### Products Module

```
GET    /api/v1/products                     — List products (search, filter, paginate)
POST   /api/v1/products                     — Create product
GET    /api/v1/products/:id                 — Get product with variants, prices
PATCH  /api/v1/products/:id                 — Update product
DELETE /api/v1/products/:id                 — Soft delete product
POST   /api/v1/products/:id/variants        — Add variant
PATCH  /api/v1/products/:id/variants/:vid   — Update variant
DELETE /api/v1/products/:id/variants/:vid   — Delete variant
GET    /api/v1/products/:id/prices          — Get price list for product
PATCH  /api/v1/products/:id/prices          — Update product prices
POST   /api/v1/products/bulk               — Bulk create/update products
```

**Product List:**
```json
// GET /api/v1/products?page=1&limit=25&filter[categoryId]=uuid&sort=-createdAt&q=indomie
// Response
{
  "success": true,
  "data": [
    {
      "id": "uuid",
      "code": "IND-001",
      "barcode": "8991234567890",
      "name": "Indomie Goreng",
      "type": "simple",
      "category": { "id": "uuid", "name": "Mie Instan" },
      "brand": { "id": "uuid", "name": "Indomie" },
      "unit": { "id": "uuid", "shortName": "pcs" },
      "taxRate": 11,
      "isActive": true,
      "createdAt": "2025-01-15T08:30:00Z"
    }
  ],
  "meta": {
    "page": 1,
    "limit": 25,
    "total": 1234,
    "totalPages": 50
  }
}
```

**Validation Rules (Product):**
| Field | Rule |
|-------|------|
| code | Required, unique within tenant, max 100 chars |
| barcode | Optional, unique within tenant |
| name | Required, max 255 chars |
| type | Required, enum: simple/variant/composite/service |
| unitId | Required, valid UUID |
| taxRate | 0-100, default 0 |
| tags | Array of strings, max 50 tags |

---

### Categories Module

```
GET    /api/v1/categories             — List categories (tree or flat)
POST   /api/v1/categories             — Create category
GET    /api/v1/categories/:id         — Get category with children
PATCH  /api/v1/categories/:id         — Update category
DELETE /api/v1/categories/:id         — Soft delete (reassign products)
```

```json
// GET /api/v1/categories
// Response (tree)
{
  "success": true,
  "data": [
    {
      "id": "uuid",
      "name": "Makanan",
      "slug": "makanan",
      "sortOrder": 1,
      "children": [
        {
          "id": "uuid",
          "name": "Mie Instan",
          "slug": "mie-instan",
          "parentId": "uuid-parent",
          "sortOrder": 1,
          "children": []
        }
      ]
    }
  ]
}
```

---

### Brands Module

```
GET    /api/v1/brands                 — List brands
POST   /api/v1/brands                 — Create brand
GET    /api/v1/brands/:id             — Get brand
PATCH  /api/v1/brands/:id             — Update brand
DELETE /api/v1/brands/:id             — Soft delete
```

---

### Units Module

```
GET    /api/v1/units                  — List units
POST   /api/v1/units                  — Create unit
PATCH  /api/v1/units/:id              — Update unit
DELETE /api/v1/units/:id              — Soft delete
POST   /api/v1/units/conversions      — Create conversion
```

---

### Customers Module

```
GET    /api/v1/customers              — List customers
POST   /api/v1/customers              — Create customer
GET    /api/v1/customers/:id          — Get customer with sales history
PATCH  /api/v1/customers/:id          — Update customer
DELETE /api/v1/customers/:id          — Soft delete
GET    /api/v1/customer-groups        — List customer groups
POST   /api/v1/customer-groups        — Create customer group
```

---

### Suppliers Module

```
GET    /api/v1/suppliers              — List suppliers
POST   /api/v1/suppliers              — Create supplier
GET    /api/v1/suppliers/:id          — Get supplier with products
PATCH  /api/v1/suppliers/:id          — Update supplier
DELETE /api/v1/suppliers/:id          — Soft delete
```

---

### Warehouses Module

```
GET    /api/v1/warehouses             — List warehouses
POST   /api/v1/warehouses             — Create warehouse
PATCH  /api/v1/warehouses/:id         — Update warehouse
DELETE /api/v1/warehouses/:id         — Soft delete
```

---

### Inventory Module

```
GET    /api/v1/inventory/balances           — List inventory balances (filterable by warehouse, product)
GET    /api/v1/inventory/balances/:id       — Get balance detail
GET    /api/v1/inventory/movements          — List stock movements
POST   /api/v1/inventory/adjust            — Adjust stock (physical count)
POST   /api/v1/inventory/transfer           — Transfer stock between warehouses
POST   /api/v1/inventory/write-off          — Write off damaged/expired stock
GET    /api/v1/inventory/low-stock          — Products below minimum stock
```

**Stock Adjustment:**
```json
// POST /api/v1/inventory/adjust
// Request
{
  "warehouseId": "uuid",
  "items": [
    {
      "productId": "uuid",
      "variantId": null,
      "batchNumber": "BATCH-001",
      "expiryDate": "2025-12-31",
      "newQuantity": 150,
      "reason": "Physical count correction"
    }
  ],
  "notes": "Monthly cycle count - Warehouse A"
}
// Response
{
  "success": true,
  "data": {
    "movementsCreated": 1,
    "balances": [
      {
        "productId": "uuid",
        "productName": "Indomie Goreng",
        "previousQuantity": 100,
        "adjustedQuantity": 150,
        "difference": 50,
        "newCostPrice": 3500.00
      }
    ]
  }
}
```

---

### Sales Module (POS)

```
POST   /api/v1/sales                    — Create sale (complete transaction)
GET    /api/v1/sales                    — List sales (filterable)
GET    /api/v1/sales/:id                — Get sale with items and payments
PATCH  /api/v1/sales/:id                — Update sale (draft only)
DELETE /api/v1/sales/:id                — Void sale (with inventory reversal)
POST   /api/v1/sales/:id/refund         — Refund specific items
POST   /api/v1/sales/hold               — Hold cart
GET    /api/v1/sales/holds              — List held carts
DELETE /api/v1/sales/holds/:id          — Discard hold
```

**Create Sale:**
```json
// POST /api/v1/sales
// Request
{
  "branchId": "uuid",
  "warehouseId": "uuid",
  "customerId": "uuid",
  "shiftId": "uuid",
  "items": [
    {
      "productId": "uuid",
      "variantId": null,
      "quantity": 5,
      "unitPrice": 4000.00,
      "discountPerItem": 0,
      "batchNumber": null,
      "expiryDate": null
    },
    {
      "productId": "uuid",
      "variantId": "uuid-variant",
      "quantity": 2,
      "unitPrice": 25000.00,
      "discountPerItem": 1000.00
    }
  ],
  "payments": [
    {
      "paymentMethodId": "uuid-cash",
      "amount": 50000.00
    }
  ],
  "notes": "Pagi ramai"
}
// Response
{
  "success": true,
  "data": {
    "id": "uuid",
    "invoiceNumber": "INV-20250604-0001",
    "subtotal": 70000.00,
    "discountTotal": 2000.00,
    "taxTotal": 7480.00,
    "grandTotal": 75480.00,
    "paidAmount": 50000.00,
    "changeAmount": 0,
    "status": "completed",
    "items": [ /* items with computed totals */ ],
    "payments": [ /* payment details */ ],
    "customer": { /* customer info */ },
    "createdAt": "2025-06-04T10:30:00Z"
  }
}
```

**Validation Rules (Sale):**
| Field | Rule |
|-------|------|
| branchId | Required, valid UUID, must be active |
| warehouseId | Required, valid UUID, must belong to branch |
| items | Required, min 1 item, max 500 items per transaction |
| items[].productId | Required, valid UUID, product must be active |
| items[].quantity | Required, > 0, must not exceed available stock |
| items[].unitPrice | Required, >= 0 |
| payments | Required, min 1 payment |
| payments[].amount | Required, sum must >= grand total |

---

### Payments Module

```
GET    /api/v1/payments               — List payments
GET    /api/v1/payments/:id           — Get payment detail
POST   /api/v1/payments/:id/refund    — Refund payment
```

---

### Purchase Orders Module

```
GET    /api/v1/purchase-orders                — List POs
POST   /api/v1/purchase-orders                — Create PO
GET    /api/v1/purchase-orders/:id            — Get PO with items
PATCH  /api/v1/purchase-orders/:id            — Update PO (draft only)
DELETE /api/v1/purchase-orders/:id            — Cancel PO
POST   /api/v1/purchase-orders/:id/submit     — Submit for approval
POST   /api/v1/purchase-orders/:id/approve    — Approve PO
POST   /api/v1/purchase-orders/:id/reject     — Reject PO
POST   /api/v1/purchase-orders/:id/receive    — Receive goods (partial/full)
```

**Receive Goods:**
```json
// POST /api/v1/purchase-orders/:id/receive
// Request
{
  "receivedDate": "2025-06-04",
  "items": [
    {
      "lineId": "uuid",
      "quantityReceived": 100,
      "batchNumber": "BATCH-JUN-001",
      "expiryDate": "2026-06-04",
      "unitCost": 3200.00
    }
  ],
  "notes": "All items received in good condition"
}
// Response
{
  "success": true,
  "data": {
    "poId": "uuid",
    "poNumber": "PO-2025-06-0001",
    "status": "partially_received",
    "receivedItems": 1,
    "inventoryUpdated": true,
    "newInventoryBalances": [ /* updated balances */ ]
  }
}
```

---

### Reports Module

```
GET    /api/v1/reports/sales/summary           — Daily/monthly sales summary
GET    /api/v1/reports/sales/detail            — Detailed sales report with filters
GET    /api/v1/reports/sales/by-product        — Sales by product
GET    /api/v1/reports/sales/by-category       — Sales by category
GET    /api/v1/reports/sales/by-customer       — Sales by customer
GET    /api/v1/reports/sales/by-cashier        — Sales by cashier
GET    /api/v1/reports/inventory/valuation     — Inventory valuation report
GET    /api/v1/reports/inventory/stock-movement — Stock movement report
GET    /api/v1/reports/inventory/low-stock     — Low stock report
GET    /api/v1/reports/profit-loss             — P&L summary
GET    /api/v1/reports/tax                     — Tax report
POST   /api/v1/reports/export                  — Export report to PDF/CSV/Excel
```

**Sales Summary:**
```json
// GET /api/v1/reports/sales/summary?filter[dateFrom]=2025-06-01&filter[dateTo]=2025-06-04&filter[branchId]=uuid
// Response
{
  "success": true,
  "data": {
    "period": { "from": "2025-06-01", "to": "2025-06-04" },
    "summary": {
      "totalSales": 125000000.00,
      "totalTransactions": 450,
      "averageTransactionValue": 277777.78,
      "totalDiscount": 5000000.00,
      "totalTax": 13750000.00,
      "totalCost": 87500000.00,
      "grossProfit": 37500000.00,
      "grossProfitMargin": 30.00
    },
    "byPaymentMethod": [
      { "method": "Cash", "amount": 62500000.00, "count": 250 },
      { "method": "QRIS", "amount": 37500000.00, "count": 150 },
      { "method": "Card", "amount": 25000000.00, "count": 50 }
    ],
    "topProducts": [
      { "productId": "uuid", "name": "Indomie Goreng", "quantity": 1250, "revenue": 5000000.00 }
    ]
  }
}
```

---

### Subscriptions Module

```
GET    /api/v1/plans                    — List available plans
GET    /api/v1/plans/:id                — Get plan details
GET    /api/v1/subscriptions            — Get current subscription
POST   /api/v1/subscriptions            — Create subscription (checkout)
PATCH  /api/v1/subscriptions/plan       — Change plan (upgrade/downgrade)
POST   /api/v1/subscriptions/cancel     — Cancel subscription
POST   /api/v1/subscriptions/reactivate — Reactivate canceled subscription
```

---

### Audit Logs Module

```
GET    /api/v1/audit-logs              — List audit logs (filterable)
GET    /api/v1/audit-logs/:id          — Get audit log detail
GET    /api/v1/audit-logs/export       — Export audit trail
```

```json
// GET /api/v1/audit-logs?filter[entityType]=product&filter[entityId]=uuid&limit=25
// Response
{
  "success": true,
  "data": [
    {
      "id": 1000001,
      "userId": "uuid",
      "userEmail": "admin@tokoku.com",
      "ipAddress": "192.168.1.100",
      "action": "UPDATE",
      "entityType": "product",
      "entityId": "uuid",
      "changedFields": ["price", "minStock"],
      "oldValues": { "price": 3500, "minStock": 5 },
      "newValues": { "price": 4000, "minStock": 10 },
      "createdAt": "2025-06-04T10:30:00Z"
    }
  ],
  "meta": { "cursor": 1000001, "limit": 25, "hasMore": true }
}
```

---

### Notifications Module

```
GET    /api/v1/notifications                    — List notifications
PATCH  /api/v1/notifications/:id/read           — Mark as read
POST   /api/v1/notifications/read-all           — Mark all as read
GET    /api/v1/notifications/unread-count       — Get unread count
```

---

### Branches Module

```
GET    /api/v1/branches                — List branches
POST   /api/v1/branches                — Create branch
PATCH  /api/v1/branches/:id            — Update branch
DELETE /api/v1/branches/:id            — Soft delete
```

---

### Shifts Module

```
POST   /api/v1/shifts                  — Open shift (with opening balance)
PATCH  /api/v1/shifts/:id/close        — Close shift (with closing balance)
GET    /api/v1/shifts                  — List shifts
GET    /api/v1/shifts/:id              — Get shift detail
GET    /api/v1/shifts/current          — Get current open shift for user
```

```json
// POST /api/v1/shifts
// Request
{
  "branchId": "uuid",
  "openingBalance": 500000.00
}
// Response
{
  "success": true,
  "data": {
    "id": "uuid",
    "branchId": "uuid",
    "cashierName": "Budi Santoso",
    "openedAt": "2025-06-04T07:00:00Z",
    "openingBalance": 500000.00,
    "status": "open"
  }
}

// PATCH /api/v1/shifts/:id/close
// Request
{
  "closingBalance": 1250000.00,
  "actualBalance": 1248000.00,
  "notes": "Short by IDR 2,000"
}
```

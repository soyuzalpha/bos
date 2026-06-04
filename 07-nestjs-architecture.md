# Phase 7: NestJS Architecture

## 7.1 Project Structure

```
src/
├── main.ts                              # Bootstrap application
├── app.module.ts                        # Root module with global imports
├── app.controller.ts                    # Health check, root endpoints
│
├── common/                              # Cross-cutting concerns
│   ├── constants/
│   │   ├── error-codes.constant.ts
│   │   ├── permissions.constant.ts
│   │   └── cache-keys.constant.ts
│   │
│   ├── decorators/
│   │   ├── current-user.decorator.ts        # @CurrentUser() parameter decorator
│   │   ├── public.decorator.ts              # @Public() — skip auth
│   │   ├── permissions.decorator.ts         # @Permissions('product.create')
│   │   ├── tenant.decorator.ts              # @Tenant() — extract tenant context
│   │   └── roles.decorator.ts               # @Roles('admin')
│   │
│   ├── dto/
│   │   ├── pagination.dto.ts                # PaginationParams
│   │   ├── api-response.dto.ts              # ApiResponse<T>, ApiError
│   │   └── filter.dto.ts                    # Generic filter DTOs
│   │
│   ├── filters/
│   │   ├── http-exception.filter.ts         # Global exception -> ApiError
│   │   └── prisma-exception.filter.ts       # Prisma errors -> ApiError
│   │
│   ├── guards/
│   │   ├── jwt-auth.guard.ts                # JWT verification
│   │   ├── tenant.guard.ts                  # Tenant validation + injection
│   │   ├── roles.guard.ts                   # Role-based access
│   │   └── permissions.guard.ts             # Permission-based access
│   │
│   ├── interceptors/
│   │   ├── tenant.interceptor.ts            # Extract + validate X-Tenant-Id
│   │   ├── audit.interceptor.ts             # Track audit-worthy requests
│   │   ├── logging.interceptor.ts           # Request/response logging
│   │   ├── transform.interceptor.ts         # Wrap in ApiResponse format
│   │   └── timeout.interceptor.ts           # Request timeout enforcement
│   │
│   ├── middleware/
│   │   ├── request-id.middleware.ts          # X-Request-Id generation
│   │   ├── cors.middleware.ts
│   │   └── helmet.middleware.ts
│   │
│   ├── pipes/
│   │   ├── uuid-validation.pipe.ts          # Validate UUID params
│   │   ├── parse-optional-uuid.pipe.ts
│   │   └── trim.pipe.ts
│   │
│   ├── interfaces/
│   │   ├── authenticated-request.interface.ts
│   │   ├── tenant-context.interface.ts
│   │   └── audit-options.interface.ts
│   │
│   └── helpers/
│       ├── slug.helper.ts
│       ├── hash.helper.ts
│       ├── date.helper.ts
│       └── number.helper.ts
│
├── config/                              # Application configuration
│   ├── config.module.ts                 # NestJS ConfigModule wrapper
│   ├── app.config.ts                    # App configuration schema
│   ├── database.config.ts               # Database configuration
│   ├── redis.config.ts                  # Redis configuration
│   ├── jwt.config.ts                    # JWT configuration
│   ├── queue.config.ts                  # BullMQ configuration
│   ├── storage.config.ts                # File storage configuration
│   └── cors.config.ts                   # CORS configuration
│
├── database/                            # Database layer
│   ├── prisma/
│   │   ├── prisma.module.ts             # Prisma provider
│   │   ├── prisma.service.ts            # Prisma client singleton + extensions
│   │   └── prisma-migration.service.ts
│   │
│   ├── seeds/
│   │   ├── seed.ts                      # Main seed runner
│   │   ├── seed-permissions.ts          # Seed all permissions
│   │   ├── seed-roles.ts                # Seed default roles
│   │   ├── seed-plans.ts                # Seed subscription plans
│   │   └── seed-demo-data.ts            # Tenant demo data
│   │
│   └── mixins/
│       ├── soft-delete.mixin.ts         # Prisma middleware for soft delete
│       ├── tenant-scope.mixin.ts        # Auto-append tenant_id to queries
│       └── audit-trail.mixin.ts         # Auto-create audit logs on mutations
│
├── modules/                             # === FEATURE MODULES ===
│   ├── auth/                            # Authentication module
│   │   ├── auth.module.ts
│   │   ├── auth.controller.ts
│   │   ├── auth.service.ts
│   │   ├── strategies/
│   │   │   ├── jwt.strategy.ts              # Passport JWT strategy
│   │   │   └── jwt-refresh.strategy.ts      # Refresh token strategy
│   │   ├── dto/
│   │   │   ├── login.dto.ts
│   │   │   ├── register.dto.ts
│   │   │   ├── refresh-token.dto.ts
│   │   │   ├── forgot-password.dto.ts
│   │   │   ├── reset-password.dto.ts
│   │   │   └── change-password.dto.ts
│   │   └── validators/
│   │       └── auth.validator.ts
│   │
│   ├── tenant/                             # Tenant management (superadmin)
│   │   ├── tenant.module.ts
│   │   ├── tenant.controller.ts
│   │   ├── tenant.service.ts
│   │   ├── tenant-registration.service.ts   # Onboarding workflow
│   │   ├── tenant-provisioning.service.ts   # Resource provisioning
│   │   ├── dto/
│   │   │   ├── create-tenant.dto.ts
│   │   │   ├── update-tenant.dto.ts
│   │   │   └── tenant-response.dto.ts
│   │   └── entities/
│   │       └── tenant.entity.ts
│   │
│   ├── user/                               # User management
│   │   ├── user.module.ts
│   │   ├── user.controller.ts
│   │   ├── user.service.ts
│   │   ├── dto/
│   │   │   ├── create-user.dto.ts
│   │   │   ├── update-user.dto.ts
│   │   │   ├── user-response.dto.ts
│   │   │   └── user-role.dto.ts
│   │   ├── entities/
│   │   │   └── user.entity.ts
│   │   └── user.mapper.ts
│   │
│   ├── role/                               # Role-based access control
│   │   ├── role.module.ts
│   │   ├── role.controller.ts
│   │   ├── role.service.ts
│   │   ├── dto/
│   │   │   ├── create-role.dto.ts
│   │   │   ├── update-role.dto.ts
│   │   │   ├── role-response.dto.ts
│   │   │   └── assign-permission.dto.ts
│   │   └── entities/
│   │       └── role.entity.ts
│   │
│   ├── permission/                         # Permission registry
│   │   ├── permission.module.ts
│   │   ├── permission.controller.ts
│   │   ├── permission.service.ts
│   │   └── dto/
│   │       └── permission-response.dto.ts
│   │
│   ├── product/                            # Product catalog
│   │   ├── product.module.ts
│   │   ├── product.controller.ts
│   │   ├── product.service.ts
│   │   ├── product-variant.service.ts
│   │   ├── product-price.service.ts
│   │   ├── dto/
│   │   │   ├── create-product.dto.ts
│   │   │   ├── update-product.dto.ts
│   │   │   ├── product-response.dto.ts
│   │   │   ├── product-list-query.dto.ts
│   │   │   ├── create-variant.dto.ts
│   │   │   ├── update-variant.dto.ts
│   │   │   ├── bulk-product.dto.ts
│   │   │   └── product-filter.dto.ts
│   │   ├── entities/
│   │   │   ├── product.entity.ts
│   │   │   └── product-variant.entity.ts
│   │   ├── mappers/
│   │   │   ├── product.mapper.ts
│   │   │   └── variant.mapper.ts
│   │   ├── validators/
│   │   │   └── product.validator.ts
│   │   ├── events/
│   │   │   ├── product-created.event.ts
│   │   │   ├── product-updated.event.ts
│   │   │   └── product-deleted.event.ts
│   │   └── listeners/
│   │       ├── product-search-index.listener.ts
│   │       └── product-cache-invalidate.listener.ts
│   │
│   ├── category/                           # Product categories (tree)
│   │   ├── category.module.ts
│   │   ├── category.controller.ts
│   │   ├── category.service.ts
│   │   ├── dto/
│   │   │   ├── create-category.dto.ts
│   │   │   ├── update-category.dto.ts
│   │   │   └── category-response.dto.ts
│   │   └── entities/
│   │       └── category.entity.ts
│   │
│   ├── brand/                              # Product brands
│   │   ├── brand.module.ts
│   │   ├── brand.controller.ts
│   │   ├── brand.service.ts
│   │   ├── dto/
│   │   │   ├── create-brand.dto.ts
│   │   │   ├── update-brand.dto.ts
│   │   │   └── brand-response.dto.ts
│   │   └── entities/
│   │       └── brand.entity.ts
│   │
│   ├── unit/                               # Units of measurement
│   │   ├── unit.module.ts
│   │   ├── unit.controller.ts
│   │   ├── unit.service.ts
│   │   ├── dto/
│   │   │   ├── create-unit.dto.ts
│   │   │   ├── update-unit.dto.ts
│   │   │   └── unit-response.dto.ts
│   │   └── entities/
│   │       └── unit.entity.ts
│   │
│   ├── customer/                           # Customer management
│   │   ├── customer.module.ts
│   │   ├── customer.controller.ts
│   │   ├── customer.service.ts
│   │   ├── customer-group.service.ts
│   │   ├── dto/
│   │   │   ├── create-customer.dto.ts
│   │   │   ├── update-customer.dto.ts
│   │   │   ├── customer-response.dto.ts
│   │   │   └── customer-filter.dto.ts
│   │   ├── entities/
│   │   │   └── customer.entity.ts
│   │   └── validators/
│   │       └── customer.validator.ts
│   │
│   ├── supplier/                           # Supplier management
│   │   ├── supplier.module.ts
│   │   ├── supplier.controller.ts
│   │   ├── supplier.service.ts
│   │   └── dto/
│   │       ├── create-supplier.dto.ts
│   │       ├── update-supplier.dto.ts
│   │       └── supplier-response.dto.ts
│   │
│   ├── warehouse/                          # Warehouse management
│   │   ├── warehouse.module.ts
│   │   ├── warehouse.controller.ts
│   │   ├── warehouse.service.ts
│   │   └── dto/
│   │       ├── create-warehouse.dto.ts
│   │       ├── update-warehouse.dto.ts
│   │       └── warehouse-response.dto.ts
│   │
│   ├── inventory/                          # Inventory management
│   │   ├── inventory.module.ts
│   │   ├── inventory.controller.ts
│   │   ├── inventory.service.ts
│   │   ├── inventory-balance.service.ts
│   │   ├── inventory-movement.service.ts
│   │   ├── inventory-adjustment.service.ts
│   │   ├── inventory-transfer.service.ts
│   │   ├── dto/
│   │   │   ├── inventory-balance-response.dto.ts
│   │   │   ├── stock-adjustment.dto.ts
│   │   │   ├── stock-transfer.dto.ts
│   │   │   ├── stock-write-off.dto.ts
│   │   │   └── inventory-movement-response.dto.ts
│   │   ├── entities/
│   │   │   ├── inventory-balance.entity.ts
│   │   │   └── inventory-movement.entity.ts
│   │   ├── events/
│   │   │   └── stock-adjusted.event.ts
│   │   ├── processors/
│   │   │   ├── cost-calculator.ts           # FIFO / Weighted Average
│   │   │   └── movement-processor.ts
│   │   └── jobs/
│   │       └── inventory-revaluation.job.ts
│   │
│   ├── sale/                               # THE CORE POS MODULE
│   │   ├── sale.module.ts
│   │   ├── sale.controller.ts
│   │   ├── sale.service.ts                  # Main entry point
│   │   ├── sale-processor.service.ts        # Orchestrates sale flow
│   │   ├── sale-cart.service.ts             # Cart management
│   │   ├── sale-hold.service.ts             # Hold/recall transactions
│   │   ├── sale-pricing.service.ts          # Pricing + discount engine
│   │   ├── sale-refund.service.ts           # Refund logic
│   │   ├── dto/
│   │   │   ├── create-sale.dto.ts
│   │   │   ├── create-sale-item.dto.ts
│   │   │   ├── create-payment.dto.ts
│   │   │   ├── sale-response.dto.ts
│   │   │   ├── sale-list-query.dto.ts
│   │   │   ├── hold-cart.dto.ts
│   │   │   └── refund.dto.ts
│   │   ├── entities/
│   │   │   ├── sale.entity.ts
│   │   │   ├── sale-item.entity.ts
│   │   │   └── sale-payment.entity.ts
│   │   ├── mappers/
│   │   │   ├── sale.mapper.ts
│   │   │   └── sale-item.mapper.ts
│   │   ├── validators/
│   │   │   ├── sale.validator.ts            # Business rules
│   │   │   └── payment.validator.ts
│   │   ├── events/
│   │   │   ├── sale-completed.event.ts
│   │   │   ├── sale-voided.event.ts
│   │   │   └── sale-refunded.event.ts
│   │   └── jobs/
│   │       └── sale-invoice-generation.job.ts
│   │
│   ├── payment/                            # Payment processing
│   │   ├── payment.module.ts
│   │   ├── payment.controller.ts
│   │   ├── payment.service.ts
│   │   ├── payment-method.service.ts
│   │   ├── gateways/
│   │   │   ├── payment-gateway.interface.ts
│   │   │   ├── cash-payment.strategy.ts
│   │   │   ├── card-payment.strategy.ts
│   │   │   ├── qris-payment.strategy.ts
│   │   │   └── ewallet-payment.strategy.ts
│   │   └── dto/
│   │       ├── process-payment.dto.ts
│   │       └── payment-response.dto.ts
│   │
│   ├── purchase-order/                     # Purchase order management
│   │   ├── purchase-order.module.ts
│   │   ├── purchase-order.controller.ts
│   │   ├── purchase-order.service.ts
│   │   ├── purchase-receiving.service.ts
│   │   ├── dto/
│   │   │   ├── create-purchase-order.dto.ts
│   │   │   ├── update-purchase-order.dto.ts
│   │   │   ├── purchase-order-response.dto.ts
│   │   │   ├── receive-goods.dto.ts
│   │   │   └── approve-purchase-order.dto.ts
│   │   ├── entities/
│   │   │   └── purchase-order.entity.ts
│   │   ├── events/
│   │   │   ├── po-created.event.ts
│   │   │   ├── po-approved.event.ts
│   │   │   └── po-received.event.ts
│   │   └── jobs/
│   │       └── po-overdue-check.job.ts
│   │
│   ├── report/                             # Reporting engine
│   │   ├── report.module.ts
│   │   ├── report.controller.ts
│   │   ├── services/
│   │   │   ├── sales-report.service.ts
│   │   │   ├── inventory-report.service.ts
│   │   │   ├── profit-loss-report.service.ts
│   │   │   ├── tax-report.service.ts
│   │   │   └── export.service.ts
│   │   ├── dto/
│   │   │   ├── sales-summary-query.dto.ts
│   │   │   ├── report-response.dto.ts
│   │   │   └── export-request.dto.ts
│   │   └── generators/
│   │       ├── pdf-generator.ts
│   │       ├── csv-generator.ts
│   │       └── excel-generator.ts
│   │
│   ├── subscription/                       # Billing & plans
│   │   ├── subscription.module.ts
│   │   ├── subscription.controller.ts
│   │   ├── subscription.service.ts
│   │   ├── plan.service.ts
│   │   ├── billing.service.ts
│   │   ├── entitlement.service.ts           # Feature flag checks
│   │   ├── dto/
│   │   │   ├── create-subscription.dto.ts
│   │   │   ├── plan-response.dto.ts
│   │   │   └── subscription-response.dto.ts
│   │   ├── webhooks/
│   │   │   └── stripe-webhook.controller.ts
│   │   └── jobs/
│   │       ├── subscription-renewal.job.ts
│   │       ├── subscription-expiry.job.ts
│   │       └── usage-metering.job.ts
│   │
│   ├── audit-log/                          # Immutable audit trail
│   │   ├── audit-log.module.ts
│   │   ├── audit-log.controller.ts
│   │   ├── audit-log.service.ts
│   │   ├── audit-log.middleware.ts          # Auto-creation on mutations
│   │   ├── dto/
│   │   │   ├── audit-log-response.dto.ts
│   │   │   └── audit-log-query.dto.ts
│   │   └── entities/
│   │       └── audit-log.entity.ts
│   │
│   ├── notification/                       # Notification channels
│   │   ├── notification.module.ts
│   │   ├── notification.controller.ts
│   │   ├── notification.service.ts
│   │   ├── channels/
│   │   │   ├── notification-channel.interface.ts
│   │   │   ├── in-app-notification.channel.ts
│   │   │   ├── email-notification.channel.ts
│   │   │   ├── whatsapp-notification.channel.ts
│   │   │   └── sms-notification.channel.ts
│   │   ├── templates/
│   │   │   ├── low-stock.template.ts
│   │   │   ├── po-approved.template.ts
│   │   │   └── invoice.template.ts
│   │   ├── dto/
│   │   │   ├── notification-response.dto.ts
│   │   │   └── mark-read.dto.ts
│   │   └── jobs/
│   │       └── send-notification.job.ts
│   │
│   ├── branch/                             # Branch management
│   │   ├── branch.module.ts
│   │   ├── branch.controller.ts
│   │   ├── branch.service.ts
│   │   └── dto/
│   │       ├── create-branch.dto.ts
│   │       ├── update-branch.dto.ts
│   │       └── branch-response.dto.ts
│   │
│   └── shift/                              # Cashier shift management
│       ├── shift.module.ts
│       ├── shift.controller.ts
│       ├── shift.service.ts
│       └── dto/
│           ├── open-shift.dto.ts
│           ├── close-shift.dto.ts
│           └── shift-response.dto.ts
│
├── queues/                                 # BullMQ queue definitions
│   ├── queue.module.ts
│   ├── queue.service.ts                    # Queue manager
│   ├── notification.queue.ts
│   ├── report.queue.ts
│   ├── email.queue.ts
│   ├── invoice.queue.ts
│   └── audit.queue.ts
│
├── jobs/                                   # Job processors
│   ├── notification-job.processor.ts
│   ├── report-job.processor.ts
│   ├── invoice-job.processor.ts
│   └── audit-cleanup-job.processor.ts
│
├── events/                                 # Event bus definitions
│   ├── event-bus.module.ts
│   └── hermes.event.ts                    # Base event class
│
├── shared/                                 # Cross-cutting utilities
│   ├── cache/
│   │   ├── cache.module.ts
│   │   ├── cache.service.ts                # Redis-backed cache abstraction
│   │   └── cache.decorator.ts              # @Cacheable decorator
│   │
│   ├── storage/
│   │   ├── storage.module.ts
│   │   ├── storage.service.ts               # Local/S3 abstraction
│   │   ├── local-storage.provider.ts
│   │   └── s3-storage.provider.ts
│   │
│   ├── search/
│   │   ├── search.module.ts
│   │   └── search.service.ts               # Full-text search abstraction
│   │
│   ├── pdf/
│   │   ├── pdf.module.ts
│   │   └── pdf.service.ts                  # PDF generation (Puppeteer/PDFKit)
│   │
│   ├── excel/
│   │   ├── excel.module.ts
│   │   └── excel.service.ts                # Excel generation (ExcelJS)
│   │
│   └── validators/
│       ├── validator.module.ts
│       └── constraint.validator.ts         # Custom class-validator decorators
│
└── templates/                              # Email/PDF templates
    ├── email/
    │   ├── invoice.hbs
    │   ├── password-reset.hbs
    │   └── low-stock-alert.hbs
    └── pdf/
        ├── invoice.hbs
        └── sales-report.hbs
```

---

## 7.2 Module Responsibilities

| Component | Responsibility |
|-----------|---------------|
| **Controller** | HTTP routing, request validation, response formatting. Thin — delegates to service. |
| **Service** | Business logic. Orchestrates repository calls, event emission, queue dispatch. One service per aggregate root. |
| **Repository** | Direct Prisma access. Encapsulates all database queries. Returns domain entities or DTOs. |
| **DTO** | Data Transfer Objects for request/response. Uses `class-validator` decorators for validation. |
| **Entity** | Domain model representation. May contain computed fields or domain methods. |
| **Mapper** | Transforms between Prisma models, domain entities, and DTOs. Keeps layers decoupled. |
| **Validator** | Business rule validation that cannot be expressed in DTO decorators. |
| **Events** | Domain events emitted after state changes. Consumed by listeners (same process) or queue jobs (async). |
| **Jobs** | BullMQ job definitions for async processing. |

---

## 7.3 Module Dependency Graph

```
app.module.ts
├── ConfigModule         (global)
├── PrismaModule         (global)
├── CacheModule          (global)
├── QueueModule          (global)
├── EventBusModule       (global)
│
├── AuthModule           ─── depends on: UserModule, TenantModule
├── TenantModule         ─── standalone (superadmin)
├── UserModule           ─── depends on: RoleModule, BranchModule
├── RoleModule           ─── depends on: PermissionModule
├── PermissionModule     ─── standalone
│
├── ProductModule        ─── depends on: CategoryModule, BrandModule, UnitModule
├── CategoryModule       ─── standalone
├── BrandModule          ─── standalone
├── UnitModule           ─── standalone
│
├── CustomerModule       ─── depends on: CustomerGroupModule
├── SupplierModule       ─── standalone
│
├── InventoryModule      ─── depends on: WarehouseModule, ProductModule
├── WarehouseModule      ─── depends on: BranchModule
│
├── SaleModule           ─── depends on: ProductModule, InventoryModule, CustomerModule, PaymentModule, ShiftModule
├── PaymentModule        ─── depends on: SaleModule
├── ShiftModule          ─── depends on: BranchModule, UserModule
│
├── PurchaseOrderModule  ─── depends on: SupplierModule, ProductModule, InventoryModule
│
├── ReportModule         ─── depends on: SaleModule, InventoryModule, PurchaseOrderModule
│
├── BranchModule         ─── depends on: TenantModule
│
├── SubscriptionModule   ─── depends on: PlanModule, TenantModule
├── PlanModule           ─── standalone
│
├── AuditLogModule       ─── global middleware, depends on: TenantModule
├── NotificationModule   ─── global, depends on: UserModule
│
└── SharedModule         ─── Cache, Storage, Search, PDF, Excel
```

---

## 7.4 Key Architectural Decisions

| Decision | Choice | Rationale |
|----------|--------|-----------|
| **Monolith vs Microservices** | Modular Monolith | Faster to build, deploy, and debug. Clearly bounded modules with well-defined interfaces. Extract to microservices only when performance demands it (e.g., report generation). |
| **CQRS** | Tactical (not full Event Sourcing) | Separate read queries from write commands within the same module. Write models validate business rules; read models optimize for display. Full Event Sourcing adds complexity that isn't justified for a POS system. |
| **Prisma Middleware** | Soft delete, tenant scope, audit | Global Prisma middleware ensures every query respects `deletedAt` and `tenantId`. No developer can accidentally bypass. |
| **Validation** | Request DTOs + Business Validators | DTOs handle type/structure validation. Separate validator classes handle business rules that require database lookups or cross-entity checks. |
| **File Uploads** | Local dev / S3 prod | Abstracted behind `StorageService`. Switch provider without changing business logic. |
| **PDF Generation** | Puppeteer (HTML→PDF) | More flexible than PDFKit. Use Handlebars templates. Queue PDF generation to avoid blocking HTTP responses. |
| **Background Jobs** | BullMQ + Redis | Report generation, email sending, invoice PDF, audit cleanup, subscription renewal. All async. |

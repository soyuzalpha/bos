# Phase 4: Prisma Schema

> Complete, production-ready Prisma schema for PostgreSQL.
> Location: `prisma/schema.prisma`

```prisma
// prisma/schema.prisma

generator client {
  provider        = "prisma-client-js"
  previewFeatures = ["postgresqlExtensions", "fullTextSearch", "extendedIndexes"]
}

datasource db {
  provider   = "postgresql"
  url        = env("DATABASE_URL")
  extensions = [uuid_ossp, pg_trgm]
}

// ============================================================
// ENUMS
// ============================================================

enum ProductType {
  simple
  variant
  composite
  service
}

enum WarehouseType {
  main
  backroom
  display
  damaged
  transit
}

enum InventoryMovementType {
  purchase_receipt
  sale
  sale_return
  purchase_return
  transfer_out
  transfer_in
  adjustment
  write_off
  opening_balance
}

enum SaleStatus {
  draft
  completed
  voided
  refunded
}

enum PurchaseOrderStatus {
  draft
  pending_approval
  approved
  ordered
  partially_received
  completed
  cancelled
}

enum PaymentStatus {
  pending
  completed
  failed
  refunded
}

enum PaymentMethodType {
  cash
  card
  e_wallet
  qris
  transfer
  credit
}

enum SubscriptionStatus {
  active
  trialing
  past_due
  canceled
  expired
  suspended
}

enum NotificationChannel {
  in_app
  email
  whatsapp
  sms
}

enum ShiftStatus {
  open
  closed
  verified
}

enum UnitCategory {
  unit
  weight
  volume
  length
  area
  time
}

enum PriceListType {
  retail
  wholesale
  special
  membership
}

enum AuditAction {
  CREATE
  UPDATE
  DELETE
  LOGIN
  LOGOUT
  FAILED_LOGIN
  EXPORT
  RESTORE
  VOID
}

// ============================================================
// MODELS
// ============================================================

model Tenant {
  id              String    @id @default(uuid()) @db.Uuid
  name            String    @db.VarChar(255)
  slug            String    @unique @db.VarChar(100)
  domain          String?   @unique @db.VarChar(255)
  logoUrl         String?   @map("logo_url") @db.Text
  address         String?   @db.Text
  phone           String?   @db.VarChar(50)
  email           String?   @db.VarChar(255)
  taxId           String?   @map("tax_id") @db.VarChar(100)
  currencyCode    String    @default("IDR") @map("currency_code") @db.VarChar(3)
  timezone        String    @default("Asia/Jakarta") @db.VarChar(50)
  locale          String    @default("id-ID") @db.VarChar(10)
  dateFormat      String    @default("DD/MM/YYYY") @map("date_format") @db.VarChar(20)
  isActive        Boolean   @default(true) @map("is_active")
  subscriptionId  String?   @map("subscription_id") @db.Uuid
  settings        Json      @default("{}")
  createdAt       DateTime  @default(now()) @map("created_at")
  updatedAt       DateTime  @updatedAt @map("updated_at")
  deletedAt       DateTime? @map("deleted_at")

  // Relations
  subscription           Subscription?        @relation(fields: [subscriptionId], references: [id])
  users                  User[]
  branches               Branch[]
  roles                  Role[]
  permissions            Permission[]
  categories             Category[]
  brands                 Brand[]
  units                  Unit[]
  unitConversions        UnitConversion[]
  products               Product[]
  productVariants        ProductVariant[]
  productPrices          ProductPrice[]
  priceLists             PriceList[]
  customers              Customer[]
  customerGroups         CustomerGroup[]
  suppliers              Supplier[]
  productSuppliers       ProductSupplier[]
  warehouses             Warehouse[]
  inventoryBalances      InventoryBalance[]
  inventoryMovements     InventoryMovement[]
  sales                  Sale[]
  saleItems              SaleItem[]
  payments               Payment[]
  paymentMethods         PaymentMethod[]
  purchaseOrders         PurchaseOrder[]
  purchaseOrderItems     PurchaseOrderItem[]
  auditLogs              AuditLog[]
  notifications          Notification[]
  shifts                 Shift[]
  userRoles              UserRole[]
  plans                  Plan[]

  @@map("tenants")
}

model User {
  id                   String   @id @default(uuid()) @db.Uuid
  tenantId             String   @map("tenant_id") @db.Uuid
  branchId             String?  @map("branch_id") @db.Uuid
  email                String   @db.VarChar(255)
  phone                String?  @db.VarChar(50)
  passwordHash         String   @map("password_hash") @db.VarChar(255)
  firstName            String   @map("first_name") @db.VarChar(100)
  lastName             String?  @map("last_name") @db.VarChar(100)
  isActive             Boolean  @default(true) @map("is_active")
  mustChangePassword   Boolean  @default(false) @map("must_change_password")
  lastLoginAt          DateTime? @map("last_login_at")
  failedLoginAttempts  Int      @default(0) @map("failed_login_attempts")
  lockedUntil          DateTime? @map("locked_until")
  refreshTokenHash     String?  @map("refresh_token_hash") @db.VarChar(255)
  mfaSecret            String?  @map("mfa_secret") @db.VarChar(255)
  mfaEnabled           Boolean  @default(false) @map("mfa_enabled")
  settings             Json     @default("{}")
  createdAt            DateTime @default(now()) @map("created_at")
  updatedAt            DateTime @updatedAt @map("updated_at")
  deletedAt            DateTime? @map("deleted_at")

  // Relations
  tenant              Tenant          @relation(fields: [tenantId], references: [id])
  branch              Branch?         @relation(fields: [branchId], references: [id])
  userRoles           UserRole[]
  sales               Sale[]
  shifts              Shift[]
  purchaseOrdersReq   PurchaseOrder[] @relation("requestedBy")
  purchaseOrdersApp   PurchaseOrder[] @relation("approvedBy")
  voidedSales         Sale[]          @relation("voidedBy")
  verifiedShifts      Shift[]         @relation("verifiedBy")
  auditLogs           AuditLog[]
  notifications       Notification[]

  @@unique([tenantId, email])
  @@unique([tenantId, phone])
  @@index([tenantId, email], name: "idx_users_tenant_email")
  @@index([tenantId, phone], name: "idx_users_tenant_phone")
  @@map("users")
}

model Role {
  id          String   @id @default(uuid()) @db.Uuid
  tenantId    String   @map("tenant_id") @db.Uuid
  name        String   @db.VarChar(100)
  slug        String   @db.VarChar(100)
  description String?  @db.Text
  isSystem    Boolean  @default(false) @map("is_system")
  isDefault   Boolean  @default(false) @map("is_default")
  createdAt   DateTime @default(now()) @map("created_at")
  updatedAt   DateTime @updatedAt @map("updated_at")
  deletedAt   DateTime? @map("deleted_at")

  tenant          Tenant           @relation(fields: [tenantId], references: [id])
  rolePermissions RolePermission[]
  userRoles       UserRole[]

  @@unique([tenantId, slug])
  @@map("roles")
}

model Permission {
  id          String   @id @default(uuid()) @db.Uuid
  tenantId    String   @map("tenant_id") @db.Uuid
  name        String   @db.VarChar(100)
  slug        String   @db.VarChar(100)
  module      String   @db.VarChar(50)
  action      String   @db.VarChar(50)
  description String?  @db.Text
  isSystem    Boolean  @default(true) @map("is_system")
  createdAt   DateTime @default(now()) @map("created_at")
  updatedAt   DateTime @updatedAt @map("updated_at")

  tenant          Tenant           @relation(fields: [tenantId], references: [id])
  rolePermissions RolePermission[]

  @@unique([tenantId, slug])
  @@index([tenantId, module])
  @@map("permissions")
}

model RolePermission {
  id           String   @id @default(uuid()) @db.Uuid
  roleId       String   @map("role_id") @db.Uuid
  permissionId String   @map("permission_id") @db.Uuid
  createdAt    DateTime @default(now()) @map("created_at")

  role       Role       @relation(fields: [roleId], references: [id], onDelete: Cascade)
  permission Permission @relation(fields: [permissionId], references: [id], onDelete: Cascade)

  @@unique([roleId, permissionId])
  @@map("role_permissions")
}

model UserRole {
  id        String   @id @default(uuid()) @db.Uuid
  userId    String   @map("user_id") @db.Uuid
  roleId    String   @map("role_id") @db.Uuid
  branchId  String?  @map("branch_id") @db.Uuid
  createdAt DateTime @default(now()) @map("created_at")

  user   User    @relation(fields: [userId], references: [id], onDelete: Cascade)
  role   Role    @relation(fields: [roleId], references: [id], onDelete: Cascade)
  branch Branch? @relation(fields: [branchId], references: [id])

  @@unique([userId, roleId, branchId])
  @@map("user_roles")
}

model Branch {
  id           String   @id @default(uuid()) @db.Uuid
  tenantId     String   @map("tenant_id") @db.Uuid
  code         String   @db.VarChar(50)
  name         String   @db.VarChar(255)
  address      String?  @db.Text
  city         String?  @db.VarChar(100)
  province     String?  @db.VarChar(100)
  postalCode   String?  @map("postal_code") @db.VarChar(20)
  phone        String?  @db.VarChar(50)
  email        String?  @db.VarChar(255)
  taxId        String?  @map("tax_id") @db.VarChar(100)
  isActive     Boolean  @default(true) @map("is_active")
  isHeadOffice Boolean  @default(false) @map("is_head_office")
  openingTime  DateTime? @map("opening_time") @db.Time
  closingTime  DateTime? @map("closing_time") @db.Time
  settings     Json     @default("{}")
  createdAt    DateTime @default(now()) @map("created_at")
  updatedAt    DateTime @updatedAt @map("updated_at")
  deletedAt    DateTime? @map("deleted_at")

  tenant     Tenant   @relation(fields: [tenantId], references: [id])
  users      User[]
  warehouses Warehouse[]
  sales      Sale[]
  shifts     Shift[]
  userRoles  UserRole[]
  purchaseOrders PurchaseOrder[]

  @@unique([tenantId, code])
  @@map("branches")
}

model Warehouse {
  id        String        @id @default(uuid()) @db.Uuid
  tenantId  String        @map("tenant_id") @db.Uuid
  branchId  String        @map("branch_id") @db.Uuid
  code      String        @db.VarChar(50)
  name      String        @db.VarChar(255)
  type      WarehouseType
  address   String?       @db.Text
  isActive  Boolean       @default(true) @map("is_active")
  isDefault Boolean       @default(false) @map("is_default")
  createdAt DateTime      @default(now()) @map("created_at")
  updatedAt DateTime      @updatedAt @map("updated_at")
  deletedAt DateTime?     @map("deleted_at")

  tenant            Tenant              @relation(fields: [tenantId], references: [id])
  branch            Branch              @relation(fields: [branchId], references: [id])
  inventoryBalances  InventoryBalance[]
  inventoryMovements InventoryMovement[]
  sales             Sale[]
  purchaseOrders    PurchaseOrder[]
  shifts            Shift?[]

  @@unique([tenantId, code])
  @@map("warehouses")
}

model Category {
  id          String    @id @default(uuid()) @db.Uuid
  tenantId    String    @map("tenant_id") @db.Uuid
  parentId    String?   @map("parent_id") @db.Uuid
  name        String    @db.VarChar(255)
  slug        String    @db.VarChar(255)
  description String?   @db.Text
  sortOrder   Int       @default(0) @map("sort_order")
  isActive    Boolean   @default(true) @map("is_active")
  createdAt   DateTime  @default(now()) @map("created_at")
  updatedAt   DateTime  @updatedAt @map("updated_at")
  deletedAt   DateTime? @map("deleted_at")

  tenant   Tenant    @relation(fields: [tenantId], references: [id])
  parent   Category? @relation("CategoryHierarchy", fields: [parentId], references: [id])
  children Category[] @relation("CategoryHierarchy")
  products Product[]

  @@unique([tenantId, slug])
  @@index([tenantId, parentId])
  @@map("categories")
}

model Brand {
  id          String    @id @default(uuid()) @db.Uuid
  tenantId    String    @map("tenant_id") @db.Uuid
  name        String    @db.VarChar(255)
  slug        String    @db.VarChar(255)
  description String?   @db.Text
  logoUrl     String?   @map("logo_url") @db.Text
  isActive    Boolean   @default(true) @map("is_active")
  createdAt   DateTime  @default(now()) @map("created_at")
  updatedAt   DateTime  @updatedAt @map("updated_at")
  deletedAt   DateTime? @map("deleted_at")

  tenant   Tenant    @relation(fields: [tenantId], references: [id])
  products Product[]

  @@unique([tenantId, slug])
  @@map("brands")
}

model Unit {
  id         String       @id @default(uuid()) @db.Uuid
  tenantId   String       @map("tenant_id") @db.Uuid
  name       String       @db.VarChar(100)
  shortName  String       @map("short_name") @db.VarChar(20)
  category   UnitCategory
  isBaseUnit Boolean      @default(false) @map("is_base_unit")
  isActive   Boolean      @default(true) @map("is_active")
  createdAt  DateTime     @default(now()) @map("created_at")
  updatedAt  DateTime     @updatedAt @map("updated_at")
  deletedAt  DateTime?    @map("deleted_at")

  tenant           Tenant            @relation(fields: [tenantId], references: [id])
  products         Product[]         @relation("sellingUnit")
  purchaseProducts Product[]         @relation("purchaseUnit")
  fromConversions  UnitConversion[]  @relation("fromUnit")
  toConversions    UnitConversion[]  @relation("toUnit")

  @@map("units")
}

model UnitConversion {
  id         String @id @default(uuid()) @db.Uuid
  tenantId   String @map("tenant_id") @db.Uuid
  fromUnitId String @map("from_unit_id") @db.Uuid
  toUnitId   String @map("to_unit_id") @db.Uuid
  factor     Decimal @db.Decimal(20, 6)
  createdAt  DateTime @default(now()) @map("created_at")

  tenant   Tenant @relation(fields: [tenantId], references: [id])
  fromUnit Unit   @relation("fromUnit", fields: [fromUnitId], references: [id])
  toUnit   Unit   @relation("toUnit", fields: [toUnitId], references: [id])

  @@unique([fromUnitId, toUnitId])
  @@map("unit_conversions")
}

model Product {
  id              String      @id @default(uuid()) @db.Uuid
  tenantId        String      @map("tenant_id") @db.Uuid
  categoryId      String?     @map("category_id") @db.Uuid
  brandId         String?     @map("brand_id") @db.Uuid
  unitId          String      @map("unit_id") @db.Uuid
  purchaseUnitId  String?     @map("purchase_unit_id") @db.Uuid
  code            String      @db.VarChar(100)
  barcode         String?     @db.VarChar(100)
  name            String      @db.VarChar(255)
  description     String?     @db.Text
  type            ProductType
  trackSerial     Boolean     @default(false) @map("track_serial")
  trackBatch      Boolean     @default(false) @map("track_batch")
  hasExpiry       Boolean     @default(false) @map("has_expiry")
  isActive        Boolean     @default(true) @map("is_active")
  taxInclusive    Boolean     @default(true) @map("tax_inclusive")
  taxRate         Decimal     @default(0) @map("tax_rate") @db.Decimal(5, 2)
  minStock        Decimal     @default(0) @map("min_stock") @db.Decimal(20, 2)
  maxStock        Decimal?    @map("max_stock") @db.Decimal(20, 2)
  weight          Decimal?    @db.Decimal(12, 4)
  tags            String[]    @default([])
  imageUrl        String?     @map("image_url") @db.Text
  createdAt       DateTime    @default(now()) @map("created_at")
  updatedAt       DateTime    @updatedAt @map("updated_at")
  deletedAt       DateTime?   @map("deleted_at")

  tenant            Tenant              @relation(fields: [tenantId], references: [id])
  category          Category?           @relation(fields: [categoryId], references: [id])
  brand             Brand?              @relation(fields: [brandId], references: [id])
  unit              Unit                @relation("sellingUnit", fields: [unitId], references: [id])
  purchaseUnit      Unit?               @relation("purchaseUnit", fields: [purchaseUnitId], references: [id])
  variants          ProductVariant[]
  prices            ProductPrice[]
  suppliers         ProductSupplier[]
  inventoryBalances InventoryBalance[]
  inventoryMovements InventoryMovement[]
  saleItems         SaleItem[]
  purchaseOrderItems PurchaseOrderItem[]

  @@unique([tenantId, code])
  @@unique([tenantId, barcode])
  @@index([tenantId, categoryId])
  @@index([tenantId, brandId])
  @@index([tenantId, isActive])
  @@index([tenantId, name])
  @@map("products")
}

model ProductVariant {
  id         String   @id @default(uuid()) @db.Uuid
  tenantId   String   @map("tenant_id") @db.Uuid
  productId  String   @map("product_id") @db.Uuid
  code       String   @db.VarChar(100)
  barcode    String?  @db.VarChar(100)
  name       String   @db.VarChar(255)
  attributes Json
  price      Decimal? @db.Decimal(20, 2)
  costPrice  Decimal? @map("cost_price") @db.Decimal(20, 2)
  isActive   Boolean  @default(true) @map("is_active")
  sortOrder  Int      @default(0) @map("sort_order")
  createdAt  DateTime @default(now()) @map("created_at")
  updatedAt  DateTime @updatedAt @map("updated_at")
  deletedAt  DateTime? @map("deleted_at")

  tenant            Tenant              @relation(fields: [tenantId], references: [id])
  product           Product             @relation(fields: [productId], references: [id], onDelete: Cascade)
  prices            ProductPrice[]
  inventoryBalances InventoryBalance[]
  inventoryMovements InventoryMovement[]
  saleItems         SaleItem[]
  purchaseOrderItems PurchaseOrderItem[]

  @@unique([tenantId, code])
  @@index([productId])
  @@map("product_variants")
}

model PriceList {
  id                String       @id @default(uuid()) @db.Uuid
  tenantId          String       @map("tenant_id") @db.Uuid
  name              String       @db.VarChar(100)
  slug              String       @db.VarChar(100)
  type              PriceListType
  isDefault         Boolean      @default(false) @map("is_default")
  markupPercentage  Decimal?     @map("markup_percentage") @db.Decimal(10, 2)
  createdAt         DateTime     @default(now()) @map("created_at")
  updatedAt         DateTime     @updatedAt @map("updated_at")
  deletedAt         DateTime?    @map("deleted_at")

  tenant   Tenant        @relation(fields: [tenantId], references: [id])
  prices   ProductPrice[]

  @@unique([tenantId, slug])
  @@map("price_lists")
}

model ProductPrice {
  id          String  @id @default(uuid()) @db.Uuid
  tenantId    String  @map("tenant_id") @db.Uuid
  productId   String  @map("product_id") @db.Uuid
  variantId   String? @map("variant_id") @db.Uuid
  priceListId String  @map("price_list_id") @db.Uuid
  price       Decimal @db.Decimal(20, 2)
  minQty      Int     @default(1) @map("min_qty")
  createdAt   DateTime @default(now()) @map("created_at")
  updatedAt   DateTime @updatedAt @map("updated_at")

  tenant    Tenant         @relation(fields: [tenantId], references: [id])
  product   Product        @relation(fields: [productId], references: [id], onDelete: Cascade)
  variant   ProductVariant? @relation(fields: [variantId], references: [id], onDelete: Cascade)
  priceList PriceList       @relation(fields: [priceListId], references: [id])

  @@unique([productId, variantId, priceListId, minQty])
  @@map("product_prices")
}

model CustomerGroup {
  id                  String   @id @default(uuid()) @db.Uuid
  tenantId            String   @map("tenant_id") @db.Uuid
  name                String   @db.VarChar(100)
  slug                String   @db.VarChar(100)
  discountPercentage  Decimal  @default(0) @map("discount_percentage") @db.Decimal(5, 2)
  createdAt           DateTime @default(now()) @map("created_at")
  updatedAt           DateTime @updatedAt @map("updated_at")

  tenant    Tenant    @relation(fields: [tenantId], references: [id])
  customers Customer[]

  @@unique([tenantId, slug])
  @@map("customer_groups")
}

model Customer {
  id              String    @id @default(uuid()) @db.Uuid
  tenantId        String    @map("tenant_id") @db.Uuid
  groupId         String?   @map("group_id") @db.Uuid
  branchId        String?   @map("branch_id") @db.Uuid
  code            String    @db.VarChar(50)
  name            String    @db.VarChar(255)
  email           String?   @db.VarChar(255)
  phone           String?   @db.VarChar(50)
  whatsapp        String?   @db.VarChar(50)
  address         String?   @db.Text
  city            String?   @db.VarChar(100)
  province        String?   @db.VarChar(100)
  postalCode      String?   @map("postal_code") @db.VarChar(20)
  taxId           String?   @map("tax_id") @db.VarChar(100)
  creditLimit     Decimal   @default(0) @map("credit_limit") @db.Decimal(20, 2)
  totalPurchases  Decimal   @default(0) @map("total_purchases") @db.Decimal(20, 2)
  loyaltyPoints   Int       @default(0) @map("loyalty_points")
  birthDate       DateTime? @map("birth_date") @db.Date
  notes           String?   @db.Text
  isActive        Boolean   @default(true) @map("is_active")
  createdAt       DateTime  @default(now()) @map("created_at")
  updatedAt       DateTime  @updatedAt @map("updated_at")
  deletedAt       DateTime? @map("deleted_at")

  tenant   Tenant        @relation(fields: [tenantId], references: [id])
  group    CustomerGroup? @relation(fields: [groupId], references: [id])
  branch   Branch?       @relation(fields: [branchId], references: [id])
  sales    Sale[]

  @@unique([tenantId, code])
  @@unique([tenantId, email])
  @@index([tenantId, name])
  @@index([tenantId, phone])
  @@map("customers")
}

model Supplier {
  id              String    @id @default(uuid()) @db.Uuid
  tenantId        String    @map("tenant_id") @db.Uuid
  code            String    @db.VarChar(50)
  name            String    @db.VarChar(255)
  contactPerson   String?   @map("contact_person") @db.VarChar(255)
  email           String?   @db.VarChar(255)
  phone           String?   @db.VarChar(50)
  whatsapp        String?   @db.VarChar(50)
  address         String?   @db.Text
  city            String?   @db.VarChar(100)
  province        String?   @db.VarChar(100)
  taxId           String?   @map("tax_id") @db.VarChar(100)
  paymentTerms    String?   @map("payment_terms") @db.VarChar(100)
  leadTimeDays    Int?      @map("lead_time_days")
  isActive        Boolean   @default(true) @map("is_active")
  createdAt       DateTime  @default(now()) @map("created_at")
  updatedAt       DateTime  @updatedAt @map("updated_at")
  deletedAt       DateTime? @map("deleted_at")

  tenant          Tenant            @relation(fields: [tenantId], references: [id])
  products        ProductSupplier[]
  purchaseOrders  PurchaseOrder[]

  @@unique([tenantId, code])
  @@map("suppliers")
}

model ProductSupplier {
  id           String   @id @default(uuid()) @db.Uuid
  tenantId     String   @map("tenant_id") @db.Uuid
  productId    String   @map("product_id") @db.Uuid
  supplierId   String   @map("supplier_id") @db.Uuid
  supplierCode String?  @map("supplier_code") @db.VarChar(100)
  costPrice    Decimal? @map("cost_price") @db.Decimal(20, 2)
  isPreferred  Boolean  @default(false) @map("is_preferred")
  minOrderQty  Decimal? @map("min_order_qty") @db.Decimal(20, 2)
  leadTimeDays Int?     @map("lead_time_days")
  createdAt    DateTime @default(now()) @map("created_at")

  tenant   Tenant   @relation(fields: [tenantId], references: [id])
  product  Product  @relation(fields: [productId], references: [id], onDelete: Cascade)
  supplier Supplier @relation(fields: [supplierId], references: [id], onDelete: Cascade)

  @@unique([productId, supplierId])
  @@map("product_suppliers")
}

model InventoryBalance {
  id                String   @id @default(uuid()) @db.Uuid
  tenantId          String   @map("tenant_id") @db.Uuid
  warehouseId       String   @map("warehouse_id") @db.Uuid
  productId         String   @map("product_id") @db.Uuid
  variantId         String?  @map("variant_id") @db.Uuid
  batchNumber       String?  @map("batch_number") @db.VarChar(100)
  expiryDate        DateTime? @map("expiry_date") @db.Date
  quantity          Decimal  @default(0) @db.Decimal(20, 2)
  reservedQuantity  Decimal  @default(0) @map("reserved_quantity") @db.Decimal(20, 2)
  availableQuantity Decimal  @default(0) @map("available_quantity") @db.Decimal(20, 2)
  costPrice         Decimal? @map("cost_price") @db.Decimal(20, 2)
  lastMovementAt    DateTime? @map("last_movement_at")
  createdAt         DateTime @default(now()) @map("created_at")
  updatedAt         DateTime @updatedAt @map("updated_at")

  tenant    Tenant    @relation(fields: [tenantId], references: [id])
  warehouse Warehouse @relation(fields: [warehouseId], references: [id])
  product   Product   @relation(fields: [productId], references: [id])
  variant   ProductVariant? @relation(fields: [variantId], references: [id])

  @@unique([warehouseId, productId, variantId, batchNumber, expiryDate])
  @@index([tenantId, warehouseId, productId])
  @@index([tenantId, productId])
  @@map("inventory_balances")
}

model InventoryMovement {
  id              BigInt               @id @default(autoincrement())
  tenantId        String               @map("tenant_id") @db.Uuid
  movementType    InventoryMovementType @map("movement_type")
  referenceType   String?              @map("reference_type") @db.VarChar(50)
  referenceId     String?              @map("reference_id") @db.Uuid
  warehouseId     String               @map("warehouse_id") @db.Uuid
  productId       String               @map("product_id") @db.Uuid
  variantId       String?              @map("variant_id") @db.Uuid
  batchNumber     String?              @map("batch_number") @db.VarChar(100)
  expiryDate      DateTime?            @map("expiry_date") @db.Date
  quantity        Decimal              @db.Decimal(20, 2)
  costBefore      Decimal?             @map("cost_before") @db.Decimal(20, 2)
  costAfter       Decimal?             @map("cost_after") @db.Decimal(20, 2)
  unitCost        Decimal?             @map("unit_cost") @db.Decimal(20, 2)
  notes           String?              @db.Text
  createdBy       String               @map("created_by") @db.Uuid
  createdAt       DateTime             @default(now()) @map("created_at")

  tenant    Tenant           @relation(fields: [tenantId], references: [id])
  warehouse Warehouse        @relation(fields: [warehouseId], references: [id])
  product   Product          @relation(fields: [productId], references: [id])
  variant   ProductVariant?  @relation(fields: [variantId], references: [id])
  user      User             @relation(fields: [createdBy], references: [id])

  @@index([tenantId, warehouseId, productId])
  @@index([referenceType, referenceId])
  @@index([tenantId, createdAt])
  @@map("inventory_movements")
}

model Shift {
  id                String      @id @default(uuid()) @db.Uuid
  tenantId          String      @map("tenant_id") @db.Uuid
  branchId          String      @map("branch_id") @db.Uuid
  userId            String      @map("user_id") @db.Uuid
  cashierName       String      @map("cashier_name") @db.VarChar(255)
  openedAt          DateTime    @map("opened_at")
  closedAt          DateTime?   @map("closed_at")
  openingBalance    Decimal     @map("opening_balance") @db.Decimal(20, 2)
  closingBalance    Decimal?    @map("closing_balance") @db.Decimal(20, 2)
  actualBalance     Decimal?    @map("actual_balance") @db.Decimal(20, 2)
  totalSales        Decimal     @default(0) @map("total_sales") @db.Decimal(20, 2)
  totalTransactions Int         @default(0) @map("total_transactions")
  status            ShiftStatus @default(open)
  notes             String?     @db.Text
  verifiedBy        String?     @map("verified_by") @db.Uuid
  createdAt         DateTime    @default(now()) @map("created_at")
  updatedAt         DateTime    @updatedAt @map("updated_at")

  tenant   Tenant  @relation(fields: [tenantId], references: [id])
  branch   Branch  @relation(fields: [branchId], references: [id])
  user     User    @relation("shiftCashier", fields: [userId], references: [id])
  verifier User?   @relation("shiftVerifier", fields: [verifiedBy], references: [id])

  @@index([tenantId, branchId, openedAt])
  @@index([tenantId, userId])
  @@map("shifts")
}

model Sale {
  id             String     @id @default(uuid()) @db.Uuid
  tenantId       String     @map("tenant_id") @db.Uuid
  branchId       String     @map("branch_id") @db.Uuid
  warehouseId    String     @map("warehouse_id") @db.Uuid
  customerId     String?    @map("customer_id") @db.Uuid
  userId         String     @map("user_id") @db.Uuid
  shiftId        String?    @map("shift_id") @db.Uuid
  invoiceNumber  String     @map("invoice_number") @db.VarChar(50)
  status         SaleStatus @default(completed)
  subtotal       Decimal    @db.Decimal(20, 2)
  discountTotal  Decimal    @default(0) @map("discount_total") @db.Decimal(20, 2)
  taxTotal       Decimal    @default(0) @map("tax_total") @db.Decimal(20, 2)
  grandTotal     Decimal    @map("grand_total") @db.Decimal(20, 2)
  rounding       Decimal    @default(0) @db.Decimal(10, 2)
  paidAmount     Decimal    @default(0) @map("paid_amount") @db.Decimal(20, 2)
  changeAmount   Decimal    @default(0) @map("change_amount") @db.Decimal(20, 2)
  notes          String?    @db.Text
  voidReason     String?    @map("void_reason") @db.Text
  voidedBy       String?    @map("voided_by") @db.Uuid
  voidedAt       DateTime?  @map("voided_at")
  createdAt      DateTime   @default(now()) @map("created_at")
  updatedAt      DateTime   @updatedAt @map("updated_at")
  deletedAt      DateTime?  @map("deleted_at")

  tenant    Tenant    @relation(fields: [tenantId], references: [id])
  branch    Branch    @relation(fields: [branchId], references: [id])
  warehouse Warehouse @relation(fields: [warehouseId], references: [id])
  customer  Customer? @relation(fields: [customerId], references: [id])
  user      User      @relation(fields: [userId], references: [id])
  shift     Shift?    @relation(fields: [shiftId], references: [id])
  voidedByUser User?  @relation("voidedBy", fields: [voidedBy], references: [id])
  items     SaleItem[]
  payments  Payment[]

  @@unique([tenantId, invoiceNumber])
  @@index([tenantId, createdAt])
  @@index([tenantId, branchId, createdAt])
  @@index([tenantId, customerId])
  @@index([tenantId, userId, createdAt])
  @@map("sales")
}

model SaleItem {
  id              String   @id @default(uuid()) @db.Uuid
  tenantId        String   @map("tenant_id") @db.Uuid
  saleId          String   @map("sale_id") @db.Uuid
  productId       String   @map("product_id") @db.Uuid
  variantId       String?  @map("variant_id") @db.Uuid
  lineNumber      Int      @map("line_number")
  quantity        Decimal  @db.Decimal(20, 2)
  unitPrice       Decimal  @map("unit_price") @db.Decimal(20, 2)
  discountPerItem Decimal  @default(0) @map("discount_per_item") @db.Decimal(20, 2)
  taxPerItem      Decimal  @default(0) @map("tax_per_item") @db.Decimal(20, 2)
  totalPrice      Decimal  @map("total_price") @db.Decimal(20, 2)
  costPrice       Decimal? @map("cost_price") @db.Decimal(20, 2)
  batchNumber     String?  @map("batch_number") @db.VarChar(100)
  expiryDate      DateTime? @map("expiry_date") @db.Date
  serialNumber    String?  @map("serial_number") @db.VarChar(100)
  notes           String?  @db.Text
  createdAt       DateTime @default(now()) @map("created_at")

  tenant  Tenant         @relation(fields: [tenantId], references: [id])
  sale    Sale           @relation(fields: [saleId], references: [id], onDelete: Cascade)
  product Product        @relation(fields: [productId], references: [id])
  variant ProductVariant? @relation(fields: [variantId], references: [id])

  @@index([saleId])
  @@index([tenantId, productId, createdAt])
  @@map("sale_items")
}

model PaymentMethod {
  id               String           @id @default(uuid()) @db.Uuid
  tenantId         String           @map("tenant_id") @db.Uuid
  code             String           @db.VarChar(50)
  name             String           @db.VarChar(100)
  type             PaymentMethodType
  requiresReference Boolean         @default(false) @map("requires_reference")
  isActive         Boolean          @default(true) @map("is_active")
  sortOrder        Int              @default(0) @map("sort_order")
  createdAt        DateTime         @default(now()) @map("created_at")

  tenant   Tenant   @relation(fields: [tenantId], references: [id])
  payments Payment[]

  @@unique([tenantId, code])
  @@map("payment_methods")
}

model Payment {
  id              String        @id @default(uuid()) @db.Uuid
  tenantId        String        @map("tenant_id") @db.Uuid
  saleId          String        @map("sale_id") @db.Uuid
  paymentMethodId String        @map("payment_method_id") @db.Uuid
  amount          Decimal       @db.Decimal(20, 2)
  referenceNumber String?       @map("reference_number") @db.VarChar(255)
  gatewayResponse Json?         @map("gateway_response")
  status          PaymentStatus @default(completed)
  paidAt          DateTime      @default(now()) @map("paid_at")
  createdAt       DateTime      @default(now()) @map("created_at")

  tenant        Tenant        @relation(fields: [tenantId], references: [id])
  sale          Sale          @relation(fields: [saleId], references: [id])
  paymentMethod PaymentMethod @relation(fields: [paymentMethodId], references: [id])

  @@index([saleId])
  @@map("payments")
}

model PurchaseOrder {
  id             String             @id @default(uuid()) @db.Uuid
  tenantId       String             @map("tenant_id") @db.Uuid
  branchId       String             @map("branch_id") @db.Uuid
  warehouseId    String             @map("warehouse_id") @db.Uuid
  supplierId     String             @map("supplier_id") @db.Uuid
  poNumber       String             @map("po_number") @db.VarChar(50)
  status         PurchaseOrderStatus
  orderDate      DateTime           @map("order_date") @db.Date
  expectedDate   DateTime?          @map("expected_date") @db.Date
  receivedDate   DateTime?          @map("received_date") @db.Date
  subtotal       Decimal            @db.Decimal(20, 2)
  discountTotal  Decimal            @default(0) @map("discount_total") @db.Decimal(20, 2)
  taxTotal       Decimal            @default(0) @map("tax_total") @db.Decimal(20, 2)
  grandTotal     Decimal            @map("grand_total") @db.Decimal(20, 2)
  notes          String?            @db.Text
  terms          String?            @db.Text
  requestedBy    String             @map("requested_by") @db.Uuid
  approvedBy     String?            @map("approved_by") @db.Uuid
  approvedAt     DateTime?          @map("approved_at")
  createdAt      DateTime           @default(now()) @map("created_at")
  updatedAt      DateTime           @updatedAt @map("updated_at")
  deletedAt      DateTime?          @map("deleted_at")

  tenant    Tenant    @relation(fields: [tenantId], references: [id])
  branch    Branch    @relation(fields: [branchId], references: [id])
  warehouse Warehouse @relation(fields: [warehouseId], references: [id])
  supplier  Supplier  @relation(fields: [supplierId], references: [id])
  requester User      @relation("requestedBy", fields: [requestedBy], references: [id])
  approver  User?     @relation("approvedBy", fields: [approvedBy], references: [id])
  items     PurchaseOrderItem[]

  @@unique([tenantId, poNumber])
  @@index([tenantId, status])
  @@index([tenantId, supplierId, status])
  @@index([tenantId, branchId, status])
  @@map("purchase_orders")
}

model PurchaseOrderItem {
  id                String   @id @default(uuid()) @db.Uuid
  tenantId          String   @map("tenant_id") @db.Uuid
  purchaseOrderId   String   @map("purchase_order_id") @db.Uuid
  productId         String   @map("product_id") @db.Uuid
  variantId         String?  @map("variant_id") @db.Uuid
  lineNumber        Int      @map("line_number")
  quantityOrdered   Decimal  @map("quantity_ordered") @db.Decimal(20, 2)
  quantityReceived  Decimal  @default(0) @map("quantity_received") @db.Decimal(20, 2)
  quantityInvoiced  Decimal  @default(0) @map("quantity_invoiced") @db.Decimal(20, 2)
  unitCost          Decimal  @map("unit_cost") @db.Decimal(20, 2)
  discountPerItem   Decimal  @default(0) @map("discount_per_item") @db.Decimal(20, 2)
  taxPerItem        Decimal  @default(0) @map("tax_per_item") @db.Decimal(20, 2)
  totalCost         Decimal  @map("total_cost") @db.Decimal(20, 2)
  batchNumber       String?  @map("batch_number") @db.VarChar(100)
  expiryDate        DateTime? @map("expiry_date") @db.Date
  notes             String?  @db.Text

  tenant        Tenant         @relation(fields: [tenantId], references: [id])
  purchaseOrder PurchaseOrder  @relation(fields: [purchaseOrderId], references: [id], onDelete: Cascade)
  product       Product        @relation(fields: [productId], references: [id])
  variant       ProductVariant? @relation(fields: [variantId], references: [id])

  @@index([purchaseOrderId])
  @@map("purchase_order_items")
}

model Plan {
  id            String   @id @default(uuid()) @db.Uuid
  name          String   @db.VarChar(100)
  slug          String   @unique @db.VarChar(100)
  description   String?  @db.Text
  priceMonthly  Decimal  @map("price_monthly") @db.Decimal(20, 2)
  priceYearly   Decimal  @map("price_yearly") @db.Decimal(20, 2)
  maxUsers      Int      @map("max_users")
  maxBranches   Int      @map("max_branches")
  maxProducts   Int      @map("max_products")
  features      Json
  isActive      Boolean  @default(true) @map("is_active")
  sortOrder     Int      @map("sort_order")
  createdAt     DateTime @default(now()) @map("created_at")
  updatedAt     DateTime @updatedAt @map("updated_at")

  subscriptions Subscription[]

  @@map("plans")
}

model Subscription {
  id                      String             @id @default(uuid()) @db.Uuid
  tenantId                String             @unique @map("tenant_id") @db.Uuid
  planId                  String             @map("plan_id") @db.Uuid
  status                  SubscriptionStatus
  trialEndsAt             DateTime?          @map("trial_ends_at")
  currentPeriodStartsAt   DateTime           @map("current_period_starts_at")
  currentPeriodEndsAt     DateTime           @map("current_period_ends_at")
  canceledAt              DateTime?          @map("canceled_at")
  canceledAtPeriodEnd     Boolean            @default(false) @map("canceled_at_period_end")
  stripeSubscriptionId    String?            @map("stripe_subscription_id") @db.VarChar(255)
  stripeCustomerId        String?            @map("stripe_customer_id") @db.VarChar(255)
  metadata                Json               @default("{}")
  createdAt               DateTime           @default(now()) @map("created_at")
  updatedAt               DateTime           @updatedAt @map("updated_at")

  tenant Tenant @relation(fields: [tenantId], references: [id])
  plan   Plan   @relation(fields: [planId], references: [id])

  @@map("subscriptions")
}

model AuditLog {
  id              BigInt      @id @default(autoincrement())
  tenantId        String      @map("tenant_id") @db.Uuid
  userId          String?     @map("user_id") @db.Uuid
  ipAddress       String?     @map("ip_address") @db.VarChar(45)
  userAgent       String?     @db.Text
  action          AuditAction
  entityType      String      @map("entity_type") @db.VarChar(100)
  entityId        String      @map("entity_id") @db.Uuid
  oldValues       Json?       @map("old_values")
  newValues       Json?       @map("new_values")
  changedFields   String[]    @map("changed_fields")
  correlationId   String?     @map("correlation_id") @db.Uuid
  createdAt       DateTime    @default(now()) @map("created_at")

  tenant Tenant @relation(fields: [tenantId], references: [id])
  user   User?  @relation(fields: [userId], references: [id])

  @@index([tenantId, entityType, entityId])
  @@index([tenantId, userId])
  @@index([tenantId, createdAt])
  @@map("audit_logs")
}

model Notification {
  id        String             @id @default(uuid()) @db.Uuid
  tenantId  String             @map("tenant_id") @db.Uuid
  userId    String?            @map("user_id") @db.Uuid
  roleSlug  String?            @map("role_slug") @db.VarChar(100)
  type      String             @db.VarChar(50)
  channel   NotificationChannel
  title     String             @db.VarChar(255)
  body      String             @db.Text
  data      Json?              @default("{}")
  isRead    Boolean            @default(false) @map("is_read")
  readAt    DateTime?          @map("read_at")
  createdAt DateTime           @default(now()) @map("created_at")

  tenant Tenant @relation(fields: [tenantId], references: [id])
  user   User?  @relation(fields: [userId], references: [id])

  @@index([tenantId, userId, isRead])
  @@index([tenantId, createdAt])
  @@map("notifications")
}
```

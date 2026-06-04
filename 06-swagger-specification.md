# Phase 6: Swagger Specification (OpenAPI 3.0)

> Structure for the OpenAPI spec document. This is the architectural skeleton — you'll build the full YAML from this.

```yaml
# openapi.yaml — Structural Outline

openapi: 3.0.3
info:
  title: Hermes POS — Business Operating System API
  description: |
    Enterprise-grade POS and Business Management System.
    Multi-tenant, multi-branch, production-ready.
  version: 1.0.0
  contact:
    name: API Support
    email: api@hermespos.com

servers:
  - url: https://api.hermespos.com/api/v1
    description: Production server
  - url: https://staging-api.hermespos.com/api/v1
    description: Staging server

components:
  securitySchemes:
    BearerAuth:
      type: http
      scheme: bearer
      bearerFormat: JWT
    TenantHeader:
      type: apiKey
      in: header
      name: X-Tenant-Id

  parameters:
    PageParam:
      name: page
      in: query
      schema: { type: integer, minimum: 1, default: 1 }
      description: Page number (offset pagination)
    LimitParam:
      name: limit
      in: query
      schema: { type: integer, minimum: 1, maximum: 100, default: 25 }
      description: Items per page
    SortParam:
      name: sort
      in: query
      schema: { type: string, example: "-createdAt" }
      description: Sort field. Prefix with - for descending.
    SearchParam:
      name: q
      in: query
      schema: { type: string }
      description: Full-text search query
    CursorParam:
      name: cursor
      in: query
      schema: { type: string }
      description: Cursor for keyset pagination
    TenantHeaderParam:
      name: X-Tenant-Id
      in: header
      required: true
      schema: { type: string, format: uuid }
      description: Tenant UUID

  schemas:
    # =================== GLOBAL ===================
    ApiSuccess:
      type: object
      properties:
        success: { type: boolean, example: true }
        data: {}
        meta:
          type: object
          properties:
            page: { type: integer }
            limit: { type: integer }
            total: { type: integer }
            totalPages: { type: integer }

    ApiError:
      type: object
      properties:
        success: { type: boolean, example: false }
        error:
          type: object
          properties:
            code: { type: string, example: "VALIDATION_ERROR" }
            message: { type: string, example: "Validation failed" }
            details:
              type: array
              items: { $ref: '#/components/schemas/ValidationError' }
            statusCode: { type: integer, example: 422 }

    ValidationError:
      type: object
      properties:
        field: { type: string, example: "email" }
        message: { type: string, example: "Email is required" }

    PaginatedResponse:
      type: object
      properties:
        success: { type: boolean, example: true }
        data:
          type: array
          items: {}
        meta:
          type: object
          properties:
            page: { type: integer, example: 1 }
            limit: { type: integer, example: 25 }
            total: { type: integer, example: 1234 }
            totalPages: { type: integer, example: 50 }

    # =================== AUTH ===================
    LoginRequest:
      type: object
      required: [email, password, tenantSlug]
      properties:
        email: { type: string, format: email, example: "admin@tokoku.com" }
        password: { type: string, format: password, example: "securePassword123!" }
        tenantSlug: { type: string, example: "tokoku-jkt" }

    LoginResponse:
      type: object
      properties:
        accessToken: { type: string }
        refreshToken: { type: string }
        expiresIn: { type: integer }
        user: { $ref: '#/components/schemas/UserProfile' }

    UserProfile:
      type: object
      properties:
        id: { type: string, format: uuid }
        email: { type: string }
        firstName: { type: string }
        lastName: { type: string }
        roles: { type: array, items: { type: string } }
        permissions: { type: array, items: { type: string } }

    RefreshTokenRequest:
      type: object
      required: [refreshToken]
      properties:
        refreshToken: { type: string }

    ForgotPasswordRequest:
      type: object
      required: [email]
      properties:
        email: { type: string, format: email }

    ResetPasswordRequest:
      type: object
      required: [token, password]
      properties:
        token: { type: string }
        password: { type: string, minLength: 8 }

    ChangePasswordRequest:
      type: object
      required: [currentPassword, newPassword]
      properties:
        currentPassword: { type: string }
        newPassword: { type: string, minLength: 8 }

    # =================== TENANTS ===================
    CreateTenantRequest:
      type: object
      required: [name, slug, email]
      properties:
        name: { type: string, maxLength: 255, example: "Toko ABC" }
        slug: { type: string, pattern: "^[a-z0-9-]+$", maxLength: 100, example: "toko-abc" }
        email: { type: string, format: email }
        phone: { type: string, nullable: true }
        currencyCode: { type: string, default: "IDR" }
        timezone: { type: string, default: "Asia/Jakarta" }

    TenantResponse:
      type: object
      properties:
        id: { type: string, format: uuid }
        name: { type: string }
        slug: { type: string }
        currencyCode: { type: string }
        timezone: { type: string }
        isActive: { type: boolean }
        createdAt: { type: string, format: date-time }

    # =================== USERS ===================
    CreateUserRequest:
      type: object
      required: [email, firstName, roleIds]
      properties:
        email: { type: string, format: email }
        phone: { type: string, nullable: true }
        firstName: { type: string, maxLength: 100 }
        lastName: { type: string, nullable: true, maxLength: 100 }
        roleIds: { type: array, items: { type: string, format: uuid }, minItems: 1 }
        branchId: { type: string, format: uuid, nullable: true }

    UpdateUserRequest:
      type: object
      properties:
        firstName: { type: string, maxLength: 100 }
        lastName: { type: string, nullable: true }
        phone: { type: string, nullable: true }
        branchId: { type: string, format: uuid, nullable: true }
        isActive: { type: boolean }

    UserResponse:
      type: object
      properties:
        id: { type: string, format: uuid }
        email: { type: string }
        firstName: { type: string }
        lastName: { type: string, nullable: true }
        isActive: { type: boolean }
        roles:
          type: array
          items:
            type: object
            properties:
              id: { type: string, format: uuid }
              name: { type: string }
              slug: { type: string }
        branch:
          type: object
          nullable: true
          properties:
            id: { type: string, format: uuid }
            name: { type: string }
            code: { type: string }
        lastLoginAt: { type: string, format: date-time, nullable: true }
        createdAt: { type: string, format: date-time }

    # =================== PRODUCTS ===================
    CreateProductRequest:
      type: object
      required: [code, name, type, unitId]
      properties:
        code: { type: string, maxLength: 100, example: "IND-001" }
        barcode: { type: string, maxLength: 100, nullable: true }
        name: { type: string, maxLength: 255, example: "Indomie Goreng" }
        description: { type: string, nullable: true }
        type:
          type: string
          enum: [simple, variant, composite, service]
        categoryId: { type: string, format: uuid, nullable: true }
        brandId: { type: string, format: uuid, nullable: true }
        unitId: { type: string, format: uuid }
        purchaseUnitId: { type: string, format: uuid, nullable: true }
        trackSerial: { type: boolean, default: false }
        trackBatch: { type: boolean, default: false }
        hasExpiry: { type: boolean, default: false }
        taxInclusive: { type: boolean, default: true }
        taxRate: { type: number, maximum: 100, default: 0 }
        minStock: { type: number, default: 0 }
        maxStock: { type: number, nullable: true }
        weight: { type: number, nullable: true }
        tags: { type: array, items: { type: string } }
        imageUrl: { type: string, nullable: true }

    ProductResponse:
      type: object
      properties:
        id: { type: string, format: uuid }
        code: { type: string, example: "IND-001" }
        barcode: { type: string, nullable: true }
        name: { type: string, example: "Indomie Goreng" }
        description: { type: string, nullable: true }
        type: { type: string, enum: [simple, variant, composite, service] }
        categoryId: { type: string, format: uuid, nullable: true }
        brandId: { type: string, format: uuid, nullable: true }
        unitId: { type: string, format: uuid }
        purchaseUnitId: { type: string, format: uuid, nullable: true }
        trackSerial: { type: boolean }
        trackBatch: { type: boolean }
        hasExpiry: { type: boolean }
        isActive: { type: boolean }
        taxInclusive: { type: boolean }
        taxRate: { type: number, example: 11 }
        minStock: { type: number, example: 10 }
        maxStock: { type: number, nullable: true }
        weight: { type: number, nullable: true }
        tags: { type: array, items: { type: string } }
        imageUrl: { type: string, nullable: true }
        category: { type: object, nullable: true }
        brand: { type: object, nullable: true }
        unit: { type: object }
        variants:
          type: array
          items: { $ref: '#/components/schemas/ProductVariantResponse' }
        createdAt: { type: string, format: date-time }
        updatedAt: { type: string, format: date-time }

    ProductVariantResponse:
      type: object
      properties:
        id: { type: string, format: uuid }
        code: { type: string }
        barcode: { type: string, nullable: true }
        name: { type: string }
        attributes: { type: object }
        price: { type: number, nullable: true }
        costPrice: { type: number, nullable: true }
        isActive: { type: boolean }
        sortOrder: { type: integer }

    # =================== CUSTOMERS ===================
    CreateCustomerRequest:
      type: object
      required: [code, name]
      properties:
        code: { type: string, maxLength: 50, example: "CUST-001" }
        name: { type: string, maxLength: 255, example: "John Doe" }
        email: { type: string, format: email, nullable: true }
        phone: { type: string, nullable: true }
        whatsapp: { type: string, nullable: true }
        address: { type: string, nullable: true }
        city: { type: string, nullable: true }
        province: { type: string, nullable: true }
        groupId: { type: string, format: uuid, nullable: true }
        creditLimit: { type: number, default: 0 }

    # =================== SALES ===================
    CreateSaleRequest:
      type: object
      required: [branchId, warehouseId, items, payments]
      properties:
        branchId: { type: string, format: uuid }
        warehouseId: { type: string, format: uuid }
        customerId: { type: string, format: uuid, nullable: true }
        shiftId: { type: string, format: uuid, nullable: true }
        notes: { type: string, nullable: true }
        items:
          type: array
          minItems: 1
          items:
            type: object
            required: [productId, quantity, unitPrice]
            properties:
              productId: { type: string, format: uuid }
              variantId: { type: string, format: uuid, nullable: true }
              quantity: { type: number, minimum: 0.01 }
              unitPrice: { type: number, minimum: 0 }
              discountPerItem: { type: number, minimum: 0, default: 0 }
              batchNumber: { type: string, nullable: true }
              expiryDate: { type: string, format: date, nullable: true }
              serialNumber: { type: string, nullable: true }
        payments:
          type: array
          minItems: 1
          items:
            type: object
            required: [paymentMethodId, amount]
            properties:
              paymentMethodId: { type: string, format: uuid }
              amount: { type: number, minimum: 0.01 }
              referenceNumber: { type: string, nullable: true }

    SaleResponse:
      type: object
      properties:
        id: { type: string, format: uuid }
        invoiceNumber: { type: string }
        branchId: { type: string, format: uuid }
        warehouseId: { type: string, format: uuid }
        customerId: { type: string, format: uuid, nullable: true }
        userId: { type: string, format: uuid }
        status: { type: string, enum: [draft, completed, voided, refunded] }
        subtotal: { type: number }
        discountTotal: { type: number }
        taxTotal: { type: number }
        grandTotal: { type: number }
        rounding: { type: number }
        paidAmount: { type: number }
        changeAmount: { type: number }
        items:
          type: array
          items: { $ref: '#/components/schemas/SaleItemResponse' }
        payments:
          type: array
          items: { $ref: '#/components/schemas/PaymentResponse' }
        customer: { type: object, nullable: true }
        createdAt: { type: string, format: date-time }

    SaleItemResponse:
      type: object
      properties:
        id: { type: string, format: uuid }
        productId: { type: string, format: uuid }
        productName: { type: string }
        variantId: { type: string, format: uuid, nullable: true }
        variantName: { type: string, nullable: true }
        lineNumber: { type: integer }
        quantity: { type: number }
        unitPrice: { type: number }
        discountPerItem: { type: number }
        taxPerItem: { type: number }
        totalPrice: { type: number }
        costPrice: { type: number, nullable: true }

    PaymentResponse:
      type: object
      properties:
        id: { type: string, format: uuid }
        paymentMethodId: { type: string, format: uuid }
        paymentMethodName: { type: string }
        amount: { type: number }
        referenceNumber: { type: string, nullable: true }
        status: { type: string, enum: [pending, completed, failed, refunded] }
        paidAt: { type: string, format: date-time }

    # =================== INVENTORY ===================
    StockAdjustmentRequest:
      type: object
      required: [warehouseId, items]
      properties:
        warehouseId: { type: string, format: uuid }
        notes: { type: string, nullable: true }
        items:
          type: array
          minItems: 1
          items:
            type: object
            required: [productId, newQuantity, reason]
            properties:
              productId: { type: string, format: uuid }
              variantId: { type: string, format: uuid, nullable: true }
              batchNumber: { type: string, nullable: true }
              expiryDate: { type: string, format: date, nullable: true }
              newQuantity: { type: number, minimum: 0 }
              reason: { type: string }

    # =================== REPORTS ===================
    SalesSummaryResponse:
      type: object
      properties:
        period:
          type: object
          properties:
            from: { type: string, format: date }
            to: { type: string, format: date }
        summary:
          type: object
          properties:
            totalSales: { type: number }
            totalTransactions: { type: integer }
            averageTransactionValue: { type: number }
            totalDiscount: { type: number }
            totalTax: { type: number }
            totalCost: { type: number }
            grossProfit: { type: number }
            grossProfitMargin: { type: number }
        byPaymentMethod:
          type: array
          items:
            type: object
            properties:
              method: { type: string }
              amount: { type: number }
              count: { type: integer }

  # =================== PATHS ===================
paths:
  /auth/login:
    post:
      tags: [Authentication]
      summary: Login
      description: Authenticate user with email, password, and tenant slug
      operationId: login
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/LoginRequest'
            example:
              email: admin@tokoku.com
              password: securePassword123!
              tenantSlug: tokoku-jkt
      responses:
        '200':
          description: Login successful
          content:
            application/json:
              schema:
                type: object
                properties:
                  success: { type: boolean, example: true }
                  data: { $ref: '#/components/schemas/LoginResponse' }
        '401':
          description: Invalid credentials
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ApiError'
        '429':
          description: Too many login attempts

  /auth/logout:
    post:
      tags: [Authentication]
      summary: Logout
      description: Invalidate current session
      operationId: logout
      security:
        - BearerAuth: []
      responses:
        '200':
          description: Logged out successfully

  /auth/refresh:
    post:
      tags: [Authentication]
      summary: Refresh token
      description: Get new access token using refresh token
      operationId: refreshToken
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/RefreshTokenRequest'
      responses:
        '200':
          description: Token refreshed

  /auth/me:
    get:
      tags: [Authentication]
      summary: Current user
      description: Get authenticated user's profile and permissions
      operationId: getCurrentUser
      security:
        - BearerAuth: []
      responses:
        '200':
          description: Current user profile

  /products:
    get:
      tags: [Products]
      summary: List products
      description: Get paginated list of products with search, filter, and sort
      operationId: listProducts
      security:
        - BearerAuth: []
        - TenantHeader: []
      parameters:
        - { $ref: '#/components/parameters/PageParam' }
        - { $ref: '#/components/parameters/LimitParam' }
        - { $ref: '#/components/parameters/SortParam' }
        - { $ref: '#/components/parameters/SearchParam' }
        - { name: 'filter[categoryId]', in: query, schema: { type: string, format: uuid } }
        - { name: 'filter[brandId]', in: query, schema: { type: string, format: uuid } }
        - { name: 'filter[isActive]', in: query, schema: { type: boolean } }
        - { name: 'filter[type]', in: query, schema: { type: string, enum: [simple, variant, composite, service] } }
      responses:
        '200':
          description: Products list
          content:
            application/json:
              schema:
                allOf:
                  - type: object
                    properties:
                      data:
                        type: array
                        items:
                          $ref: '#/components/schemas/ProductResponse'
                  - $ref: '#/components/schemas/PaginatedResponse'

    post:
      tags: [Products]
      summary: Create product
      description: Create a new product with all attributes
      operationId: createProduct
      security:
        - BearerAuth: []
        - TenantHeader: []
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/CreateProductRequest'
      responses:
        '201':
          description: Product created
          content:
            application/json:
              schema:
                type: object
                properties:
                  success: { type: boolean, example: true }
                  data: { $ref: '#/components/schemas/ProductResponse' }
        '409':
          description: Product code or barcode already exists
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ApiError'

  /sales:
    post:
      tags: [Sales]
      summary: Create sale
      description: Process a complete POS sale transaction with inventory deduction
      operationId: createSale
      security:
        - BearerAuth: []
        - TenantHeader: []
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/CreateSaleRequest'
      responses:
        '201':
          description: Sale completed
          content:
            application/json:
              schema:
                type: object
                properties:
                  success: { type: boolean, example: true }
                  data: { $ref: '#/components/schemas/SaleResponse' }
        '422':
          description: Validation error or insufficient stock

  /reports/sales/summary:
    get:
      tags: [Reports]
      summary: Sales summary
      description: Get aggregated sales summary for a date range
      operationId: salesSummary
      security:
        - BearerAuth: []
        - TenantHeader: []
      parameters:
        - { name: 'filter[dateFrom]', in: query, required: true, schema: { type: string, format: date } }
        - { name: 'filter[dateTo]', in: query, required: true, schema: { type: string, format: date } }
        - { name: 'filter[branchId]', in: query, schema: { type: string, format: uuid } }
      responses:
        '200':
          description: Sales summary report
```

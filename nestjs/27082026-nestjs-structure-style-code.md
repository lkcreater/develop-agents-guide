# NestJS Project Structure — Best Practices

## 1. Recommended Structure

For a medium-to-large NestJS project, a **domain/module-oriented structure** is recommended over a layer-based structure.

```text
src/
├── main.ts
├── app.module.ts
│
├── config/
│   ├── configuration.ts
│   ├── database.config.ts
│   └── app.config.ts
│
├── common/
│   ├── decorators/
│   ├── guards/
│   ├── interceptors/
│   ├── filters/
│   ├── pipes/
│   ├── middleware/
│   ├── exceptions/
│   ├── constants/
│   ├── types/
│   └── utils/
│
├── database/
│   ├── database.module.ts
│   ├── migrations/
│   └── seeds/
│
├── auth/
│   ├── auth.module.ts
│   ├── auth.controller.ts
│   ├── auth.service.ts
│   ├── guards/
│   ├── strategies/
│   ├── dto/
│   │   ├── login.dto.ts
│   │   ├── register.dto.ts
│   │   └── refresh-token.dto.ts
│   └── types/
│
├── users/
│   ├── users.module.ts
│   ├── users.controller.ts
│   ├── users.service.ts
│   ├── dto/
│   │   ├── create-user.dto.ts
│   │   └── update-user.dto.ts
│   └── types/
│
├── products/
│   ├── products.module.ts
│   ├── products.controller.ts
│   ├── products.service.ts
│   ├── dto/
│   └── types/
│
├── orders/
│   ├── orders.module.ts
│   ├── orders.controller.ts
│   ├── orders.service.ts
│   ├── dto/
│   └── types/
│
├── integrations/
│   ├── integrations.module.ts
│   ├── facebook/
│   │   ├── facebook.service.ts
│   │   ├── facebook.client.ts
│   │   └── types/
│   ├── whatsapp/
│   │   ├── whatsapp.service.ts
│   │   ├── whatsapp.client.ts
│   │   └── types/
│   └── line/
│       ├── line.service.ts
│       ├── line.client.ts
│       └── types/
│
└── health/
    ├── health.module.ts
    └── health.controller.ts
```

---

## 2. Core Architecture

Recommended dependency flow:

```text
Controller
    ↓
Service
    ↓
Repository (optional)
    ↓
Database
```

### Controller

Responsible for HTTP/API concerns:

- Routing
- Request parameters
- DTO validation
- Authentication/authorization decorators
- Calling application services
- Returning responses

Controllers should **not contain business logic**.

Example:

```ts
@Post()
create(@Body() dto: CreateOrderDto) {
  return this.ordersService.create(dto);
}
```

### Service

Responsible for business logic:

```ts
@Injectable()
export class OrdersService {
  constructor(
    private readonly ordersRepository: OrdersRepository,
  ) {}

  async create(dto: CreateOrderDto) {
    // Business rules
    // Validation beyond DTO validation
    // Transaction orchestration
    // Calling repositories/services

    return this.ordersRepository.create(dto);
  }
}
```

### Repository

Use a repository when database access becomes complex or when you want to isolate persistence logic.

```text
orders/
├── orders.controller.ts
├── orders.service.ts
├── orders.repository.ts
├── dto/
└── orders.module.ts
```

For simple CRUD projects, a repository layer is optional.

---

## 3. Module-Oriented Design

Each business domain should normally have its own module.

```text
src/
├── auth/
├── users/
├── products/
├── orders/
├── payments/
├── notifications/
└── ...
```

Each module owns its own:

- Controller
- Service
- DTO
- Types
- Repository
- Domain-specific utilities

Example:

```text
orders/
├── orders.module.ts
├── orders.controller.ts
├── orders.service.ts
├── orders.repository.ts
├── dto/
│   ├── create-order.dto.ts
│   ├── update-order.dto.ts
│   └── order-query.dto.ts
└── types/
```

This makes the codebase easier to navigate and maintain.

---

## 4. Common Directory

Use `common/` only for code genuinely shared across multiple modules.

```text
common/
├── decorators/
├── guards/
├── interceptors/
├── filters/
├── pipes/
├── middleware/
├── exceptions/
├── constants/
├── types/
└── utils/
```

Examples:

```text
common/guards/jwt-auth.guard.ts
common/interceptors/response.interceptor.ts
common/filters/http-exception.filter.ts
common/pipes/validation.pipe.ts
```

### Rule

If code is specific to one module, keep it inside that module.

```text
orders/
└── utils/
```

Do not move it into `common/` just because it is technically reusable.

---

## 5. External API Integrations

For systems that integrate with external providers such as Meta, WhatsApp, LINE, OpenAI, payment providers, or other SaaS APIs, use an `integrations/` directory.

```text
integrations/
├── meta/
│   ├── meta.module.ts
│   ├── meta.client.ts
│   ├── meta.service.ts
│   └── types/
│
├── whatsapp/
│   ├── whatsapp.module.ts
│   ├── whatsapp.client.ts
│   ├── whatsapp.service.ts
│   └── types/
│
├── line/
│   ├── line.module.ts
│   ├── line.client.ts
│   ├── line.service.ts
│   └── types/
│
└── openai/
    ├── openai.module.ts
    ├── openai.client.ts
    ├── openai.service.ts
    └── types/
```

Recommended responsibility:

```text
Business Module
      ↓
Integration Service
      ↓
Integration Client
      ↓
External API
```

For example:

```text
WhatsAppService
      ↓
WhatsAppClient
      ↓
Meta Graph API
```

### Client

Responsible for:

- HTTP requests
- Authentication headers
- API URL
- External API response handling
- Retry/timeouts where appropriate

### Service

Responsible for:

- Business logic
- Mapping external data to internal models
- Business-level validation
- Orchestration

This separation makes it easier to replace an external provider later.

---

## 6. DTO Best Practices

Separate DTOs by operation.

Recommended:

```text
users/
└── dto/
    ├── create-user.dto.ts
    ├── update-user.dto.ts
    ├── user-query.dto.ts
    └── user-response.dto.ts
```

Avoid:

```text
users/
└── dto.ts
```

A single DTO file becomes difficult to maintain as the module grows.

Example:

```ts
export class CreateUserDto {
  @IsEmail()
  email: string;

  @IsString()
  @MinLength(8)
  password: string;
}
```

Use `class-validator` and `ValidationPipe` for request validation.

---

## 7. API Versioning

For production APIs, consider URI versioning:

```ts
app.enableVersioning({
  type: VersioningType.URI,
});
```

API examples:

```text
/api/v1/users
/api/v1/orders
/api/v1/products
```

Future versions:

```text
/api/v2/users
```

Version only APIs that actually need different contracts. Avoid creating unnecessary versions.

---

## 8. Database Layer

Keep database infrastructure separate from business modules.

```text
database/
├── database.module.ts
├── migrations/
└── seeds/
```

For Prisma:

```text
src/
├── database/
│   └── prisma.service.ts
│
└── users/
    └── users.service.ts
```

For simple CRUD:

```text
Controller
    ↓
Service
    ↓
PrismaService
    ↓
PostgreSQL
```

For complex systems:

```text
Controller
    ↓
Service
    ↓
Repository
    ↓
PrismaService
    ↓
PostgreSQL
```

Do not introduce repositories everywhere unless they provide real value.

---

## 9. Authentication

Authentication should normally be isolated as its own module.

```text
auth/
├── auth.module.ts
├── auth.controller.ts
├── auth.service.ts
├── dto/
│   ├── login.dto.ts
│   ├── register.dto.ts
│   └── refresh-token.dto.ts
├── guards/
│   ├── jwt-auth.guard.ts
│   └── roles.guard.ts
├── strategies/
│   ├── jwt.strategy.ts
│   └── local.strategy.ts
└── types/
```

Keep authentication logic out of `users.service.ts` unless the logic is specifically user-domain logic.

---

## 10. Health Check

Production services should have a dedicated health module.

```text
health/
├── health.module.ts
└── health.controller.ts
```

Typical endpoints:

```text
GET /health
GET /health/ready
GET /health/live
```

Health checks can cover:

- Application
- PostgreSQL
- Redis
- External dependencies where appropriate

---

## 11. What to Avoid

### Layer-Based Structure

Avoid organizing the whole application like this:

```text
src/
├── controllers/
├── services/
├── repositories/
├── dto/
└── entities/
```

This becomes difficult to navigate as the application grows.

Prefer:

```text
src/
├── users/
├── orders/
├── products/
└── payments/
```

### God Folder

Avoid putting everything into:

```text
common/
├── services/
├── helpers/
├── utils/
├── repositories/
└── ...
```

This eventually becomes a dumping ground.

### Huge Services

Avoid files such as:

```text
users.service.ts
5000+ lines
```

Split responsibilities when a service becomes too large:

```text
users/
├── users.service.ts
├── user-profile.service.ts
├── user-permission.service.ts
└── user-session.service.ts
```

---

## 12. Recommended Production Structure

For a production application using:

- NestJS
- PostgreSQL
- Redis
- Authentication
- External APIs
- Background jobs
- REST API

A good starting point is:

```text
src/
├── main.ts
├── app.module.ts
│
├── config/
│
├── common/
│   ├── decorators/
│   ├── guards/
│   ├── interceptors/
│   ├── filters/
│   ├── pipes/
│   ├── exceptions/
│   ├── constants/
│   ├── types/
│   └── utils/
│
├── database/
│   ├── database.module.ts
│   ├── migrations/
│   └── seeds/
│
├── auth/
├── users/
├── products/
├── orders/
├── payments/
├── notifications/
│
├── integrations/
│   ├── meta/
│   ├── whatsapp/
│   ├── line/
│   ├── openai/
│   └── ...
│
├── jobs/
│   ├── jobs.module.ts
│   ├── processors/
│   └── queues/
│
└── health/
```

---

## 13. Architecture Rules

Use these rules as project guidelines:

| Rule | Recommendation |
|---|---|
| Business domain | One root module per domain |
| HTTP/API | Controller |
| Business logic | Service |
| Database access | Repository when useful |
| Shared code | `common/` |
| External APIs | `integrations/` |
| Database infrastructure | `database/` |
| Authentication | `auth/` |
| Background jobs | `jobs/` |
| Health checks | `health/` |
| Request validation | DTO + ValidationPipe |
| API evolution | API versioning |
| Large services | Split by responsibility |

---

## 14. Recommended Dependency Direction

Keep dependencies flowing inward toward business/application logic.

```text
HTTP
 │
 ▼
Controller
 │
 ▼
Service
 │
 ├──────────────► Repository
 │                    │
 │                    ▼
 │                 Database
 │
 └──────────────► Integration Service
                         │
                         ▼
                  Integration Client
                         │
                         ▼
                   External API
```

Avoid having controllers directly call:

```text
Controller
   ↓
Prisma
```

or:

```text
Controller
   ↓
External API
```

Controllers should remain thin.

---

## 15. Practical Rule of Thumb

> **Business domain → root module**

> **External provider → integrations**

> **Reusable infrastructure → common**

> **Database infrastructure → database**

> **HTTP logic → controller**

> **Business logic → service**

> **Complex DB access → repository**

> **External API communication → client**

The goal is not to create the most abstract architecture possible. The goal is to create a structure that remains easy to understand, test, change, and scale.

For most NestJS production projects, a **module-oriented architecture with clear boundaries** is a strong balance between Clean Architecture principles and practical maintainability.

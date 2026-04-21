# logpilot-gateway — API Gateway

> **Central entry point for all client requests. Handles JWT authentication, rate limiting, CORS, and routes traffic to all internal microservices.**

---

## 📋 Service Overview

| Property | Value |
|----------|-------|
| **Service Name** | `logpilot-gateway` |
| **Technology** | Spring Boot 3.x, Spring Cloud Gateway, Spring Security |
| **Type** | API Gateway |
| **Port** | `8080` (primary, public-facing) |
| **Communication** | REST (inbound from frontend/external) → internal services |
| **Team Owner** | Backend Engineer #1 |
| **Build Priority** | Month 1, Week 1–2 |

---

## 🎯 Responsibilities

1. **JWT Authentication** — Issue and validate JWT tokens (15-min access + 7-day refresh)
2. **User Management** — Registration, login, password reset, role management
3. **RBAC Authorization** — Admin, Developer, Viewer roles per organization
4. **Rate Limiting** — 100 requests/min per tenant (configurable)
5. **Request Routing** — Proxy requests to internal services based on path prefix
6. **CORS Policy** — Configurable cross-origin resource sharing
7. **Multi-Tenancy** — Every request scoped to `org_id` extracted from JWT
8. **Billing Management** — Stripe subscription integration, usage metering
9. **Audit Logging** — Log all user actions to PostgreSQL audit table

---

## 📦 Codebase Structure

```
logpilot-gateway/
├── src/
│   └── main/
│       ├── java/com/logpilot/gateway/
│       │   ├── GatewayApplication.java
│       │   ├── config/
│       │   │   ├── SecurityConfig.java              # Spring Security + JWT filter chain
│       │   │   ├── CorsConfig.java                  # CORS settings
│       │   │   ├── RateLimitConfig.java             # Rate limiting filter
│       │   │   ├── RouteConfig.java                 # Gateway route definitions
│       │   │   └── StripeConfig.java                # Stripe API keys
│       │   ├── controller/
│       │   │   ├── AuthController.java              # POST /auth/login, /auth/register
│       │   │   ├── UserController.java              # User profile CRUD
│       │   │   ├── OrgController.java               # Organization management
│       │   │   ├── BillingController.java           # Subscription & billing
│       │   │   └── AuditController.java             # Query audit logs
│       │   ├── security/
│       │   │   ├── JwtTokenProvider.java            # Generate/validate JWT
│       │   │   ├── JwtAuthFilter.java               # OncePerRequestFilter
│       │   │   ├── UserDetailsServiceImpl.java      # Load user from DB
│       │   │   └── RoleEnum.java                    # ADMIN, DEVELOPER, VIEWER
│       │   ├── service/
│       │   │   ├── AuthService.java                 # Login, register, refresh token
│       │   │   ├── UserService.java                 # User CRUD
│       │   │   ├── OrgService.java                  # Org CRUD, invite members
│       │   │   ├── BillingService.java              # Stripe integration
│       │   │   ├── UsageMeterService.java           # Track log volume per tenant
│       │   │   └── AuditService.java                # Log user actions
│       │   ├── model/
│       │   │   ├── User.java                        # User entity
│       │   │   ├── Organization.java                # Org entity
│       │   │   ├── Subscription.java                # Billing subscription entity
│       │   │   └── AuditLog.java                    # Audit trail entity
│       │   └── repository/
│       │       ├── UserRepository.java
│       │       ├── OrgRepository.java
│       │       ├── SubscriptionRepository.java
│       │       └── AuditLogRepository.java
│       └── resources/
│           ├── application.yml
│           └── logback-spring.xml
├── src/test/java/
├── Dockerfile
├── pom.xml
└── README.md
```

---

## 🔗 Dependencies

| Dependency | Purpose |
|------------|---------|
| Spring Cloud Gateway | API routing and filtering |
| Spring Boot Starter Security | Authentication & authorization |
| Spring Boot Starter Data JPA | PostgreSQL for users, orgs, billing |
| jjwt (io.jsonwebtoken) | JWT token generation & validation |
| Stripe Java SDK | Subscription billing |
| Spring Boot Starter Actuator | Health checks |
| Bucket4j + Redis | Distributed rate limiting |
| PostgreSQL Driver | Database |

---

## 🔀 Route Configuration

| Path Pattern | Routes To | Service |
|-------------|-----------|---------|
| `/api/v1/logs/**` | `http://logpilot-ingest:8081` | logpilot-ingest-service |
| `/api/v1/incidents/**` | `http://logpilot-query:8086` | logpilot-query-service |
| `/api/v1/query/**` | `http://logpilot-ai:8083` | logpilot-ai-engine |
| `/api/v1/alerts/**` | `http://logpilot-alerting:8084` | logpilot-alerting-service |
| `/api/v1/services/**` | `http://logpilot-ingest:8081` | logpilot-ingest-service |
| `/api/v1/analytics/**` | `http://logpilot-query:8086` | logpilot-query-service |
| `/auth/**` | Handled locally | logpilot-gateway (self) |
| `/api/v1/billing/**` | Handled locally | logpilot-gateway (self) |

---

## 🗄️ Database Tables Owned

| Table | Purpose |
|-------|---------|
| `organizations` | id, name, plan, created_at |
| `users` | id, org_id, email, role, password_hash |
| `subscriptions` | id, org_id, plan, stripe_subscription_id, status |
| `audit_logs` | id, user_id, org_id, action, resource, timestamp |

---

## 📡 Auth Endpoints

| Method | Path | Description |
|--------|------|-------------|
| POST | `/auth/register` | Create new org + admin user |
| POST | `/auth/login` | Login, returns JWT access + refresh tokens |
| POST | `/auth/refresh` | Refresh access token |
| POST | `/auth/logout` | Invalidate refresh token |
| GET | `/auth/me` | Get current user profile |

---

## ✅ Testing Checklist

- [ ] Unit tests for JwtTokenProvider (generate, validate, expire)
- [ ] Unit tests for AuthService (register, login, wrong password)
- [ ] Integration test: full auth flow (register → login → access protected endpoint)
- [ ] Rate limit test: exceed 100 req/min → 429 response
- [ ] RBAC test: viewer cannot access admin endpoints
- [ ] Route test: verify all path prefixes route to correct services
- [ ] Stripe webhook test: subscription created → DB updated
- [ ] Audit log test: API call → audit entry in DB


# CloudKart — Microservices E-Commerce (Spring Boot)

CloudKart is a learning/project template for building an **e-commerce system using microservices** in **Java + Spring Boot**. It demonstrates service boundaries, API design, resiliency patterns, and environment-friendly configs (dev/prod). ([GitHub][1])

> Status: Initial scaffold present (`pom.xml`, `src/`, Maven wrapper). Endpoints and services are being added incrementally. ([GitHub][1])

---

## ✨ Goals / What You’ll Learn

* Clear **service boundaries** (catalog, cart, order, payment, inventory)
* **API gateway** + **service discovery** for routing & discovery
* **Resiliency** (timeouts, retries, circuit breakers) and **idempotency**
* **Persistence** with JPA and migrations
* **Testing** (unit, slice, integration)
* **Observability** (logs, metrics, traces) — optional add-ons

---

## 🧱 Tech Stack

* **Language:** Java 17+
* **Framework:** Spring Boot 3.x (Web, Validation, Data JPA)
* **Microservices:** Spring Cloud (Gateway, Eureka/Discovery, OpenFeign/RestClient)
* **DB:** PostgreSQL/MySQL (prod), H2 (dev)
* **Build:** Maven (wrapper included)
* **Testing:** JUnit 5, Mockito, Spring Boot Test
* **Optional:** Resilience4j, Testcontainers, Flyway/Liquibase, Prometheus + Grafana, Zipkin/Tempo

> GitHub shows the project is Java. Add/remove items to match your actual code as services land. ([GitHub][1])

---

## 🧭 Service Design (suggested modules)

| Service                      | Purpose                                  | Key Endpoints (examples)                                    |
| ---------------------------- | ---------------------------------------- | ----------------------------------------------------------- |
| **catalog-service**          | Products, categories, pricing, search    | `GET /api/catalog/items`, `POST /api/catalog/items`         |
| **cart-service**             | User carts, add/remove/update items      | `GET /api/carts/{userId}`, `POST /api/carts/{userId}/items` |
| **order-service**            | Place orders from carts, order lifecycle | `POST /api/orders`, `GET /api/orders/{id}`                  |
| **payment-service (mock)**   | Initiate + verify payment, idempotency   | `POST /api/payments`, callback `/api/payments/{id}/verify`  |
| **inventory-service**        | Stock levels, reservations, releases     | `GET /api/inventory/{sku}`                                  |
| **gateway**                  | Single entry point, routing, CORS        | `/{service}/…` proxied routes                               |
| **discovery**                | Service registry (Eureka/Consul)         | N/A                                                         |
| **config-server** (optional) | Centralized config                       | N/A                                                         |

**Simple architecture (ASCII):**

```
[ client ] -> [ API Gateway ] -> { catalog | cart | order | payment | inventory }
                           \-> [ Discovery ]
                           \-> [ Config Server ] (optional)

Services -> DBs (per service)   +   Observability: metrics/logs/traces (optional)
```

---

## 🚀 Getting Started (Dev, quick)

### Prerequisites

* Java 17+, Maven 3.8+
* Docker (optional, for DBs/Zipkin/Prometheus)

### Clone & run a single service (scaffold)

```bash
git clone https://github.com/chaitanyashree25/CloudKart.git
cd CloudKart
./mvnw spring-boot:run
```

### Suggested `application.yml` (dev profile example)

```yaml
spring:
  datasource:
    url: jdbc:h2:mem:cloudkart;DB_CLOSE_DELAY=-1;MODE=PostgreSQL
    driverClassName: org.h2.Driver
    username: sa
    password:
  jpa:
    hibernate:
      ddl-auto: update
    show-sql: true

server:
  port: 8080
```

> When you split into multiple services, use different ports (e.g., 8081..8085), register with discovery, and route via the gateway.

---

## 🔌 API Contracts (examples to adapt)

### Catalog

* `GET /api/catalog/items?category=&q=`
* `GET /api/catalog/items/{id}`
* `POST /api/catalog/items` *(ADMIN)*

### Cart

* `GET /api/carts/{userId}`
* `POST /api/carts/{userId}/items` `{ "itemId": "SKU123", "qty": 2 }`
* `PATCH /api/carts/{userId}/items/{itemId}` `{ "qty": 3 }`
* `DELETE /api/carts/{userId}/items/{itemId}`

### Orders

* `POST /api/orders` `{ "userId": 1, "addressId": 10, "paymentMode": "UPI" }`
* `GET /api/orders/{id}`
* `PATCH /api/orders/{id}/status` *(ops/admin)*

### Payments (mock)

* `POST /api/payments` `{ "orderId": 123, "amount": 49900, "currency": "INR" }`
* `POST /api/payments/{id}/verify` (callback simulation)

**Validation**: use `@NotNull`, `@Positive`, `@Size`, and enforce server-side pricing totals (never trust client totals).

---

## 🧪 Testing

```bash
./mvnw test
```

* **Unit tests:** services, mappers, validators
* **Slice tests:** `@WebMvcTest`, `@DataJpaTest`
* **Integration:** Testcontainers (spin up Postgres & run repo/service tests)

**Example dependency (Testcontainers + Postgres):**

```xml
<dependency>
  <groupId>org.testcontainers</groupId>
  <artifactId>postgresql</artifactId>
  <scope>test</scope>
</dependency>
```

---

## 🧰 Dev Tooling (optional but recommended)

* **OpenAPI/Swagger UI:** springdoc-openapi → `/swagger-ui.html`
* **Resilience4j:** timeouts, retries, circuit breakers on downstream calls
* **Flyway/Liquibase:** versioned schema changes per service
* **Observability:**

  * Metrics: Micrometer + Prometheus
  * Traces: Zipkin/Tempo + OpenTelemetry
  * Logs: Structured JSON + correlation IDs

---

## 🧑‍🍳 Data Model (starter idea)

* **Catalog**: `Product(id, sku, title, price, category, stock)`
* **Cart**: `Cart(userId)`, `CartItem(cartId, productId, qty, priceSnapshot)`
* **Order**: `Order(id, userId, total, status, createdAt)`, `OrderItem(orderId, productId, qty, price)`
* **Inventory**: `Stock(productId, qtyAvailable, reserved)`
* **Payment**: `Payment(id, orderId, amount, status, txnRef)`

---

## 🗺️ Roadmap

* [ ] Split monorepo into modules/services (catalog, cart, order, payment, inventory, gateway, discovery)
* [ ] Add OpenAPI specs for each service
* [ ] Add Resilience4j policies and idempotency keys
* [ ] Add Docker Compose (services + DBs + Zipkin + Prometheus)
* [ ] CI: GitHub Actions (build, test) + code coverage badge
* [ ] AuthN/Z (JWT, roles), rate limiting at gateway

---

## 📂 Project Layout (suggested when you split into modules)

```
cloudkart/
  gateway/
  discovery/
  catalog-service/
  cart-service/
  order-service/
  payment-service/
  inventory-service/
  common-libs/        # shared DTOs, error model (avoid DB sharing)
```

---

## 🤝 Contributing

This is a learning project. PRs that add **small, focused** improvements (tests, docs, service scaffolds) are welcome.

## 📜 License

Choose one (MIT/Apache-2.0) and add `LICENSE`.


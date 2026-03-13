# ⚡ HotelCore — Concurrent Hotel Booking Engine

> A **high-concurrency hotel booking engine** built with Java Spring Boot — engineered to handle real-world problems like race conditions in inventory reservation, fraud-safe payment confirmation, and a multi-layered dynamic pricing pipeline using the Decorator design pattern.
>
> Not a tutorial project. Built to solve the same engineering challenges that power **Booking.com, OYO, and Airbnb** at scale.

---

## 🚀 Why This Project Stands Out

Most backend projects are simple CRUD apps. This one solves **real engineering problems**:

| Problem | Solution Used |
|---|---|
| Two users booking the same room simultaneously | Pessimistic Locking (`PESSIMISTIC_WRITE`) |
| Prices should change based on demand & season | Decorator Pattern — 4-layer pricing pipeline |
| Expensive search queries on large inventory | `HotelMinPrice` materialized summary table |
| Booking confirmed only after payment, never before | Stripe webhook — server-side confirmation only |
| JWT exceptions bypass `@ControllerAdvice` | `HandlerExceptionResolver` injected into filter |
| Stale prices across 1000s of rooms | Hourly `@Scheduled` batch pricing update job |

---

## 🏗️ Architecture

```
Client Request
      │
      ▼
  JWT Auth Filter  ──── validates token, injects User into SecurityContext
      │
      ▼
  Controller Layer  ──── REST endpoints (8 controllers, 26 APIs)
      │
      ▼
  Service Layer  ──── business logic, transactions, pricing
      │         │
      │         └──── PricingService (Decorator Chain)
      │                   Base → Surge → Occupancy → Urgency → Holiday
      ▼
  Repository Layer  ──── JPA + custom JPQL queries, pessimistic locks
      │
      ▼
  PostgreSQL  ──── 8 entities, unique constraints, inventory tables
      │
  Stripe API  ──── Checkout session + Webhook + Refund
```

---

## ✨ Key Features

### 👤 User-Facing
- Signup / Login with **JWT** (access token: 10 min, refresh token: 6 months via `HttpOnly` cookie)
- Search hotels by **city and date range** — paginated results with dynamic pricing
- View detailed hotel info with **room-wise availability and pricing**
- Complete **3-step booking flow**: Reserve → Add Guests → Pay
- Secure Stripe Checkout — booking confirmed only via **server-side webhook**
- Cancel booking with **automatic Stripe refund**
- View booking history and manage guest profiles

### 🏨 Hotel Manager (Admin)
- Create and manage hotels, rooms, and pricing
- Activate hotels (auto-generates 365-day inventory)
- Manage room inventory per date range (open/close/surge)
- View all bookings and generate **revenue reports** with date filters

### 💳 Payment Flow (Fraud-Safe)
```
User clicks Pay
     │
     ▼
POST /bookings/{id}/payments  ──── creates Stripe Checkout session
     │
     ▼
User completes payment on Stripe
     │
     ▼
Stripe → POST /webhook/payment  ──── validates Stripe-Signature header
     │
     ▼
Booking status → CONFIRMED  ──── only server can confirm, never frontend
```
> ⚠️ Booking is **NEVER confirmed from the frontend**. Only the verified Stripe webhook confirms it. This prevents payment fraud and tampering.

---

## 🧠 Engineering Deep Dive

### 1. Decorator Pattern — Dynamic Pricing Pipeline
Prices are not static. Every room's price is calculated by chaining 4 strategies:

```java
PricingStrategy chain = new BasePricingStrategy();
chain = new SurgePricingStrategy(chain);       // admin-controlled surge multiplier
chain = new OccupancyPricingStrategy(chain);   // +20% if occupancy > 80%
chain = new UrgencyPricingStrategy(chain);     // +15% if booking within 7 days
chain = new HolidayPricingStrategy(chain);     // +25% on holidays
```
Each strategy **wraps the previous one** (Decorator Pattern). Adding a new pricing rule requires zero changes to existing code (Open/Closed Principle).

### 2. Pessimistic Locking — Preventing Double Booking
When two users try to book the same room at the same time, only one wins:

```java
@Lock(LockModeType.PESSIMISTIC_WRITE)
List<Inventory> findAndLockAvailableInventory(roomId, startDate, endDate, roomsCount);
```
The database row is locked for the duration of the transaction. The second request waits, then finds zero availability and fails gracefully.

### 3. HotelMinPrice — Search Performance Optimization
Searching "hotels in Delhi from Dec 1–5" across millions of inventory rows would be slow. Instead:
- A separate `HotelMinPrice` table stores the **cheapest price per hotel per day**
- Search queries run on this **lightweight summary table** — not the full inventory
- This table is refreshed every hour by a scheduled batch job

### 4. Hourly Batch Pricing Scheduler
```java
@Scheduled(cron = "0 0 * * * *")  // every hour
public void updatePrices() {
    // Fetch hotels in pages of 100 (batch processing)
    // Recalculate dynamic prices for all rooms
    // Rebuild HotelMinPrice summary table in bulk
}
```
Processes hotels in **batches of 100** using pagination to avoid memory issues.

### 5. JWT Exception Handling in Security Filter
JWT validation errors occur **inside the filter**, before `@RestControllerAdvice` can handle them. Standard Spring would return an unformatted error. Fix:

```java
// Inject HandlerExceptionResolver into the JWT filter
handlerExceptionResolver.resolveException(request, response, null, jwtException);
```
This routes the exception through the normal exception handler, returning a clean JSON error response.

---

## 📡 Complete API Reference

### 🔐 Auth (3 APIs)
| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/auth/signup` | Register new user |
| POST | `/auth/login` | Login — returns JWT + refresh token cookie |
| POST | `/auth/refresh` | Get new access token using refresh token cookie |

### 🏨 Browse Hotels — Public (2 APIs)
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/hotels/search` | Search hotels by city, dates, rooms (paginated) |
| GET | `/hotels/{id}/info` | Get hotel details with room pricing |

### 📅 Booking Flow (5 APIs)
| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/bookings/init` | Initialize booking — reserves inventory |
| POST | `/bookings/{id}/addGuests` | Add guest details to booking |
| POST | `/bookings/{id}/payments` | Create Stripe checkout session |
| POST | `/bookings/{id}/cancel` | Cancel booking + trigger Stripe refund |
| GET | `/bookings/{id}/status` | Check current booking status |

### 👤 User Profile (6 APIs)
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/users/profile` | Get my profile |
| PATCH | `/users/profile` | Update profile |
| GET | `/users/myBookings` | Get all my bookings |
| GET | `/users/guests` | Get saved guest profiles |
| POST | `/users/guests` | Add new guest profile |
| PUT | `/users/guests/{id}` | Update guest profile |
| DELETE | `/users/guests/{id}` | Remove guest profile |

### 🏗️ Admin — Hotels (6 APIs)
| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/admin/hotels` | Create new hotel |
| GET | `/admin/hotels` | Get all my hotels |
| GET | `/admin/hotels/{id}` | Get hotel by ID |
| PUT | `/admin/hotels/{id}` | Update hotel |
| DELETE | `/admin/hotels/{id}` | Delete hotel |
| PATCH | `/admin/hotels/{id}/activate` | Activate hotel (creates 365-day inventory) |
| GET | `/admin/hotels/{id}/bookings` | Get all bookings for a hotel |
| GET | `/admin/hotels/{id}/reports` | Revenue report with date filters |

### 🛏️ Admin — Rooms (5 APIs)
| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/admin/hotels/{id}/rooms` | Add room to hotel |
| GET | `/admin/hotels/{id}/rooms` | List all rooms |
| GET | `/admin/hotels/{id}/rooms/{roomId}` | Get room by ID |
| PUT | `/admin/hotels/{id}/rooms/{roomId}` | Update room |
| DELETE | `/admin/hotels/{id}/rooms/{roomId}` | Delete room |

### 📦 Admin — Inventory (2 APIs)
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/admin/inventory/rooms/{id}` | Get inventory by room |
| PATCH | `/admin/inventory/rooms/{id}` | Update inventory (surge, open/close dates) |

### 💳 Webhook (1 API)
| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/webhook/payment` | Stripe webhook — confirms or cancels booking |

**Total: 26 REST APIs**

---

## 🗄️ Database Design

```
app_user          ← User accounts, roles (GUEST / HOTEL_MANAGER)
hotel             ← Hotel info, owner, city, amenities
room              ← Room type, base price, capacity
inventory         ← Availability per room per date (with surge factor)
hotel_min_price   ← Cheapest price per hotel per day (search optimization)
booking           ← Booking records with status lifecycle
guest             ← Saved guest profiles
booking_guest     ← Many-to-many: bookings ↔ guests
```

**Booking Status Lifecycle:**
```
RESERVED → GUESTS_ADDED → PAYMENTS_PENDING → CONFIRMED
                                           └──→ CANCELLED
```

---

## 🛠️ Tech Stack

| Category | Technology |
|---|---|
| Language | Java 17 |
| Framework | Spring Boot 3 |
| Security | Spring Security + JWT (JJWT) |
| Database | PostgreSQL |
| ORM | Spring Data JPA / Hibernate |
| Payments | Stripe API (Checkout + Webhooks + Refunds) |
| API Docs | Swagger / OpenAPI (`@Operation` annotations) |
| Mapping | ModelMapper |
| Build | Maven |
| Utils | Lombok |

---

## ⚙️ Local Setup

### Prerequisites
- Java 17+
- PostgreSQL
- Maven
- Stripe account (for payment testing)

### Steps

**1. Clone the repository**
```bash
git clone https://github.com/ShivamNayak-dev/airBnbApp.git
cd airBnbApp
```

**2. Create PostgreSQL database**
```sql
CREATE DATABASE airbnb_db;
```

**3. Configure environment variables**  
Create `src/main/resources/application.properties` or set environment variables:
```properties
spring.datasource.url=jdbc:postgresql://localhost:5432/airbnb_db
spring.datasource.username=your_db_username
spring.datasource.password=your_db_password

jwt.secretKey=your_jwt_secret_key_min_32_characters

stripe.secret.key=sk_test_your_stripe_secret_key
stripe.webhook.secret=whsec_your_webhook_secret

frontend.url=http://localhost:3000
```

**4. Run the application**
```bash
mvn spring-boot:run
```

**5. Access Swagger UI**
```
http://localhost:8080/swagger-ui.html
```

---

## 📁 Project Structure

```
src/main/java/com/codingshuttle/projects/airBnbApp/
├── controller/        # 8 REST controllers — 26 APIs
├── service/           # Business logic + PricingUpdateService (scheduler)
├── strategy/          # Pricing strategies (Decorator Pattern)
├── repository/        # JPA repositories + custom JPQL queries
├── entity/            # 8 JPA entities + 4 enums
├── dto/               # Request/Response DTOs
├── security/          # JWT filter, JWT service, WebSecurityConfig
├── config/            # CORS, Stripe, ModelMapper config
├── advice/            # Global exception handler + response wrapper
└── exception/         # Custom exception classes
```

---

## 🔮 Planned Improvements

- [ ] Scheduled job to auto-release expired bookings (currently manual check only)
- [ ] Real holiday detection API for `HolidayPricingStrategy`
- [ ] Unit and integration tests (JUnit 5 + Mockito)
- [ ] Docker + Docker Compose setup
- [ ] Redis caching for search results
- [ ] Email notifications on booking confirmation/cancellation

---

## 📄 License

MIT License — free to use, modify, and distribute.

---

*Built by [Shivam Nayak](https://github.com/ShivamNayak-dev) | [LinkedIn](https://linkedin.com/in/your-linkedin)*

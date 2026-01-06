# 🏨 Hotel Booking Management System

A backend system that simulates how real hotel booking platforms (like OYO, Booking.com, Airbnb) work —  
from hotel management and room availability to secure payments and booking workflows.

This project focuses on **backend architecture**, not UI.  
It is designed as a **production-ready system**:

- users can search hotels, check availability, and book rooms  
- admins can manage hotels, rooms, and inventory  
- payments are processed securely through Stripe  
- bookings are confirmed only after verified payment  
- concurrency is controlled so two people cannot book the same room

It demonstrates real concepts used in industry — authentication, transactions, webhooks, validation,
and layered architecture — instead of just CRUD APIs.

---



A **production-style hotel booking backend** with:

- Secure JWT authentication  
- Real-time room availability & inventory
- Dynamic pricing behavior  
- Safe booking workflows  
- Stripe payment integration (with webhooks)  
- Admin hotel/room management  
- Transactional concurrency control (prevents double booking)

Built using **Spring Boot, PostgreSQL, Stripe, and Spring Security** — designed like a real SaaS backend.

---

## 📸 Architecture Overview

> (Replace with your diagram if available)

![Architecture](docs/images/architecture.png)

```
Client → API → Controller → Service → Repository → PostgreSQL
                              │
                              └── Stripe (Checkout + Webhooks)
```

---

## ✨ Features

### 👤 User Features
- Register & Login (JWT)
- Browse hotels & available rooms
- Add guest details
- Create bookings
- Secure Stripe payments
- View booking history
- Cancel bookings

---

### 🏨 Admin Features
- Create & manage hotels
- Define rooms & pricing
- Manage room inventory (per date)
- Track and manage bookings

---

### 💳 Payments
- Stripe Checkout integration
- Payment verification via webhook
- Confirm booking **only after successful payment**
- Safe handling of failed payments  
- Refund-ready design

> Booking is NEVER confirmed from frontend — **only webhook** confirms it.  
> Prevents tampering & fraud.

---

### 🔒 Security
- JWT authentication
- Role-based access control
- Validation on all inputs
- Secrets stored via environment variables
- Restricted admin APIs

---

## 🛠 Tech Stack

**Backend:** Spring Boot 3, Spring Security, Hibernate/JPA  
**Database:** PostgreSQL  
**Payments:** Stripe API + Webhooks  
**Other:** Lombok, Maven, Validation API

---

## 🧠 Booking Flow (End-to-End)

![Booking Flow](docs/images/booking-flow.png)

1️⃣ User selects hotel + rooms  
2️⃣ System checks and locks availability  
3️⃣ Booking saved as **PENDING**  
4️⃣ Stripe payment session generated  
5️⃣ User completes payment  
6️⃣ Stripe sends webhook  
7️⃣ Booking becomes **CONFIRMED**  
8️⃣ Failed payment → booking cancelled safely

---

## ⚙️ Environment Variables

Create `.env` or configure host:

```
DB_URL=jdbc:postgresql://localhost:5432/airbnb
DB_USERNAME=postgres
DB_PASSWORD=your_password

JWT_SECRET=your_jwt_secret

STRIPE_SECRET=sk_test_xxx
STRIPE_WEBHOOK_SECRET=whsec_xxx
```

> 🚫 **Never commit secrets to GitHub.**

---

## ▶️ Local Setup (Development)

### 1️⃣ Clone the project
```bash
git clone https://github.com/<your-username>/hotel-booking-Management-system.git
cd hotel-booking-Management-system
```

### 2️⃣ Create PostgreSQL database
```sql
CREATE DATABASE airbnb;
```

### 3️⃣ Run backend
```bash
mvn spring-boot:run
```

API base URL:

```
http://localhost:8080/api/v1
```

---

## 📡 API Overview (High Level)

### 🔐 Auth
```
POST /auth/signup
POST /auth/login
GET  /auth/me
```

### 🏨 Hotels
```
GET /hotels/search
GET /hotels/{id}
```

### 🛒 Bookings
```
POST /bookings/init
POST /bookings/{id}/payments
GET  /bookings/{id}/status
POST /bookings/{id}/cancel
```

### 🛠 Admin APIs
```
/admin/hotels/**
/admin/rooms/**
/admin/inventory/**
```

👉 Full details available inside controller classes.

---

## 🏗 Project Structure

```text
src/
 ├── controller/        # REST endpoints
 ├── service/           # Business logic
 ├── repository/        # Data persistence layer
 ├── dto/               # Request/response models
 ├── entity/            # JPA entity classes
 ├── security/          # JWT & auth filters
 ├── config/            # Application configuration
 └── exception/         # Global exception handling
```

---

## 🧠 Key Engineering Concepts

### 🔁 Concurrency Control
Prevents multiple users from booking same room:

- transactions
- optimistic validation
- rollback on failure

> Only one booking succeeds — others safely fail.

---

### 💲 Dynamic Pricing Behavior
Displayed prices adjust based on:

- remaining room categories
- inventory availability
- admin price adjustments

Uses cached **HotelMinPrice** to avoid expensive queries.

---

### 🧯 Global Error Handling
Centralized handler returns clean responses like:

```
400 Bad Request
401 Unauthorized
404 Not Found
409 Conflict
500 Internal Server Error
```

---

## 🚀 Deployment

Supports deployment to:

- Render
- Railway
- AWS
- Docker (planned)

Make sure to configure environment variables.

---

## 🧭 Future Enhancements

- Docker & CI/CD
- Email/SMS booking notifications
- Full Admin dashboard UI
- Redis caching for search
- Automatic refunds
- Performance metrics & monitoring

---

## 📌 Notes

- This repository contains **backend only**
- Frontend can be developed independently and integrated via APIs

---

## 🤝 Contributing

1️⃣ Fork this repo  
2️⃣ Create feature branch  
3️⃣ Submit PR  

---

## 📜 License

MIT — free to use and modify.

---

## 🙏 Acknowledgements

Inspired by real hotel booking platforms & payment integrations.  
Thanks to Stripe, Spring & PostgreSQL communities.


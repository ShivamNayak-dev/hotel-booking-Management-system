🏨 Hotel Booking Management System

A complete hotel booking platform with authentication, room inventory, secure payments, and admin management — built using Spring Boot, JWT Security, Stripe Payments, and PostgreSQL.

Designed to behave like a real-world production system with layered architecture, validation, webhooks, and booking workflows.

🚀 Features
👤 User

Register & login (JWT)

Browse available hotels & rooms

Create bookings

Add guest details

Pay securely

View booking history

Cancel bookings

🏨 Admin

Manage hotels

Manage rooms & pricing

View booking reports

Control room availability (inventory)

💳 Payments

Create payment session

Verify Stripe webhook callbacks

Confirm booking after successful payment

Handle failed payments safely

🔒 Security

JWT authentication

Role-based access control

Request validation

Restricted admin APIs

🛠 Tech Stack

Spring Boot 3

Spring Security (JWT)

PostgreSQL

Hibernate / JPA

Stripe API

Lombok

Maven

🏗 Architecture
Client → Controller → Service → Repository → Database
                      |
                      └── Payment Gateway (Stripe)


Controllers handle REST APIs

Services contain business logic

Repositories access the database

Stripe webhooks update payment status safely

📡 API Overview (High Level)
🔐 Auth
POST /auth/signup
POST /auth/login
POST /auth/refresh

🛒 Booking
POST /bookings/init
POST /bookings/{id}/payments
GET  /bookings/{id}/status
POST /bookings/{id}/cancel

🏨 Hotels
GET /hotels/search
GET /hotels/{id}/info

🛠 Admin
/admin/hotels/**
/admin/rooms/**
/admin/inventory/**


👉 Full API details are inside the controller files.

⚙️ Environment Variables

Secrets should NOT be stored in source code.

DB_PASSWORD=...
JWT_SECRET=...
STRIPE_SECRET=...
STRIPE_WEBHOOK=...

▶️ How to Run (development)

1️⃣ Start PostgreSQL and create DB:

CREATE DATABASE airBnb;


2️⃣ Run backend:

mvn spring-boot:run


API base URL:

http://localhost:8080/api/v1

🧭 Future Enhancements

Docker support

Email booking notifications

Admin dashboard UI

Caching for hotel search

Load testing & metrics

📌 Notes

This project focuses on backend implementation only.
Frontend can be added as a separate service later.

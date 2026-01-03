🏨 Hotel Booking Management System

A complete hotel booking platform with authentication, room inventory, secure payments, and admin management — built using Spring Boot, JWT Security, Stripe Payments, and PostgreSQL.

Designed to behave like a real-world production system with layered architecture, validation, webhooks, and booking workflows.

🚀 Features:
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
Track booking reports
Control availability (inventory)

💳 Payments
Initialize payment session
Verify webhook callbacks
Confirm bookings after successful payment
Handle failed payments gracefully
Handle failed payments gracefully

🔒 Security
JWT authentication
Role-based access control
Input validation
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
 
                     
                     └→ Payment Gateway (Stripe)
📡 API Overview (high level)
Auth
/auth/login, /auth/signup, /auth/refresh

Booking
/bookings/init, /bookings/{id}/payments, /bookings/{id}/status, /bookings/{id}/cancel

Hotels
/hotels/search, /hotels/{id}/info

Admin
/admin/hotels/**, /admin/rooms/**, /admin/inventory/**

👉 Full API list is inside controllers.


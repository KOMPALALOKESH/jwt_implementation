# 🛡️ Spring Boot JWT Authentication (MySQL + Maven)

A fully functional **Spring Boot 3.1** project implementing **JWT-based Authentication & Authorization** with **MySQL**.
This project demonstrates how to secure REST APIs using **Spring Security**, **JWT tokens**, and **role-based access control (RBAC)**.

---

## 📁 Project Overview

### 🧱 Core Stack

| Component         | Technology              |
| ----------------- | ----------------------- |
| Language          | Java 17                 |
| Framework         | Spring Boot 3.1         |
| Build Tool        | Maven                   |
| Security          | Spring Security + JWT   |
| Database          | MySQL                   |
| ORM               | JPA (Hibernate)         |
| Password Encoding | BCrypt                  |
| Token Type        | JWT (Access Token only) |
| API Design        | RESTful APIs            |

---

## ⚙️ Project Modules & Structure

```
springboot-jwt-auth/
│
├── pom.xml
├── README.md
│
├── src/main/java/com/example/jwt/
│   ├── SpringbootJwtAuthApplication.java        # Application entry point
│   ├── config/                                  # Security configuration & filters
│   │   ├── SecurityConfig.java
│   │   └── JwtAuthenticationFilter.java
│   ├── controller/                              # REST Controllers
│   │   ├── AuthController.java
│   │   ├── PublicController.java
│   │   └── UserController.java
│   ├── dto/                                     # Data Transfer Objects
│   │   ├── LoginRequest.java
│   │   ├── RegisterRequest.java
│   │   └── AuthResponse.java
│   ├── entity/                                  # Database Entities
│   │   ├── User.java
│   │   └── Role.java
│   ├── repository/                              # Spring Data JPA Repositories
│   │   └── UserRepository.java
│   ├── service/                                 # Business Logic Layer
│   │   ├── UserService.java
│   │   └── CustomUserDetailsService.java
│   └── util/                                   # JWT Utility
│       └── JwtUtil.java
│
└── src/main/resources/
    ├── application.properties
    └── data.sql (optional seed data)
```

---

## 🧩 Application Flow (End-to-End)

1. **User registers** using `/api/auth/register`.

   * Password is encoded with BCrypt.
   * Assigned default role `ROLE_USER`.
   * Data stored in MySQL.
2. **User logs in** via `/api/auth/login`.

   * If credentials are valid, a **JWT token** is generated and returned.
3. **User accesses APIs** by including `Authorization: Bearer <token>` header.
4. **JwtAuthenticationFilter** validates token on each request.
5. **Spring Security** ensures role-based access (User/Admin).
6. **Public APIs** are accessible without token.

---

## 🧠 Layer-by-Layer Explanation

### 1️⃣ Entity Layer

#### `User.java`

* Represents a registered user.
* Fields:

  * `id`: Primary Key
  * `username`: Unique
  * `email`: Unique
  * `password`: Encrypted using BCrypt
  * `roles`: Set of roles (User/Admin)

#### `Role.java`

```java
public enum Role {
    ROLE_USER,
    ROLE_ADMIN
}
```

* Simple enum representing application roles.

---

### 2️⃣ Repository Layer

#### `UserRepository.java`

* Extends `JpaRepository<User, Long>`.
* Custom queries:

  * `findByUsername(String username)`
  * `existsByUsername(String username)`
  * `existsByEmail(String email)`

---

### 3️⃣ DTO Layer

Used for clean data exchange between client and server.

| DTO                 | Purpose                                                  |
| ------------------- | -------------------------------------------------------- |
| **RegisterRequest** | Captures registration details: username, email, password |
| **LoginRequest**    | Captures login credentials: username, password           |
| **AuthResponse**    | Returns JWT token after successful login                 |

---

### 4️⃣ Service Layer

#### `UserService.java`

* Handles user registration logic.
* Validates uniqueness (username/email).
* Encrypts passwords.
* Assigns `ROLE_USER` by default.
* Can also be used by Admin to create users.

#### `CustomUserDetailsService.java`

* Implements `UserDetailsService` for Spring Security.
* Loads user details (username, password, roles) from DB.
* Converts roles to `GrantedAuthority` for Spring Security.

---

### 5️⃣ Utility Layer

#### `JwtUtil.java`

Handles:

* Token generation
* Token validation
* Username extraction

Configuration:

* `jwt.secret` → secret key for signing tokens.
* `jwt.expiration-ms` → token expiry time (e.g., 1 hour).

---

### 6️⃣ Configuration Layer

#### `SecurityConfig.java`

* Configures Spring Security.
* Disables session (stateless).
* Enables JWT filter.
* Configures endpoint access rules:

  * `/api/auth/**`, `/api/public/**` → permit all
  * `/api/admin/**` → only `ROLE_ADMIN`
  * All others → authenticated
* Defines beans:

  * `PasswordEncoder`
  * `AuthenticationManager`
  * `JwtAuthenticationFilter`

#### `JwtAuthenticationFilter.java`

* Custom `OncePerRequestFilter`.
* Extracts `Authorization` header.
* Validates and parses JWT.
* Sets authentication context if valid.

---

### 7️⃣ Controller Layer

#### `AuthController.java`

Handles Authentication APIs:

| Method | Endpoint             | Description                     | Access |
| ------ | -------------------- | ------------------------------- | ------ |
| `POST` | `/api/auth/register` | Register new user               | Public |
| `POST` | `/api/auth/login`    | Authenticate & return JWT token | Public |

#### `PublicController.java`

Simple open endpoints (no token required):

| Method | Endpoint           | Description            |
| ------ | ------------------ | ---------------------- |
| `GET`  | `/api/public/info` | Public endpoint (test) |

#### `UserController.java`

Secured user & admin APIs:

| Method | Endpoint               | Description              | Role       |
| ------ | ---------------------- | ------------------------ | ---------- |
| `GET`  | `/api/user/profile`    | Fetch user profile info  | USER/ADMIN |
| `POST` | `/api/admin/create`    | Admin creates a new user | ADMIN      |
| `GET`  | `/api/admin/dashboard` | Admin dashboard          | ADMIN      |

---

### 8️⃣ Database Layer

#### `application.properties`

```properties
spring.datasource.url=jdbc:mysql://localhost:3306/jwt_db?useSSL=false
spring.datasource.username=root
spring.datasource.password=your_password
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true

jwt.secret=ReplaceThisSecretWithAStrongOneChangeMe
jwt.expiration-ms=3600000
```

#### Default Admin (Auto-created)

At startup, the app seeds:

```plaintext
username: admin
password: admin123
role: ROLE_ADMIN
```

---

## 🚀 How to Run the Project

### 🧰 Prerequisites

* Java 17
* Maven 3+
* MySQL (running locally)

### ⚙️ Steps

1. Create database:

   ```sql
   CREATE DATABASE jwt_db;
   ```
2. Update MySQL credentials in `application.properties`.
3. Run the app:

   ```bash
   mvn spring-boot:run
   ```
4. App runs on:
   👉 `http://localhost:8080`

---

## 🔑 API Endpoints Summary

| API                    | Method | Access  | Description                |
| ---------------------- | ------ | ------- | -------------------------- |
| `/api/auth/register`   | POST   | Public  | Register a new user        |
| `/api/auth/login`      | POST   | Public  | Login & get JWT token      |
| `/api/public/info`     | GET    | Public  | Test endpoint              |
| `/api/user/profile`    | GET    | Secured | User profile               |
| `/api/admin/dashboard` | GET    | Admin   | Admin-only data            |
| `/api/admin/create`    | POST   | Admin   | Create new users with role |

---

## 🧪 Example Workflow (Using Postman)

1. **Register User**

   ```
   POST /api/auth/register
   Body:
   {
     "username": "lokesh",
     "email": "lokesh@example.com",
     "password": "123456"
   }
   ```

2. **Login User**

   ```
   POST /api/auth/login
   Body:
   {
     "username": "lokesh",
     "password": "123456"
   }
   ```

   → Response: `{ "token": "eyJhbGciOi..." }`

3. **Access Secured API**

   ```
   GET /api/user/profile
   Header: Authorization: Bearer eyJhbGciOi...
   ```

4. **Admin Create User**

   ```
   POST /api/admin/create
   Header: Authorization: Bearer <admin_token>
   Body:
   {
     "username": "newuser",
     "email": "new@x.com",
     "password": "userpass"
   }
   ```

---

## 🧰 Key Features

✅ JWT Token-based authentication
✅ BCrypt password hashing
✅ Role-based authorization (Admin/User)
✅ Stateless session handling
✅ Secure endpoints with filters
✅ Clean layered architecture
✅ MySQL persistence
✅ Auto-admin creation

---

## 🧩 Future Enhancements

* Add **JWT refresh token** support
* Add **logout/revoke** functionality
* Implement **exception handling** (global error handler)
* Swagger/OpenAPI documentation
* Dockerize (App + MySQL)
* Unit/Integration Tests (JUnit + Mockito)

---

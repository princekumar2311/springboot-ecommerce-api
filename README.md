# 🛒 Spring Boot E-Commerce API

A RESTful e-commerce backend built with **Spring Boot**, **Spring Security (JWT)** and **PostgreSQL**. It supports role-based access (User / Seller / Admin), category and product management, a shopping cart, addresses, order placement with payment details, product image upload, and an admin analytics endpoint. Interactive API docs are available through Swagger UI.

![Java](https://img.shields.io/badge/Java-17-orange)
![Spring Boot](https://img.shields.io/badge/Spring%20Boot-4.1.1-brightgreen)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-DB-blue)
![JWT](https://img.shields.io/badge/Auth-JWT-purple)
![Maven](https://img.shields.io/badge/Build-Maven-red)

---

## ✨ Features

- **Authentication & Authorization**
    - Sign up, sign in and sign out
    - JWT is issued as a cookie on sign-in and is also accepted as an `Authorization: Bearer <token>` header
    - Role-based access control: `ROLE_USER`, `ROLE_SELLER`, `ROLE_ADMIN`
    - Passwords hashed with BCrypt
    - Roles and demo users are created automatically on startup
- **Categories** – public paginated listing; admin create, update and delete
- **Products**
    - Public listing with pagination, sorting, keyword search and category filter
    - Admin and seller product creation, update and delete
    - Sellers can list the products they created
    - Product image upload (multipart); files are saved to the local `images/` folder
    - Special price is calculated automatically from price and discount
    - Duplicate product names within the same category are rejected
- **Cart** – create a cart, add products, increase/decrease quantity, remove items, view your own cart
- **Addresses** – create, read, update and delete addresses for the logged-in user
- **Orders** – place an order from the cart with payment details; on success the order is saved with status `Accepted`, product stock is reduced and the cart is emptied; admin and seller order lists; order status updates
- **Analytics** – admin endpoint returning product count, total orders and total revenue
- **Validation & error handling** – Bean Validation plus a global exception handler (`APIException`, `ResourceNotFoundException`)
- **API documentation** – Swagger UI / OpenAPI 3 via springdoc

---

## 🧰 Tech Stack

| Layer | Technology |
|---|---|
| Language | Java 17 |
| Framework | Spring Boot 4.1.1 (Web MVC, Data JPA, Validation) |
| Security | Spring Security, JWT (jjwt 0.13.0) |
| Database | PostgreSQL (Hibernate `ddl-auto=update`) |
| Mapping | ModelMapper 3.0.0 |
| API Docs | springdoc-openapi 3.0.0 (Swagger UI) |
| Payments | Stripe Java SDK 29.3.0 (dependency included, see Known Limitations) |
| Utilities | Lombok |
| Build | Maven (wrapper included) |

---

## 📁 Project Structure

```
src/main/java/com/ecommerce/project
├── config/          # AppConfig (ModelMapper), AppConstants, SwaggerConfig
├── controller/      # Auth, Category, Product, Cart, Address, Order, Analytics
├── exceptions/      # APIException, ResourceNotFoundException, global handler
├── model/           # JPA entities: User, Role, Category, Product, Cart, CartItem,
│                    #   Address, Order, OrderItem, Payment
├── payload/         # DTOs and response wrappers
├── repositories/    # Spring Data JPA repositories
├── security/
│   ├── jwt/         # JwtUtils, AuthTokenFilter, AuthEntryPointJwt
│   ├── request/     # LoginRequest, SignupRequest
│   ├── response/    # UserInfoResponse, MessageResponse
│   ├── services/    # UserDetailsImpl, UserDetailsServiceImpl
│   └── WebSecurityConfig.java
├── service/         # Service interfaces and implementations
└── util/            # AuthUtil
```

---

## 🚀 Getting Started

### Prerequisites

- Java 17+
- Maven (or the included `mvnw` wrapper)
- PostgreSQL 13+

### 1. Clone the repository

```bash
git clone https://github.com/princekumar2311/springboot-ecommerce-api.git
cd springboot-ecommerce-api
```

### 2. Create the database

```sql
CREATE DATABASE ecommerce;
```

### 3. Configure the application

Edit `src/main/resources/application.properties`:

```properties
spring.datasource.url=jdbc:postgresql://localhost:5432/ecommerce
spring.datasource.username=postgres
spring.datasource.password=<your-db-password>

spring.app.jwtSecret=<a-long-random-secret>
spring.app.jwtExpirationMs=300000000
spring.ecom.app.jwtCookieName=springBootEcom

project.image=images/
image.base.url=http://localhost:9090/images

server.port=9090
```

Notes:

- `spring.jackson.deserialization.fail-on-null-for-primitives=false` and `server.error.include-message=always` are already set in the project, so a `null` for a numeric field becomes `0` and error responses include the message.

> ⚠️ **Never commit real passwords or secrets.** Use environment variables:
> ```properties
> spring.datasource.password=${DB_PASSWORD}
> spring.app.jwtSecret=${JWT_SECRET}
> ```

### 4. Run

```bash
./mvnw spring-boot:run
```

On Windows:

```bash
mvnw.cmd spring-boot:run
```

The API starts at **http://localhost:9090**.

### 5. Open the API docs

**http://localhost:9090/swagger-ui/index.html**

---

## 👤 Default Users (development only)

Created automatically on startup:

| Username | Email | Password | Roles |
|---|---|---|---|
| `user1` | user1@example.com | `password1` | USER |
| `seller1` | seller1@example.com | `password2` | SELLER |
| `admin` | admin@example.com | `adminPass` | USER, SELLER, ADMIN |

> The startup seeder resets the roles of these accounts on every restart, so changing `user1` into an admin in the database will not last. To get another admin, register a new username through signup. **Change or remove these accounts before deploying.**

### Signup roles

`POST /api/auth/signup` accepts an optional `role` array:

| Value in `role` | Role assigned |
|---|---|
| `"admin"` | ROLE_ADMIN |
| `"seller"` | ROLE_SELLER |
| anything else, or omitted | ROLE_USER |

Username must be 3–20 characters and password 6–40 characters. Signup fails if the username or email already exists, so use a **new** username when you want a new admin.

---

## 🔌 API Endpoints

Base URL: `http://localhost:9090/api`

### Access rules

| Path pattern | Who can access |
|---|---|
| `/api/auth/**`, `/api/public/**` | Everyone |
| `/api/seller/**` | Seller, Admin |
| `/api/admin/**` | Admin only |
| Everything else | Any authenticated user |

### Authentication

| Method | Endpoint | Description |
|---|---|---|
| POST | `/auth/signup` | Register a new user |
| POST | `/auth/signin` | Sign in (sets the JWT cookie and returns user info with the token) |
| POST | `/auth/signout` | Sign out (clears the cookie) |
| GET | `/auth/user` | Current user details (id, username, email, roles) |
| GET | `/auth/username` | Current username |
| GET | `/auth/sellers` | List sellers (paginated, `pageNumber`) |

### Categories

| Method | Endpoint | Access | Description |
|---|---|---|---|
| GET | `/public/categories` | Public | List categories (`pageNumber`, `pageSize`, `sortBy`, `sortOrder`) |
| POST | `/admin/categories` | Admin | Create a category |
| PUT | `/admin/categories/{categoryId}` | Admin | Update a category |
| DELETE | `/admin/categories/{categoryId}` | Admin | Delete a category |

### Products

| Method | Endpoint | Access | Description |
|---|---|---|---|
| GET | `/public/products` | Public | List products (`keyword`, `category`, `pageNumber`, `pageSize`, `sortBy`, `sortOrder`) |
| GET | `/public/categories/{categoryId}/products` | Public | Products in a category |
| GET | `/public/products/keyword/{keyword}` | Public | Search by keyword |
| POST | `/admin/categories/{categoryId}/product` | Admin | Add a product |
| POST | `/seller/categories/{categoryId}/product` | Seller/Admin | Add a product |
| PUT | `/admin/products/{productId}` | Admin | Update a product |
| PUT | `/seller/products/{productId}` | Seller/Admin | Update a product |
| PUT | `/admin/products/{productId}/image` | Admin | Upload product image (multipart, field `image`) |
| PUT | `/seller/products/{productId}/image` | Seller/Admin | Upload product image |
| DELETE | `/admin/products/{productId}` | Admin | Delete a product |
| DELETE | `/seller/products/{productId}` | Seller/Admin | Delete a product |
| GET | `/admin/products` | Admin | All products (admin view) |
| GET | `/seller/products` | Seller/Admin | Products created by the logged-in seller |

### Cart

| Method | Endpoint | Description |
|---|---|---|
| POST | `/cart/create` | Create / sync a cart |
| POST | `/carts/products/{productId}/quantity/{quantity}` | Add a product to the cart |
| GET | `/carts` | List all carts |
| GET | `/carts/users/cart` | Current user's cart |
| PUT | `/cart/products/{productId}/quantity/{operation}` | `delete` decreases by 1, any other value (e.g. `add`) increases by 1 |
| DELETE | `/carts/{cartId}/product/{productId}` | Remove a product from the cart |

### Addresses

| Method | Endpoint | Description |
|---|---|---|
| POST | `/addresses` | Create an address |
| GET | `/addresses` | List all addresses |
| GET | `/addresses/{addressId}` | Get an address |
| GET | `/users/addresses` | Current user's addresses |
| PUT | `/addresses/{addressId}` | Update an address |
| DELETE | `/addresses/{addressId}` | Delete an address |

### Orders

| Method | Endpoint | Access | Description |
|---|---|---|---|
| POST | `/order/users/payments/{paymentMethod}` | Authenticated | Place an order from the cart |
| GET | `/admin/orders` | Admin | All orders (paginated) |
| GET | `/seller/orders` | Seller/Admin | Seller orders (paginated) |
| PUT | `/admin/orders/{orderId}/status` | Admin | Update order status |
| PUT | `/seller/orders/{orderId}/status` | Seller/Admin | Update order status |

### Analytics

| Method | Endpoint | Access | Description |
|---|---|---|---|
| GET | `/admin/app/analytics` | Admin | Product count, total orders, total revenue |

---

## 🧪 Example Requests (Postman / cURL)

### Sign in as admin

```http
POST /api/auth/signin
Content-Type: application/json

{
  "username": "admin",
  "password": "adminPass"
}
```

The response sets a cookie named `springBootEcom` (path `/api`, 24-hour lifetime). Postman stores and re-sends it automatically. You can also copy the `jwtToken` from the response and send `Authorization: Bearer <token>`.

### Create a category (Admin)

```http
POST /api/admin/categories
Content-Type: application/json

{ "categoryName": "Electronics" }
```

### Add a product (Admin)

```http
POST /api/admin/categories/1/product
Content-Type: application/json

{
  "productName": "iPhone 15",
  "description": "Latest Apple smartphone",
  "quantity": 10,
  "price": 80000,
  "discount": 10
}
```

Do **not** send `productId`, `image` or `specialPrice` when creating a product. The server generates the ID, sets a default image and calculates the special price.

### Place an order

```http
POST /api/order/users/payments/Online
Content-Type: application/json

{
  "addressId": 1,
  "pgName": "Stripe",
  "pgPaymentId": "pi_123",
  "pgStatus": "succeeded",
  "pgResponseMessage": "Payment successful"
}
```

---

## ✅ Validation Rules

| Field | Rule |
|---|---|
| Category name | Not blank, at least 5 characters |
| Product name | Not blank, at least 3 characters |
| Product description | Not blank, at least 6 characters |
| Address street / building / pincode | At least 5 characters each |
| Address city | At least 4 characters |
| Address state / country | At least 2 characters each |
| Payment method | At least 4 characters |
| Signup username | 3–20 characters |
| Signup password | 6–40 characters |

---

## ⚠️ Known Limitations

- **Open admin signup.** Anyone can call `/api/auth/signup` with `"role": ["admin"]` and become an admin. Restrict this before deploying (for example, ignore the `role` field on public signup and add an admin-only endpoint for creating admins/sellers).
- **Uploaded images are not served over HTTP.** Images are saved to the `images/` folder, but there is no resource handler mapping `/images/**` to that folder. To serve them, add this class under `config/`:

  ```java
  @Configuration
  public class WebMvcConfig implements WebMvcConfigurer {
      @Override
      public void addResourceHandlers(ResourceHandlerRegistry registry) {
          registry.addResourceHandler("/images/**").addResourceLocations("file:images/");
      }
  }
  ```
- **Cookie flags.** The JWT cookie is created with `httpOnly=false` and `secure=false`. Set both to `true` when deploying over HTTPS.
- **CORS** is not configured explicitly, and the `frontend.url` property is unused. Add a CORS configuration before connecting a browser frontend on another origin.
- **Stripe.** The Stripe SDK is a dependency, but there is no Stripe payment-intent endpoint yet. Payment gateway details are sent by the client and stored with the order.
- **Secrets in `application.properties`.** Move the database password and JWT secret to environment variables.

---

## 🩺 Troubleshooting

| Symptom | Likely cause |
|---|---|
| `401 Unauthorized` on `/api/admin/...` | Not signed in, or the cookie/token expired |
| `403 Forbidden` | Signed in without the required role (use `admin`) |
| `405 Method Not Allowed` | Wrong HTTP method for the URL (e.g. POST to a GET-only endpoint) |
| `400 Bad Request` on create | Malformed JSON, missing body, or a duplicate name |
| `500` when saving | A field fails entity validation (e.g. name too short) |
| Sign-up says username is taken | Choose a new username; existing users keep their roles |

---

## 🗺️ Roadmap

- [ ] Restrict admin/seller role assignment on signup
- [ ] Serve uploaded images via `/images/**`
- [ ] Stripe payment-intent endpoint
- [ ] CORS configuration for a frontend app
- [ ] Unit and integration tests
- [ ] Docker / Docker Compose setup
- [ ] Move all secrets to environment variables

---

## 👨‍💻 Author

**Prince Kumar**
GitHub: [@princekumar2311](https://github.com/princekumar2311)
Email: aboutprincekumar@gmail.com

---

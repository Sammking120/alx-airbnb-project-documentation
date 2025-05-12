
# Requirements specifications for the backend

## 1. User Authentication

### Endpoints
- **POST** `/api/v1/auth/register`
- **POST** `/api/v1/auth/login`
- **POST** `/api/v1/auth/logout`
- **GET**  `/api/v1/auth/refresh`

### Request / Response
- **Register**  
  - Request:
    ```json
    { "first_name":"string", "last_name":"string", "email":"string", "password":"string" }
    ```
  - Response `201 Created`:
    ```json
    { "user_id":"UUID", "created_at":"timestamp" }
    ```
- **Login**  
  - Request:
    ```json
    { "email":"string", "password":"string" }
    ```
  - Response `200 OK`:
    ```json
    { "access_token":"JWT", "expires_in":3600 }
    ```

### Validation
- `email`: valid format, unique
- `password`: ≥12 chars, mixed case, digits, symbols
- Rate limit: 5 login attempts/min per IP

### Security & Performance
- TLS required
- JWT signed with RS256
- 95th-percentile response < 200 ms at 100 RPS

---

## 2. Property Management

### Endpoints
- **POST**   `/api/v1/properties`
- **GET**    `/api/v1/properties`
- **GET**    `/api/v1/properties/{id}`
- **PUT**    `/api/v1/properties/{id}`
- **DELETE** `/api/v1/properties/{id}`

### Request / Response
- **Create**  
  - Request:
    ```json
    {
      "host_id":"UUID",
      "name":"string",
      "description":"string",
      "location":"string",
      "price_per_night":0.00,
      "amenities":["string"]
    }
    ```
  - Response `201 Created`:
    ```json
    { "property_id":"UUID", "created_at":"timestamp" }
    ```
- **List/Search**  
  - Query: `location`, `min_price`, `max_price`, `amenities`, `page`, `limit`
  - Response `200 OK`: paginated list

### Validation
- `name`: 5–100 chars
- `location`: non-empty, ≤ 200 chars
- `price_per_night`: positive, two decimals max

### Performance
- Default `limit=20`, max `limit=100`
- Index on `location`, `price_per_night`
- Rate limit: 50 req/min per user

---

## 3. Booking System

### Endpoints
- **POST**   `/api/v1/bookings`
- **GET**    `/api/v1/bookings`
- **GET**    `/api/v1/bookings/{id}`
- **PATCH**  `/api/v1/bookings/{id}/cancel`
- **PUT**    `/api/v1/bookings/{id}`

### Request / Response
- **Create**  
  - Request:
    ```json
    {
      "property_id":"UUID",
      "user_id":"UUID",
      "start_date":"YYYY-MM-DD",
      "end_date":"YYYY-MM-DD"
    }
    ```
  - Response `201 Created`:
    ```json
    {
      "booking_id":"UUID",
      "total_price":0.00,
      "status":"pending"
    }
    ```

### Validation
- `start_date` < `end_date`
- No overlapping bookings per property
- `status`: pending | confirmed | canceled | completed

### Performance
- 95th-percentile response < 250 ms at 200 RPS

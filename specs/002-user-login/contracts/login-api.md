# API Contract: User Login Endpoint

This contract specifies the interface exposed by the Java backend for user authentication.

## Endpoint: Login

- **Method**: `POST`
- **Path**: `/api/auth/login`
- **Headers**:
  - `Content-Type: application/json`

### Request Payload

```json
{
  "username": "john_doe",
  "password": "SecurePassword123"
}
```

- **Validation Rules**:
  - `username`: Required, non-empty, non-blank string.
  - `password`: Required, non-empty, non-blank string.

---

### Response Payloads

#### 1. Success Response (200 OK)
Returned when credentials match the stored hash.

```json
{
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiJqb2huX2RvZSIsImV4cCI6MTc4MjMzNjYwMH0.signature",
  "type": "Bearer",
  "expiresAt": "2026-06-26T09:49:57Z"
}
```

- **Key Fields**:
  - `token`: The signed JSON Web Token to be used for authorization in subsequent requests.
  - `type`: The authorization scheme prefix (always `Bearer`).
  - `expiresAt`: UTC expiration timestamp (24 hours after creation).

#### 2. Authentication Failure (401 Unauthorized)
Returned if the username does not exist or password comparison fails.

```json
{
  "timestamp": "2026-06-25T09:49:57Z",
  "status": 401,
  "error": "Unauthorized",
  "message": "Invalid username or password"
}
```

#### 3. Validation Failure (400 Bad Request)
Returned if inputs are missing or blank.

```json
{
  "timestamp": "2026-06-25T09:49:57Z",
  "status": 400,
  "error": "Bad Request",
  "message": "Username and password are required"
}
```

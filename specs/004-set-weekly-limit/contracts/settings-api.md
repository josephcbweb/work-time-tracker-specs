# API Contract: User Settings Endpoints

This contract specifies the interfaces exposed by the Java backend to support retrieving and configuring user-specific weekly time limits.

---

## 1. Endpoint: Retrieve Settings

Retrieves the current weekly time limit for the authenticated user. If no limit is configured, returns the default settings.

- **Method**: `GET`
- **Path**: `/api/user/settings`
- **Headers**:
  - `Authorization: Bearer <token>` (Required)

### Response Payloads

#### Success Response (200 OK)
Returned when the settings are successfully retrieved.

```json
{
  "username": "john_doe",
  "weeklyLimit": 40.0
}
```

#### Authentication Failure (401 Unauthorized)
Returned if the request is missing a token or the token is invalid/expired.

```json
{
  "timestamp": "2026-06-25T10:32:00Z",
  "status": 401,
  "error": "Unauthorized",
  "message": "Full authentication is required to access this resource"
}
```

---

## 2. Endpoint: Update Settings

Updates the weekly time limit for the authenticated user.

- **Method**: `PUT`
- **Path**: `/api/user/settings`
- **Headers**:
  - `Authorization: Bearer <token>` (Required)
  - `Content-Type: application/json`

### Request Payload
The body must contain the new weekly limit value.

```json
{
  "weeklyLimit": 37.5
}
```

### Response Payloads

#### Success Response (200 OK)
Returned when the settings are successfully updated in the database.

```json
{
  "username": "john_doe",
  "weeklyLimit": 37.5
}
```

#### Validation Failure (400 Bad Request)
Returned if the provided limit is invalid (non-numeric, outside the 0.5 to 168.0 range, or has more than 1 decimal place).

```json
{
  "timestamp": "2026-06-25T10:32:00Z",
  "status": 400,
  "error": "Bad Request",
  "message": "Weekly limit must be between 0.5 and 168.0 hours with up to 1 decimal place."
}
```

#### Authentication Failure (401 Unauthorized)
Returned if the request is missing a token or the token is invalid/expired.

```json
{
  "timestamp": "2026-06-25T10:32:00Z",
  "status": 401,
  "error": "Unauthorized",
  "message": "Full authentication is required to access this resource"
}
```

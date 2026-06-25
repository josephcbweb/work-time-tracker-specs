# API Contract: User Logout Endpoint

This contract specifies the interface exposed by the Java backend to support user session termination.

## Endpoint: Logout

- **Method**: `POST`
- **Path**: `/api/auth/logout`
- **Headers**:
  - `Authorization: Bearer <token>` (Required)

### Request Payload
- No request body is required. The token to be invalidated is extracted directly from the `Authorization` header.

---

### Response Payloads

#### 1. Success Response (200 OK)
Returned when the session token is successfully invalidated and blacklisted.

```json
{
  "message": "Successfully logged out"
}
```

#### 2. Authentication Failure (401 Unauthorized)
Returned if the request is missing a token or the token is malformed.

```json
{
  "timestamp": "2026-06-25T10:29:00Z",
  "status": 401,
  "error": "Unauthorized",
  "message": "Full authentication is required to access this resource"
}
```

#### 3. Already Revoked / Expired Token (200 OK)
If a user sends a logout request for a token that is already blacklisted or expired, the backend should return a `200 OK` status with the success message. This ensures the logout operation remains idempotent and robust.

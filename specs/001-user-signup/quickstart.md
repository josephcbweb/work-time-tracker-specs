# Quickstart & Validation Guide: User Signup

This document describes how to validate the User Signup feature end-to-end after implementation.

## Prerequisites

1. **Backend**: Java 17 SDK installed. The backend server should be running on `http://localhost:8080`.
2. **Frontend**: Node.js installed. The web client should be running on `http://localhost:5173`.
3. **Data**: The `users.csv` file should exist in the backend database directory.

---

## Validation Scenarios

Refer to the API definition in [signup.json](contracts/signup.json) and data rules in [data-model.md](data-model.md) for full contract details.

### Scenario 1: Successful User Registration (API)

Verify that the backend creates a user record correctly.

**Execution**:
Run the following curl command:
```bash
curl -X POST http://localhost:8080/api/auth/signup \
  -H "Content-Type: application/json" \
  -d '{"username": "new_user_1", "password": "supersecurepassword123"}'
```

**Expected Outcome**:
* **HTTP Status**: `201 Created`
* **Response Body**:
  ```json
  {
    "id": 1,
    "username": "new_user_1",
    "createdAt": "2026-06-24T09:45:00Z"
  }
  ```
* **Database Verification**: Open `work-time-tracker-backend/data/users.csv` and verify `new_user_1` is written, and its password is BCrypt hashed (begins with `$2a$`).

---

### Scenario 2: Registration with Duplicate Username (API)

Verify that the system blocks registering an existing username.

**Execution**:
Attempt to register the same username again:
```bash
curl -X POST http://localhost:8080/api/auth/signup \
  -H "Content-Type: application/json" \
  -d '{"username": "new_user_1", "password": "anotherpassword123"}'
```

**Expected Outcome**:
* **HTTP Status**: `400 Bad Request`
* **Response Body**:
  ```json
  {
    "timestamp": "2026-06-24T09:46:00Z",
    "status": 400,
    "error": "Bad Request",
    "message": "Username 'new_user_1' is already registered"
  }
  ```

---

### Scenario 3: Validation Constraints (API)

Verify that invalid inputs are rejected by the backend.

**Execution (Weak Password)**:
```bash
curl -X POST http://localhost:8080/api/auth/signup \
  -H "Content-Type: application/json" \
  -d '{"username": "new_user_2", "password": "abc"}'
```

**Expected Outcome**:
* **HTTP Status**: `400 Bad Request`
* **Response Body**: Contains a message indicating the password must be at least 8 characters.

---

### Scenario 4: UI End-to-End Validation

Verify that the user interface matches requirements.

**Execution**:
1. Open the browser and navigate to `http://localhost:5173/signup`.
2. Click **Sign Up** with empty fields.
   * *Outcome*: Inputs are highlighted, error messages display below inputs, and no HTTP call is sent.
3. Enter username `user_test` and password `password_test`. Click **Sign Up**.
   * *Outcome*: Successfully redirected to the login page (`/login`) with a green success message: `"Sign up successful! Please log in."`

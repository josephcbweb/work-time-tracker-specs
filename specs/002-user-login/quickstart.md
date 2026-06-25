# Quickstart Validation Guide: User Login

This guide describes how to verify the end-to-end functionality of the User Login feature once implemented.

## Prerequisites

1. **Test User Existence**: The `users.csv` database file must contain at least one registered user.
   Example entry in `work-time-tracker-backend/src/main/resources/db/users.csv`:
   ```csv
   1,john_doe,$2a$10$tMh4fJkZ3N3j3K5Lz4EwOefT00XpX7YtB8VvA2C9d8S7q6p5o4n3m,2026-06-25T09:47:00Z
   ```
   *(Note: The above BCrypt hash corresponds to the password `SecurePassword123`)*

2. **Backend Server running**:
   ```bash
   cd work-time-tracker-backend
   ./mvnw spring-boot:run
   ```
   *(Running on http://localhost:8080)*

3. **Frontend Server running**:
   ```bash
   cd work-time-tracker-frontend
   npm run dev
   ```
   *(Running on http://localhost:5173)*

---

## 1. Backend API Validation (cURL)

Verify the API contract specified in [login-api.md](file:///Users/joseph.cb/Documents/test-collab/work-time-tracker-specs/specs/002-user-login/contracts/login-api.md).

### Scenario A: Successful Login
Run the following cURL request to authenticate with valid credentials:
```bash
curl -X POST http://localhost:8080/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"username": "john_doe", "password": "SecurePassword123"}'
```
**Expected Response (200 OK)**:
A JSON payload containing `token`, `type` ("Bearer"), and `expiresAt` ISO timestamp.

### Scenario B: Failed Login (Invalid Credentials)
Run a request with incorrect credentials:
```bash
curl -X POST http://localhost:8080/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"username": "john_doe", "password": "WrongPassword!"}'
```
**Expected Response (401 Unauthorized)**:
A standard error envelope with status `401` and message `Invalid username or password`.

### Scenario C: Failed Login (Missing Fields)
Run a request with missing username or password:
```bash
curl -X POST http://localhost:8080/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"username": ""}'
```
**Expected Response (400 Bad Request)**:
A standard error envelope with status `400` and validation error message.

---

## 2. Frontend Interface Validation

1. Open your browser and navigate to the login page: `http://localhost:5173/login`.
2. **Empty Validation**: Click the "Login" button without entering credentials. The form must prevent submission and highlight the inputs.
3. **Invalid Feedback**: Enter a wrong username or password and submit. Verify that the form displays "Invalid username or password" under or near the credentials input.
4. **Successful Login Redirect**: Enter `john_doe` and `SecurePassword123`, then click "Login". Verify that you are redirected to `http://localhost:5173/dashboard`.
5. **Session Check**: Open the browser's developer console and inspect local storage (`localStorage.getItem('token')`) to confirm the JWT is securely stored.

---

## 3. Running Automated Tests

### Backend Unit & Integration Tests
Run unit tests in the backend package:
```bash
cd work-time-tracker-backend
./mvnw test
```

### Frontend Tests
Run unit/component tests in the frontend package:
```bash
cd work-time-tracker-frontend
npm run test
```

# Quickstart Validation Guide: User Logout

This guide describes how to verify the end-to-end functionality of the User Logout feature once implemented.

## Prerequisites

1. **Active Session Token**: You need a valid JWT token generated via the login endpoint (refer to [login-api.md](file:///Users/joseph.cb/Documents/test-collab/work-time-tracker-specs/specs/002-user-login/contracts/login-api.md)).
2. **Backend Server Running**:
   ```bash
   cd work-time-tracker-backend
   ./mvnw spring-boot:run
   ```
3. **Frontend Server Running**:
   ```bash
   cd work-time-tracker-frontend
   npm run dev
   ```

---

## 1. Backend API Validation (cURL)

Verify the API contract specified in [logout-api.md](file:///Users/joseph.cb/Documents/test-collab/work-time-tracker-specs/specs/003-user-logout/contracts/logout-api.md).

### Scenario A: Successful Logout and Invalidation
1. Authenticate to obtain a token:
   ```bash
   TOKEN=$(curl -s -X POST http://localhost:8080/api/auth/login \
     -H "Content-Type: application/json" \
     -d '{"username": "john_doe", "password": "SecurePassword123"}' | grep -o '"token":"[^"]*' | grep -o '[^"]*$')
   ```
2. Trigger logout to blacklist the token:
   ```bash
   curl -X POST http://localhost:8080/api/auth/logout \
     -H "Authorization: Bearer $TOKEN"
   ```
   **Expected Response (200 OK)**:
   ```json
   {
     "message": "Successfully logged out"
   }
   ```
3. Verify that the token is now invalidated by calling a protected endpoint (e.g., retrieving weekly time limit):
   ```bash
   curl -X GET http://localhost:8080/api/settings \
     -H "Authorization: Bearer $TOKEN"
   ```
   **Expected Response (401 Unauthorized)**:
   A standard error envelope indicating authentication failure.

### Scenario B: Logout with Missing Authorization Header
Submit a logout request without the Bearer token:
```bash
curl -X POST http://localhost:8080/api/auth/logout
```
**Expected Response (401 Unauthorized)**:
A standard error envelope indicating full authentication is required.

---

## 2. Frontend Interface Validation

1. Open your browser and navigate to `http://localhost:5173/login`.
2. Authenticate using valid credentials (`john_doe` / `SecurePassword123`) to access the dashboard.
3. **Locate UI Control**: Confirm that a visible "Logout" button/link is rendered on the dashboard header/sidebar.
4. **Trigger Logout**: Click the "Logout" button.
5. **Verify Redirection & Session Wiping**:
   - The application must immediately redirect to `http://localhost:5173/login`.
   - Open browser developer tools -> Application -> Local Storage. Confirm that the key storing the JWT token is fully removed.
6. **Verify Route Protection**:
   - Attempt to manually navigate back to `http://localhost:5173/dashboard` in the address bar.
   - The application must block the attempt and redirect you to `/login`.
7. **Verify Back Button Prevention**:
   - Click the browser's "Back" button after being redirected to `/login`.
   - Confirm that the UI remains on `/login` and does not reveal the dashboard.

---

## 3. Running Automated Tests

### Backend Tests
Execute backend unit and integration tests (including `AuthControllerTest` and `TokenBlacklistServiceTest`):
```bash
cd work-time-tracker-backend
./mvnw test
```

### Frontend Tests
Execute frontend linting and compilation checks:
```bash
cd work-time-tracker-frontend
npm run build
```

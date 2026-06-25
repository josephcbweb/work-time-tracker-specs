# Quickstart Validation Guide: Set Weekly Time Limit

This guide describes how to verify the end-to-end functionality of the Set Weekly Time Limit feature once implemented.

## Prerequisites

1. **Active Session Token**: You need a valid JWT token generated via the login endpoint (refer to [login-api.md](../../002-user-login/contracts/login-api.md)).
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

Verify the API contracts specified in [settings-api.md](contracts/settings-api.md).

### Scenario A: Retrieve Default Settings
1. Obtain an active session token `$TOKEN`.
2. Fetch current settings:
   ```bash
   curl -X GET http://localhost:8080/api/user/settings \
     -H "Authorization: Bearer $TOKEN"
   ```
   **Expected Response (200 OK)**:
   ```json
   {
     "username": "john_doe",
     "weeklyLimit": 40.0
   }
   ```

### Scenario B: Update Settings successfully
Update the weekly time limit to a valid decimal value:
```bash
curl -X PUT http://localhost:8080/api/user/settings \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"weeklyLimit": 37.5}'
```
**Expected Response (200 OK)**:
```json
{
  "username": "john_doe",
  "weeklyLimit": 37.5
}
```

### Scenario C: Update Settings with Invalid Value (Out of Bounds)
Attempt to set the limit above the maximum threshold (168.0 hours):
```bash
curl -X PUT http://localhost:8080/api/user/settings \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"weeklyLimit": 200.0}'
```
**Expected Response (400 Bad Request)**:
Standard error envelope (as per project constitution) containing the validation error message.

---

## 2. Frontend Interface Validation

1. Open your browser and navigate to `http://localhost:5173/login`.
2. Authenticate using valid credentials to access the dashboard.
3. **Verify Settings Load**: Verify that the dashboard displays the active weekly limit (defaults to `40.0` hours).
4. **Modify limit**:
   - Locate the weekly limit input field.
   - Enter `35` and save.
5. **Verify Recalculations**:
   - Verify the daily average target updates immediately from `8.0` hours to `7.0` hours (`35 / 5` weekdays).
   - Check that consistency calculations and remaining progress charts recalculate based on `35.0` hours.
6. **Verify Input Constraints**:
   - Try entering `0` or a negative value.
   - Confirm that the UI validates and blocks the action, displaying a clear error.
7. **Verify Persistence**:
   - Refresh the page and confirm the weekly limit displays `35.0` hours.

---

## 3. Running Automated Tests

### Backend Tests
Run Spring Boot tests (including `SettingsControllerTest` and database CSV read/write unit tests):
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

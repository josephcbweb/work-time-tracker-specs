# Quickstart Validation Guide: Time Tracking and Analytics

This guide describes how to verify the end-to-end functionality of the Time Tracking & Logging and Analytics & Recommendations features once implemented.

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

Verify the API contracts specified in [timelog-api.md](contracts/timelog-api.md).

### Scenario A: Fetch Weekly logs
```bash
curl -X GET "http://localhost:8080/api/logs/week?date=2026-06-24" \
  -H "Authorization: Bearer $TOKEN"
```
**Expected Response (200 OK)**: A JSON containing the days of the week, total hours, and list of entries.

### Scenario B: Successful Clock In
```bash
curl -X POST http://localhost:8080/api/logs/clock-in \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"date": "2026-06-24", "time": "09:00"}'
```
**Expected Response (200 OK)**: Returns the newly created active log entry. Save the returned `"id"` value as `$ENTRY_ID`.

### Scenario C: Successful Clock Out
```bash
curl -X POST http://localhost:8080/api/logs/clock-out \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d "{\"entryId\": \"$ENTRY_ID\", \"time\": \"17:00\"}"
```
**Expected Response (200 OK)**: Returns the closed log entry with `outTime` set to `"17:00"` and `durationMinutes` set to `480`.

### Scenario D: Double Clock-in Validation Error
Attempt to clock in again while an active session exists:
```bash
curl -X POST http://localhost:8080/api/logs/clock-in \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"date": "2026-06-24", "time": "18:00"}'
```
**Expected Response (400 Bad Request)**: A validation error message: `"Please clock out of your active session first."`

### Scenario E: Weekend Block Validation Error
Attempt to clock in on a Saturday (e.g., `2026-06-27`):
```bash
curl -X POST http://localhost:8080/api/logs/clock-in \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"date": "2026-06-27", "time": "09:00"}'
```
**Expected Response (400 Bad Request)**: Error message: `"Logging is blocked on weekends."`

---

## 2. Frontend Interface Validation

1. Open your browser and navigate to `http://localhost:5173/login`.
2. Authenticate using valid credentials to access the dashboard.
3. **Verify Dashboard Layout**:
   - Verify that Monday through Friday are displayed in cards/grid containers.
   - Verify the current day is visually highlighted with a colored border or background (if today is a weekday).
4. **Log time**:
   - In today's card, enter `09:00` and click "Clock In". Verify it updates to show the active session.
   - Enter `17:00` and click "Clock Out". Verify that the session is listed as `8h 0m` and the daily total is shown as `8h 0m`.
5. **Verify Weekly Progress Graph**:
   - Change your weekly limit to `40.0` hours (from Epic 2).
   - Verify the graph updates to show `8.0` hours worked (20%), `32.0` hours remaining.
6. **Verify Consistency Checking**:
   - If today is Monday:
     - Working `8.0` hours is equal to target average (`40 / 5 = 8`). Verify consistency check sentence displays: `"You are consistent! You have maintained your daily average of 8 hours throughout the week."`
   - Delete/Edit the entry to be `6.0` hours (under average).
     - Verify consistency check sentence updates to warn: `"Consistency check: You are currently behind your daily average pace of 8.0 hours."`
7. **Verify Make-up Recommendations**:
   - If today is Tuesday, and yesterday (Monday) you logged `6.0` hours (deficit of 2.0 hours):
     - Verify a make-up alert card is displayed saying: `"You logged a loss of 2.0 hours yesterday. You have to spend 10.0 hours more today to make up for the week."` (Target `8.0` + Deficit `2.0` = `10.0`).
8. **Verify Week Navigation**:
   - Click "Previous Week". Verify that the date range switches to the preceding week.
   - Verify the progress graph and daily log lists show logs for that past week.
   - **Verify Historical Scoping**: Ensure the consistency checking sentence and make-up recommendations are fully hidden when viewing this past week.

---

## 3. Running Automated Tests

### Backend Tests
```bash
cd work-time-tracker-backend
./mvnw test
```

### Frontend Tests
```bash
cd work-time-tracker-frontend
npm run build
```

# API Contract: Time Tracking and Analytics Endpoints

This contract specifies the interfaces exposed by the Java backend to support time logging, editing, deletion, and retrieval of weekly summaries.

---

## 1. Endpoint: Retrieve Weekly Logs & Summary

Fetches all logs, targets, and calculated analytics for the week containing the specified date.

- **Method**: `GET`
- **Path**: `/api/logs/week`
- **Params**:
  - `date=YYYY-MM-DD` (Required, date within the target week)
- **Headers**:
  - `Authorization: Bearer <token>` (Required)

### Response Payload (200 OK)
```json
{
  "weekStart": "2026-06-22",
  "weekEnd": "2026-06-26",
  "weeklyLimit": 40.0,
  "totalHoursWorked": 16.5,
  "hoursRemaining": 23.5,
  "completionPercentage": 41.25,
  "days": [
    {
      "date": "2026-06-22",
      "dayName": "Monday",
      "totalHours": 8.0,
      "entries": [
        {
          "id": "e30349c8-d1a1-4cf4-bc6c-7e6e22f2541a",
          "inTime": "09:00",
          "outTime": "17:00",
          "durationMinutes": 480
        }
      ]
    },
    {
      "date": "2026-06-23",
      "dayName": "Tuesday",
      "totalHours": 8.5,
      "entries": [
        {
          "id": "766efcd5-cf6b-4e12-88f5-412e2c0e86a0",
          "inTime": "08:30",
          "outTime": "17:00",
          "durationMinutes": 510
        }
      ]
    },
    {
      "date": "2026-06-24",
      "dayName": "Wednesday",
      "totalHours": 0.0,
      "entries": []
    },
    {
      "date": "2026-06-25",
      "dayName": "Thursday",
      "totalHours": 0.0,
      "entries": []
    },
    {
      "date": "2026-06-26",
      "dayName": "Friday",
      "totalHours": 0.0,
      "entries": []
    }
  ]
}
```

---

## 2. Endpoint: Clock In

Initiates an active work session.

- **Method**: `POST`
- **Path**: `/api/logs/clock-in`
- **Headers**:
  - `Authorization: Bearer <token>` (Required)
  - `Content-Type: application/json`
- **Request Payload**:
  ```json
  {
    "date": "2026-06-24",
    "time": "09:00"
  }
  ```

### Response Payload (200 OK)
```json
{
  "id": "99f3be6c-a63e-4b68-8094-1a967a5367be",
  "username": "alice",
  "date": "2026-06-24",
  "inTime": "09:00",
  "outTime": null,
  "durationMinutes": null
}
```

---

## 3. Endpoint: Clock Out

Completes an active work session.

- **Method**: `POST`
- **Path**: `/api/logs/clock-out`
- **Headers**:
  - `Authorization: Bearer <token>` (Required)
  - `Content-Type: application/json`
- **Request Payload**:
  ```json
  {
    "entryId": "99f3be6c-a63e-4b68-8094-1a967a5367be",
    "time": "17:30"
  }
  ```

### Response Payload (200 OK)
```json
{
  "id": "99f3be6c-a63e-4b68-8094-1a967a5367be",
  "username": "alice",
  "date": "2026-06-24",
  "inTime": "09:00",
  "outTime": "17:30",
  "durationMinutes": 510
}
```

---

## 4. Endpoint: Edit Time Entry

Modifies the times of an existing time entry (active or closed).

- **Method**: `PUT`
- **Path**: `/api/logs/entry/{id}`
- **Headers**:
  - `Authorization: Bearer <token>` (Required)
  - `Content-Type: application/json`
- **Request Payload**:
  ```json
  {
    "inTime": "09:15",
    "outTime": "17:15"
  }
  ```

### Response Payload (200 OK)
```json
{
  "id": "99f3be6c-a63e-4b68-8094-1a967a5367be",
  "username": "alice",
  "date": "2026-06-24",
  "inTime": "09:15",
  "outTime": "17:15",
  "durationMinutes": 480
}
```

---

## 5. Endpoint: Delete Time Entry

Deletes an existing time entry.

- **Method**: `DELETE`
- **Path**: `/api/logs/entry/{id}`
- **Headers**:
  - `Authorization: Bearer <token>` (Required)

### Response Payload (200 OK)
```json
{
  "message": "Time entry deleted successfully"
}
```

---

## Error Envelope Payload (Standard)
Returned when validation rules (e.g. overlaps, weekend logging, out-of-order times) are violated.
```json
{
  "timestamp": "2026-06-25T10:32:00Z",
  "status": 400,
  "error": "Bad Request",
  "message": "Cannot clock in: An active session already exists."
}
```

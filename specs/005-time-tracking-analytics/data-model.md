# Data Model Design: Time Tracking and Analytics

This document defines the schema, constraints, and persistence format for daily time logging entries.

---

## Entity: TimeLog

Represents a single work session logged by a user.

### Attributes

| Attribute | Type | Description | Constraints |
| :--- | :--- | :--- | :--- |
| `id` | String | Unique identifier for the entry. | Primary Key (UUID format) |
| `username` | String | Unique identifier of the user who owns this log. | Foreign Key to User |
| `date` | String | The calendar date of the session (`YYYY-MM-DD`). | Mandatory |
| `in_time` | String | The clock-in time in 24-hour format (`HH:mm`). | Mandatory |
| `out_time` | String | The clock-out time in 24-hour format (`HH:mm`). | Nullable (when active) |

### Relationships

- **User (1) <---> (*) TimeLog**: A single user can have multiple time logs over time.

---

## Storage Specification (CSV)

Time logs are persisted in a shared CSV file on the backend.

### File Details
- **Path**: `work-time-tracker-backend/data/timelogs.csv`
- **Concurrency**: Reentrant Read-Write Locking must protect this file during any read or write operations to prevent write collisions.

### Header Schema
```csv
id,username,date,in_time,out_time
```

### Sample Data
```csv
id,username,date,in_time,out_time
a1b2c3d4-e5f6-7a8b-9c0d-1e2f3a4b5c6d,alice,2026-06-22,09:00,12:00
f5e4d3c2-b1a0-9e8d-7c6b-5a4f3e2d1c0b,alice,2026-06-22,13:00,17:30
8a9b0c1d-2e3f-4a5b-6c7d-8e9f0a1b2c3d,bob,2026-06-22,08:30,17:00
```

---

## Validation & Business Rules

1. **Weekend Excluded**:
   - The date parameter of a new log MUST NOT fall on a Saturday or a Sunday.
   - Weekend check is performed using ISO weekday rules.

2. **Chronological Validity**:
   - The `out_time` MUST be chronologically after the corresponding `in_time`.
   - E.g. `In: 09:00` and `Out: 08:30` is invalid.
   - Midnight-crossing sessions (e.g. `In: 22:00` on Monday, `Out: 02:00` on Tuesday) are valid. The full duration is saved with Monday's date. The `OutTime` string is stored as-is, and the frontend/backend calculate the difference across midnight.

3. **Single Active Session**:
   - A user can only have at most one active session (i.e. an entry where `out_time` is null) at any time.
   - Creating a new clock-in is rejected if an active session exists.

4. **Overlap Prevention**:
   - A new session range `[in_time, out_time]` must not overlap with any completed sessions for the same user on the same date.
   - Overlap check formula: `Start1 < End2 AND End1 > Start2`.

5. **Auto-Expiration**:
   - If a session remains active for more than 24 hours (i.e. `current_time - in_time > 24 hours`), the background verification task will close the session by setting `out_time` equal to `in_time` (duration 0).

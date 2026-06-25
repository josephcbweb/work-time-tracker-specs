# Data Model Design: Set Weekly Time Limit

This document describes the schema, rules, and relationships for persisting weekly limit settings.

## Entity: UserSettings

Represents the configuration preferences for an individual user.

### Attributes

| Attribute | Type | Description | Constraints |
| :--- | :--- | :--- | :--- |
| `username` | String | Unique identifier of the user (owner of the settings). | Primary Key, Foreign Key to User |
| `weekly_limit` | Decimal/Double | The weekly work hour limit. | Default: `40.0`, Range: `0.5` to `168.0` (inclusive) |

### Relationships

- **User (1) <---> (1) UserSettings**: A user has exactly one settings configuration.

---

## Storage Specification (CSV)

In accordance with the project constitution, settings are stored in a CSV file on the backend.

### File Details
- **Path**: `work-time-tracker-backend/data/settings.csv`
- **Locking**: A file-level read/write lock must be acquired before any operation to prevent data corruption.

### Header Schema
```csv
username,weekly_limit
```

### Sample Data
```csv
username,weekly_limit
alice,40.0
bob,37.5
```

---

## Validation & Business Rules

1. **Authentication**: All read and write operations on `UserSettings` must occur under an authenticated context. A user can only view or modify the settings associated with their own `username`.
2. **Weekly Limit Range**:
   - The value of `weekly_limit` must be a numeric value.
   - Minimum value: `0.5` hours.
   - Maximum value: `168.0` hours (number of hours in a full week).
3. **Decimal Precision**:
   - Up to 1 decimal place is supported (e.g. `37.5`).
   - Inputs with more decimal places must be rounded or rejected (standard backend API validation rejects values with scale > 1).

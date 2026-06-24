# Data Model: User Signup

This document specifies the data model, storage representation, and validation rules for the User Signup feature.

## 1. User Entity

The `User` entity represents a registered account in the Work Time Tracker application.

### 1.1 Fields and Types

| Field Name | Type | Key | Validation / Constraints | Description |
|------------|------|-----|---------------------------|-------------|
| `id` | Long | PK | Auto-incrementing, Non-null | Unique internal identifier for the user. |
| `username` | String | Unique | 3-20 chars, Alphanumeric + `_` or `-`, Case-insensitive, Non-blank | User-facing login identifier. |
| `passwordHash` | String | - | Exactly 60 characters, Non-null | BCrypt hashed password. |
| `createdAt` | Instant | - | ISO-8601 UTC timestamp, Non-null | The date and time when the user signed up. |

### 1.2 CSV Storage Format

The entity is serialized and persisted in a CSV file named `users.csv` in the backend data directory.

* **File Path**: `work-time-tracker-backend/data/users.csv`
* **Header Structure**: `id,username,passwordHash,createdAt`
* **Sample Content**:
  ```csv
  id,username,passwordHash,createdAt
  1,alice,$2a$10$Z3.F908lB265NfA9B2vOyeGvY7uY21x.f492lO919e91.e919e919,2026-06-24T09:45:00Z
  2,bob_dev,$2a$10$X8.F908lB265NfA9B2vOyeGvY7uY21x.f492lO919e91.e919e919,2026-06-24T14:15:30Z
  ```

---

## 2. Validation & Constraints

### 2.1 Username Rules
- **Non-Blank**: Must not be null, empty, or contain only whitespace.
- **Length**: Must be between 3 and 20 characters (inclusive).
- **Format**: Must contain only letters (A-Z, a-z), numbers (0-9), underscores (`_`), and hyphens (`-`).
- **Uniqueness**: Must not match any existing username in `users.csv` (case-insensitive check, e.g., `Alice` matches and conflicts with `alice`).

### 2.2 Password Rules
- **Non-Blank**: Must not be null, empty, or contain only whitespace.
- **Length**: Must be at least 8 characters long before hashing.
- **Hashing**: Must be hashed using BCrypt before persistence. The password in its plain-text form must never be written to logs or stored in any persistent media.

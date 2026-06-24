# Overall Constitution - Work Time Tracker

This document serves as the master contract and architectural guide for the entire Work Time Tracker application. It defines overall codebase rules, project structures, and conventions that align both frontend and backend development.

---

## 1. Project Organization (Monorepo)

The repository is organized into three distinct subdirectories:
* **`work-time-tracker-specs/`**: Contains specifications, requirements, domain models, and system memory/constitutions.
* **`work-time-tracker-frontend/`**: The React + Vite client-side user interface.
* **`work-time-tracker-backend/`**: The Java-based API server with CSV-based data storage.

Developers and AI agents must preserve this division and avoid leaking dependencies or folders between these scopes.

---

## 2. API & Data Exchange Contract

* **Protocol:** HTTP/1.1 REST over SSL (in production) returning application/json.
* **Date & Time Standards:**
  * Dates must be transferred in ISO-8601 extended format: `YYYY-MM-DD`.
  * Times must be transferred in 24-hour format: `HH:mm` (e.g., `09:30` or `18:15`).
  * Creation timestamps must be UTC string representations: `YYYY-MM-DDTHH:mm:ssZ`.
* **Standard Response Envelope (for Errors):**
  * When a request fails, the API must return a structured JSON response containing:
    ```json
    {
      "timestamp": "2026-06-24T09:45:00Z",
      "status": 400,
      "error": "Bad Request",
      "message": "Detailed error message here"
    }
    ```

---

## 3. Data Integrity & Validation Rules

* Both frontend and backend share responsibility for verifying data constraints:
  1. **Authentication:** Only authenticated users can retrieve or modify day logs, time entries, and weekly settings.
  2. **Chronological Validity:** An "Out Time" must be strictly after the corresponding "In Time".
  3. **Concurrency/Overlaps:** A user cannot have overlapping logs or multiple active clock-ins.
  4. **Weekday Exclusion:** Logging is strictly prohibited on Saturdays and Sundays. The system will throw validation errors if date inputs land on weekends.
  5. **Persistence Safety:** In-memory caching or file locking on the backend must prevent concurrent data corruption of the CSV data storage files.

---

## 4. Development Workflow & Coding Standards

* **Clean Code:** Use self-documenting variables and function names. Keep functions small and single-purpose.
* **Security:** BCrypt must be used for user passwords on the backend. No secret credentials, passwords, or JWT keys should be hardcoded in the codebase; utilize environment variables or properties files instead.
* **Error Handling:** Avoid empty catch blocks. Log meaningful error details to stderr/logs.

# Backend Constitution - Work Time Tracker

This document defines the architecture, data persistence guidelines, packaging structure, and coding standards for the Java backend application.

---

## 1. Tech Stack & Architecture

* **Language:** Java 17+
* **Framework:** Spring Boot (preferred for rapid API setup) or standard Java REST/Servlets.
* **Architecture:** Controller-Service-Repository pattern.
  * **Controllers:** Handle HTTP routing, parsing request parameters, and returning response payloads.
  * **Services:** Enforce business logic calculations (weekly limits, consistency check, deficits).
  * **Repositories:** Manage persistence mechanisms (CSV reading and writing).

---

## 2. CSV Data Persistence Rules

As specified, a relational database is replaced by plain CSV files. Adhere to these storage guidelines:
* **Storage Location:** All CSV files (`users.csv`, `settings.csv`, `timelogs.csv`) must be stored in a directory within the backend folder (e.g., `src/main/resources/db/` or a configurable `./data/` folder).
* **Concurrency & Synchronization:** 
  * Because Java controllers serve concurrent requests, all file read/write operations must be thread-safe.
  * Use Java's `ReentrantReadWriteLock` or class-level/method-level `synchronized` blocks when accessing or editing files to prevent race conditions and data corruption.
* **Auto-Incrementing IDs:** The repository layer must parse the existing CSV rows on startup, determine the maximum ID, and maintain an atomic counter (`AtomicLong`) to allocate new IDs.
* **CSV Formats:**
  * `users.csv`: `id,username,passwordHash,createdAt`
  * `settings.csv`: `id,userId,weeklyHourLimit,weekStartDate`
  * `timelogs.csv`: `id,dayLogId,userId,date,dayOfWeek,inTime,outTime` (we can flatten `DayLog` and `TimeEntry` into a single file or maintain two distinct CSV files).

---

## 3. Directory & Package Structure (Spring Boot Example)

```text
com.worktime.tracker/
├── controller/         # REST API endpoints (AuthController, LogController, SettingsController)
├── service/            # Business logic (AuthService, LogService, RecommendationService)
├── repository/         # CSV File storage managers (UserRepository, LogRepository, SettingsRepository)
├── model/              # Domain models (User, WeeklySetting, DayLog, TimeEntry)
├── dto/                # Request/Response Data Transfer Objects (LoginRequest, LogResponse)
├── exception/          # Global exception handler and custom exceptions
└── TrackerApplication.java
```

---

## 4. Security & Validation

* **Passwords:** Store password hashes using BCrypt (`BCryptPasswordEncoder`). Raw passwords must never be logged or persisted.
* **Authentication:** Verify user identity via JWT tokens or a simple stateful session cookie. Return `401 Unauthorized` for endpoints accessed without valid credentials.
* **Input Sanitization:** Validate request inputs (e.g., ensure time inputs match `HH:mm` format, and limits are positive numeric values) before writing them to the CSV.

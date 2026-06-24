# Requirements Specification - Work Time Tracker

This document details the functional, non-functional, and system requirements for the Work Time Tracker application.

---

## 1. System Overview & Technology Stack

The Work Time Tracker is a lightweight web application designed to help employees track their daily work hours, monitor their weekly goals, maintain consistency, and catch up on missed hours.

* **Frontend:** React built with Vite (TypeScript preferred). Styled with Vanilla CSS (premium, modern, clean, dark-mode-first aesthetic with subtle micro-animations).
* **Backend:** Java (using a framework like Spring Boot or plain Java Servlets/REST API).
* **Database / Data Storage:** CSV files located in the backend folder (e.g., `users.csv`, `settings.csv`, `timelogs.csv`) to persist users, configurations, and time logs.

---

## 2. Functional Requirements (FR)

### 2.1 User Management & Security
* **FR-1.1 (Signup):** The system shall allow new users to register an account using a unique username and password.
* **FR-1.2 (Password Hashing):** The backend shall hash passwords securely using bcrypt (or equivalent) before storing them.
* **FR-1.3 (Authentication):** The system shall authenticate users via username and password. Upon successful login, the system must issue a secure token (e.g., JWT) to authorize subsequent API requests.
* **FR-1.4 (Authorization):** A user shall only be able to view, edit, or delete their own data (weekly limit, day logs, time entries).

### 2.2 Profile & Goal Settings
* **FR-2.1 (Weekly Target):** The system shall allow the user to set and update a weekly time limit (stored in hours, e.g., 40.0 hours).
* **FR-2.2 (Default Target):** If no weekly limit is set, the system shall default to a placeholder configuration (e.g., 40 hours) and prompt the user to configure it.

### 2.3 Time Tracking & Logging
* **FR-3.1 (Weekly Dashboard View):** The frontend shall display a dashboard listing the 5 weekdays of the current week (Monday through Friday). Weekend days (Saturday and Sunday) are blocked from logging.
* **FR-3.2 (Current Day Highlight):** The system shall detect the current local date of the user and visually highlight that day's UI container if it falls on a weekday.
* **FR-3.3 (Time Entry Logging):** 
  * The user shall be able to input "In Time" and "Out Time" values (formatted as `HH:mm` or using a 24-hour time picker).
  * Time entries must be stored chronologically under the selected day.
* **FR-3.4 (Validation Rules & Constraints):**
  * **Rule 1 (Pairing):** For every "In Time" entered, there must be a corresponding "Out Time" entered to complete the session.
  * **Rule 2 (Single Active Session):** A user cannot add a new "In Time" if they have an active "In Time" that lacks a corresponding "Out Time". 
  * **Rule 3 (Order):** The "Out Time" of a session must be chronologically after the corresponding "In Time".
  * **Rule 4 (No Overlaps):** A new "In Time" cannot be scheduled within the time range of an already completed session on the same day.
  * **Rule 5 (Overnight/Cross-day Sessions):** An active session can be clocked out on a subsequent day. The entire duration is attributed to the calendar date on which the session started.
  * **Rule 6 (Weekend Block):** The system shall refuse to save or log time entries on Saturday and Sunday.
* **FR-3.5 (Daily Duration Calculation):**
  * The system shall calculate the duration of each completed session: `Duration = Out Time - In Time`.
  * The system shall calculate the total hours worked per day as the sum of all completed session durations for that day.
  * Active sessions (with only an "In Time") do not contribute to the total hours worked until they are clocked out.
* **FR-3.6 (Edit & Delete Logs):**
  * The user shall be able to delete any time entry.
  * The user shall be able to update/edit an entry's "In Time" and "Out Time", subject to Rule 3 (Order) and Rule 4 (No Overlaps).

### 2.4 Analytics & Progress Visualizations
* **FR-4.1 (Weekly Progress Chart):**
  * The frontend shall display a visual chart (e.g., progress ring, bar chart, or gauge) depicting:
    * **Total Hours Worked:** Sum of daily totals for the current week.
    * **Target Weekly Limit:** The user's weekly time limit.
    * **Hours Remaining:** `Weekly Limit - Total Hours Worked` (or 0 if limit is exceeded).
* **FR-4.2 (Consistency Evaluation):**
  * The daily pace is defined as: `Daily Target = Weekly Limit / 5` (based on 5 working weekdays: Mon-Fri).
  * The system shall calculate the user's consistency by comparing the total hours worked up to the current day against the cumulative target for the days elapsed in the work week.
  * The system shall display a clear, dynamic message at the top of the dashboard indicating if the user is maintaining their required average.
* **FR-4.3 (Missed-Hours Recovery Recommendation):**
  * The system shall evaluate the hours worked on the **previous weekday**.
  * If the previous day's total is less than the `Daily Target` (`Weekly Limit / 5`), a loss is identified: `Loss = Daily Target - Previous Day's Total`.
  * The system shall present a prominent recommendation banner to the user saying:
    * `"You logged a loss of [X] hours yesterday. You have to spend [Y] hours more today to make up for the week."`
    * Where `X` is the yesterday's deficit, and `Y` is `Daily Target + Loss`.
* **FR-4.4 (Historical Week Navigation):**
  * The system shall allow the user to view previous or subsequent weeks' logs, limits, and stats by clicking "Previous Week" or "Next Week" navigation controls.

---

## 3. Non-Functional Requirements (NFR)

### 3.1 Usability & Design
* **NFR-3.1.1 (Premium UI):** The interface must look modern, utilizing responsive card designs, clean fonts, customized form controls, and elegant colors. Glassmorphism, smooth animations on hover/click, and consistent padding must be applied.
* **NFR-3.1.2 (Responsive Layout):** The user interface must scale fluidly across desktop, tablet, and mobile browsers.

### 3.2 Reliability & Integrity
* **NFR-3.2.1 (Input Validation):** All inputs (times, numbers, login fields) must be validated on both the client side and the server side to prevent database corruption or unexpected application errors.
* **NFR-3.2.2 (Data Persistence):** Time entries and user configurations must persist across page refreshes and server restarts.

### 3.3 Performance
* **NFR-3.3.1 (Low Latency API):** API responses for fetching weekly time logs and setting limits should be returned in under 200ms under normal network conditions.
* **NFR-3.3.2 (Optimistic UI Updates):** Frontend state should update responsively during time logging actions to preserve a high-quality user experience.

---

## 4. Key API Contracts (Tentative)

* `POST /api/auth/signup` - Register a new user
* `POST /api/auth/login` - Authenticate user, return JWT
* `GET /api/user/settings` - Retrieve weekly time limit
* `PUT /api/user/settings` - Update weekly time limit
* `GET /api/logs/week?date=YYYY-MM-DD` - Fetch all day logs and time entries for the week containing the specified date
* `POST /api/logs/clock-in` - Record an "In Time" (request body: `dayOfWeek`, `time`)
* `POST /api/logs/clock-out` - Record an "Out Time" for the active session (request body: `entryId`, `time`)
* `PUT /api/logs/entry/{id}` - Modify an existing entry's In/Out time (request body: `inTime`, `outTime`)
* `DELETE /api/logs/entry/{id}` - Delete a time entry


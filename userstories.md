# User Stories - Work Time Tracker

This document outlines the user stories and acceptance criteria for the Work Time Tracker application.

---

## Epic 1: Authentication & User Management

### US1.1: User Signup
* **As a** new user,  
  **I want to** sign up for an account using a username and password,  
  **So that** I can securely store my time tracking data.
* **Acceptance Criteria:**
  * **Scenario: Successful registration**
    * **Given** the signup page is loaded,
    * **When** I enter a unique username and a secure password, and click "Sign Up",
    * **Then** my account is created in the system, and I am redirected to the login page with a success message.
  * **Scenario: Registration with duplicate username**
    * **Given** the signup page is loaded,
    * **When** I enter a username that is already taken and click "Sign Up",
    * **Then** the system displays an error message indicating the username is already registered.
  * **Scenario: Invalid inputs**
    * **Given** the signup page is loaded,
    * **When** I leave the username or password blank,
    * **Then** the validation prevents submission and highlights the missing fields.

### US1.2: User Login
* **As a** registered user,  
  **I want to** log in with my username and password,  
  **So that** I can access my workspace.
* **Acceptance Criteria:**
  * **Scenario: Successful login**
    * **Given** the login page is loaded,
    * **When** I enter valid credentials and click "Login",
    * **Then** I am authenticated, a secure session/JWT is established, and I am redirected to the dashboard.
  * **Scenario: Failed login with invalid credentials**
    * **Given** the login page is loaded,
    * **When** I enter incorrect credentials,
    * **Then** an error message is displayed: "Invalid username or password," and I remain on the login page.

### US1.3: User Logout
* **As an** authenticated user,  
  **I want to** log out of the system,  
  **So that** my session is securely terminated.
* **Acceptance Criteria:**
  * **Scenario: Successful logout**
    * **Given** I am logged into the dashboard,
    * **When** I click the "Logout" button,
    * **Then** my session token is destroyed, and I am redirected back to the login page.

---

## Epic 2: Dashboard & Weekly Settings

### US2.1: Set Weekly Time Limit
* **As a** logged-in user,  
  **I want to** configure my weekly time limit in hours,  
  **So that** the system can calculate my daily targets and progress.
* **Acceptance Criteria:**
  * **Given** I am logged into the dashboard,
  * **When** I enter a positive numerical value (e.g., 40) in the "Weekly Limit" input field and save it,
  * **Then** the value is updated in the database, and the daily targets, progress charts, and recommendations immediately recalculate.

---

## Epic 3: Daily Time Tracking & Logging

### US3.1: View Weekly Days
* **As a** logged-in user,  
  **I want to** see the days of the week (Monday through Friday) displayed in a list/grid, with the current day visually highlighted,  
  **So that** I can easily check my tracking history and focus on today's logs.
* **Acceptance Criteria:**
  * **Given** I am logged into the dashboard,
  * **When** the page loads,
  * **Then** the weekdays (Monday through Friday) of the selected week are shown, and the system highlights the card corresponding to the current local calendar day (if it is a weekday).

### US3.2: Log "In" Time (Clock In)
* **As a** logged-in user,  
  **I want to** log an "In" time for a specific day,  
  **So that** I can record when I started working.
* **Acceptance Criteria:**
  * **Given** the selected day is a weekday (Monday-Friday) and has no active (open) "In" entry,
  * **When** I input a valid time and click "Clock In",
  * **Then** the "In" time is recorded, and the entry is marked as active/open (awaiting an "Out" time).

### US3.3: Log "Out" Time (Clock Out)
* **As a** logged-in user,  
  **I want to** log an "Out" time for a corresponding "In" entry,  
  **So that** the system can calculate the duration of that work session.
* **Acceptance Criteria:**
  * **Given** I have a logged "In" time without a corresponding "Out" time,
  * **When** I input a valid time and click "Clock Out",
  * **Then** the "Out" time is saved, the session is completed, and the duration is calculated.
  * **Note:** The "Out" time can be entered at any time (even on a subsequent day), and its entire duration is attributed to the day the session started.

### US3.4: Overlapping and Sequential Time Entry Constraints
* **As a** logged-in user,  
  **I want to** be prevented from creating malformed work sessions,  
  **So that** my logs are accurate and consistent.
* **Acceptance Criteria:**
  * **Scenario: Cannot add another "In" time before clocking out**
    * **Given** I have an "In" entry *without* an "Out" time for the current day,
    * **When** I attempt to add another "In" entry,
    * **Then** the action is blocked, and the system prompts: "Please clock out of your active session first."
  * **Scenario: Sequential logs are allowed**
    * **Given** all previous "In" entries have corresponding "Out" times,
    * **When** I add a new "In" entry (chronologically after the last "Out" time),
    * **Then** the entry is successfully saved.

### US3.5: View Total Daily Spent Time
* **As a** logged-in user,  
  **I want to** see the total time spent for each day of the week,  
  **So that** I know how many hours I logged on any particular day.
* **Acceptance Criteria:**
  * **Given** I have logged multiple closed In/Out entries on a day,
  * **When** I view that day's log,
  * **Then** the system displays the sum of durations of all completed sessions for that day (formatted in hours and minutes, e.g., `7h 30m`).

### US3.6: Delete Time Entry
* **As a** logged-in user,  
  **I want to** delete an existing time entry (whether open or closed),  
  **So that** I can remove accidental or incorrect logs.
* **Acceptance Criteria:**
  * **Given** I am viewing a day's time entries,
  * **When** I click the "Delete" button next to a specific time entry,
  * **Then** the entry is permanently removed from the system, and daily/weekly totals and progress recalculate immediately.

### US3.7: Edit Time Entry
* **As a** logged-in user,  
  **I want to** edit the "In" or "Out" time of an existing entry,  
  **So that** I can correct input mistakes.
* **Acceptance Criteria:**
  * **Given** I am viewing a day's time entries,
  * **When** I edit the "In" or "Out" time and click "Save",
  * **Then** the system validates that:
    * The new Out time (if present) is chronologically after the In time.
    * The modified entry does not overlap with any other completed time entries on the same day.
    * If valid, the times are updated and totals recalculate immediately.

### US3.8: Weekend Block
* **As a** logged-in user,  
  **I want the system to** prevent logging work hours on weekends (Saturday and Sunday),  
  **So that** logging is restricted to official working weekdays.
* **Acceptance Criteria:**
  * **Given** the user is viewing the weekly logs,
  * **When** the selected day is Saturday or Sunday,
  * **Then** the logging controls are disabled, and a notice reads: "Logging is blocked on weekends."

---

## Epic 4: Analytics & Recommendations

### US4.1: View Weekly Progress Graph
* **As a** logged-in user,  
  **I want to** see a progress graph or chart showing how much time is left for the week,  
  **So that** I can gauge my progress towards my weekly target.
* **Acceptance Criteria:**
  * **Given** I have set a weekly limit and logged work hours,
  * **When** I view the dashboard,
  * **Then** the chart displays:
    * Total hours logged this week (sum of all weekdays).
    * Total hours remaining to meet the limit.
    * A percentage completion value.

### US4.2: View Consistency Sentence
* **As a** logged-in user,  
  **I want to** read a consistency sentence at the top of my dashboard,  
  **So that** I can see if I am maintaining an equal daily average to hit my weekly limit.
* **Acceptance Criteria:**
  * **Given** my daily target is calculated as `Weekly Limit / 5` (for 5 weekdays),
  * **When** I view the dashboard,
  * **Then** the system evaluates my logged time against this average target for the elapsed weekdays of the current week:
    * If I am on or ahead of schedule (e.g. `Time Logged >= (Target Daily Average * Number of Completed Weekdays)`), display a positive sentence: `"You are consistent! You have maintained your daily average of X hours throughout the week."`
    * If I have fallen behind, display: `"Consistency check: You are currently behind your daily average pace of X hours."`

### US4.3: View Make-up Recommendation
* **As a** logged-in user,  
  **I want to** see a recommendation if I worked less than the target on the previous day,  
  **So that** I know how much extra time I need to spend today to make up for the week.
* **Acceptance Criteria:**
  * **Given** the previous weekday's logged hours were less than the daily average target (`Weekly Limit / 5`),
  * **When** the page loads,
  * **Then** a warning alert or card is displayed: `"You logged a loss of X hours yesterday. You have to spend Y hours more today to make up for the week."`
  * **Note:** 
    * `X = (Weekly Limit / 5) - Previous Day's Logged Hours`.
    * `Y = Today's Target (Weekly Limit / 5) + Yesterday's Loss (X)`.

### US4.4: Historical Week Navigation
* **As a** logged-in user,  
  **I want to** navigate to previous and next weeks,  
  **So that** I can view my historical time tracking logs, limits, and consistency statistics.
* **Acceptance Criteria:**
  * **Given** I am on the dashboard,
  * **When** I click the "Previous Week" navigation button,
  * **Then** the calendar updates to display Monday to Friday of the preceding week, along with the logs, progress chart, and settings associated with that week.
  * **When** I click the "Next Week" navigation button,
  * **Then** the dashboard updates to the subsequent week's data.


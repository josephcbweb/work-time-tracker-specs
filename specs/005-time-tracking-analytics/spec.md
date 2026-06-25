# Feature Specification: Time Tracking and Analytics

**Feature Branch**: `005-time-tracking-analytics`

**Created**: 2026-06-25

**Status**: Draft

**Input**: User description: "implement EPIC3 and EPIC4 from userstories.md file"

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Weekday Highlighting & Daily Time Logging (Priority: P1)

As a logged-in user,  
I want to see weekdays (Monday through Friday) with the current day highlighted, and be able to clock in and clock out to record my sessions.

**Why this priority**: Core time-tracking functionality. Without the ability to log times, no analytics can be computed.

**Independent Test**: Log in on a weekday, verify that weekday card is highlighted, clock in by entering a time, verify it shows active, clock out by entering a later time, and verify the session duration is computed and accumulated.

**Acceptance Scenarios**:

1. **Given** the user is logged into the dashboard and today is Wednesday,  
   **When** the dashboard loads,  
   **Then** the Wednesday card is visually highlighted, and other weekdays are shown normally.
2. **Given** the user has no active session on Monday,  
   **When** the user enters an "In" time (e.g. `09:00`) and clicks "Clock In",  
   **Then** the entry is saved as active and the UI shows "Clocked In at 09:00".
3. **Given** the user has an active session starting at `09:00`,  
   **When** the user enters an "Out" time (e.g. `17:00`) and clicks "Clock Out",  
   **Then** the session is marked closed, the duration of `8h 0m` is calculated, and the day's total updates to `8h 0m`.

---

### User Story 2 - Validation Rules & Entry Constraints (Priority: P2)

As a logged-in user,  
I want the system to enforce logical constraints on time entries (no overlapping sessions, single active session, chronological order) and block weekend logging.

**Why this priority**: Ensures data integrity. Prevents users from inputting impossible hours (e.g. overlapping work or working on weekends).

**Independent Test**: Try to clock in when already clocked in, try to clock out with a time earlier than clock-in, and check that Saturday/Sunday views disable logging.

**Acceptance Scenarios**:

1. **Given** an active session exists without an "Out" time,  
   **When** the user tries to clock in again,  
   **Then** the action is blocked and a message displays: "Please clock out of your active session first."
2. **Given** a completed session from `09:00` to `12:00`,  
   **When** the user tries to add an "In" time of `10:00`,  
   **Then** the system blocks the update due to overlap.
3. **Given** the selected day is Saturday,  
   **When** the page loads,  
   **Then** logging controls are disabled, and a notice reads: "Logging is blocked on weekends."

---

### User Story 3 - Analytics & Consistency Tracking (Priority: P3)

As a logged-in user,  
I want to see my weekly progress graph, consistency sentence, and make-up recommendations to keep myself on track.

**Why this priority**: Provides key insights and motivation to the user based on logged hours and limits.

**Independent Test**: Set limit to 40 hours. Log 8 hours on Monday and Tuesday. Verify progress graph shows 16 hours logged (40%), consistency sentence says consistent, and no make-up banner is shown.

**Acceptance Scenarios**:

1. **Given** the weekly limit is `40` hours and the user logged `16` hours,  
   **When** viewing the dashboard,  
   **Then** the progress graph displays 16 hours logged (40% completion) and 24 hours remaining.
2. **Given** the weekly limit is `40` hours and the user logged `6` hours on Monday (deficit of 2 hours),  
   **When** viewing the dashboard on Tuesday,  
   **Then** the make-up recommendation reads: "You logged a loss of 2.0 hours yesterday. You have to spend 10.0 hours more today to make up for the week."

---

### User Story 4 - Historical Week Navigation (Priority: P4)

As a logged-in user,  
I want to navigate to previous and next weeks to view logs and statistics.

**Why this priority**: Required for tracking long-term productivity and reviewing past entries.

**Independent Test**: Click "Previous Week", verify dates display the preceding Monday-Friday, and that logged entries/charts load past data.

**Acceptance Scenarios**:

1. **Given** the dashboard is displaying the current week,  
   **When** the user clicks "Previous Week",  
   **Then** the calendar updates to the preceding week, and logs and stats for that week are fetched and rendered.

---

### Edge Cases

- **Midnight-crossing sessions**: A user clocks in at `22:00` on Monday and clocks out at `02:00` on Tuesday. The system must attribute the full `4h 0m` duration to Monday (the day the session started) as per the user stories.
- **Deleting the only session**: When a user deletes a time entry that was contributing to the daily total, the daily total, weekly progress, and consistency message must immediately recalculate and reflect the change.
- **Editing an active session**: If a user edits the "In" time of an active session, the system must validate that the new "In" time does not overlap with any completed sessions on that day.
- **Auto-Expired Sessions**: If a session has been auto-closed because it was open for more than 24 hours, the user can edit or delete it just like any other closed session to correct their logged hours.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: The system MUST display a weekly grid/list showing Monday through Friday of the currently selected week.
- **FR-002**: The system MUST highlight the card for the current local weekday (Monday through Friday).
- **FR-003**: The system MUST support clocking in and clocking out for any weekday of the selected week.
- **FR-004**: The system MUST prevent entering an "In" time if another session is currently active. If a session remains active for more than 24 hours, the system will automatically close it (setting the Out time equal to the In time, resulting in a 0-hour duration) to permit new clock-ins.
- **FR-005**: The system MUST validate that any "Out" time is chronologically after its corresponding "In" time.
- **FR-006**: The system MUST prevent any session from overlapping with an existing completed session on the same day.
- **FR-007**: The system MUST block all logging inputs on Saturday and Sunday and display a weekend notice.
- **FR-008**: The system MUST compute and display the sum of durations of all completed sessions per day, formatted as `Xh Ym`.
- **FR-009**: The system MUST allow users to delete any time entry, immediately updating totals.
- **FR-010**: The system MUST allow users to edit the "In" and "Out" times of existing entries, subject to ordering and overlap validation.
- **FR-011**: The system MUST display a weekly progress visualization containing: Total hours logged, hours remaining, and percentage completed.
- **FR-012**: The system MUST display a dynamic consistency sentence:
  - Positive if `Logged Hours >= (Weekly Limit / 5) * Completed Weekdays`
  - Warning if `Logged Hours < (Weekly Limit / 5) * Completed Weekdays`
- **FR-013**: The system MUST display a make-up recovery banner on weekdays if the previous weekday's logged hours were less than the daily average target.
- **FR-014**: The system MUST allow navigating to previous and next weeks, loading corresponding settings and logs. When viewing a historical (past) week, the dashboard MUST display the weekly progress graph and daily logs, but MUST hide the consistency sentence and make-up recovery recommendations since those weeks are already concluded.

### Key Entities *(include if feature involves data)*

- **TimeLog**: Represents a single work session. Attributes:
  - **Id**: Unique identifier (UUID or sequential integer).
  - **Username**: Owner of the log.
  - **Date**: ISO-8601 date (`YYYY-MM-DD`) on which the session started.
  - **InTime**: 24-hour time representation (`HH:mm`) for clock-in.
  - **OutTime**: 24-hour time representation (`HH:mm`) for clock-out (nullable).
  - **Duration**: Computed difference in minutes between OutTime and InTime (nullable).

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: Page navigation and week switching must retrieve past logs and update the UI in under 500ms.
- **SC-002**: 100% of clock-in, clock-out, delete, and edit actions must reflect in the database and recalculate dashboard analytics within 1 second.
- **SC-003**: 100% of overlapping session inputs or weekend logging attempts must be blocked by validation on both frontend and backend.

## Assumptions

- Weekly limits (defaulting to 40.0 if not configured) are fetched and used for all target and consistency calculations.
- All time logging calculations are based on the user's local timezone.
- Deleted time entries cannot be recovered.

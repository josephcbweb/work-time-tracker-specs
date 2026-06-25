# Research and Technical Decisions: Time Tracking and Analytics

This document details the analysis and key technical decisions made for Epic 3 (Time Tracking & Logging) and Epic 4 (Analytics & Recommendations).

---

## Decision 1: Handling Long-Running Active Sessions

- **Decision**: Auto-close active sessions after 24 hours (Option B).
- **Rationale**: Prevents users who forget to clock out from being locked out of the tracking system indefinitely. If a session's start time (`InTime`) is older than 24 hours relative to the current server system time, the system will automatically close it by setting `OutTime = InTime` (resulting in a 0-minute duration). The user is then allowed to clock in again and can edit the auto-closed session's times if they need to correct it.
- **Alternatives Considered**:
  - *No Auto-closure (Option A)*: Rejected because if a user forgets to clock out on Friday, they would be unable to clock in on Monday without first editing or deleting Friday's session, which degrades usability.
  - *Auto-close at end-of-day (Option C)*: Rejected because it blocks valid night shifts or cross-day sessions that cross the midnight boundary (23:59).

---

## Decision 2: Scoping Analytics on Historical Weeks

- **Decision**: Hide consistency check sentences and make-up recommendations for historical weeks (Option B).
- **Rationale**: Dynamic consistency statements (pace check against elapsed workdays) and make-up recovery messages (deficits from "yesterday") are only actionable in real-time during the current week. Displaying active recommendations for a week in the past is confusing and irrelevant. Historical weeks will display the static weekly progress graph (total completed hours vs. that week's limit) and the history of daily logs.
- **Alternatives Considered**:
  - *Historical Calculations (Option A)*: Rejected because showing make-up recommendations for a Wednesday in a week that ended a year ago is not useful or actionable.
  - *Static Summary (Option C)*: Rejected to keep the UI clean and focused on historical records without generating complex retro-active statuses.

---

## Decision 3: TimeLog Schema on CSV

- **Decision**: Persist logs to `/work-time-tracker-backend/data/timelogs.csv` using columns: `id,username,date,in_time,out_time`.
- **Rationale**: Simple flat format that fits standard relational data logs. Easy to parse using Apache Commons CSV and lock-protected during operations.
- **Alternatives Considered**:
  - *Weekly files (e.g. `timelogs_week_YYYY_WW.csv`)*: Rejected due to unnecessary file management overhead. A single file with proper indexing/filtering is highly efficient for the expected scale.

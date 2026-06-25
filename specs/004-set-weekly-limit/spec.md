# Feature Specification: Set Weekly Time Limit

**Feature Branch**: `004-set-weekly-limit`

**Created**: 2026-06-25

**Status**: Draft

**Input**: User description: "implement us2.1"

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Configure Weekly Limit (Priority: P1)

As a logged-in user,  
I want to configure my weekly time limit in hours,  
So that the system can calculate my daily targets and progress.

**Why this priority**: Crucial configuration setting. All dashboard calculations, consistency metrics, and make-up recovery recommendations depend on the daily average target, which is derived directly from the weekly limit.

**Independent Test**: Log in, edit the "Weekly Limit" input field with a valid positive number, click save, verify the UI updates target indicators immediately, and check that a reload preserves the changes.

**Acceptance Scenarios**:

1. **Given** the user is logged into the dashboard,  
   **When** the user enters a positive numerical value (e.g., `40` or `37.5`) in the "Weekly Limit" input field and clicks save,  
   **Then** the value is updated in the database, and the daily targets, progress charts, and recommendations immediately recalculate.
2. **Given** the user is logged into the dashboard,  
   **When** the user enters a value that is not a positive number (e.g., `0`, `-5`, or `abc`) and attempts to save,  
   **Then** the client-side validation prevents submission, highlights the invalid field with an error message, and does not perform an API update.
3. **Given** the user has logged in for the first time,  
   **When** the dashboard page loads,  
   **Then** the system defaults the weekly limit to `40.0` hours, displays it, and prompts the user to configure their custom weekly limit.

---

### Edge Cases

- **Extremely High or Low Values**: What happens if the user inputs a limit of `0.1` hours or `168` hours (the absolute maximum hours in a week)? The system must support fractional hours (e.g., `37.5`) but enforce a maximum validation limit (e.g., `168.0` hours) and a minimum limit (e.g., `0.5` hours) to prevent nonsensical configurations.
- **Save during offline status / network failure**: If the user saves the weekly limit while offline or experiencing a network failure, the application should display a clear error message, keep the unsaved value in the input field, and avoid overwriting local state with outdated values.
- **Decimal Precision**: What happens if a user inputs a value with multiple decimal places (e.g. `37.555`)? The system must round or validate the input to a maximum of one decimal place (e.g. `37.6` or restrict entry to one decimal place).

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: The system MUST retrieve and display the authenticated user's current weekly time limit on the dashboard load.
- **FR-002**: The system MUST default the weekly limit to 40.0 hours if no prior limit has been configured.
- **FR-003**: The system MUST allow users to update their weekly time limit via a dedicated numerical input field on the dashboard.
- **FR-004**: The system MUST validate that the weekly time limit is a positive number (including decimals, up to one decimal place) between `0.5` and `168.0` inclusive.
- **FR-005**: The system MUST recalculate daily targets (`Weekly Limit / 5` weekdays) and weekly progress visualizations immediately upon a successful limit update.
- **FR-006**: The weekly limit MUST be saved securely and associated with the authenticated user ID in the backend data storage (`settings.csv`).
- **FR-007**: The weekly limit is a single global setting per user. Updates to the weekly limit will apply to the user's dashboard calculations across all weeks (current and navigated historical weeks).

### Key Entities *(include if feature involves data)*

- **User Settings**: Represents the user's configuration preferences. Key attributes include:
  - **User Reference**: Association linking settings to a specific registered user.
  - **Weekly Limit**: Numeric value in hours (e.g., `40.0`) representing the global weekly work hour goal.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: The save operation for updating the weekly time limit must complete and update the database in under 1.5 seconds under standard network conditions.
- **SC-002**: 100% of invalid inputs (non-numeric, negative, 0, or greater than 168.0 hours) must be rejected with appropriate feedback before saving.
- **SC-003**: Recalculated dashboard values (daily target, consistency sentence, and progress chart) must update on the screen within 200ms of a successful API response.

## Assumptions

- The frontend and backend communicate via JSON-based REST APIs.
- The default limit of 40.0 hours is sufficient for users who have not yet configured a limit.
- Settings are persisted on the backend in a file named `settings.csv` using the user's unique username/ID as the primary key.

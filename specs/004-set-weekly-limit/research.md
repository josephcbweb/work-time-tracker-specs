# Research and Technical Decisions: Set Weekly Time Limit

## Decision 1: Weekly Limit Persistence Scope

- **Decision**: Single Global Setting. The weekly time limit is defined as a profile-level setting of the user. Any changes to the limit will update the setting globally and will apply to calculations on the dashboard for all weeks (current and historically navigated weeks).
- **Rationale**: Keeps the data model and storage mechanism simple and direct. Avoids complex schema changes in `settings.csv` and simplifies the synchronization of the frontend state (no need to fetch different limits for different weeks). 
- **Alternatives Considered**: 
  - *Week-specific Settings (Option B)*: Rejected because storing a configuration per week increases CSV parsing and writing complexity, and creates overhead when navigating between weeks.
  - *Historical Snapshots (Option C)*: Rejected to avoid unnecessary database rows and state logic complexity for the MVP.

## Decision 2: Backend Storage Schema and Format

- **Decision**: Store setting records in `/work-time-tracker-backend/data/settings.csv` with fields: `username`, `weekly_limit`.
- **Rationale**: Aligns with the overall project architecture which requires CSV persistence in the backend directory.
- **Alternatives Considered**: 
  - *JSON file*: Rejected because the project constitution specifically mandates CSV-based storage files.

## Decision 3: Input Validation and Precision

- **Decision**: Support single decimal place precision (e.g. `37.5` hours). Valid range: `0.5` to `168.0` inclusive.
- **Rationale**: Standard work schedules can include half hours (e.g., 37.5 hours per week). Enforcing a maximum limit of 168.0 prevents configuring more hours than exist in a week.
- **Alternatives Considered**: 
  - *Integer hours only*: Rejected because it restricts realistic flexibility (many part-time or European work weeks are fractional).

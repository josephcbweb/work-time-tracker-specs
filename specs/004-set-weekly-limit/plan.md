# Implementation Plan: Set Weekly Time Limit

**Branch**: `004-set-weekly-limit` | **Date**: 2026-06-25 | **Spec**: [spec.md](spec.md)

**Input**: Feature specification from `/specs/004-set-weekly-limit/spec.md`

## Summary

Implement a feature to allow authenticated users to set and update their global weekly time limit in hours. The React frontend will provide an input field on the dashboard (defaulting to 40.0 hours and prompting the user if not configured), validate the input, and save the settings by calling `PUT /api/user/settings`. The Java backend will store settings per-user in `settings.csv`, validate that the input is a positive number between 0.5 and 168.0, and return the updated settings. Any update will trigger immediate recalculations of the daily average target (`Weekly Limit / 5`) and progress/consistency metrics on the dashboard.

## Technical Context

**Language/Version**: Java 17 (Backend), TypeScript 6.0 / React 19 (Frontend)

**Primary Dependencies**:
- Backend: Spring Boot 3.2.5 (Web, Security, Validation), io.jsonwebtoken (jjwt) 0.11.5, commons-csv 1.10.0
- Frontend: React 19.2.7, Vite 8.1.0, Lucide React (icons)

**Storage**: CSV-based data storage (`settings.csv` for user configuration and targets).

**Testing**:
- Backend: JUnit 5, Spring Security Test
- Frontend: Oxlint for linting, TypeScript compilation validation

**Target Platform**: JVM 17+, Modern Web Browsers

**Project Type**: Monorepo Web Application

**Performance Goals**: Update settings API responds in < 200ms, UI recalculates and renders within 200ms of API success.

**Constraints**:
- Access restricted to authenticated users.
- Settings are saved per-user in CSV format.
- Strictly Vanilla CSS for styling (no Tailwind CSS or UI component libraries).
- Native browser `fetch` API for all backend communication.

**Scale/Scope**: Single global weekly limit setting per user, CSV file storage on backend.

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

| Gate / Rule | Status | Notes |
|-------------|--------|-------|
| Monorepo project structure | Pass | Frontend in `work-time-tracker-frontend/`, Backend in `work-time-tracker-backend/`, Specs in `work-time-tracker-specs/`. |
| API & Date Standards | Pass | Protocol is HTTP/1.1 REST returning application/json. Standard error JSON envelope used on failure. Dates/times follow standards. |
| Authentication & Data Validation | Pass | Ensures that only authenticated users can access/update settings. Checks limit constraints (0.5 to 168.0 hours). |
| Vanilla CSS ONLY on Frontend | Pass | Only custom CSS rules in plain CSS files are permitted. |
| Native fetch on Frontend | Pass | Uses native `fetch` requests inside frontend services without external libraries. |

## Project Structure

### Documentation (this feature)

```text
specs/004-set-weekly-limit/
├── plan.md              # This file
├── research.md          # Phase 0 output
├── data-model.md        # Phase 1 output
├── quickstart.md        # Phase 1 output
├── contracts/
│   └── settings-api.md  # Phase 1 output
└── checklists/
    └── requirements.md  # Spec checklist
```

### Source Code (repository root)

```text
work-time-tracker-backend/
├── src/main/java/com/worktime/tracker/
│   ├── controller/
│   │   └── SettingsController.java          # Handles GET/PUT /api/user/settings
│   ├── service/
│   │   └── SettingsService.java             # Settings validation & CSV storage orchestration
│   ├── repository/
│   │   └── SettingsRepository.java          # Reads/writes user settings to settings.csv
│   ├── model/
│   │   └── UserSettings.java                # Represents setting entity
│   └── dto/
│       ├── SettingsDto.java                 # Request/response payload for settings
│       └── ErrorResponse.java               # Standard error response payload
└── src/test/java/com/worktime/tracker/
    └── controller/
        └── SettingsControllerTest.java      # Integration/unit tests for settings API

work-time-tracker-frontend/
├── src/
│   ├── components/
│   │   └── WeeklyLimitInput.tsx             # Interactive weekly limit editing component
│   ├── context/
│   │   └── SettingsContext.tsx              # Context providing active limit and recalculation functions
│   ├── pages/
│   │   └── DashboardPage.tsx                # Context consumer, contains weekly metrics
│   └── services/
│       └── api.ts                           # API fetch helper for settings
```

**Structure Decision**: Monorepo separation is respected. All backend Java files reside under `work-time-tracker-backend/` and all frontend React/TypeScript files under `work-time-tracker-frontend/`.

## Complexity Tracking

No violations to track. The architecture conforms fully to the monorepo, backend, and frontend constitutions.

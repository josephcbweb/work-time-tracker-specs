# Implementation Plan: Time Tracking and Analytics

**Branch**: `005-time-tracking-analytics` | **Date**: 2026-06-25 | **Spec**: [spec.md](spec.md)

**Input**: Feature specification from `/specs/005-time-tracking-analytics/spec.md`

## Summary

Implement the complete dashboard system for weekdays time-logging, navigation, validation, and analytics. Authenticated users can navigate weeks, clock-in, and clock-out. Daily logs and metrics recalculate automatically. The backend handles REST endpoints, verifies chronological/concurrency/weekend constraints, manages an auto-expiration of active sessions after 24 hours, and persists data to `timelogs.csv`. The frontend renders weekdays Monday-Friday, highlights the current day, displays progress charts, a consistency sentence, and a make-up recovery recommendation (which are hidden when viewing historical weeks).

## Technical Context

**Language/Version**: Java 17 (Backend), TypeScript 6.0 / React 19 (Frontend)

**Primary Dependencies**:
- Backend: Spring Boot 3.2.5 (Web, Security, Validation), io.jsonwebtoken (jjwt) 0.11.5, commons-csv 1.10.0
- Frontend: React 19.2.7, Vite 8.1.0, Lucide React (icons)

**Storage**: CSV-based data storage (`timelogs.csv` for time logs, `settings.csv` for weekly limits).

**Testing**:
- Backend: JUnit 5, Spring Security Test
- Frontend: Oxlint for linting, TypeScript compilation validation

**Target Platform**: JVM 17+, Modern Web Browsers

**Performance Goals**: Log retrieval and week navigation in < 500ms, logging/edit/delete updates database & UI analytics in < 1 second.

**Constraints**:
- Authentication required.
- Chronological validity (Out > In), single active session, overlap prevention, weekend block.
- Auto-closure of active sessions after 24 hours (creates 0-duration closed session).
- Hide active consistency checking/make-up recovery for historical weeks.
- Strictly Vanilla CSS for styling (no Tailwind CSS or UI libraries).
- Native browser `fetch` API for all backend communication.

**Scale/Scope**: Session tracking and analytics, CSV storage file on backend.

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

| Gate / Rule | Status | Notes |
|-------------|--------|-------|
| Monorepo project structure | Pass | Frontend in `work-time-tracker-frontend/`, Backend in `work-time-tracker-backend/`, Specs in `work-time-tracker-specs/`. |
| API & Date Standards | Pass | Protocol is HTTP/1.1 REST returning application/json. Date format: `YYYY-MM-DD`. Time format: `HH:mm`. Timestamps: UTC. Standard error JSON envelope used on failure. |
| Authentication & Data Validation | Pass | Restricts updates to own data. Checks overlap constraints, chronological order, weekend exclusion. |
| Vanilla CSS ONLY on Frontend | Pass | Only custom CSS rules in plain CSS files are permitted. |
| Native fetch on Frontend | Pass | Uses native `fetch` requests inside frontend services without external libraries. |

## Project Structure

### Documentation (this feature)

```text
specs/005-time-tracking-analytics/
├── plan.md              # This file
├── research.md          # Phase 0 output
├── data-model.md        # Phase 1 output
├── quickstart.md        # Phase 1 output
├── contracts/
│   └── timelog-api.md   # Phase 1 output
└── checklists/
    └── requirements.md  # Spec checklist
```

### Source Code (repository root)

```text
work-time-tracker-backend/
├── src/main/java/com/worktime/tracker/
│   ├── controller/
│   │   └── TimeLogController.java           # API endpoints (GET week, POST clock-in/out, PUT, DELETE)
│   ├── service/
│   │   ├── TimeLogService.java              # Logging logic, overlap/weekend validation, auto-expiration task
│   │   └── AnalyticsService.java            # Consistency sentence, make-up target and progress math
│   ├── repository/
│   │   └── TimeLogRepository.java           # Reads/writes logs to timelogs.csv with locking
│   ├── model/
│   │   └── TimeLog.java                     # Represents a work session log
│   └── dto/
│       ├── TimeLogDto.java                  # Request/response payload for logging
│       ├── WeeklySummaryDto.java            # Aggregated weekly logs and progress analytics
│       └── ErrorResponse.java               # Standard error response payload
└── src/test/java/com/worktime/tracker/
    └── controller/
        └── TimeLogControllerTest.java       # Integration tests for time logs endpoints

work-time-tracker-frontend/
├── src/
│   ├── components/
│   │   ├── DayCard.tsx                      # Individual weekday container with logs list & forms
│   │   ├── TimeLogEntry.tsx                 # View, edit, delete buttons for a logged session
│   │   ├── WeeklyProgressGraph.tsx          # Progress ring or bar for hours worked vs. limit
│   │   ├── ConsistencyPanel.tsx             # Shows dynamic average checks & make-up warnings
│   │   └── WeekNavigation.tsx               # Prev/Next week controls with date selection
│   ├── pages/
│   │   └── DashboardPage.tsx                # Integrates weekday list, analytics, and navigation
│   └── services/
│       └── timelogService.ts                # API fetch helpers for logs
```

**Structure Decision**: Monorepo structure is followed. All Java code is placed in `work-time-tracker-backend/` and React components in `work-time-tracker-frontend/`.

## Complexity Tracking

No violations to track.

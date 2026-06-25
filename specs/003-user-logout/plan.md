# Implementation Plan: User Logout

**Branch**: `003-user-logout` | **Date**: 2026-06-25 | **Spec**: [spec.md](file:///Users/joseph.cb/Documents/test-collab/work-time-tracker-specs/specs/003-user-logout/spec.md)

**Input**: Feature specification from `/specs/003-user-logout/spec.md`

## Summary

Implement a secure user logout feature that invalidates the user's active session. The React frontend will clear the JWT token from local/session storage, update the global AuthContext, and redirect the user to the login page. To meet server-side session termination requirements, the client will send a POST request to `/api/auth/logout`. The Java backend will intercept this, validate the token, and add it to an in-memory token blacklist service, preventing the token from being reused in future authenticated requests.

## Technical Context

**Language/Version**: Java 17 (Backend), TypeScript 6.0 / React 19 (Frontend)

**Primary Dependencies**:
- Backend: Spring Boot 3.2.5 (Web, Security, Validation), io.jsonwebtoken (jjwt) 0.11.5
- Frontend: React 19.2.7, Vite 8.1.0, Lucide React (icons)

**Storage**: In-memory token blacklist for JWT invalidation (Backend).

**Testing**:
- Backend: JUnit 5, Spring Security Test
- Frontend: Oxlint for linting, TypeScript compilation validation

**Target Platform**: JVM 17+, Modern Web Browsers (Chrome, Firefox, Safari)

**Project Type**: Monorepo Web Application

**Performance Goals**: Logout execution and page redirection completed in under 1 second.

**Constraints**:
- Must invalidate JWT tokens server-side (using a blacklisting mechanism).
- Must clear client-side token storage and application context state.
- Strictly Vanilla CSS for styling (no Tailwind CSS or UI component libraries).
- Native browser `fetch` API for all backend communication.

**Scale/Scope**: Session-level token management, in-memory cache appropriate for single/multi-user deployment.

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

| Gate / Rule | Status | Notes |
|-------------|--------|-------|
| Monorepo project structure | Pass | Frontend in `work-time-tracker-frontend/`, Backend in `work-time-tracker-backend/`, Specs in `work-time-tracker-specs/`. |
| API & Date Standards | Pass | Protocol is HTTP/1.1 REST returning application/json. Standard error JSON envelope used on failure. |
| Authentication & Data Validation | Pass | Ensures that only valid session tokens are blacklisted and no actions are allowed with blacklisted tokens. |
| Vanilla CSS ONLY on Frontend | Pass | Only custom CSS rules in plain CSS files are permitted. |
| Native fetch on Frontend | Pass | Uses native `fetch` requests inside frontend services without external libraries. |

## Project Structure

### Documentation (this feature)

```text
specs/003-user-logout/
├── plan.md              # This file
├── research.md          # Phase 0 output
├── data-model.md        # Phase 1 output
├── quickstart.md        # Phase 1 output
├── contracts/
│   └── logout-api.md    # Phase 1 output
└── checklists/
    └── requirements.md  # Spec checklist
```

### Source Code (repository root)

```text
work-time-tracker-backend/
├── src/main/java/com/worktime/tracker/
│   ├── controller/
│   │   └── AuthController.java               # Adds logout endpoint
│   ├── service/
│   │   ├── AuthService.java                  # Integrates logout logic
│   │   └── TokenBlacklistService.java        # Manages blacklisted JWTs
│   ├── security/
│   │   ├── JwtAuthenticationFilter.java      # Checks blacklist before authorizing
│   │   └── JwtTokenProvider.java             # Helper methods for JWT handling
│   └── dto/
│       └── ErrorResponse.java                # Standard error response payload
└── src/test/java/com/worktime/tracker/
    └── controller/
        └── AuthControllerTest.java           # Integration/unit tests for logout

work-time-tracker-frontend/
├── src/
│   ├── components/
│   │   └── LogoutButton.tsx                  # Interactive logout button
│   ├── context/
│   │   └── AuthContext.tsx                   # Triggers logout context action
│   ├── pages/
│   │   └── DashboardPage.tsx                 # Integrates LogoutButton in UI
│   └── services/
│       └── api.ts                            # API call helper for logout
```

**Structure Decision**: Monorepo separation is respected. All backend Java files reside under `work-time-tracker-backend/` and all frontend React/TypeScript files under `work-time-tracker-frontend/`.

## Complexity Tracking

No violations to track. The architecture conforms fully to the monorepo, backend, and frontend constitutions.

---
description: "Task list for User Logout backend implementation"
---

# Tasks: User Logout

**Input**: Design documents from `/specs/specs/003-user-logout/`

**Prerequisites**: plan.md (required), spec.md (required for user stories), research.md, data-model.md, contracts/

**Tests**: Tests are included to verify token invalidation and endpoint security as outlined in plan.md and quickstart.md.

**Organization**: Tasks are grouped by setup, foundational, user story, and polish phases to enable independent implementation and testing of the backend logout functionality.

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Can run in parallel (different files, no dependencies)
- **[Story]**: Which user story this task belongs to (e.g., US1, US2, US3)
- Include exact file paths in descriptions

## Path Conventions

- **Backend**: `work-time-tracker-backend/src/` or `work-time-tracker-backend/src/test/` at repository root.

---

## Phase 1: Setup (Shared Infrastructure)

**Purpose**: Verification of initial backend project status and build stability before modification.

- [ ] T001 Verify backend project build and run tests using work-time-tracker-backend/pom.xml

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Core in-memory blacklist infrastructure that MUST be complete before the User Story can be implemented.

**⚠️ CRITICAL**: No user story work can begin until this phase is complete.

- [ ] T002 Implement thread-safe TokenBlacklistService in work-time-tracker-backend/src/main/java/com/worktime/tracker/service/TokenBlacklistService.java
- [ ] T003 [P] Create unit tests for TokenBlacklistService in work-time-tracker-backend/src/test/java/com/worktime/tracker/service/TokenBlacklistServiceTest.java

**Checkpoint**: Foundation ready - user story implementation can now begin.

---

## Phase 3: User Story 1 - Secure User Session Termination (Priority: P1) 🎯 MVP

**Goal**: Implement backend session invalidation via token blacklisting, update security filters, and expose the logout endpoint.

**Independent Test**:
Using cURL or integration test, verify that:
1. Log in returns a valid JWT.
2. GET request on protected endpoint `/api/settings` using the JWT returns 200 OK.
3. POST request on `/api/auth/logout` using the JWT returns 200 OK with success message.
4. GET request on `/api/settings` using the same JWT returns 401 Unauthorized.

### Implementation for User Story 1

- [ ] T004 [US1] Update JwtAuthenticationFilter in work-time-tracker-backend/src/main/java/com/worktime/tracker/security/JwtAuthenticationFilter.java to reject blacklisted tokens
- [ ] T005 [US1] Update AuthService in work-time-tracker-backend/src/main/java/com/worktime/tracker/service/AuthService.java to support logout token invalidation
- [ ] T006 [US1] Implement logout endpoint in AuthController in work-time-tracker-backend/src/main/java/com/worktime/tracker/controller/AuthController.java
- [ ] T007 [P] [US1] Add unit/integration tests for logout endpoint in work-time-tracker-backend/src/test/java/com/worktime/tracker/controller/AuthControllerTest.java
- [ ] T008 [P] [US1] Create integration tests for token blacklist verification in work-time-tracker-backend/src/test/java/com/worktime/tracker/controller/AuthControllerIntegrationTest.java

**Checkpoint**: At this point, User Story 1 (backend portion) should be fully functional and testable independently.

---

## Phase 4: Polish & Cross-Cutting Concerns

**Purpose**: Cleanup, validation, and quality verification.

- [ ] T009 Run Maven test command on work-time-tracker-backend/pom.xml to verify all backend tests pass
- [ ] T010 Perform code review and check thread-safety under work-time-tracker-backend/src/main/java/com/worktime/tracker/

---

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: Can start immediately.
- **Foundational (Phase 2)**: Depends on Setup (Phase 1) completion - BLOCKS User Story 1.
- **User Story 1 (Phase 3)**: Depends on Foundational (Phase 2) completion.
- **Polish (Phase 4)**: Depends on User Story 1 (Phase 3) completion.

### User Story Dependencies

- **User Story 1 (P1)**: Can start after Foundational (Phase 2) - No dependencies on other stories.

### Within Each User Story

- Service layer (T005) and Filter (T004) changes should be done before exposing the Endpoint (T006).
- Unit and integration tests (T007, T008) can be developed in parallel with the implementation but depend on the endpoint structure.

### Parallel Opportunities

- Foundational test task (T003) can be written in parallel with service task (T002).
- Unit tests (T007) and integration tests (T008) can run in parallel.

---

## Parallel Example: User Story 1

```bash
# Implement the filters and service modifications concurrently:
Task: "Update JwtAuthenticationFilter in work-time-tracker-backend/src/main/java/com/worktime/tracker/security/JwtAuthenticationFilter.java to reject blacklisted tokens"
Task: "Update AuthService in work-time-tracker-backend/src/main/java/com/worktime/tracker/service/AuthService.java to support logout token invalidation"

# Add tests in parallel:
Task: "Add unit/integration tests for logout endpoint in work-time-tracker-backend/src/test/java/com/worktime/tracker/controller/AuthControllerTest.java"
Task: "Create integration tests for token blacklist verification in work-time-tracker-backend/src/test/java/com/worktime/tracker/controller/AuthControllerIntegrationTest.java"
```

---

## Implementation Strategy

### MVP First (User Story 1 Only)

1. Complete Phase 1: Setup.
2. Complete Phase 2: Foundational.
3. Complete Phase 3: User Story 1.
4. **STOP and VALIDATE**: Run integration test suite using MockMvc to verify token invalidation.

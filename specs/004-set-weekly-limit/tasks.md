---
description: "Task list for Set Weekly Time Limit backend implementation"
---

# Tasks: Set Weekly Time Limit

**Input**: Design documents from `/specs/specs/004-set-weekly-limit/`

**Prerequisites**: plan.md (required), spec.md (required for user stories), research.md, data-model.md, contracts/

**Tests**: Tests are included to verify settings retrieval, persistence in CSV, input validation bounds, and API controller security.

**Organization**: Tasks are grouped by setup, foundational, user story, and polish phases to enable independent implementation and testing of the settings configuration on the backend.

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

**Purpose**: Core model, DTO, and CSV storage setup that MUST be complete before the User Story can be implemented.

**⚠️ CRITICAL**: No user story work can begin until this phase is complete.

- [ ] T002 Create domain model UserSettings in work-time-tracker-backend/src/main/java/com/worktime/tracker/model/UserSettings.java
- [ ] T003 Create SettingsDto in work-time-tracker-backend/src/main/java/com/worktime/tracker/dto/SettingsDto.java
- [ ] T004 [P] Create CSV repository SettingsRepository in work-time-tracker-backend/src/main/java/com/worktime/tracker/repository/SettingsRepository.java

**Checkpoint**: Foundation ready - user story implementation can now begin.

---

## Phase 3: User Story 1 - Configure Weekly Limit (Priority: P1) 🎯 MVP

**Goal**: Implement backend logic to retrieve, validate, and save the weekly work hour limit per user.

**Independent Test**:
Using cURL or integration test, verify that:
1. Log in to obtain a valid JWT.
2. GET request on `/api/user/settings` using the JWT returns the default `40.0` hours with status 200.
3. PUT request on `/api/user/settings` with invalid limit (e.g. `-5` or `169.0`) returns `400 Bad Request`.
4. PUT request on `/api/user/settings` with valid limit (e.g. `37.5`) returns `200 OK` and saves to `settings.csv`.
5. Subsequent GET request on `/api/user/settings` returns `37.5`.

### Implementation for User Story 1

- [ ] T005 [US1] Implement SettingsService in work-time-tracker-backend/src/main/java/com/worktime/tracker/service/SettingsService.java
- [ ] T006 [US1] Create SettingsController in work-time-tracker-backend/src/main/java/com/worktime/tracker/controller/SettingsController.java
- [ ] T007 [P] [US1] Write controller tests in work-time-tracker-backend/src/test/java/com/worktime/tracker/controller/SettingsControllerTest.java
- [ ] T008 [P] [US1] Write integration/repository tests for SettingsRepository and SettingsService verification in work-time-tracker-backend/src/test/java/com/worktime/tracker/service/SettingsServiceTest.java

**Checkpoint**: At this point, User Story 1 (backend settings) should be fully functional and testable independently.

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

- Service layer (T005) and Controller (T006) should be implemented in sequence.
- Unit and integration tests (T007, T008) can be developed in parallel but depend on model/DTO structure.

### Parallel Opportunities

- CSV repository creation (T004) can run in parallel with the model (T002) and DTO (T003) setup.
- Controller unit tests (T007) and repository integration tests (T008) can run in parallel.

---

## Parallel Example: User Story 1

```bash
# Implement the endpoint and write tests concurrently:
Task: "Create SettingsController in work-time-tracker-backend/src/main/java/com/worktime/tracker/controller/SettingsController.java"

# Add tests in parallel:
Task: "Write controller tests in work-time-tracker-backend/src/test/java/com/worktime/tracker/controller/SettingsControllerTest.java"
Task: "Write integration/repository tests for SettingsRepository and SettingsService verification in work-time-tracker-backend/src/test/java/com/worktime/tracker/service/SettingsServiceTest.java"
```

---

## Implementation Strategy

### MVP First (User Story 1 Only)

1. Complete Phase 1: Setup.
2. Complete Phase 2: Foundational.
3. Complete Phase 3: User Story 1.
4. **STOP and VALIDATE**: Run integration test suite using MockMvc to verify settings GET/PUT functions.

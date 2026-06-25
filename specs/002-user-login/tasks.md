# Tasks: User Login

**Input**: Design documents from `specs/specs/002-user-login/`

**Prerequisites**: [plan.md](file:///Users/joseph.cb/Documents/test-collab/work-time-tracker-backend/specs/specs/002-user-login/plan.md) (required), [spec.md](file:///Users/joseph.cb/Documents/test-collab/work-time-tracker-backend/specs/specs/002-user-login/spec.md) (required)

**Conventions**: [constitution-backend.md](file:///Users/joseph.cb/Documents/test-collab/work-time-tracker-backend/specs/.specify/memory/constitution-backend.md)

**Scope**: Backend implementation tasks only

**Format**: `[ID] [P?] [Story] Description`
- **[P]**: Can run in parallel (different files, no dependencies)
- **[Story]**: Which user story this task belongs to (e.g., US1, US2, US3)
- Contains exact file paths relative to backend root

---

## Phase 1: Setup (Shared Infrastructure)

**Purpose**: Backend configuration and JWT dependencies setup

- [X] T001 Add JWT dependencies (`jjwt-api`, `jjwt-impl`, and `jjwt-jackson`) in `pom.xml`
- [X] T002 Add JWT secret key and expiration properties in `src/main/resources/application.properties`

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Core authentication framework infrastructure

**⚠️ CRITICAL**: No user story work can begin until this phase is complete

- [X] T003 [P] Implement token generation, extraction, and validation logic in `src/main/java/com/worktime/tracker/security/JwtTokenProvider.java`
- [X] T004 [P] Implement authentication request interception filter in `src/main/java/com/worktime/tracker/security/JwtAuthenticationFilter.java`
- [X] T005 Update HTTP security rules to permit `/api/auth/login` and register `JwtAuthenticationFilter` in `src/main/java/com/worktime/tracker/security/SecurityConfig.java`

**Checkpoint**: Foundation ready - user story implementation can now begin in parallel

---

## Phase 3: User Story 1 - User Authentication / Login (Priority: P1) 🎯 MVP

**Goal**: Authenticate users, verify password hash via BCrypt, and return JWT token.

**Independent Test**: Verify by making a POST request to `/api/auth/login` with valid/invalid credentials and checking response status/envelope.

### Tests for User Story 1
- [X] T006 [P] [US1] Create unit tests for login business logic in `src/test/java/com/worktime/tracker/service/AuthServiceTest.java`
- [X] T007 [P] [US1] Create integration tests for authentication endpoints in `src/test/java/com/worktime/tracker/controller/AuthControllerIntegrationTest.java`

### Implementation for User Story 1
- [X] T008 [P] [US1] Create login credentials request payload class in `src/main/java/com/worktime/tracker/dto/LoginRequest.java`
- [X] T009 [P] [US1] Create authentication response token envelope class in `src/main/java/com/worktime/tracker/dto/LoginResponse.java`
- [X] T010 [US1] Implement user verification and token generation service in `src/main/java/com/worktime/tracker/service/AuthService.java`
- [X] T011 [US1] Expose the POST `/api/auth/login` endpoint in `src/main/java/com/worktime/tracker/controller/AuthController.java`
- [X] T012 [US1] Map authentication failure exception to 401 Unauthorized in `src/main/java/com/worktime/tracker/exception/GlobalExceptionHandler.java`

**Checkpoint**: User Story 1 is fully functional and testable independently.

---

## Phase 4: Polish & Cross-Cutting Concerns

**Purpose**: Cleanup, validation and test verification

- [X] T013 Run full test suite using Maven to verify login and signup endpoints
- [X] T014 [P] Verify API responses against the contract defined in `specs/specs/002-user-login/contracts/login-api.md`

---

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: No dependencies - can start immediately
- **Foundational (Phase 2)**: Depends on Setup completion - BLOCKS all user stories
- **User Stories (Phase 3)**: Depends on Foundational phase completion
- **Polish (Phase 4)**: Depends on all desired user stories being complete

### User Story Dependencies

- **User Story 1 (P1)**: Can start after Foundational (Phase 2) - No dependencies on other stories

### Within Each User Story

- Tests (T006, T007) should be written first and verify they fail prior to service and controller changes.
- DTO models (T008, T009) before service (T010)
- Service (T010) before Controller (T011)
- Controller (T011) and Exception Handler (T012) before Integration Tests can pass.

---

## Parallel Example: User Story 1

```bash
# Launch DTO tasks in parallel
Task: "Create login request DTO in src/main/java/com/worktime/tracker/dto/LoginRequest.java"
Task: "Create login response DTO in src/main/java/com/worktime/tracker/dto/LoginResponse.java"
```

---

## Implementation Strategy

### MVP First (User Story 1 Only)

1. Complete Phase 1: Setup
2. Complete Phase 2: Foundational
3. Complete Phase 3: User Story 1
4. **STOP and VALIDATE**: Test User Story 1 independently using cURL or tests
5. Complete Phase 4: Polish

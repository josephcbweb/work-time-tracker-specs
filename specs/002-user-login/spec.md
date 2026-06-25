# Feature Specification: User Login

**Feature Branch**: `002-user-login`

**Created**: 2026-06-25

**Status**: Draft

**Input**: User description: "use @[userstories.md] to implement us1.2"

## User Scenarios & Testing *(mandatory)*

### User Story 1 - User Authentication / Login (Priority: P1)

As a registered user, I want to log in with my username and password, so that I can access my workspace.

**Why this priority**: User login is a fundamental security barrier. It ensures that user logs, settings, and dashboards are protected, private, and accessible only to their rightful owners. Login is a hard dependency for almost all post-signup features.

**Independent Test**: Can be fully tested by navigating to `/login`, entering the credentials of a user created in US1.1, and clicking "Login". Successful authentication redirects the user to the dashboard `/dashboard` and establishes a secure session.

**Acceptance Scenarios**:

1. **Given** the login page is loaded,  
   **When** I enter valid credentials and click "Login",  
   **Then** I am authenticated, a secure session/JWT is established, and I am redirected to the dashboard.
2. **Given** the login page is loaded,  
   **When** I enter incorrect credentials and click "Login",  
   **Then** an error message is displayed: "Invalid username or password," and I remain on the login page.

---

### Edge Cases

- **Missing Fields**: If either the username or password input is empty, validation must prevent form submission and highlight the missing fields.
- **Non-existent Username**: If a login attempt uses a username that does not exist in the database, the system must return the identical generic error message: "Invalid username or password" to prevent username enumeration attacks.
- **Session/Token Expiration**: When the authentication token/session expires, any subsequent request to access a protected route (like the dashboard) must be denied and redirect the user back to the login page.
- **Already Authenticated**: If a user who is already authenticated navigates directly to the login page, they must be automatically redirected to the dashboard.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: The system MUST provide a user login interface containing inputs for username and password, and a submit action.
- **FR-002**: The system MUST validate that both username and password fields are non-empty and non-blank prior to submitting the login request.
- **FR-003**: The backend authentication service MUST verify the login credentials by matching the provided username and comparing the provided password with the stored password hash using BCrypt.
- **FR-004**: The system MUST return a cryptographically signed token (JWT) or establish a secure HTTP-only cookie-based session upon successful authentication.
- **FR-005**: The system MUST return a generic, user-friendly error message "Invalid username or password" when login fails due to incorrect credentials or non-existent username.
- **FR-006**: The frontend application MUST store the authentication token securely and automatically include it in the headers of all subsequent API requests.
- **FR-007**: The system MUST redirect the user to the dashboard upon successful login.
- **FR-008**: The system MUST block unauthenticated requests to the dashboard and redirect them to the login screen.

### Key Entities *(include if feature involves data)*

- **User**: Represents the user account being authenticated.
  - `username` (String): Unique case-insensitive identifier.
  - `passwordHash` (String): Securely hashed password.
- **AuthToken**: Represents the temporary authorization session generated upon successful login.
  - `token` (String): Cryptographically signed token containing user identifier and expiration claim.
  - `expiresAt` (Timestamp): Timestamp when the token becomes invalid.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: A user with valid credentials can complete the login flow (credentials input to dashboard redirect) in under 1.5 seconds under standard network conditions.
- **SC-002**: 100% of failed login attempts display the same generic error message, ensuring zero leakage of user existence (verified via test suite).
- **SC-003**: Access to all authenticated pages and endpoints is blocked with a 401 Unauthorized status code if the request is missing a valid token.
- **SC-004**: The session token must expire after 24 hours of inactivity.

## Assumptions

- User registration (US1.1) is fully implemented and the user database contains hashed passwords.
- HTTPS is enforced for transferring credentials securely in non-development environments.
- The token signing secret key is managed via secure environment variables.

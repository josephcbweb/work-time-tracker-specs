# Research & Decisions: User Logout

This document outlines the design decisions, rationale, and alternatives considered for implementing the User Logout (US1.3) feature.

## 1. Server-Side Token Invalidation

- **Decision**: In-memory thread-safe Token Blacklist on the backend.
- **Rationale**:
  - Since JWT is stateless, the token remains signature-valid until it naturally expires. To meet `FR-002` ("immediately terminate the session on the server"), we must track revoked tokens.
  - An in-memory cache using a thread-safe `ConcurrentHashMap` tracks blacklisted tokens alongside their expiration times.
  - The security filter `JwtAuthenticationFilter` is updated to check this blacklist on every incoming request.
  - A scheduled cleanup thread or eviction strategy removes expired tokens from the blacklist to prevent memory leaks.
- **Alternatives Considered**:
  - **Client-only invalidation**: Just deleting the token from client-side storage. Rejected because if the token is intercepted or copied before logout, it remains valid and usable by third parties until it naturally expires.
  - **CSV File Persistence for Blacklisted Tokens**: Storing blacklisted tokens in a CSV file. Rejected due to performance constraints. I/O latency for checking and writing CSV files on every API call is too high and unnecessary for short-lived session tokens.

## 2. Client-Side Session Cleanup Flow

- **Decision**: Immediate removal of local tokens, state reset in `AuthContext`, and redirect to `/login` regardless of API call outcome.
- **Rationale**:
  - The priority of logout is securing the physical terminal. If the user clicks "Logout" while offline, the local token must be deleted immediately.
  - Even if the network call to `/api/auth/logout` fails (due to timeout or server offline), the client-side state cleanup must succeed and redirect the user.
- **Alternatives Considered**:
  - **Awaiting Server Confirmation**: Waiting for the API response before clearing local storage. Rejected because if the network is disconnected, the logout action would fail, leaving the user's session active on the device.

## 3. Handling Concurrent Browser Tabs

- **Decision**: The backend API will return `401 Unauthorized` for any request presenting a blacklisted token, which the client-side API layer will intercept to clear local state and force redirect.
- **Rationale**:
  - If a user has the app open in Tab A and Tab B, logging out from Tab A blacklists the token on the server.
  - The next action in Tab B will make an API call, receive a `401 Unauthorized` status (due to token blacklist check on the server), and the frontend `fetch` interceptor will automatically wipe Tab B's local state and redirect to `/login`.
- **Alternatives Considered**:
  - **Broadcast Channel API / Storage Event Listener**: Broadcasting logout event across tabs locally. This is a nice-to-have optimization but not strictly necessary as a first step; the backend-enforced 401 interception provides a robust security fallback.

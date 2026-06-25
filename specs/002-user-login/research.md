# Research & Decisions: User Login

This document outlines the design decisions, rationale, and alternatives considered for implementing the User Login (US1.2) feature.

## 1. Authentication Mechanism

- **Decision**: JSON Web Tokens (JWT) for session management.
- **Rationale**: 
  - Allows stateless authentication between the React frontend and Java backend.
  - The backend generates a signed token containing the authenticated user's details (username) and expiration date.
  - The frontend stores the token in `localStorage` or `sessionStorage` and attaches it as a `Bearer` token in the HTTP `Authorization` header for protected API calls.
  - Fits the constitution's recommendation.
- **Alternatives Considered**:
  - **Stateful Session Cookies**: Requires session database tracking or file persistence on the backend. Harder to configure for local development when client and server run on different ports (CORS issues).

## 2. Password Verification Method

- **Decision**: BCrypt hash verification on the backend.
- **Rationale**: 
  - Required by the overall and backend constitutions.
  - Leverages standard secure hashing (`BCryptPasswordEncoder`) to verify credentials without storing or logging plain-text passwords.
- **Alternatives Considered**:
  - **SHA-256 / PBKDF2**: Rejected to maintain strict alignment with the project's BCrypt constraint.

## 3. CSV File Thread-Safety for Login Lookup

- **Decision**: Synchronized wrapper around the CSV parsing code in `UserRepository`.
- **Rationale**:
  - Since multiple users might attempt to log in simultaneously, concurrent read requests on `users.csv` must be thread-safe.
  - A thread-safe file reading utility prevents concurrent access collisions.
- **Alternatives Considered**:
  - **ReentrantReadWriteLock**: Offers better read performance but adds unnecessary complexity for a simple CSV files backend with a low scale.

## 4. Frontend UI Design and Styling

- **Decision**: Vanilla CSS with custom properties (CSS variables) for theme colors, spacing, and transition durations.
- **Rationale**:
  - Strictly aligns with the Frontend Constitution prohibiting CSS frameworks like Tailwind CSS.
  - Custom variables allow easy dark-mode-first styling, glassmorphism UI elements, and interactive micro-animations.
- **Alternatives Considered**:
  - **Tailwind CSS**: Explicitly banned by the constitution.

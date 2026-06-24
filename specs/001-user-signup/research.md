# Research Notes: User Signup

This document details the architectural decisions, technology selections, and rationales for implementing the User Signup feature (US1.1) in the Work Time Tracker application.

## Key Technical Decisions

### 1. Backend Framework & REST API
* **Decision**: Java 17 with Spring Boot (Spring Boot Starter Web, Spring Boot Starter Security).
* **Rationale**: Spring Boot provides a lightweight and robust environment for building REST APIs. It provides out-of-the-box support for REST endpoints, exception handling, validation, and security primitives (e.g. BCrypt hashing).
* **Alternatives Considered**: 
  * *Plain Java HTTP Server / Servlets*: Rejected due to high development overhead for routing, JSON serialization/deserialization, and validation.
  * *Other JVM Frameworks (e.g., Quarkus/Micronaut)*: Rejected as Spring Boot is standard and has the largest community and documentation footprint.

### 2. Password Hashing & Security
* **Decision**: BCrypt algorithm using Spring Security's `BCryptPasswordEncoder` (strength factor 10).
* **Rationale**: BCrypt is specifically designed for password hashing, utilizing a configurable work factor to protect against brute-force attacks. It is a strict mandate of the project constitution.
* **Alternatives Considered**:
  * *PBKDF2 or Argon2*: Excellent alternatives, but BCrypt is explicitly specified in the project constitution and requirements.
  * *SHA-256 (with salt)*: Rejected because SHA-256 is a fast cryptographic hash and is highly vulnerable to GPU-accelerated brute-force attacks if used for password storage.

### 3. CSV File Storage Thread-Safety & Concurrency
* **Decision**: Use a synchronized file lock or repository wrapper pattern utilizing Java's `ReentrantReadWriteLock` for reading/writing the `users.csv` file, along with `Apache Commons CSV` for robust parsing.
* **Rationale**: Since the backend uses CSV files for persistence, multiple concurrent registration requests could lead to race conditions or file corruption. Applying a thread-safe read/write lock in the database manager layer ensures that only one write operation happens at a time while allowing concurrent reads.
* **Alternatives Considered**:
  * *Raw Java File I/O (buffered writers)*: Rejected because it does not handle CSV escaping, quoting, and newlines correctly under concurrent access.
  * *Plain java `synchronized` blocks*: Safe but limits read concurrency. A `ReadWriteLock` is more performant.

### 4. UI Design & Styling
* **Decision**: React with Vanilla CSS using CSS Custom Properties (Variables) for tokens (electric indigo `#6366f1` and deep grey `#0f172a` for a dark-mode-first aesthetic). Smooth micro-animations for input focus and button hover states.
* **Rationale**: Custom Vanilla CSS gives us maximum control over layout and aesthetics, ensuring we deliver a premium, modern feel without Tailwind CSS bloat.
* **Alternatives Considered**:
  * *Tailwind CSS*: Rejected as vanilla CSS is requested for maximum design flexibility and styling control.

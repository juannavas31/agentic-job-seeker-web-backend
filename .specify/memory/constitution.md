# agentic-job-seeker-web-backend Constitution

This is a backend server for the agentic-job-seeker web application.

## Core Principles

### I. User Authentication via Keycloak

User authentication is based on Keycloak, which acts as the issuer of JWTs used by this backend to validate an authenticated user. The backend MUST verify the token's signature using Keycloak's public keys (JWKS endpoint) and extract user claims from the validated token. All protected endpoints MUST enforce this authentication. The user's email address MUST be used as the preferred user identifier.

### II. RESTful External APIs

The product MUST use only external APIs that are compliant with the RESTful principles. Before adopting an API, verify that it is well documented and offers an OpenAPI documentation.

### III. SOLID Architectural Principles and Separation of Concerns

The system MUST follow SOLID architectural principles and keep responsibilities separate across API, business-logic, and data-access code. Components MUST depend on clear contracts and avoid coupling domain rules to transport or persistence details. Design choices that materially depart from these principles MUST be documented and justified.

### IV. Unit and Integration Tests Are Mandatory

Every user story and implemented task MUST include relevant automated unit tests and integration tests. Tests MUST cover the behavior changed by the work, and the work is not complete until both test types pass. A task that changes no executable behavior still requires tests that validate its relevant contract or effect.

## Technical Constraints

The backend is a Python web service built with FastAPI. New backend work MUST follow the repository's documented structure (see [README.md](../../README.md)) and keep HTTP endpoint handling separate from business logic. External-service integrations must be based on RESTful APIs.

## Development Workflow and Quality Gates

User stories and implementation tasks MUST identify their expected behavior and test coverage. Reviewers MUST verify that changes comply with these principles and that the required unit and integration tests pass. Exceptions require a constitution amendment; a feature plan or implementation task alone cannot waive a principle.

## Governance

This constitution governs project specifications, plans, implementation tasks, and code changes. Amendments MUST be reviewed and recorded in this file. Each amendment MUST update the version according to semantic versioning: MAJOR for incompatible governance changes, MINOR for new or materially expanded principles or sections, and PATCH for clarifications that do not change requirements. Project changes MUST be reviewed for compliance with the current version.

**Version**: 2.0.0 | **Ratified**: 2026-10-29 | **Last Amended**: 2026-10-31

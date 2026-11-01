# Feature Specification: Job Seeker Backend API

**Feature Branch**: `001-job-seeker-api`  
**Created**: 2026-10-31
**Status**: Draft  
**Input**: User description: "Backend server offering a RESTful API (FastAPI) with MongoDB/Beanie persistence, Keycloak JWT authentication, and an external LangGraph agent for job-offer fetching, evaluation, resume adaptation, and cover-letter generation."

## User Scenarios & Testing *(mandatory)*

### User Story 1 — Upload a Resume (Priority: P1)

As a user of an API client, I want to upload my resume for the first time to the backend server, so that I can use it later.

**Why this priority**: A stored resume is the prerequisite for every other feature (job offers, adapted resumes, cover letters). Without it, nothing else works.

**Independent Test**: Can be fully tested by calling `POST /resumes` with a valid file and JWT, and verifying the resume is persisted and retrievable.

**Acceptance Scenarios**:

1. **Given** a resume in markdown, docx, or pdf format and an authenticated user with a valid Keycloak JWT, **When** the user calls `POST /resumes` using multipart/form-data, **Then** the resume is stored in the backend and a success response is returned.
2. **Given** a resume and a non-authenticated user, **When** the user calls `POST /resumes`, **Then** the response is 401 Unauthorized.
3. **Given** a resume in a format other than docx, pdf, or markdown and an authenticated user, **When** the user calls `POST /resumes`, **Then** the response is 400 Bad Request (invalid data).
4. **Given** a valid resume that has already been uploaded and an authenticated user, **When** the user calls `POST /resumes` with that resume, **Then** the response is 409 Conflict.

---

### User Story 2 — Replace a Resume (Priority: P1)

As a user of an API client, I want to upload an updated version of my resume to the backend server, so that I can use it later.

**Why this priority**: Users must be able to keep their resume current; this is equally critical to the initial upload.

**Independent Test**: Can be fully tested by uploading a resume via `POST /resumes`, then replacing it via `PUT /resumes`, and verifying the new version is stored.

**Acceptance Scenarios**:

1. **Given** a resume and an authenticated user with a valid JWT, **When** the user calls `PUT /resumes` using multipart/form-data, **Then** the resume is stored, replacing the existing one.
2. **Given** a resume and a non-authenticated user, **When** the user calls `PUT /resumes`, **Then** the response is 401 Unauthorized.
3. **Given** a resume in a format other than docx, pdf, or markdown and an authenticated user, **When** the user calls `PUT /resumes`, **Then** the response is 400 Bad Request.
4. **Given** an authenticated user who has previously uploaded a resume, **When** the user uploads a new resume in a different format via `PUT /resumes`, **Then** the backend deletes the previous resume and stores the new one.
5. **Given** an authenticated user who has not previously uploaded any resume, **When** the user calls `PUT /resumes`, **Then** the response is 404 Not Found.

---

### User Story 3 — Download Own Resume (Priority: P2)

As a user of an API client, I want to download my resume from the backend server, so that I can use it later.

**Why this priority**: Downloading the stored resume is important for user convenience but is not required before job-offer or cover-letter flows can deliver value.

**Independent Test**: Can be fully tested by uploading a resume, then calling `GET /resumes/{user_identifier}` and verifying the returned file matches the uploaded content.

**Acceptance Scenarios**:

1. **Given** an authenticated user with a valid JWT who has previously uploaded a resume, **When** the user calls `GET /resumes/{user_identifier}`, **Then** the backend returns the user's resume in pdf format using multipart/form-data.
2. **Given** a non-authenticated user, **When** the user calls `GET /resumes/{user_identifier}`, **Then** the response is 401 Unauthorized.
3. **Given** an authenticated user who has not previously uploaded any resume, **When** the user calls `GET /resumes/{user_identifier}`, **Then** the response is 404 Not Found.

---

### User Story 4 — Get Qualified Job Offers (Priority: P2)

As a job seeker, I want to get job offers for a role published in a past time range enriched with an updated resume, so that I can easily apply.

**Why this priority**: This is the core value proposition — matching the user's resume against job offers via the external agent — but it depends on a stored resume (US1).

**Independent Test**: Can be fully tested by uploading a resume, then calling `GET /job-offers` with mandatory query parameters and verifying the response matches the defined schema.

**Acceptance Scenarios**:

1. **Given** an authenticated user who has previously uploaded a resume, **When** the user calls `GET /job-offers` with `role_title` and `time_range` (and optionally `min_score`), **Then** the backend returns a list of qualified job offers matching the defined JSON schema.
2. **Given** a non-authenticated user, **When** the user calls `GET /job-offers`, **Then** the response is 401 Unauthorized.
3. **Given** an authenticated user, **When** the user calls `GET /job-offers` and misses any mandatory parameter, **Then** the response is 400 Bad Request.
4. **Given** an authenticated user who has not uploaded any resume, **When** the user calls `GET /job-offers`, **Then** the response is 404 Not Found (resume missing).

---

### User Story 5 — Download a Cover Letter (Priority: P3)

As a job seeker, I want to download a cover letter generated for a specific job offer, so that I can easily apply.

**Why this priority**: Cover letters enhance the application but depend on job offers already being fetched (US4) and a resume being stored (US1).

**Independent Test**: Can be fully tested by uploading a resume, fetching job offers, then calling `GET /cover-letter?job_offer_id=<id>` and verifying a plain-text file is returned.

**Acceptance Scenarios**:

1. **Given** an authenticated user with a valid `job_offer_id`, **When** the user calls `GET /cover-letter?job_offer_id=<id>`, **Then** the backend returns a cover letter file in plain-text format using multipart/form-data.
2. **Given** an authenticated user, **When** the user calls `GET /cover-letter` with a non-existing `job_offer_id`, **Then** the response is 404 Not Found.
3. **Given** an authenticated user, **When** the user calls `GET /cover-letter` with no `job_offer_id` parameter, **Then** the response is 400 Bad Request.
4. **Given** a non-authenticated user, **When** the user calls `GET /cover-letter`, **Then** the response is 401 Unauthorized.

---

### User Story 6 — Download an Adapted Resume (Priority: P3)

As a job seeker, I want to download a resume adapted to a specific job offer, so that I can easily apply.

**Why this priority**: Adapted resumes are a premium output that depends on job offers (US4) and a stored resume (US1).

**Independent Test**: Can be fully tested by uploading a resume, fetching job offers, then calling `GET /resumes/{user_identifier}/job_offer/{job_offer_id}` and verifying a pdf file is returned.

**Acceptance Scenarios**:

1. **Given** an authenticated user with a stored resume and a valid `job_offer_id`, **When** the user calls `GET /resumes/{user_identifier}/job_offer/{job_offer_id}`, **Then** the backend returns an adapted resume in pdf format using multipart/form-data.
2. **Given** an authenticated user, **When** the user calls the endpoint with an optional `user_prompt` query parameter, **Then** the backend incorporates the prompt into the adaptation.
3. **Given** a non-authenticated user, **When** the user calls the endpoint, **Then** the response is 401 Unauthorized.

---

### Edge Cases

- What happens when the uploaded file is empty (0 bytes)? → 400 Bad Request.
- What happens when the JWT is expired or has an invalid signature? → 401 Unauthorized.
- What happens when the Keycloak server is unreachable during token validation? → 503 Service Unavailable.
- What happens when the external agent (LangGraph) is unreachable or times out? → 503 Service Unavailable.
- What happens when `time_range` is in an unrecognized format? → 400 Bad Request.
- What happens when `min_score` is outside the valid range (e.g., negative or > 1)? → 400 Bad Request.
- What happens when a user requests another user's resume by providing a different `user_identifier`? → 403 Forbidden (users may only access their own resources).

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: The backend MUST expose a RESTful API implemented with FastAPI.
- **FR-002**: The backend MUST expose an OpenAPI specification endpoint at `GET /openapi`.
- **FR-003**: The backend MUST use MongoDB to store and retrieve data, accessed via the Beanie ODM library.
- **FR-004**: The backend MUST authenticate users via JWTs issued by Keycloak, verifying the token signature using Keycloak's public keys (JWKS endpoint).
- **FR-005**: The backend MUST use the user's email address as the preferred user identifier, extracted from the validated JWT claims.
- **FR-006**: The backend MUST integrate with an external agent via the LangGraph SDK to fetch job offers, evaluate job offers, create adapted resumes, and generate cover letters.
- **FR-007**: `POST /resumes` MUST accept a resume file (markdown, docx, or pdf) via multipart/form-data with `UploadFile`, `Form`, and `File` from FastAPI, and store it for the authenticated user.
- **FR-008**: `PUT /resumes` MUST replace an existing resume for the authenticated user, accepting the same formats and multipart approach as `POST /resumes`.
- **FR-009**: `GET /resumes/{user_identifier}` MUST return the user's resume in pdf format via multipart/form-data.
- **FR-010**: `GET /job-offers` MUST accept `role_title` (mandatory), `time_range` (mandatory), and `min_score` (optional) as query parameters, and return a list of qualified job offers matching the defined response schema.
- **FR-011**: `GET /cover-letter` MUST accept `job_offer_id` (mandatory) and `user_prompt` (optional) as query parameters, and return a plain-text cover letter file via multipart/form-data.
- **FR-012**: `GET /resumes/{user_identifier}/job_offer/{job_offer_id}` MUST return a resume adapted to the specified job offer in pdf format via multipart/form-data, with an optional `user_prompt` query parameter.
- **FR-013**: All endpoints MUST be protected by Keycloak JWT authentication via a FastAPI dependency that extracts the `Authorization` header, verifies the token signature, and extracts user claims.
- **FR-014**: The backend MUST use `python-jose[cryptography]` for JWT handling and `httpx` for communicating with the Keycloak server.
- **FR-015**: Keycloak configuration MUST be defined via environment variables: `KEYCLOAK_URL`, `REALM`, `JWKS_URL`, and `EXPECTED_ISSUER`.
- **FR-016**: Users MUST NOT be able to access resources belonging to other users (enforced by matching the JWT email claim to the requested `user_identifier`).

### Key Entities

- **Resume**: Represents a user's uploaded resume. Key attributes: user identifier (email), file content, file format (markdown/docx/pdf), upload timestamp, last-updated timestamp.
- **Job Offer**: Represents a job posting fetched and evaluated by the external agent. Key attributes: job_offer_id, job_title, company, date, job_description, salaries_range, employees, score, adapted_resume (URL).
- **Cover Letter**: Represents a generated cover letter for a specific job offer. Key attributes: job_offer_id, user identifier, content (plain text), generation timestamp.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: A user can upload a resume in any supported format and receive a success response within 5 seconds.
- **SC-002**: A user can retrieve their stored resume and receive the file in pdf format within 3 seconds.
- **SC-003**: A user can request job offers and receive a qualified list within 30 seconds (accounting for external agent latency).
- **SC-004**: A user can request a cover letter for a specific job offer and receive the file within 15 seconds.
- **SC-005**: A user can request an adapted resume for a specific job offer and receive the pdf within 15 seconds.
- **SC-006**: All endpoints return 401 for unauthenticated requests, with 100% consistency.
- **SC-007**: All endpoints return appropriate 400/404/409 error codes for invalid input, missing resources, and conflicts, respectively.
- **SC-008**: The OpenAPI specification endpoint returns a valid OpenAPI document accessible at `GET /openapi`.

## Assumptions

- The Keycloak server is already deployed and configured with a realm and client for this backend.
- The external LangGraph agent is a separate service accessible via the LangGraph SDK; its deployment and availability are outside the scope of this backend.
- MongoDB is available and accessible from the backend; connection details are provided via environment variables.
- The `time_range` query parameter format (e.g., "last_7_days", "last_30_days") will be defined during implementation planning.
- The `score` field in job offers is a numeric value between 0 and 1 (inclusive), representing the agent's evaluation confidence.
- File size limits for resume uploads will be defined during implementation planning (default: 10 MB).

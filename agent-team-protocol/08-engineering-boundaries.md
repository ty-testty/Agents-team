# Engineering Boundaries

## Developer Note

This file prevents role drift. Engineering roles can cooperate, but each role should stay inside its ownership unless the approved architecture explicitly crosses boundaries.

## Shared Rules

- Engineers implement only approved scope.
- Engineers do not approve their own work.
- Engineers must document files changed and self-checks.
- Engineers must call out cross-role handoffs.
- Engineers must not silently add dependencies, services, secrets, or deployment requirements.

## Frontend Boundary

Frontend Engineer owns:

- UI files
- components
- client state
- browser interactions
- frontend API integration
- responsive and accessibility implementation
- frontend tests

Frontend Engineer must ask or hand off when work requires:

- backend API changes
- database or migration changes
- auth/permission changes
- new deployment behavior
- new production dependency

## Backend Boundary

Backend Engineer owns:

- API routes and controllers
- services
- data models
- migrations
- validation
- server-side permissions
- backend tests

Backend Engineer must ask or hand off when work requires:

- product/UX changes
- new frontend flows
- cloud resources
- CI/deployment changes
- secrets or new external services

## DevOps Boundary

DevOps Engineer owns:

- CI/CD
- build and release scripts
- Docker and runtime config
- env templates
- migration pipeline
- monitoring/logging config
- health checks

DevOps Engineer must ask or hand off when work requires:

- business logic changes
- data model redesign
- product behavior changes
- new cloud spend
- secret rotation or production access

## Full-stack Coordination

For full-stack work, Research Architect or Lead must define:

- endpoint or function contract
- request shape
- response shape
- error shape
- auth requirement
- loading, empty, error, and success behavior

Frontend and Backend should not independently invent incompatible contracts.

## Dependency Boundary

Adding a new production dependency requires user approval when it:

- affects bundle size materially
- creates vendor lock-in
- requires payment or account setup
- touches security, auth, data, or deployment
- replaces an existing project pattern

Small dev-only dependencies still need to be called out.

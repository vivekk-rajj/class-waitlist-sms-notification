# Prompt Traceability

This file records the prompt sequence used to produce the architecture-only deliverable for ENG-135180.

1. **Requirements intake**

   > Review the ENG-135180 Class Waitlist SMS Notification Job assignment. Extract the happy path, unhappy path, accessibility, telemetry, security, database-schema, API-contract, and design-system requirements. Respect the explicit constraint: Capstone 1 architectural planning only, with no feature code.

2. **Schema design**

   > Design a production-ready PostgreSQL schema for classes, students, waitlist entries, notification jobs, notification attempts, and audit events. Include types, primary/foreign keys, uniqueness, indexes, statuses, timestamps, idempotency, retry state, consent, privacy, retention, and transaction boundaries. Produce a Mermaid ERD.

3. **API contract design**

   > Define versioned JSON API contracts for listing and mutating waitlists, creating asynchronous notification jobs, reading job status, claiming worker attempts, and simulating analytics. Include request/response examples, validation, error format, pagination, idempotency, Twilio mock behavior, retries, and slow-connectivity behavior.

4. **Edge-case and NFR review**

   > Audit the architecture against empty states, invalid inputs, slow 3G/loading indicators, keyboard accessibility and ARIA, XSS-safe input handling, telemetry logging, secret/PII protection, deterministic retries, and duplicate prevention. Document any requirement that belongs to the future UI rather than this architecture-only phase.

5. **Final verification**

   > Verify that the repository contains a README, architecture/schema document, Mermaid ERD, API contract document, and this PROMPTS.md file; that no real credentials or PII are present; and that the documents clearly state that no production feature code is included.

The resulting files are `docs/architecture.md`, `docs/api-contracts.md`, this `PROMPTS.md`, and the root `README.md`.

# Class Waitlist SMS Notification Job

Architecture-only deliverable for ENG-135180. This Capstone 1 submission defines the relational data model, ERD, and API contracts for a Twilio-compatible SMS notification workflow. It intentionally contains no production feature UI or real Twilio integration.

## Deliverables

- [Architecture and database schema](docs/architecture.md)
- [API contracts](docs/api-contracts.md)
- [Prompt traceability log](PROMPTS.md)

## Scope

The design supports staff-managed class waitlists, deterministic promotion ordering, notification job creation, a Twilio mock provider, retries, idempotency, auditability, and operational recovery during poor connectivity. The future client should implement the UI with keyboard navigation, visible loading states, accessible empty/error states, sanitized inputs, and a monochromatic design system using 16px/32px spacing tokens.

## Validation

This repository is documentation-only. There is no application build or test command to run yet. Review the Mermaid ERD and example requests/responses in the linked documents before implementation begins.

No credentials, phone numbers, or other real personal information are included.

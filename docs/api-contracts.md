# API Contracts

Base path: `/api/v1`. All responses are JSON and include `requestId`. Authentication and authorization are assumed to be provided by the hosting platform; every mutation records the authenticated actor in `audit_events`.

## Common conventions

- `Content-Type: application/json`.
- UUID identifiers and ISO-8601 UTC timestamps.
- Mutations require `Idempotency-Key` (1–200 printable characters).
- List endpoints support `limit` (1–100, default 25), `cursor`, and stable sort order.
- Validation errors return HTTP 422; malformed JSON returns 400; unauthenticated/unauthorized requests return 401/403; missing resources return 404; conflicting state returns 409; transient server/provider failures return 503.

```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "One or more fields are invalid.",
    "fields": {"phone": "Use an E.164 phone number."}
  },
  "requestId": "req_01J..."
}
```

## List a class waitlist

`GET /api/v1/classes/{classId}/waitlist?status=waiting&limit=25&cursor=...`

Response `200`:

```json
{
  "items": [
    {
      "id": "entry-uuid",
      "position": 1,
      "status": "waiting",
      "student": {"id": "student-uuid", "displayName": "Example Student"},
      "joinedAt": "2026-10-14T10:00:00Z"
    }
  ],
  "nextCursor": null,
  "requestId": "req_01J..."
}
```

An empty result is still `200` with `items: []`; the client displays “No data found.” Phone numbers are never returned by list endpoints.

## Add a student to a waitlist

`POST /api/v1/classes/{classId}/waitlist`

Request:

```json
{
  "student": {
    "externalKey": "student-123",
    "displayName": "Example Student",
    "phone": "+15555550123",
    "smsConsent": true
  }
}
```

Validation: required fields, display name 1–200 characters, E.164 phone, consent must be true for SMS, and all text is trimmed/normalized and rejected if it contains control characters. The server stores only sanitized values and a phone hash for duplicate detection.

Response `201`: the waitlist entry, student-safe display fields, position, and timestamps. Repeating the same idempotency key returns the original `201` response. An existing student/class pair returns `409 WAITLIST_ENTRY_EXISTS` unless the entry is already removed and an explicit reactivation flow is added.

## Remove a waitlist entry

`DELETE /api/v1/classes/{classId}/waitlist/{entryId}`

Requires `Idempotency-Key`. Response `204` on the first successful removal and on an idempotent repeat. The record is soft-removed with an audit event.

## Create notification jobs

`POST /api/v1/classes/{classId}/notification-jobs`

Request:

```json
{
  "maxRecipients": 1,
  "messageTemplate": "A spot opened in {{className}}. Reply YES to enroll."
}
```

The service locks eligible entries in position order, verifies consent and current status, sanitizes the template, and creates at most one queued SMS job per eligible entry. `maxRecipients` is 1–100. The response is `202 Accepted` because delivery is asynchronous:

```json
{
  "jobIds": ["job-uuid"],
  "created": 1,
  "skipped": [{"entryId": "entry-2", "reason": "SMS_NOT_CONSENTED"}],
  "requestId": "req_01J..."
}
```

## Get notification job status

`GET /api/v1/notification-jobs/{jobId}`

Response `200` exposes status, attempt count, timestamps, sanitized error code, and provider message ID when available. It never exposes a phone number, raw provider payload, or message body.

```json
{
  "id": "job-uuid",
  "status": "sent",
  "attemptCount": 1,
  "provider": "twilio_mock",
  "providerMessageId": "SM-mock-uuid",
  "createdAt": "2026-10-14T10:01:00Z",
  "sentAt": "2026-10-14T10:01:02Z",
  "requestId": "req_01J..."
}
```

## Run/claim a notification job (worker contract)

`POST /api/v1/notification-jobs/{jobId}/attempts`

Requires an internal worker credential and idempotency key. The worker claims the job, calls the Twilio mock adapter, records an attempt, and transitions the job atomically.

Twilio mock behavior:

- Valid consent and valid normalized phone: return deterministic `SM-mock-{jobId}` and mark `sent`.
- Configured transient simulation: return `PROVIDER_UNAVAILABLE`; retry with exponential backoff.
- Invalid/blocked/opted-out recipient: return a permanent error and mark `failed`.
- The mock must never make a network call or require a real API key.

Response `200` for a completed attempt, `409 JOB_ALREADY_PROCESSING` when another worker owns the lease, and `503 PROVIDER_UNAVAILABLE` only when the caller should retry the worker operation.

## Analytics simulation

`POST /api/v1/analytics/interactions`

This optional internal endpoint accepts an allow-listed event name only:

```json
{"event": "class_waitlist_primary_action_completed"}
```

The server logs `[Analytics] User interacted with Class Waitlist SMS Notification Job (Twilio Mock)` and returns `204`. No student, phone, message, or request payload is sent to an analytics vendor.

## Slow connectivity and client behavior

Clients should preserve the last successful list response while a refresh is pending, show a visible non-blocking loading indicator, disable duplicate submits while a mutation is in flight, and retry safe GET requests with bounded backoff. Mutations rely on `Idempotency-Key` so a timeout followed by a retry cannot create duplicate waitlist entries or SMS jobs. Keyboard focus should move to a success/error summary after completion without trapping focus.

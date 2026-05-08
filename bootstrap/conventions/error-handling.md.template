# Error handling

How errors flow across the stack — wire shape, classification, and
client expectations.

---

## Wire shape

All API errors return a single shape. Pick one and hold the line:

```json
{
  "error": {
    "code": "user.not_found",
    "message": "User does not exist.",
    "details": { ... }
  }
}
```

- `code` — stable string identifier. Consumers branch on this.
- `message` — human-readable, English, not localized at API layer.
- `details` — optional object, error-specific.

## HTTP status mapping

- `400` — validation failure (request shape wrong, field invalid)
- `401` — auth missing or invalid
- `403` — auth valid, action forbidden
- `404` — resource not found
- `409` — conflict (idempotency violation, concurrent modification)
- `422` — business rule violation (request shape valid, semantic invalid)
- `429` — rate limit
- `500` — unexpected server error
- `503` — dependency unavailable

Avoid `400` as a catch-all. Pick the specific code.

## Client expectations

- **iOS / web** must branch on `error.code`, not on `error.message`.
- **iOS / web** must handle 5xx with retry + backoff for safe (idempotent)
  operations only. Non-idempotent operations require an idempotency key
  before retry.
- Display `error.message` to users only when it's safe and useful;
  otherwise translate `error.code` to localized copy on the client.

## Server discipline

- Never leak stack traces, internal paths, or DB error messages into
  `error.message` in production.
- Log at `error` level for 5xx, `warn` for unexpected 4xx, `info` for
  expected 4xx.
- Every 5xx generates a request ID returned in `details.request_id` for
  client-side logging.

## Cross-cutting errors

*Errors that aren't tied to one endpoint — auth expiry, rate limits,
maintenance windows. Document the global handling here.*

- _(populate during /audit)_

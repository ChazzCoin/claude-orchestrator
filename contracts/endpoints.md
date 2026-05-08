# Endpoints

Canonical API surface across the stack. Markdown for now → executable
spec (OpenAPI / GraphQL schema) once the audit settles.

---

## Conventions

- **Versioning** — see ADR if one exists; otherwise document the chosen
  approach here (path-based `/v1/`, header-based, content-type negotiated).
- **Authentication** — see `conventions/auth.md` for the auth model. This
  file lists which endpoints are public vs authenticated vs internal.
- **Error model** — see `conventions/error-handling.md`. All endpoints
  return errors in the documented shape.
- **Idempotency** — non-GET endpoints document their idempotency key
  expectation, given pod-restart can replay requests.
- **Pagination** — pick one shape (cursor, page+size, since+limit) and
  hold the line.
- **Long-running work** — sync endpoints stay under the ingress timeout
  (see `platform-constraints.md`). Anything longer uses the async-job
  pattern: `POST /<resource> → 202 + job_id`, `GET /jobs/<id>`.

---

## Endpoints

*Group by resource. One section per endpoint. Format:*

```
### POST /v1/users

**Purpose:** <one sentence>

**Owner:** <which sub-repo implements>

**Consumed by:** <which sub-repo(s) call this>

**Auth:** <public | bearer-token | service-to-service>

**Request:**
```json
{ ... }
```

**Response 200:**
```json
{ ... }
```

**Errors:** <which error codes are real, not just "any 4xx">

**Notes:** <idempotency, timeouts, side effects, scaling concerns>
```

_(populate during /audit)_

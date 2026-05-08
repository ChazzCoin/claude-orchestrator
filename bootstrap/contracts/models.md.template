# Data models

Canonical data shapes used across the stack. Markdown is the source of
truth **for now** — the long-term direction is an executable schema
(shared types package, OpenAPI components, or protobuf) with generated
clients per platform.

When that migration happens, this file becomes a pointer to the
generated source.

---

## Conventions

- **Field names** — see `conventions/naming.md`. Pick one casing per
  surface (e.g. `snake_case` on the wire, `camelCase` in TS, `camelCase`
  in Swift) and document the boundary where transformation happens.
- **IDs** — `UUID` v4 unless there's a reason. Strings on the wire.
- **Timestamps** — ISO-8601 UTC strings on the wire (`2026-05-07T18:51:00Z`).
- **Nullability** — explicit. A field is required, optional, or nullable;
  pick one and say which.
- **Enums** — string enums on the wire, never numeric. Document the full
  value set here, not in the consumer.

---

## Core entities

*One section per first-class entity. Format:*

```
### Entity name

**Purpose:** <one sentence — what this represents in the domain>

**Owners:** <which sub-repo owns the canonical record>

| Field          | Type            | Required | Notes                          |
|----------------|-----------------|----------|--------------------------------|
| id             | string (UUID)   | yes      | server-generated               |
| created_at     | string (ISO-8601)| yes     | server-generated               |
| ...            | ...             | ...      | ...                            |

**Lifecycle:** <created where, mutated where, deleted where>

**Indexes / queries:** <hot-path queries this shape supports>

**Open questions:** <anything unresolved about this entity>
```

_(populate during /audit)_

---

## Relationships

*How the entities relate. Cardinality (1:1, 1:N, N:M), foreign keys,
ownership boundaries.*

- _(populate)_

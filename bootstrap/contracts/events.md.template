# Events

Async messages: queues, pub/sub topics, webhooks, websocket frames —
anything that crosses sub-repos without a direct HTTP request/response.

If your stack is purely synchronous today, this file stays empty.
That's fine — note it under "current state" below so it's not mistaken
for "we forgot to fill this in."

---

## Conventions

- **Event names** — see `conventions/naming.md`. Past-tense verb on the
  resource (`user.created`, `payment.captured`).
- **Schemas** — every event has a versioned schema. Schemas live with
  the data models (`models.md`) or in a separate registry — document
  which.
- **Delivery semantics** — at-least-once is the default. Consumers
  document idempotency.
- **Ordering** — document per topic. If ordering matters, document
  partitioning key.
- **Retention** — document per topic. Replay window matters for
  consumer recovery.

---

## Current state

- _(populate: "no async messaging today" or list the systems in use —
  Service Bus, EventHub, Kafka, RabbitMQ, SQS, etc.)_

---

## Events

*One section per event. Format:*

```
### user.created

**Topic / queue:** <name>
**Producer:** <which sub-repo emits>
**Consumers:** <which sub-repo(s) subscribe, what they do with it>
**Schema:**
```json
{ ... }
```
**Delivery:** at-least-once | exactly-once | at-most-once
**Ordering:** none | per <key> | global
**Retention:** <duration>
```

_(populate during /audit)_

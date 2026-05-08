# Platform constraints

Runtime, infra, and operational realities that **constrain architectural
design decisions**. The devops kit (or your own knowledge) populates this.
Contract decisions in `contracts/` reference back to here.

The point: when designing an endpoint, an event, or a data shape, the
constraints below are not aspirations — they are the rules the running
system enforces. Designs that ignore them break in production.

---

## Compute

*e.g. Kubernetes pod limits, function cold starts, ingress timeouts,
restart behavior, autoscaling triggers.*

- _(populate during /audit)_

## Network

*e.g. ingress TLS termination, public vs private endpoints, egress
restrictions, DNS, regional topology.*

- _(populate)_

## Identity & secrets

*e.g. workload identity model, token TTLs, secret rotation, KMS, where
service-to-service auth lives.*

- _(populate)_

## Data

*e.g. DB engine, sharding model, backup cadence, RPO/RTO, transaction
guarantees, connection pool size × replica count, hot path query
patterns.*

- _(populate)_

## Observability

*e.g. log retention, trace sampling, metric cardinality limits, alerting
shape.*

- _(populate)_

## Deployment

*e.g. CI/CD shape, deploy cadence, blue/green vs rolling, rollback
mechanism, feature flag system if any.*

- _(populate)_

## Compliance & regional

*e.g. data residency, PII handling, audit trail requirements, geographic
restrictions on services.*

- _(populate)_

---

## Design rules that fall out

A short list of "given the above, this is non-negotiable." Updated as the
constraints above are filled in.

*Example:*

- Ingress timeout is 30s → no synchronous endpoint may exceed 25s budget;
  longer work uses the async-job + status pattern documented in
  `contracts/endpoints.md`.

- _(populate as the constraints get specific)_

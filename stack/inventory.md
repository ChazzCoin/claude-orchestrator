# Stack inventory

Every service / app / repo in this company's stack, with the runtime
truth needed to reason about it. Populated by `/audit`, kept current as
things change.

This is the answer to "what do we actually have."

---

## Sub-repos

*One section per repo. Format:*

```
### <name>
- **Repo:** <git remote>
- **Local path:** <where it lives on disk>
- **Role:** <api | ios | web | devops | other>
- **Stack:** <language, framework, runtime version>
- **Deploy target:** <where it runs — k8s cluster, App Store, CDN, etc.>
- **Exposes:** <endpoints, UI surfaces, infra resources, events>
- **Consumes:** <which other components — by name>
- **Status:** <production | beta | in-development>
- **Notes:** <anything that's a gotcha for design decisions>
```

_(populate during /audit)_

---

## Out-of-stack dependencies

*Third-party services the stack depends on — auth providers, payment,
analytics, observability, etc. Each one is a coupling point worth
recording.*

- _(populate)_

---

## Cross-cutting components

*Shared libraries, shared types packages, shared design system — things
consumed by more than one repo. List by name, point at the source of
truth.*

- _(populate)_

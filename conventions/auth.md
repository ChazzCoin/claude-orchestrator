# Authentication & authorization

How identity flows across the stack. The single biggest source of
cross-platform drift if it's not pinned.

---

## Identity model

- **Authentication provider:** _(e.g. Auth0, Firebase Auth, Cognito,
  custom)_
- **Token type:** _(JWT bearer | opaque token | session cookie)_
- **Token TTL:** _(access token + refresh token TTLs)_
- **Refresh model:** _(rotating refresh tokens | sliding session |
  none)_
- **Storage:**
  - iOS: _(Keychain | secure enclave | …)_
  - Web: _(httpOnly cookie | localStorage with risk acknowledged | …)_
  - API server-to-server: _(workload identity | service principal | …)_

## Authorization model

- **Granularity:** _(roles | scopes | per-resource ACLs | tenant-scoped)_
- **Where it's enforced:** _(API gateway | per-endpoint middleware |
  per-handler — pick one and stay there)_
- **Who can grant what:** _(admin user | self-service | invitation
  flow)_

## Multi-tenancy

*If applicable.*

- **Tenant identifier:** _(field name, type, where it lives in the
  token / claims)_
- **Tenant scoping:** _(every query is implicitly scoped | explicit
  tenant_id parameter)_
- **Cross-tenant access:** _(is it allowed, who, how audited)_

## Service-to-service

- **Inter-service calls:** _(workload identity | shared secret | mTLS)_
- **Devops / pipeline access to APIs:** _(personal token | service
  principal)_

## Open questions

*Auth gets re-litigated every 18 months. Capture what's parked.*

- _(populate during /audit)_

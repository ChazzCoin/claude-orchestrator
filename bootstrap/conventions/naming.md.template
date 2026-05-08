# Naming conventions

Cross-stack naming rules. The point is uniformity at the boundaries —
sub-kits should not need to negotiate naming on a feature-by-feature
basis.

---

## Wire format (HTTP / events)

- **JSON keys:** _(snake_case | camelCase — pick and document)_
- **Path segments:** kebab-case, plural for collections (`/api/v1/users`)
- **Query parameters:** _(snake_case | camelCase — match JSON keys for
  consistency)_
- **HTTP headers:** standard kebab-case (`X-Request-Id`)

## Per-platform internal

- **TypeScript / web:** camelCase fields, PascalCase types
- **Swift / iOS:** camelCase properties, PascalCase types
- **API server:** _(language convention — e.g. snake_case for Python,
  camelCase for Node)_

## Boundary transformation

State explicitly which layer in each repo converts wire format to
internal format. This avoids the "everyone transforms ad hoc" anti-pattern.

- _(populate: "API server uses snake_case end-to-end; iOS converts at
  the network layer in NetworkService.swift; web converts at the API
  client layer in src/api/")_

## Identifiers

- **Entity IDs:** `<entity>_id` on the wire (e.g. `user_id`, `payment_id`)
- **Database tables:** plural, snake_case (`users`, `payment_methods`)
- **K8s resources:** kebab-case (`api-deployment`, `web-ingress`)

## Event names

- Past-tense verb on resource: `user.created`, `payment.captured`,
  `order.shipped`
- Lowercase, dot-separated.

## File / repo / branch

- File names: kebab-case
- Repo names: kebab-case (`<company>-api`, `<company>-ios`, `<company>-web`)
- Feature branches: `<type>/<short-slug>` (`feat/multi-tenant-users`)

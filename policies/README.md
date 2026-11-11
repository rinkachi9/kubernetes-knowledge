# Policies

Policies that configure groups of resources in a namespace or on a node: defaults and bounds for container resources, quotas on namespace totals, and limits on process IDs. Corresponds to the "Policies" chapter of the Kubernetes documentation Concepts.

TODO: Write the introduction to policies (what a policy object is, how it differs from RBAC and admission policies).

## Notes in reading order

1. [Limit ranges](limit-ranges.md): default requests and limits, minimum and maximum, interaction with resource quotas.

## Planned topics

- Resource quotas (compute, storage and object count quotas, scopes, priority class quotas)
- Process ID limits and reservations
- Validating admission policies (CEL)

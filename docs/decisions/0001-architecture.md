# Decision record: Ports and adapters for the runtime

- Status: accepted
- Date: 2026-09-11
- Owners: tllib maintainers
- Affected domains: master (initial application)
- Affected features: none; no handoff contract changes

## Context

`tllib` is the downstream runtime for contracts prepared in `tllib-specs`. The
runtime needs to integrate external providers without coupling domain behavior
to a database, network client, filesystem, or framework. The initial `master`
domain already exposes `MasterProvider` as a protocol, but the boundary was not
yet documented or protected by an automated check.

This is a repository-level decision because it governs every domain module and
the location of future integrations.

## Scientific boundary

The scientific meaning and observable obligations remain authoritative in the
published handoff contracts and their upstream sources. This decision changes
only runtime organization. It does not define new master semantics, algorithms,
entities, equations, or provider behavior beyond the existing protocol.

## Decision

The runtime uses a ports-and-adapters architecture:

- Domain modules contain models, domain errors, operations, and port contracts.
- A port is an internal protocol describing the capability the domain needs.
- Adapters implement ports and live under `src/tllib/adapters/` or a documented
  domain-specific adapter package when a concrete integration is introduced.
- Domain modules must not import adapters, infrastructure packages, databases,
  network clients, filesystems, or application frameworks.
- Adapters may depend on domain models and ports, but domain code must not
  depend on a concrete adapter.
- Application composition owns provider construction, registration, and
  lifetime. The domain does not discover or construct external services.

The implementation strategy, concrete adapter technology, and composition
mechanism remain open unless a later decision constrains them.

## Alternatives considered

### Alternative A: direct infrastructure imports

Each domain could call a selected database or HTTP client directly. This would
be simple for one integration, but it would make domain behavior difficult to
test and would couple every consumer to an infrastructure choice. It was
rejected because the dependency direction would be inverted.

### Alternative B: a framework-first service layer

The runtime could require a web or dependency-injection framework and place
domain operations inside framework services. This would provide conventions for
composition, but would add a global runtime dependency before one is required
and would make library use outside that framework harder. It was rejected in
favor of a framework-neutral boundary.

## Consequences

### Positive

- Domain operations can be tested with in-memory protocol implementations.
- External integrations can change without changing domain code.
- The dependency direction is explicit and mechanically checked.
- The runtime remains usable without a database, network, or framework.

### Negative or costly

- Each integration requires a small adapter and an explicit composition step.
- Contributors must maintain port contracts and architecture tests.
- A very small feature may have more files than a direct integration.

### Neutral or deferred

- No concrete adapter technology is selected by this decision.
- Adapter configuration, retries, transactions, and observability remain
  integration-specific concerns.

## Compatibility and migration

This is a package-internal organizational decision and introduces no new
handoff obligation or breaking public API. Existing `MasterProvider`,
`MasterRegistry`, and domain models remain compatible. New integrations should
implement the existing port and be wired by the application composition root.

If existing domain code later needs an external capability, introduce or extend
a protocol first, then implement the adapter separately. A future decision may
move an adapter package without changing the port contract.

## Validation evidence

- `tests/test_architecture.py` accepts the current domain imports.
- The same test rejects a temporary domain module importing an infrastructure
  dependency.
- A unit test calls `list_masters` with an in-memory provider.
- The complete `pytest` suite is the required runtime validation command.

## Traceability

- Runtime port: `tllib/src/tllib/domains/master/providers.py`
- Runtime operation: `tllib/src/tllib/domains/master/operations.py`
- Upstream authority: `tllib-specs/handoff/` and its contract authority order
- No feature handoff, IR, mathematical contract, or oracle is changed.

## Follow-up

- Add concrete adapters only when an integration requirement is accepted.
- Add conformance tests when a published handoff package supplies observable
  behavior for the master domain.
- Revisit the adapter package layout if domain-specific integrations require a
  stronger convention.
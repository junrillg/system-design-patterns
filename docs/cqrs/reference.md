# CQRS: boundaries and recovery

This note describes a design scenario. It does not claim that an inspected source application implements CQRS.

## What is essential

CQRS separates models used to change information and retrieve it. It can live inside a modular monolith with a shared database. A GET/POST split, a mediator library, or separate folders is insufficient evidence by itself. A command can return an identifier, outcome and committed version; it need not return no data.

Event sourcing stores authoritative state as an event history. CQRS can instead store ordinary current-state records. An outbox is a delivery mechanism, not necessarily a complete event history. A saga coordinates a workflow across local transactions; the projection in this example only derives a read view.

## Concurrency, retries and delivery

- **Approval conflict:** check the authenticated actor's scope and expected onboarding-record version within the write transaction. A conditional update or equivalent concurrency control must prevent two commands from both committing against the same version. Reject conflicts for reconciliation; do not blindly reapply an old decision.
- **Request retry:** if the response is lost, retry with the same scoped request identity and compare meaningful inputs. Persist deduplication with the business result. Reusing an identity with changed input is a conflict. Outbox deduplication does not itself protect command acceptance.
- **Durable handoff:** save the authoritative update and event in one local transaction. A relay/projector worker reads only committed events. Mark delivered after the read-store commit. A crash between these commits causes another delivery, not atomicity across both databases.
- **Version rule:** this example uses a full snapshot of every field owned by this projection, with a monotonically increasing version per tenant/employee. Apply only a strictly newer version using an atomic guarded write. Older or duplicate snapshots become no-ops. This rule is unsafe for arbitrary delta events: increment/decrement updates need ordered processing, gap detection and/or an inbox.
- **Failure:** retry transient errors with backoff. Retain repeatedly failing events for operator recovery, alert on lag and oldest outstanding event, and fix/replay failed work. Eventual consistency requires a working recovery path; it is not a fixed delivery-time guarantee.
- **Rebuild:** seed a fresh projection from authoritative current-state snapshots and catch up concurrent changes with version guards before switching readers. Retain deletion tombstones through rebuild/catch-up so old events cannot resurrect removed rows. Do not assume the outbox retains enough history for a full replay.

## User-visible consistency

The command response carries the committed version. The UI acknowledges that save immediately and refetches until the relevant summary reaches it. If catching up takes too long, show that the summary is delayed and retain the confirmed outcome. A read projection never authorizes an irreversible business decision; the command side validates current authoritative state. Do not confuse this UI acknowledgement with a guarantee that every read is immediately current.

Query authorization and tenant filters remain mandatory. Only the projector writes its view; users issue commands to change business facts. No read-store technology or production latency is asserted.

## Primary sources

- [Microsoft: CQRS pattern](https://learn.microsoft.com/en-us/azure/architecture/patterns/cqrs) — shared-store and separate-store variants, optional messaging and event sourcing.
- [Martin Fowler: CQRS](https://martinfowler.com/bliki/CQRS.html) — separate update/display models, scope and complexity tradeoffs.
- [Microsoft: Materialized View pattern](https://learn.microsoft.com/en-us/azure/architecture/patterns/materialized-view) — derived read views, refresh, rebuilding and staleness.
- [AWS: Transactional outbox](https://docs.aws.amazon.com/prescriptive-guidance/latest/cloud-design-patterns/transactional-outbox.html) — durable handoff and duplicate-delivery considerations.

Sources checked 3 November 2026. Versioning, UI acknowledgement and recovery choices are explicit design policies for this example. They require implementation and testing before any production guarantee.

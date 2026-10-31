# Saga: recovery and boundaries

The [main note](README.md) uses an employee-onboarding design scenario. Its service boundaries, ordering, laptop reservation and cancellation policy are illustrative assumptions. This entry does not establish a saga implementation in either application covered by the [implementation catalogue](../pattern-catalogue/README.md).

## What the coordinator owns

A durable workflow record keeps the saga ID, current phase, participant operation IDs, confirmed outcomes and outstanding compensation. A restart resumes from that record. A single logical coordinator can have redundant instances; competing instances need ownership or version checks so they do not advance the same workflow inconsistently.

The ordered steps are not one transaction spanning every system. Each participant commits locally. If a participant commits but its reply is lost, its result and coordinator progress can temporarily disagree. An operation identity and reconciliation close that gap.

## Payroll result and recovery

| Result | Action |
| --- | --- |
| Enrollment accepted | Record success and reliably complete onboarding confirmation. |
| Definitive rejection; nothing enrolled | Release reservation R9, then cancel the pending HR onboarding. Persist unfinished cleanup until both succeed. |
| Timeout or missing reply | Query by the operation ID or safely repeat the same command. Do not infer that enrollment failed. |
| Compensation unavailable | Keep `COMPENSATING`, retry according to policy, and surface the unresolved action for operator recovery. |

If payroll actually enrolled the employee and policy requires cancellation, this design needs a payroll cancellation action too. Releasing only the laptop would leave an unresolved effect. A late success must be reconciled against the saga's current phase rather than silently reviving canceled onboarding.

## Safe commands and provisional state

- Use stable operation identities such as `(saga_id, step)`. Repeating a command with the same intended inputs reuses its outcome; different inputs under that identity are rejected.
- Make release target **R9 belonging to this onboarding**, rather than setting a laptop globally to available. Conditional ownership checks protect a later reservation from stale cleanup.
- Keep pending employees inactive until required setup is confirmed. Concurrent work must respect that provisional state; sagas do not provide automatic isolation between participants.
- Make final confirmation durable and retryable. A lost confirmation reply must not trigger cancellation after enrollment has already been accepted under the stated policy.
- Treat irreversible actions separately. A sent email cannot be unsent, and a salary transfer cannot be treated like a provisional reservation. The process may require a correction, refund, forward recovery or human decision.

## Orchestration and neighboring patterns

This example uses **orchestration**: one coordinator chooses the next action. **Choreography** distributes progression across event handlers. Either still needs explicit failure recovery.

[Request idempotency](../request-idempotency/README.md) protects repeated acceptance at an API boundary; saga participants also need protection for their own repeated commands. [Transactional outbox](../transactional-outbox/README.md) can atomically save local state and a message intent. It does not make the whole saga atomic or prevent every duplicate effect. These notes describe cooperating mechanisms, not verified integration code.

## Primary references

Checked **31 October 2026**:

- [Microsoft: saga distributed transactions](https://learn.microsoft.com/en-us/azure/architecture/patterns/saga) — local transactions, orchestration/choreography, lack of global isolation.
- [Microsoft: compensating transaction](https://learn.microsoft.com/en-us/azure/architecture/patterns/compensating-transaction) — business-specific compensation, concurrent updates and resumable recovery.
- [AWS: saga orchestration](https://docs.aws.amazon.com/prescriptive-guidance/latest/cloud-design-patterns/saga-orchestration.html) — coordinator, eventual consistency, idempotent participants and operational costs.
- [AWS: making retries safe with idempotent APIs](https://aws.amazon.com/builders-library/making-retries-safe-with-idempotent-APIs/) — caller-supplied identities and retries after ambiguous outcomes.

The public capability motivating this example is abstracted into generic business roles. No source-company architecture, proprietary code, deployment behavior or executed application tests are claimed. The main figures were checked as documentation assets.

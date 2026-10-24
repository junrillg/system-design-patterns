# Patterns found in the two applications

**Nine more topics are supported by the code.** Start with request idempotency after the [outbox lesson](../transactional-outbox/README.md). It answers the next question: what happens when a request arrives again?

Reviewed on **24 October 2026**: [Candidate Screening with Kafka](https://github.com/junrillg/candidate-screening-kafka/tree/458372974d8c6463a396ef165cd2fe7b93113e65) and [Résumé Match with RabbitMQ/Celery](https://github.com/junrillg/resume-match-bedrock/tree/66c0985efe2f1fb4ca04871b877d3de5e850cd47). Both revisions matched public `main` at review time. This is an implementation catalogue; these nine topics do not yet have finished illustrated lessons. Source and test inspection does not prove deployed behavior.

## Learn the delivery protections first

These mechanisms cooperate, but protect different steps. Read the short question first; open its evidence only when you want the code.

| Topic | Question it answers | Where found |
|---|---|---|
| **Transactional outbox — [finished lesson](../transactional-outbox/README.md)** | How can saving work also preserve the intent to send it? | Both apps; the lesson follows Résumé Match. |
| **1. [Request idempotency](reference.md#request-idempotency)** | What if the caller retries the same submission? Same owner/key and matching content reuse one logical request. | Both APIs. |
| **2. [Worker leases and guarded completion](reference.md#worker-leases)** | What if workers overlap or an old worker returns late? Claim work temporarily and check ownership when saving. | Both job processors, with different expiry checks. |
| **3. [Transactional inbox](reference.md#transactional-inbox)** | What if Kafka delivers an event again? Save its identity and the local database change together. | Kafka consumers. |
| **4. [Claim check](reference.md#claim-check)** | How can a message stay small? Send a reference, then retrieve the payload when needed. | Both: Kafka uses internal HTTP; Résumé Match retrieves the stored original. |

**Keep this distinction:** outbox protects the intent to send; request idempotency protects repeated API acceptance; inbox protects repeated event effects; worker ownership checks protect result commits. None alone promises that an AI call runs exactly once. See the [definition references](reference.md#primary-sources).

## Then learn recovery and keeping the UI current

| Topic | Question it answers | Where found |
|---|---|---|
| **5. [Bounded retries and recovery](reference.md#bounded-retries)** | What if a failure is temporary? Schedule another attempt, but stop at an attempt/deadline limit. | Both; their retry policies differ. |
| **6. [Admission control](reference.md#admission-control)** | What if work arrives faster than it can finish? Reject excess new work before accepting more. | Both APIs. |
| **7. [Deletion tombstones and reconciliation](reference.md#deletion-tombstones)** | What if delayed work arrives after deletion? Remember deletion and retry unfinished cleanup. | Both, across different storage boundaries. |
| **8. [Durable notification inbox](reference.md#durable-notification-inbox)** | What if the browser is offline when work finishes? Save the notification with the result. | Both. This is a user's inbox, distinct from a message-deduplication inbox. |
| **9. [Replay and reconciliation](reference.md#replay-and-reconciliation)** | What if a live update is missed? Recover from durable records or refetch current state. | Résumé Match replays retained revisions; Kafka app reconnects and refetches. |

## Choose one next session

Recommended: **request idempotency → worker leases → transactional inbox**. Study one topic at a time, using its actual named process, one diagram with role labels/colors/logos, and one matching everyday illustration. The remaining entries are a menu, not required reading for that session.

We did not label these apps as event sourcing, a general saga, a circuit breaker, or end-to-end exactly-once processing: the reviewed mechanisms do not establish those guarantees. Separate GET/POST routes alone are not evidence of CQRS.

[Optional code evidence, important differences and primary sources](reference.md)

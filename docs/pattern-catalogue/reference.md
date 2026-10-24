# Pattern catalogue: optional code reference

Static source review, **24 October 2026**. The remote heads were confirmed at these exact revisions; links stay pinned even when the apps change:

| Example app | Reviewed revision |
| --- | --- |
| Résumé Match | [`66c0985`](https://github.com/junrillg/resume-match-bedrock/tree/66c0985efe2f1fb4ca04871b877d3de5e850cd47) |
| Candidate Screening | [`4583729`](https://github.com/junrillg/candidate-screening-kafka/tree/458372974d8c6463a396ef165cd2fe7b93113e65) |

These mechanisms are implemented, rather than inferred from plans. **Tests were read, not run.** This review establishes neither deployed behavior nor a complete security/performance assessment. Source links may require repository access.

## Request idempotency

[AWS defines safe API retries](https://aws.amazon.com/builders-library/making-retries-safe-with-idempotent-APIs/) around recognizing the same caller intent. Both apps compare owner-scoped request keys and payload fingerprints: [Résumé Match acceptance](https://github.com/junrillg/resume-match-bedrock/blob/66c0985efe2f1fb4ca04871b877d3de5e850cd47/backend/matching/views.py#L296-L352), [Candidate Screening acceptance](https://github.com/junrillg/candidate-screening-kafka/blob/458372974d8c6463a396ef165cd2fe7b93113e65/services/Candidates.Api/CandidatesFlow.cs#L32-L86).

Résumé Match returns current job state; Candidate Screening replays original acceptance. Protection lasts while identity is retained; a new key can create new work. [Concurrent-request test source](https://github.com/junrillg/resume-match-bedrock/blob/66c0985efe2f1fb4ca04871b877d3de5e850cd47/backend/matching/test_jobs.py#L298-L317).

## Worker leases

A temporary claim permits recovery; a destination-side check must reject obsolete writes ([fencing explanation](https://martin.kleppmann.com/2016/02/08/how-to-do-distributed-locking.html)). Compare [Résumé Match claim/completion](https://github.com/junrillg/resume-match-bedrock/blob/66c0985efe2f1fb4ca04871b877d3de5e850cd47/backend/matching/jobs.py#L51-L101) and [Candidate Screening claim/completion](https://github.com/junrillg/candidate-screening-kafka/blob/458372974d8c6463a396ef165cd2fe7b93113e65/services/Screening.Worker/ScreeningFlow.cs#L105-L153).

Résumé Match checks lease identity **and expiry** at completion; Candidate Screening checks identity/live state, so replacement fences the old claimant, not expiry alone. Neither prevents every repeated AI call. [Stale/expired-lease test source](https://github.com/junrillg/resume-match-bedrock/blob/66c0985efe2f1fb4ca04871b877d3de5e850cd47/backend/matching/test_jobs.py#L185-L203).

## Transactional inbox

An [idempotent consumer](https://microservices.io/patterns/communication-style/idempotent-consumer.html) can record message identity with its database effect. Candidate Screening commits [inbox identity and queued job](https://github.com/junrillg/candidate-screening-kafka/blob/458372974d8c6463a396ef165cd2fe7b93113e65/services/Screening.Worker/ScreeningFlow.cs#L29-L53), then [commits the Kafka offset after the handler](https://github.com/junrillg/candidate-screening-kafka/blob/458372974d8c6463a396ef165cd2fe7b93113e65/services/Common/FlowTransport.cs#L137-L145).

The later AI loop is separate; this transaction does not make inference execute once. Résumé Match's worker lease is a different consumer-protection mechanism. [Duplicate-event test source](https://github.com/junrillg/candidate-screening-kafka/blob/458372974d8c6463a396ef165cd2fe7b93113e65/tests/integration/recovery.py#L491-L497).

## Claim check

[Claim check](https://learn.microsoft.com/en-us/azure/architecture/patterns/claim-check) separates payload storage from its message reference. Résumé Match sends a [job ID](https://github.com/junrillg/resume-match-bedrock/blob/66c0985efe2f1fb4ca04871b877d3de5e850cd47/backend/matching/jobs.py#L175-L185) and [retrieves the original document](https://github.com/junrillg/resume-match-bedrock/blob/66c0985efe2f1fb4ca04871b877d3de5e850cd47/backend/matching/documents.py#L147-L183); Candidate Screening [fetches source over internal HTTP](https://github.com/junrillg/candidate-screening-kafka/blob/458372974d8c6463a396ef165cd2fe7b93113e65/services/Screening.Worker/ScreeningFlow.cs#L70-L84).

These are reference-based payload handoffs, not worker ownership claims. Processing depends on payload availability, authorization and retention; the event is not a complete historical snapshot. [Original-reuse test source](https://github.com/junrillg/resume-match-bedrock/blob/66c0985efe2f1fb4ca04871b877d3de5e850cd47/backend/matching/test_documents.py#L42-L66).

## Bounded retries

[Retry budgets and backoff](https://aws.amazon.com/builders-library/timeouts-retries-and-backoff-with-jitter/) limit repeated load. Compare [Résumé Match delayed requeue](https://github.com/junrillg/resume-match-bedrock/blob/66c0985efe2f1fb4ca04871b877d3de5e850cd47/backend/matching/jobs.py#L41-L68) and [Candidate Screening outbox retries](https://github.com/junrillg/candidate-screening-kafka/blob/458372974d8c6463a396ef165cd2fe7b93113e65/services/Common/FlowTransport.cs#L43-L84).

Résumé Match job retries use exponential delay without jitter; Candidate Screening's outbox adds jitter, while its worker uses a separate fixed-delay policy. Neither implies a circuit breaker or universal retry algorithm. [Job retry-budget test source](https://github.com/junrillg/resume-match-bedrock/blob/66c0985efe2f1fb4ca04871b877d3de5e850cd47/backend/matching/test_jobs.py#L159-L182).

## Admission control

[Load shedding](https://aws.amazon.com/builders-library/using-load-shedding-to-avoid-overload/) rejects work to protect finite capacity. [Résumé Match reservations](https://github.com/junrillg/resume-match-bedrock/blob/66c0985efe2f1fb4ca04871b877d3de5e850cd47/backend/matching/admission.py#L25-L68) check sampled supply, queue age and limits; [Candidate Screening admission](https://github.com/junrillg/candidate-screening-kafka/blob/458372974d8c6463a396ef165cd2fe7b93113e65/services/Candidates.Api/Admission.cs#L8-L42) uses a shared lock, pending cap and fixed-minute window.

Sampled health can change after acceptance; fixed windows permit boundary bursts. These gates constrain new acceptance, not autoscaling. [Rejection/quota test source](https://github.com/junrillg/resume-match-bedrock/blob/66c0985efe2f1fb4ca04871b877d3de5e850cd47/backend/matching/test_admission.py#L41-L67).

## Deletion tombstones

[Résumé Match lifecycle reconciliation](https://github.com/junrillg/resume-match-bedrock/blob/66c0985efe2f1fb4ca04871b877d3de5e850cd47/backend/matching/documents.py#L211-L285) records deletion intent before retrying object cleanup. Candidate Screening [redacts and dispatches deletion](https://github.com/junrillg/candidate-screening-kafka/blob/458372974d8c6463a396ef165cd2fe7b93113e65/services/Candidates.Api/CandidatesFlow.cs#L128-L160); its [worker retains deleted identity](https://github.com/junrillg/candidate-screening-kafka/blob/458372974d8c6463a396ef165cd2fe7b93113e65/services/Screening.Worker/ScreeningFlow.cs#L29-L53).

Access denial/redaction precedes eventual physical or cross-service cleanup. Tombstones have finite retention; this is not proof of synchronous erasure everywhere or a general saga. [Deletion-retry test source](https://github.com/junrillg/resume-match-bedrock/blob/66c0985efe2f1fb4ca04871b877d3de5e850cd47/backend/matching/test_documents.py#L105-L123).

## Durable notification inbox

[Résumé Match persists a per-job notice](https://github.com/junrillg/resume-match-bedrock/blob/66c0985efe2f1fb4ca04871b877d3de5e850cd47/backend/matching/notifications.py#L14-L29) within its outcome transaction. [Candidate Screening completion](https://github.com/junrillg/candidate-screening-kafka/blob/458372974d8c6463a396ef165cd2fe7b93113e65/services/Candidates.Api/CandidatesCompletion.cs#L60-L83) commits status, notification and change intent together.

This is a durable **user inbox**, distinct from a consumer's processed-message inbox. A saved notice proves neither human reading nor email/push delivery. [Outcome/notice rollback test source](https://github.com/junrillg/resume-match-bedrock/blob/66c0985efe2f1fb4ca04871b877d3de5e850cd47/backend/matching/test_notifications.py#L34-L68).

## Replay and reconciliation

[Résumé Match revision replay](https://github.com/junrillg/resume-match-bedrock/blob/66c0985efe2f1fb4ca04871b877d3de5e850cd47/backend/matching/events.py#L17-L61) rereads retained database change hints and resets on gaps. Candidate Screening [registers SSE then requests reconciliation](https://github.com/junrillg/candidate-screening-kafka/blob/458372974d8c6463a396ef165cd2fe7b93113e65/services/Candidates.Api/CandidatesFlow.cs#L219-L255); its [browser refetches authoritative state](https://github.com/junrillg/candidate-screening-kafka/blob/458372974d8c6463a396ef165cd2fe7b93113e65/frontend/src/useWorkbench.ts#L129-L178).

These recover missed live signals differently: bounded event replay versus state reconciliation. Neither is event sourcing or sufficient evidence of CQRS. [Replay-gap test source](https://github.com/junrillg/resume-match-bedrock/blob/66c0985efe2f1fb4ca04871b877d3de5e850cd47/backend/matching/test_events.py#L52-L75).

## Outbox is already covered

The existing lesson follows Résumé Match. Candidate Screening also [saves candidate and dispatch intent together](https://github.com/junrillg/candidate-screening-kafka/blob/458372974d8c6463a396ef165cd2fe7b93113e65/services/Candidates.Api/CandidatesFlow.cs#L71-L80), then its [relay publishes and marks sent](https://github.com/junrillg/candidate-screening-kafka/blob/458372974d8c6463a396ef165cd2fe7b93113e65/services/Common/FlowTransport.cs#L27-L88). This is a second implementation example, not another pattern in the count.

## Primary sources

Primary definitions are linked beside their respective topics above.

[Kafka's delivery-semantics definition](https://kafka.apache.org/41/design/design/) also limits exactly-once claims: coordinating Kafka records and offsets does not automatically make unrelated database/provider effects occur once.

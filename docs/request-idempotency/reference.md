# Request idempotency: optional reference

Read the [lesson](README.md) first. This page records the implementation boundaries behind its example.

## Pinned application evidence

Résumé Match source reviewed on **24 October 2026**, revision [`66c0985`](https://github.com/junrillg/resume-match-bedrock/tree/66c0985efe2f1fb4ca04871b877d3de5e850cd47):

- [Our API entry points](https://github.com/junrillg/resume-match-bedrock/blob/66c0985efe2f1fb4ca04871b877d3de5e850cd47/backend/matching/views.py#L273-L307) call the acceptance logic below.
- [Acceptance](https://github.com/junrillg/resume-match-bedrock/blob/66c0985efe2f1fb4ca04871b877d3de5e850cd47/backend/matching/views.py#L296-L352): a UUID `Idempotency-Key`, input fingerprint, owner lock and existing-job lookup. The hash is stored as `payload_hash`; `fingerprint` is a local value, not a separate process or function. A matching request reuses the job; conflicting input is rejected with 409. Only new logical requests reserve capacity and consume daily quota.
- [Response helper](https://github.com/junrillg/resume-match-bedrock/blob/66c0985efe2f1fb4ca04871b877d3de5e850cd47/backend/matching/views.py#L202-L206): `accepted_job` serializes the current job and returns 202 with its location. This is not a cached original response.
- [Job model and owner/key uniqueness](https://github.com/junrillg/resume-match-bedrock/blob/66c0985efe2f1fb4ca04871b877d3de5e850cd47/backend/matching/models.py#L68-L95): job records retain the request identity. `Job` is stored data, not a worker process.
- [Replay/conflict test source](https://github.com/junrillg/resume-match-bedrock/blob/66c0985efe2f1fb4ca04871b877d3de5e850cd47/backend/matching/test_jobs.py#L52-L74) and [concurrent same-key test source](https://github.com/junrillg/resume-match-bedrock/blob/66c0985efe2f1fb4ca04871b877d3de5e850cd47/backend/matching/test_jobs.py#L298-L317) exercise these cases. Tests were read, not run for this lesson.

## Where the example stops

The main picture follows acceptance using an **already-saved document**. [A fresh upload](https://github.com/junrillg/resume-match-bedrock/blob/66c0985efe2f1fb4ca04871b877d3de5e850cd47/backend/matching/views.py#L339-L393) instead stages an uploading job, writes the original outside the database transaction, then finalizes queued state and dispatch intent together. The whole upload is not one database transaction.

Reusing a retained key does not waive input validation or document-access checks. The daily-quota statement concerns accepted job creation, not a guarantee that every retry must succeed indefinitely.

## Pattern versus exact response replay

[AWS: making retries safe with idempotent APIs](https://aws.amazon.com/builders-library/making-retries-safe-with-idempotent-APIs/) explains caller-supplied request identifiers, conflicting intent and retention boundaries. This lesson reuses the verified primary reference from the [catalogue audit](../pattern-catalogue/reference.md#request-idempotency).

Résumé Match returns the job’s **current state**. The [Candidate Screening implementation](https://github.com/junrillg/candidate-screening-kafka/blob/458372974d8c6463a396ef165cd2fe7b93113e65/services/Candidates.Api/CandidatesFlow.cs#L32-L86) instead replays original acceptance. Both recognize one logical request; they need not return identical bytes or offer identical retention policies.

This pattern alone does not make broker delivery, worker execution or external provider effects exactly once. No running application, broker or cloud service was exercised to publish these notes.

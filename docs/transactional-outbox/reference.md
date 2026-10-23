# Outbox: optional details

Read the [short lesson](README.md) first. These notes explain the boundaries; they are not extra steps to memorize.

## What can go wrong?

If RabbitMQ accepts a message and the relay crashes before committing `sent_at`, the reminder still looks pending. Retrying can send it twice. Marking it sent before publishing would instead risk losing the message. This is why consumers need **idempotency**: repeated delivery must not cause repeated harmful effects.

A publisher confirmation means broker acceptance, not worker completion. A consumer acknowledgement concerns handling a delivery; job status records the business outcome. The outbox provides neither a shared database-and-RabbitMQ transaction nor exactly-once effects.

## Résumé Match boundaries

Source inspected on 24 October 2026 at [`66c0985`](https://github.com/junrillg/resume-match-bedrock/tree/66c0985efe2f1fb4ca04871b877d3de5e850cd47):

- [Fresh upload](https://github.com/junrillg/resume-match-bedrock/blob/66c0985efe2f1fb4ca04871b877d3de5e850cd47/backend/matching/views.py#L339-L393): the file upload happens outside finalization. Finalization updates the reserved job to `QUEUED` and adds its outbox row in one transaction.
- [Relay](https://github.com/junrillg/resume-match-bedrock/blob/66c0985efe2f1fb4ca04871b877d3de5e850cd47/backend/matching/jobs.py#L175-L216): selects a due, unsent row; publishes; saves `sent_at`. This relay holds a separate database transaction while publishing. `sent_at` excludes that same row from ordinary sends; retry/recovery may create a new row for the same job.
- [Worker protection](https://github.com/junrillg/resume-match-bedrock/blob/66c0985efe2f1fb4ca04871b877d3de5e850cd47/backend/matching/jobs.py#L51-L101): job state and a time-limited claim protect database results from competing workers. They do not guarantee one external provider call.

Delivery still needs a running relay, recovery and a working broker. Pending reminders need monitoring and cleanup; processing order is not guaranteed.

## Sources and history

Primary references checked on 24 October 2026: [pattern author’s explanation](https://microservices.io/patterns/data/transactional-outbox.html) and [RabbitMQ confirmations and acknowledgements](https://www.rabbitmq.com/docs/confirms).

[Historical full lesson](https://github.com/junrillg/system-design-patterns/blob/f67c2653dde2c19d6bb6e0359d452c66269439c3/docs/transactional-outbox/README.md) preserves the superseded diagrams and source map. Start with the current short lesson.

This documentation does not establish learner mastery or prove runtime behavior. No application or broker tests were run for this simplification.

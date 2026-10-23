# System Design Patterns

A personal learning notebook for understanding how reliable applications work: the problem, a small diagram, a real code example, and what happens when something fails.

**Start here:** [Transactional outbox](docs/transactional-outbox/README.md) — save a business change and the intent to send work together, then deliver later.

## System design or software architecture?

Both terms fit. **System design** considers how databases, queues, workers and services cooperate. **Software architecture** describes the structures, boundaries and decisions inside and between those parts. Transactional outbox is an architectural pattern for reliable messaging, commonly studied in system design.

This notebook uses “system design patterns” as the broad home for those lessons. A pattern is a response to a particular problem, with costs; it is not a requirement to add more infrastructure.

## Topic index

| Topic | Learning material | Worked example |
| --- | --- | --- |
| Transactional outbox | [Explanation, diagrams, failures and tradeoffs](docs/transactional-outbox/README.md) | Résumé Match: Django → PostgreSQL → RabbitMQ → Celery |

Possible later topics: idempotency, leases and fencing, retries with backoff, dead-letter handling, and CQRS. These are ideas for future lessons, not completed material.

## Add a lesson

Use the [short topic guide](docs/topic-guide.md). Keep Markdown and editable Mermaid together with an inspected static image fallback. Pin real implementation links to a commit and distinguish source inspection from runtime evidence.

This is a public, docs-only learning repository. The first lesson references selected application source; it contains no résumé files, credentials or copied application repository.

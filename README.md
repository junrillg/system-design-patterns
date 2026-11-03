# System Design Patterns

Engineering notes on architecture patterns, with clear process diagrams, concise explanations and matching illustrations.

## Illustrated patterns

| Pattern | Problem it addresses |
| --- | --- |
| [Transactional outbox](docs/transactional-outbox/README.md) | Save work and the intent to send it together. |
| [Request idempotency](docs/request-idempotency/README.md) | Recognize a repeated submission without creating another job. |
| [Saga](docs/saga/README.md) | Coordinate employee onboarding across HR, IT and payroll, with recovery when setup cannot finish. |
| [CQRS](docs/cqrs/README.md) | Keep document approval rules separate from the onboarding dashboard's read model. |

The outbox and idempotency entries follow inspected application code. The saga and CQRS entries are design scenarios with explicit assumptions.

## Reading order

Each entry connects one concrete problem to a named process, a matching illustration, and the main benefit and cost. Optional references hold deeper boundaries and source evidence.

Diagram labels and colors distinguish application code, stored data, message brokers and framework roles where they appear.

“Architecture” describes how the parts fit together. “System design” considers how those parts cooperate. This notebook covers both.

## More patterns from the applications

The [reviewed pattern catalogue](docs/pattern-catalogue/README.md) identifies nine more topics in Candidate Screening and Résumé Match, with pinned code evidence and limits. The independent saga and CQRS scenarios do not change that implementation audit.

New entries follow the [short topic guide](docs/topic-guide.md).

Public engineering notes; examples use selected source links, without personal résumé files or credentials.


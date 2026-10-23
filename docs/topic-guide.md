# Writing the next pattern lesson

Create `docs/<pattern-name>/README.md`, with figures in `assets/`. Add the finished lesson to the root index.

Make the explanation easy to follow without removing the useful process:

1. Start with one concrete problem and a short, accurate memory phrase. The phrase supports the explanation; it does not replace it.
2. Keep one example. Define unfamiliar terms beside their first use, and preserve important component and function names when explaining actual code. Explicitly distinguish running processes/services, our functions, framework/library APIs, and stored data. A small role table can prevent these being confused.
3. Group the process into a few numbered stages. Match their names and numbers in the text and diagram. Include the boundary that makes the pattern work, the important handoffs, and what each actor actually does.
4. Use consistent colors plus written role labels: blue `#EAF2FF` for our application code, green `#E7F5EC` for data stores/rows, orange `#FFF0D9` for the message broker, and purple `#F1EAFE` for framework/library/runtime roles. In this outbox example, the broker is RabbitMQ and the task framework is Celery. Keep dark text/borders and a visible legend; never make color the only explanation. Group code inside its actual process without inventing deployment counts.
5. Use a process diagram and a familiar illustration when requested or useful. Give each a clear purpose, caption and reading order. Map the illustration to the same components and explain its limits. Avoid adding a third view unless it resolves a particular question.
6. Give the main benefit and cost, including essential failure risks. Keep sentences short; use enough words to explain the whole mechanism. Word and node counts are guides, never deletion rules.

Put detailed code mapping, edge cases and verified primary sources in an optional `reference.md`. Keep editable diagram source collapsed and provide inspected static images. Preserve superseded material through an immutable history link, clearly marked historical.

Before publishing, read it as a beginner: is the starting point obvious, does each step explain what happens next, and is the memory phrase still true? Check figure readability and links. The reading order must be **problem → named process → familiar illustration → benefit and cost → optional reference**. Remove competing current versions and redundant diagrams, while keeping requested teaching material.

Do not present planned topics as finished lessons. Publishing notes does not prove production behavior or learner understanding.

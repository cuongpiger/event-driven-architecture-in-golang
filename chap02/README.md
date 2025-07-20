Chapter 2, "Supporting Patterns in Brief," introduces several helpful software patterns that support excellent event-driven application design and development. These patterns, when used appropriately, can improve production time and reduce bug rates. The main knowledge areas covered include:

*   **Domain-Driven Design (DDD)**:
    *   DDD is a comprehensive approach to modeling complex business ideas into software by deeply understanding the problem domain.
    *   It emphasizes collaboration between **domain experts and developers** to sketch and discuss business concepts, which are then modeled and refined to eliminate misunderstandings. This is an ongoing process, not a one-time activity.
    *   A core principle is the **Ubiquitous Language (UL)**, which requires every domain-specific term to have a single meaning within a bounded context. This shared language fosters a better understanding of the domain and should be used everywhere, including in code.
    *   DDD helps tackle complexity by breaking the domain into subdomains, categorizing them as **Core, Supporting, or Generic domains** based on their criticality and competitive advantage to the business.
    *   The domain model focuses solely on data and behaviors relevant to solving problems within the domain, free from technical complexities.
    *   **Bounded Contexts** define the boundaries around parts of the application, ensuring that each model is safe from outside influences and external control. Each bounded context has its own UL, meaning terms might have different meanings across contexts.
    *   **Context mapping** is used to define relationships between models and contexts that need to interact, categorized into Upstream, Midway, and Downstream patterns.
    *   DDD is particularly useful for EDA by facilitating the creation of **better event names** and identifying **integration events** that form part of a bounded context's contract.

*   **Domain-Centric Architectures**:
    *   These architectures place the **domain at the center**, surrounded by layers for application logic and then infrastructure or external concerns, aiming to keep the domain free from external influences like database specifics or framework concerns.
    *   They address issues of tight coupling seen in traditional layered architectures, where changes in one layer often necessitate changes across others.
    *   Concepts like **Hexagonal Architecture** (ports and adapters pattern), **Onion Architecture**, and **Clean Architecture** evolved to solve these coupling problems by enforcing a "dependency rule" where source code dependencies only point inwards towards the domain.
    *   **Ports and adapters** are key, where ports are abstractions (interfaces) known to the application, and adapters are concrete implementations that interact with external dependencies (e.g., UIs, databases).
    *   Benefits include **high testability** and **lower long-term maintenance costs**, especially for large applications using DDD, as well as **portability and reuse** due to independence from framework or infrastructure choices.
    *   Drawbacks include a **larger upfront investment** and potential complexity for less experienced developers, or over-engineering if rigidly or incorrectly applied.
    *   Domain-centric architectures generally benefit EDA by keeping services small and independent of infrastructure choices.

*   **Command and Query Responsibility Segregation (CQRS)**:
    *   CQRS is a pattern that **splits objects into two new objects**: one responsible for commands (mutating application state) and the other for queries (returning application state). An action can be either a command or a query, but not both.
    *   It addresses the problem of complex domain models being unsuitable or too much for queries, or complex queries forcing modifications to domain models.
    *   CQRS can be applied at different levels: to the **application code only**, to the **database** (e.g., using different data stores for reads and writes), or to the **entire service**, splitting it into two distinct services that can be scaled and maintained independently.
    *   Consider CQRS when: read operations significantly outnumber write operations; security needs to be applied differently for reads and writes; **event-driven patterns like event sourcing** are used (as it allows generating multiple read models from events); complex read patterns bloat the domain model; or data needs to be available even when writing is not possible.
    *   CQRS and event sourcing work well together because event sourcing's append-only log for writes allows for the creation of numerous read models from the same events.

*   **Application Architectures**:
    *   **Monolithic Architecture**: A single codebase deployed as a single resource, easy to deploy and manage for simpler applications, but can become hard to develop efficiently as it grows.
    *   **Modular Monolith Architecture**: Applies DDD and domain-centric architecture to refactor a monolith into independent modules, offering a balance of benefits from both monoliths and microservices, with fewer drawbacks. Communication between modules should be treated as external concerns.
    *   **Microservices Architecture**: Involves building individual services, ideally aligned with bounded contexts, to create a distributed application. Offers independent deployability, scalability, and fault isolation, but introduces complexity in management and eventual consistency issues.
    *   **Recommendation for greenfield projects**: A **modular monolith** is recommended to start projects of reasonable complexity, allowing teams to focus on domain model implementation. Modules can then be easily extracted into microservices as the application outgrows the modular monolith architecture.
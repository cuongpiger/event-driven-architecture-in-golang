Chapter 5 of "Event-Driven Architecture in Golang," titled "Tracking Changes with Event Sourcing," builds upon the foundation of domain events introduced in the previous chapter. Its primary goal is to teach how to record these events in a database to maintain a complete historical record of modifications made to an aggregate, and how to query this data effectively using Command and Query Responsibility Segregation (CQRS).

The main ideas and key knowledge from Chapter 5 include:

- **What is Event Sourcing?**

  - Event sourcing is a pattern where **each change to an aggregate is recorded as an event into an append-only stream**.
  - Unlike traditional Create, Read, Update, and Delete (CRUD) systems that overwrite prior versions of data, event sourcing retains **the entire history of changes**. For example, when a product price changes, a CRUD system updates the price directly, losing the old price and the intent of the change. In event sourcing, a new event records the change, preserving the old price and valuable metadata.
  - Implementations should use **event stores that provide strong consistency guarantees and optimistic concurrency control** to ensure that concurrent modifications are handled correctly.
  - While it can be used independently, event sourcing **works very well with Domain-Driven Design (DDD)** and domain events.
  - **Distinction from Event Streaming**: It's crucial to understand that event sourcing and event streaming are different.
    - **Event Sourcing** is an _implementation detail within a single bounded context_, used to keep a history of changes. It can be **strongly consistent** if used with an ACID-compliant database.
    - **Event Streaming** is for _communicating state changes across context boundaries_, is typically **asynchronous**, and uses message brokers.
  - Event sourcing requires a non-traditional approach to data, as data cannot be searched with complex queries directly from the event stream, necessitating other access methods. It also adds complexity to a system.

- **Adding Event Sourcing to the Monolith**

  - **Richer Events**: The initial simple `ddd.Event` interface from Chapter 4 is expanded to include needs like aggregate details, and support for **serialization and deserialization**.
    - The actual data now resides in an `EventPayload` (which has no defined methods), allowing flexibility.
    - A `newEvent` constructor is introduced, using `EventOption` (like `Metadata`) to embed details such as `AggregateNameKey`, `AggregateIDKey`, and `AggregateVersionKey`.
  - **Refactoring Aggregates**: Aggregates (like `Order` or `Store`) are updated to use the new event constructor and to **ensure that all state changes originate from applying events**. Direct assignments to aggregate fields are removed, and changes are made via `ApplyEvent()` methods that react to specific event payloads.
  - **Event Versioning**: Once an event is persisted, its content cannot change. If the structure of event data needs to evolve, a **new version of the event (and payload) must be created**, and the `ApplyEvent()` method must continue to handle older versions.
  - **Aggregate Repositories and Event Stores**:
    - A generic `AggregateRepository` is introduced with `Load()` and `Save()` methods. `Load()` reconstructs an aggregate by reading and applying its historical events. `Save()` applies new events to the aggregate, persists them to the event store, and then clears the events from the aggregate.
    - The **event store (e.g., PostgreSQL `events` table)** uses a **compound primary key (`stream_id`, `stream_name`, `stream_version`) to enforce optimistic concurrency control**.
    - A **data types registry** is crucial for handling the serialization and deserialization of various aggregate and event types, ensuring type safety when converting data to and from byte slices.
    - The `AggregateStore` interface defines the lower-level interaction with the specific persistence infrastructure.

- **Using Just Enough CQRS**

  - **Necessity**: Event sourcing, while powerful for historical changes, is **not efficient or suitable for complex queries** that require scanning or filtering large numbers of events (e.g., retrieving lists of stores or products). This limitation is why CQRS is almost always used in conjunction with event sourcing.
  - **Read Models**: To support complex queries, **separate read models** are created. For MallBots, `MallRepository` (for store-related queries) and `CatalogRepository` (for product-related queries) are introduced. These read models project domain events into dedicated tables optimized for querying, sometimes reusing discarded tables.
  - **Event Handler Refactoring**: The `ddd.EventHandler` interface is refined to be more flexible, allowing a single handler to process different event types, reducing boilerplate code when handling many events for read model projections.
  - **Connecting Domain Events with the Read Model (Middleware)**: A key challenge identified is that domain events are cleared from an aggregate _after_ it's saved to the event store, meaning external handlers (like those updating read models) could miss them.
    - The solution is to use the **Chain of Responsibility pattern (middleware)** with `AggregateStore`.
    - An `EventPublisher` middleware is created, which ensures that domain events are published _after_ they have been successfully persisted to the event store, but _before_ they are cleared from the aggregate. This decouples the core persistence logic from the event publishing side effect, ensuring reliability.

- **Aggregate Event Stream Lifetimes (Snapshots)**
  - Aggregates are categorized as **short-lived** (few events, e.g., `Order`) or **long-lived** (many events, e.g., `Store`).
  - For **long-lived aggregates**, **snapshots are used to improve performance** by reducing the number of historical events that need to be read and applied to reconstruct the aggregate's state. A snapshot is a serialized representation of the aggregate's state at a specific version.
  - Aggregates using snapshots must implement `ApplySnapshot()` (to load state from a snapshot) and `ToSnapshot()` (to create a snapshot) methods.
  - Snapshots are stored in a separate `snapshots` table and integrated into the `AggregateStore` via middleware, similar to the event publisher.
  - **Downsides**: Snapshots act as a cache and introduce issues like data duplication and invalidation. They are considered a **performance optimization** and should only be used when necessary to avoid premature optimization.

In essence, Chapter 5 is like a master chef meticulously documenting every single ingredient added and every action taken to create a dish, instead of just noting the final product. It then shows how to use this detailed history to prepare new, specialized versions of the dish for different purposes (read models) and how to occasionally take a "photo" (snapshot) of the dish at a certain stage to speed up future preparations, all while maintaining a consistent and traceable history.

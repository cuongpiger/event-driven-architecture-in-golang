Chapter 4, "Event Foundations," introduces the foundational concepts of event usage and communication patterns in the MallBots application, beginning its transformation from a purely synchronous system into an event-driven one. The main idea of this chapter is to lay the groundwork for event-driven architecture by detailing the application's structure and module communication, and critically, to demonstrate how **domain events** can be used to refactor and manage side effects within a bounded context.

Key knowledge and concepts from Chapter 4 include:

- **Technical Requirements**: To run the application and examples, users need Go version 1.17+ and Docker.
- **MallBots Application Structure**:
  - The application is designed as a **modular monolith**.
  - Its root directory is organized using **"screaming architecture,"** where top-level directories are named after application components (e.g., `baskets`, `stores`, `depot`) rather than generic layers (e.g., `controllers`), making the application's purpose immediately clear.
  - It also includes standard Go project directories like `/cmd` (for executables), `/internal` (for private packages), `/docs`, and `/docker`.
- **Module Code Organization**:
  - Each module exposes a **protocol buffer API** and has a **composition root**.
  - Dependencies between modules are controlled by using **internal packages**, which prevent unintentional imports and help manage relationships.
  - The design adheres to the Go idiom **"Accept interfaces, return structs,"** where consumers define small, specific interfaces for their needs, promoting loose coupling between components and their concrete implementations.
  - The **composition root** is the central place where infrastructure, configuration, and application components are assembled and where **dependency injection** takes place. It systematically constructs Driven adapters, then the application, and finally Driver adapters.
- **Current Module Communication and Integration**:
  - Initially, all interactions between MallBots modules are **entirely synchronous**, relying on **gRPC** and **protocol buffers**.
  - Modules integrate to either pull data from another bounded context or command another bounded context to perform an action.
  - **Data Pulling**: Pulling data (e.g., Shopping Baskets pulling product/store data from Store Management) is generally preferred as it avoids maintaining lists of interested components and allows local components to handle failures with retry logic.
  - **Command Pushing**: Pushing commands (e.g., CheckoutBasket handler calling Ordering, which then calls multiple other services like Customers, Payments, Depot, Notifications) is common, but can lead to "extensive call chains" and tight coupling in synchronous systems.
- **Introduction to Event Types**: The chapter introduces different categories of events:
  - **Domain Events**: Conceptual events from Domain-Driven Design (DDD) that signify state changes _within a single bounded context_. They are typically handled **synchronously and in-process**, are immutable, but do not require serialization or versioning as they are transient and not shared externally.
  - **Event Sourcing Events**: Events that _record state changes for an aggregate_ by being serialized and stored in an event stream. These are retained indefinitely and require versioning and metadata. (Detailed implementation in Chapter 5).
  - **Integration Events**: Events that _communicate state changes across context boundaries_. They are serialized for sharing, **strictly asynchronous**, and utilize an event broker to decouple producers and consumers. (Detailed implementation in Chapter 6).
- **Refactoring Side Effects with Domain Events**: This is the core practical application in the chapter, demonstrating how to break down complex handlers that perform multiple implicit actions (e.g., a single `CreateOrder` function that also handles notifications and rewards).
  - By raising **domain events** (e.g., `OrderCreated`), the system explicitly signals that something important has happened.
  - **Aggregates** (e.g., `Order` and its `AggregateBase`) are modified to capture and raise these events.
  - An **`EventDispatcher`** (implementing the **Observer pattern**) is introduced to publish these domain events to various subscribed **domain event handlers**.
  - This refactoring decouples the core action (e.g., saving an order) from its side effects (e.g., sending a notification), making the code cleaner, more maintainable, and reflective of distinct business rules.
- **DDD for Simple Modules**: The chapter notes that not every module needs to rigorously apply DDD or domain events; forcing it on a simple domain can be counterproductive.

In essence, Chapter 4 begins the journey of transforming a tightly coupled application by introducing the fundamental concept of domain events. It's like an office upgrading from everyone directly yelling at each other to using an internal memo system: now, when something important happens, a memo is sent out, and only the relevant internal departments (domain event handlers within the same bounded context) act on it, making the core operations much clearer and more manageable.

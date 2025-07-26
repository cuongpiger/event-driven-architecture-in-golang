Chapter 6 of the source, "Asynchronous Connections," focuses on transforming a synchronous application into an asynchronous one by introducing **messages** and **integration events**.

Here are the main knowledge points from the chapter:

- **Asynchronous Integration with Messages**:

  - A **message** is a container with a payload, which can be an event, an instruction (command), a request for information (query), or a response (reply).
  - **Event Types and Their Scopes**: The chapter differentiates between three types of events based on their scope, lifetime, versioning, and handling:
    - **Domain Events**: Exist for the shortest time, stay within the application/bounded context, do not require versioning, and are typically handled synchronously. They inform other parts of the application about changes.
    - **Event-Sourced Events**: Exist for the longest time, stay within the service boundary, require versioning, and are handled synchronously. They maintain a record of every state change to an aggregate.
    - **Integration Events**: Exist for an unknown amount of time, are used by an unknown number of consumers, require versioning, and are typically handled asynchronously. They communicate significant decisions or changes across context boundaries. Both notification and event-carried state transfer events are types of integration events.
  - **Integration with Notification Events**: These events carry minimal state, often just an ID, and are useful when volume is high or data is large. Consumers are expected to make **callbacks** to the originating component for more information, which means they do not completely decouple components and still have temporal coupling.
  - **Integration with Event-Carried State Transfer**: This is a push model where data changes are sent to interested components, allowing them to create **local cached copies**. This approach significantly **decouples consumers temporally** from producers, enhancing resiliency. It's crucial to balance the amount of state included in events and avoid including information from other domains.
  - **Eventual Consistency**: This is a constant in distributed and event-driven applications, being a trade-off for performance and resiliency gains. It means that while changes may not be immediately available everywhere, the system will eventually converge to the same state if modifications cease. Problems like **read-after-write inconsistency** can arise when immediate reads follow writes in a distributed system.
  - **Message-Delivery Guarantees**:
    - **At-most-once**: Messages may be lost if the producer doesn't wait for acknowledgment or the broker deletes messages before consumer processing.
    - **At-least-once**: Messages are guaranteed to be published and delivered until acknowledged, meaning a consumer might receive the same message more than once. This requires consumers to implement **message deduplication** or other **idempotency measures**.
    - **Exactly-once**: Extremely difficult to achieve, typically involves an additional component to hold a message copy before consumer processing.
  - **Idempotent Message Delivery**: Achieved by adding deduplication to at-least-once delivery, often using a message's identity and database transactions to ensure a message is processed only once.
  - **Ordered Message Delivery**: Maintaining message order can be challenging with multiple consumers. A **single consumer** on a FIFO queue maintains order. **Multiple competing consumers** can lead to out-of-order processing of related messages. **Partitioned queues** can help by delivering messages with the same partition key in order to a single consumer per partition. Processing messages out of order can sometimes be acceptable if it doesn't lead to inconsistent states.

- **Implementing Messaging with NATS JetStream**:

  - **NATS JetStream** provides durable streams and allows consumers to track their place on a stream using cursors. Features include message deduplication, message replay, and additional retention policies.
  - **Components**: A **Stream** stores published messages for subjects, and a **Consumer** acts as a view on the message store.
  - **Packages**: The `am` package provides general asynchronous messaging interfaces like `Message`, `MessageHandler`, `MessagePublisher`, `MessageSubscriber`, and `EventStream`. It handles the serialization and deserialization of events into a generic `RawMessage` format. The `jetstream` package provides NATS JetStream-specific implementations of the `RawMessageStream` interface, abstracting away the nuances of NATS itself.

- **Making the Store Management Module Asynchronous**:
  - The monolith's configuration was modified to include NATS JetStream settings, and a **`JetStreamContext`** was provided to the modules for NATS connection.
  - **Graceful shutdown** of the NATS connection was implemented using the `Drain()` method to ensure messages are not lost.
  - **Public integration events** for the Store Management module were defined using protocol buffer messages in the `storespb` package, including unique IDs and relevant data, ensuring they are public and stand alone. These events were made registerable with the application's registry using constants and a `Registrations` function.
  - An **event stream instance** was created in the module's composition root for publishing and subscribing to event messages.
  - **Integration event handlers** were added to the Store Management module to publish these new integration events from domain event handlers, decoupling the module's internal logic from external communication.
  - The **Shopping Baskets module** was updated to receive these messages, including registering `storespb` events with its registry and creating an event stream instance. New store integration event handlers were added to process the incoming messages.
  - The module now subscribes to the `storespb.StoreAggregateChannel` to receive these messages.
  - The successful asynchronous communication between the Store Management and Shopping Baskets modules was verified through logs, demonstrating that messages are published and received even if their order of appearance in logs differs due to asynchronicity.

Chapter 7, "Event-Carried State Transfer," focuses on **refactoring synchronous module interactions into asynchronous communication** patterns, particularly by utilizing **event-carried state transfer** to create **local cached copies of data** and enable the creation of new read models from multiple sources.

Here are the main and important knowledge points:

- **Refactoring to Asynchronous Communication**:

  - The chapter builds on previous work by replacing direct synchronous calls between modules with **asynchronous message-based communication**, specifically using **event-carried state transfer**. This involves adding new asynchronous APIs to modules.
  - **Store Management State Transfer**:
    - The Store Management module is the **origin of all Store and Product data**.
    - Data that was previously "pulled" (e.g., by Shopping Baskets and Depot modules) or "pushed" (e.g., to Order Processing) can now be replaced with **local cached copies**.
    - **Local Cache (Cache Repositories)**: These are _not_ temporary copies, but rather act as an **anti-corruption layer** to protect the consuming module from external changes in the producer's data structure. Implementing these involves updating existing repositories to process data updates from events and creating new tables for the cache.
    - **Synchronous State Fallbacks**: Existing gRPC calls are retained as fallbacks for when local cached data is missing. This involves retrieving data from the fallback and then adding it to the local cache, carefully handling race conditions during inserts.
  - **Customer State Transfer**:
    - A crucial caution is highlighted: **transferring state with events does not transfer domain responsibilities** (e.g., authorization logic remains with the owning service).
    - The Customers module is updated to publish integration events, and modules like Notifications are updated to receive and cache this customer data, removing the need for synchronous callbacks.
  - **Order Processing State Transfer**:
    - NotificationHandlers in the Order Processing module are replaced with new **IntegrationEventHandlers**, leading to **complete decoupling** from the Notifications module. These handlers publish events that trigger notifications or invoice creation in other modules.
  - **Payments State Transfer**:
    - The Payments module is updated to push invoice status changes (e.g., `InvoicePaid` event) to the Order Processing module, thereby removing the last dependency Payments had on other modules.
  - **Other Refactoring Considerations**: The chapter notes that synchronous gRPC endpoints may be deprecated or removed after asynchronous alternatives are in place, but this requires careful communication with API users.

- **Documenting the Asynchronous API**:

  - Given the decoupling in EDA, it's easy for consumers to "crawl source code" to understand published events, leading to an "unknown asynchronous messaging landscape".
  - The importance of **maintaining documentation for asynchronous APIs** is emphasized.
  - **AsyncAPI**: Introduced as a **structured specification** (similar to OpenAPI) for documenting channels and messages in event-driven architectures. It can generate documentation and boilerplate code.
  - **EventCatalog**: Another tool that uses Markdown files to generate static sites for documenting events and asynchronous APIs, capable of visualizing service relationships and state flow.

- **Adding a New Order Search Module**:

  - A new `Search` module is introduced to provide **advanced order search capabilities**, leveraging the newly available event-carried state.
  - This module consumes events from multiple sources (Customers, Store Management, Order Processing) to **build a rich local read model** that provides detailed search results (e.g., customer, product, and store names associated with orders).
  - The `Search` module is implemented as a standalone component, demonstrating how new functionalities can be developed and integrated in an EDA without interrupting other teams or development schedules.

- **Building Read Models from Multiple Sources**:
  - The primary goal is to **return useful order data without requiring additional queries to other services**.
  - The `Search` module supports various search filters, such as `customer_id`, `product_ids`, `store_ids` (stored as arrays), date ranges, total ranges, and status.
  - The PostgreSQL schema for the read model is designed to facilitate these searches, prioritizing data types that have fewer issues with changes in incoming data.
  - **Read Model Record Creation**: When an `OrderCreated` event is received, data from this event is combined with data from other handlers (e.g., cached customer, store, product data) to create a **rich search model**.
  - This read model is **eventually consistent**, meaning data updates will propagate over time, and under normal conditions, users may not notice immediate inconsistencies.

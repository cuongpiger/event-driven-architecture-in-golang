Chapter 8, "Message Workflows," focuses on enabling **complex work to be performed in a distributed and asynchronous manner** within event-driven architectures. It introduces and compares various methods for handling distributed transactions and culminates in the implementation of an **orchestrated saga** for the application's order creation process.

Here are the main knowledge points from the chapter:

- **Understanding Distributed Transactions**:

  - A distributed transaction aims to provide **Atomicity, Consistency, Isolation, and Durability (ACID) guarantees** across multiple independent components, similar to a local database transaction.
  - Unlike local transactions confined to a single database, distributed transactions can span an **entire system, involving different databases or even third-party services**.
  - They are necessary for complex operations that cannot be contained within a single component, preventing issues like vanished inventory or unbilled reservations if parts of an operation fail.

- **Comparing Distributed Transaction Methods**:

  - **Two-Phase Commit (2PC)**:
    - Offers the **strongest consistency** (all ACID guarantees) via a centralized coordinator and participants.
    - Participants first _prepare_ a local transaction (e.g., `PREPARE TRANSACTION` in PostgreSQL) and, if all are ready, the coordinator sends a _commit_ message; otherwise, an _abort_ message is sent.
    - **Major drawback**: Participants hold open prepared transactions, consuming resources for extended periods, which can **limit scalability and lead to issues if the coordinator fails**.
  - **Saga**:
    - A **sequence of steps** defining actions and **compensating actions** for participating system components.
    - Does **not rely on prepared transactions** and **drops the isolation guarantee** (making them ACD transactions), allowing for long-lived operations and use with NoSQL databases.
    - **Choreographed Saga**: Each participant knows its role, listening for events that signal its turn to act. Coordination logic is **spread out**, requiring no extra services. Compensation is initiated by participants reacting to failure events. Good for a **low number of participants** with easy-to-follow coordination.
    - **Orchestrated Saga**: Uses a **Saga Execution Coordinator (SEC)** to send commands to participants, centralizing orchestration. When a failed reply is received, the SEC initiates compensation by sending compensating commands. This approach is often **better for complex, multi-step transactions**.

- **Implementing Distributed Transactions with Sagas**:
  - The chapter details adding **Command and Reply message types** to the `ddd` (domain-driven design) and `am` (asynchronous messaging) packages. `CommandHandler` returns a `Reply` and an error, with the system generating generic Success/Failure replies if none are provided. Commands include special headers for replies, and replies include headers for outcome determination.
  - A new **`sec` (Saga Execution Coordinator) package** is introduced, containing the `Orchestrator`, `Saga` definition, and `Step` components.
    - The **Orchestrator** handles incoming replies to determine the next step, or if compensation is needed, and publishes commands to participants.
    - The **Saga definition** centralizes the logic and sequence of operations, holding metadata and steps.
    - **Steps** encapsulate the logic for actions and compensating actions, potentially modifying saga data.
  - **Converting the Order Creation Process to a Saga**:
    - The existing synchronous order creation process is refactored into an **orchestrated saga** triggered by the `OrderCreated` event.
    - Participant modules (Customers, Depot, Order Processing, Payments) are updated to **implement new Command handlers** (e.g., `AuthorizeCustomer`, `CreateShoppingList`, `ApproveOrder`, `ConfirmPayment`) that are called by the SEC.
    - A new **`cosec` module** (Create-Order-Saga-Execution-Coordinator) is created to orchestrate this saga. It registers external command and reply types, defines a saga data model (`CreateOrderData`), implements a saga repository, and defines the `createOrderSaga` with its sequence of steps and associated actions.
    - The SEC module handles `OrderCreated` events by **starting a new saga instance** and reacting to replies from saga participants.
    - The original `CreateOrderHandler` in the Order Processing module is simplified to **no longer make direct calls to external systems**, publishing only its `OrderCreated` event, thus becoming **more resilient**.

Chapter 12, "Monitoring and Observability," serves as the **concluding chapter** of the book, focusing on how to **monitor the performance and health of services and the application as a whole**. It introduces the concepts of monitoring and observability, delves into distributed tracing, and provides practical steps for instrumenting the MallBots application with OpenTelemetry and Prometheus, culminating in a demonstration of viewing the collected data.

Here's a summary of the main knowledge points from Chapter 12:

- **What are Monitoring and Observability?**

  - **Monitoring** involves tracking an application's performance, usage, and health, primarily using **logging and metrics**. Its purpose is to **react to data analysis** and **detect failure states**. Examples include Kubernetes health checks, tracking query performance, and sending alerts for high error rates. Monitoring answers "What happened?" and "Why did it happen?" but struggles with **unexpected issues** or "unknown unknowns".
  - **Observability** goes beyond traditional monitoring by providing instrumentation to **answer questions about unknown unknowns** or offering insights without explicit queries. It becomes increasingly crucial as application complexity grows, especially in distributed systems where requests traverse multiple services and communication methods.

- **The Three Pillars of Observability**:

  - Observability is built upon three pillars: **logs, metrics, and traces**.
  - **Logs** explain **why** an application is in a given state.
  - **Metrics** indicate **how long** an application has been in a given state.
  - **Traces** reveal **what is impacted** by being in a given state.
  - Together, these pillars offer a **complete picture of the application's state**.

- **How Tracing Works**:

  - A trace records a request's journey through an application.
  - It begins with an **identifier** (e.g., `abcd`) when a new request enters the backend.
  - **Correlation identifiers** link all requests back to a single originating request and remain unchanged throughout the trace.
  - **Causation identifiers** point back from a follow-up request to the immediate preceding request.
  - These identifiers can be logged alongside other messages to manually track request flow.
  - **Spans** represent individual calls or units of work within a trace and can be visualized to show different processes across components and their duration. Spans can be annotated with **attributes** (information to diagnose requests) and **events** (time-component annotations).

- **Instrumenting the Application with OpenTelemetry and Prometheus**:

  - **OpenTelemetry** is a vendor-agnostic project aiming to unify instrumentation for logging, metrics, and tracing into a single API. At the time of writing, its Go SDK for tracing is stable, so it's used for **distributed tracing**.
    - The application is configured with `OTEL_SERVICE_NAME` and `OTEL_EXPORTER_OTLP_ENDPOINT` environment variables to connect to an OpenTelemetry collector.
    - **gRPC client and server interceptors** are added to automatically create and propagate trace contexts for gRPC requests.
    - **Custom middleware** is created for `MessagePublisher` (injector) and `MessageSubscriber` (extractor) to propagate trace contexts for asynchronous messages.
    - Relevant request data is annotated to spans using `span.SetAttributes()` in gRPC server calls.
    - Domain event handlers are updated to record events on the span (`span.AddEvent()`) at the start and end of processing, including error information.
  - **Prometheus** is used for **metrics reporting**.
    - An endpoint (`/metrics`) is added to the HTTP server for Prometheus to **pull metrics data**.
    - **Custom metrics** are set up, such as `sent_messages_count` (using `promauto.NewCounterVec` for total and message-specific counts) and `received_messages_latency_seconds` (using `promauto.NewHistogramVec` for request duration). These metrics include labels for partitioning (e.g., `message`, `handled`).
    - Middleware is used (`amprom.SentMessagesCounter`, `amprom.ReceivedMessagesCounter`) to automatically record these metrics for message publishing and receiving.
    - Application wrappers (e.g., `instrumentedApp`) are used to instrument existing application logic (e.g., `customers_registered_count`) without directly modifying the core application code.

- **Viewing the Monitoring Data**:
  - The Docker Compose environment is extended with four new services for data visualization:
    - **OpenTelemetry collector**: Gathers trace and span data.
    - **Jaeger**: Used to **visualize traces**. It allows searching for traces based on service, showing spans along a timeline, and detailing attributes and events recorded on each span.
    - **Prometheus**: Collects and displays **metrics data**. It requires configuration to scrape metrics from each microservice.
    - **Grafana**: Renders **dashboards** based on Prometheus metrics, offering visual insights into application activity and message rates.
  - A `busywork` application is provided to simulate user requests, generating data for observation in Jaeger, Prometheus, and Grafana.

In essence, Chapter 12 concludes the book by demonstrating how to make a complex, event-driven, distributed application truly **production-ready** through a robust implementation of **observability** via unified tracing, comprehensive metrics, and effective visualization tools.

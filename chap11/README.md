Chapter 11, "Deploying Applications to The Cloud," focuses on **transforming the MallBots modular monolith application into microservices** and **deploying it to a cloud environment**, specifically Amazon Web Services (AWS).

The main knowledge points covered include:

- **Converting the Modular Monolith to Microservices**:

  - **Refactoring the monolith construct**: The monolithic application's `app` struct, which provided resources to modules, is refactored into a **shared `System` struct** in a new `/internal/system` directory. This new `System` type duplicates much of the original `app`'s functionality, with new exported methods, and now implements a `Service` interface that is a renamed version of the old `Monolith` interface. This allows the monolith to continue running while enabling individual services.
  - **Updating the composition root of each module**: The `Startup()` method in each module is modified to call a new `Root()` function, allowing the same composition root code to be reused for both monolithic and standalone microservice execution.
  - **Making each module run as a standalone service**: Each module gets a new `/cmd/service` directory with a `main` package, which is essentially a stripped-down copy of the monolith's main package, focusing only on running that specific service.
  - **Updates to Dockerfile build processes**: A new multi-stage `Dockerfile.microservices` is introduced to compile individual services using a build argument (`svc`), allowing for smaller container images.
  - **Updates to the Docker Compose file**: The `docker-compose.yml` file is updated to include each microservice with its build configuration and uses **Docker Compose profiles** (`monolith` and `microservices`) to selectively start either the monolith or the individual microservices.
  - **Addressing host port conflicts**: Initially, running multiple microservices locally fails due to port allocation issues. This is resolved by assigning **unique, but memorable, host ports** to each microservice in the Docker Compose file, while keeping the container port consistent.
  - **Adding a reverse proxy**: To restore the consistent Swagger UI experience from the monolith, **Nginx is added as a reverse proxy** to the Docker Compose environment. It routes requests to the correct microservice based on the path, allowing a single entry point for all APIs and the Swagger UI.
  - **Fixing gRPC connections**: A new `Services` type with a custom decoder is introduced to handle the new service addresses. The `RpcConfig` struct is updated with a `Service()` method to easily fetch the correct gRPC service address, and the Shopping Baskets and Notifications modules are updated to use this new configuration for their gRPC connections.

- **Installing Necessary DevOps Tools**:

  - The chapter lists essential tools for cloud deployment: **AWS CLI, Helm, PostgreSQL CLI (`psql`), K9s (Terminal UI for Kubernetes), and Make**.
  - Two options for installation are provided: **installing all tools into a Docker container** (using a `deploytools` image) for a cleaner local system, or **installing them directly onto the local system**.
  - Instructions for **creating and configuring AWS credentials** via the IAM console and `aws configure` are detailed.

- **Using Terraform to Configure an AWS Environment**:

  - Terraform is introduced as an **Infrastructure as Code (IaC) tool** to define and deploy AWS resources.
  - The AWS infrastructure includes **Elastic Container Service (ECS) Docker repositories, an Elastic Kubernetes Service (EKS) cluster, a PostgreSQL Relational Database Service (RDS) database, a Virtual Private Cloud (VPC) with subnets, security groups, and an Application Load Balancer (ALB)**.
  - The **`make deploy`** command is used to execute the Terraform plan and apply it to AWS, a process that takes significant time (15-20 minutes) and **incurs usage costs**.
  - Once deployed, the Kubernetes environment can be viewed using **K9s** or `kubectl`.

- **Deploying the Application to AWS with Terraform**:

  - The application deployment is a separate, second step after infrastructure is ready.
  - **Database setup**: The shared triggers for the database are initialized.
  - **Kubernetes setup**: Services are deployed into a dedicated `mallbots` namespace. **ConfigMaps** are used for common environment variables, and an **Ingress** is set up for the Swagger UI on the ALB, providing a single URL for access.
  - **NATS setup**: NATS is deployed with a persistent volume claim and exposed via a service component.
  - **Microservices setup**: Each microservice is deployed with **randomly generated database passwords** stored in Kubernetes secrets (instead of ConfigMaps). Each microservice also gets a deployment and service component, and most expose an Ingress.
  - The deployment process also uses the **`make deploy`** command and takes 5-10 minutes. Deployments can be monitored using K9s or `kubectl`.

- **Tearing Down the Application and Infrastructure**:
  - It is crucial to **tear down resources using `make destroy`** to stop incurring AWS usage costs.
  - The destruction process should be done in reverse order: **application deployment first, then infrastructure deployment**, with Terraform managing the state to locate and remove resources.

In summary, Chapter 11 equips the reader with the knowledge to **transition an event-driven modular monolith to independently deployable microservices and manage their lifecycle in a cloud environment using Infrastructure as Code (Terraform) and containerization (Docker)**, ensuring the same functional experience from local development to cloud deployment.

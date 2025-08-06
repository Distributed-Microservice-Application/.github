# Distributed Microservice Application

This project demonstrates a reliable microservice architecture that's built to scale. It features asynchronous messaging with Apache Kafka, guaranteed delivery using the Outbox Pattern, and a complete monitoring stack with Prometheus and Grafana for full visibility into the system.

## 🌟 Core Concepts

- **Asynchronous Communication**: Services are decoupled and communicate asynchronously via Kafka, enhancing scalability and resilience.
- **Guaranteed Message Delivery**: The **Outbox Pattern** combined with **Change Data Capture (CDC)** using **Debezium** ensures that messages are never lost, even if messaging systems are temporarily down.
- **Observability**: Gain deep insights into system behavior with real-time metrics, logs, and beautiful dashboards.
- **Scalability**: The architecture is designed to scale horizontally. Service A runs with multiple instances, load-balanced by Nginx.

---

## 🚀 Architecture Flow

The system is designed to reliably calculate the sum of two numbers and maintain a running total, even under load.

1.  **👤 User Interaction (gRPC & HTTP Request)**
    *   A user (or a K6 load test script) sends a request to the **Nginx Load Balancer**.
    *   gRPC requests are routed to `service-a` on port `50051`.
    *   HTTP requests are routed to `service-a` on port `8090`.

2.  **🧠 Service A (Go)**
    *   Receives the request and calculates the sum.
    *   **Outbox Pattern**: Instead of publishing directly to Kafka, it atomically saves the result to an `outbox` table in its own PostgreSQL database. This guarantees the operation is recorded.

3.  **⚡ Debezium & Kafka Connect**
    *   **Debezium**, a Change Data Capture (CDC) platform, monitors the `outbox` table.
    *   When a new record appears, Debezium captures this change and publishes it as a message to an **Apache Kafka** topic (`user-events`).

4.  **🔁 Kafka - The Reliable Messenger**
    *   Acts as the central nervous system, decoupling Service A from Service B. It queues the result messages for asynchronous processing.

5.  **🧮 Service B (Java/Spring Boot)**
    *   Consumes messages from the Kafka topic.
    *   For each message, it updates a running total in its own dedicated PostgreSQL databases (Leader/Replica pattern).

6.  **📊 Observability Stack**
    *   **Prometheus**: Scrapes and stores time-series metrics data.
    *   **Grafana**: Visualizes the collected metrics on pre-configured, real-time dashboards.

---

## 🛠️ Technology Stack

| Category          | Technology                                                                                             |
| ----------------- | ------------------------------------------------------------------------------------------------------ |
| **Services**      | Go, Java 21 (Spring Boot)                                                                              |
| **Communication** | gRPC, REST                                                                                             |
| **Messaging**     | Apache Kafka, Debezium (CDC)                                                                           |
| **Database**      | PostgreSQL (with Leader/Replica pattern for Service B)                                                 |
| **Orchestration** | Docker Compose                                                                                         |
| **Load Balancing**| Nginx                                                                                                  |
| **Observability** | Prometheus, Grafana, Kafka UI                                                           |
| **Load Testing**  | K6                                                                                                     |

---

## 📂 Project Structure

```
.
├── Docker-Compose/      # Docker Compose setup, configs, and Makefile for orchestration.
├── Service-A/           # Go-based microservice for calculations (gRPC server).
├── Service-B/           # Java/Spring-based microservice for consumption and aggregation.
├── grafana/             # Grafana dashboard and datasource configurations.
└── prometheus/          # Prometheus scrape configurations.

```

---

## 🏁 Getting Started

### Prerequisites

- **Docker**: [Install Docker](https://docs.docker.com/get-docker/)
- **Docker Compose**: Included with Docker Desktop. For Linux, [install the plugin](https://docs.docker.com/compose/install/).
- **make**: A build automation tool.

### Installation & Setup

1.  **Clone the repository:**
    ```sh
    git clone <repository-url>
    cd Distributed-Microservice-Application
    ```

2.  **Navigate to the Docker Compose directory:**
    ```sh
    cd Docker-Compose
    ```

3.  **Start all services:**
    This command will start all containers, including Kafka, databases, and services, in detached mode.
    ```sh
    make start
    ```

4.  **Set up the Debezium Connector:**
    After the services are up, deploy the Debezium connector to start the Change Data Capture process.
    ```sh
    make setup-debezium
    ```
    You can verify its status with `make check-debezium`.

---

## ⚙️ Usage

All primary operations are managed via the `Makefile` in the `Docker-Compose` directory.

| Command                 | Description                                                              |
| ----------------------- | ------------------------------------------------------------------------ |
| `make help`             | Shows a list of all available commands.                                  |
| `make start`            | Starts all services in the background.                                   |
| `make stop`             | Stops and removes all containers.                                        |
| `make logs-all`         | Tails the logs for all application services (Service A instances & B).   |
| `make logs-service-a`   | Shows logs for all Service A instances.                                  |
| `make logs-service-b`   | Shows logs for the Service B instance.                                   |
| `make logs-kafka-connect`| Tails the logs for the Debezium/Kafka Connect container.                 |
| `make setup-debezium`   | Deploys the Debezium connector for the outbox table.                     |
| `make check-debezium`   | Checks the status of the Debezium connector.                             |
| `make test-workflow`    | Runs a complete end-to-end test to validate the Debezium data pipeline.  |
| `make fix-debezium`     | Runs a script to troubleshoot and fix common Debezium issues.            |
| `make clean-all`        | Cleans up all unused Docker resources (containers, networks, volumes).   |

---

## 🔬 Observability

Access the monitoring and management tools via your browser:

- **Grafana**: `http://localhost:3000`
  - **Login**: `admin` / `admin123`
  - A pre-configured dashboard "DMA (Improved)" is available to monitor the system.

- **Prometheus**: `http://localhost:9090`
  - Explore metrics and service discovery status.

- **Kafka UI**: `http://localhost:8085`
  - A web UI for managing and monitoring Kafka topics, messages, and consumers.

---

## ⚡ Load Testing

This project uses **K6** to simulate realistic load scenarios and test system performance.

- **Service A (HTTP)**: The test script is located at `Service-A/internal/test/k6_http.js`.
- **Service B (HTTP)**: The test script is located at `Service-B/k6_http.js`.

**To run a test:**

1.  [Install K6](https://k6.io/docs/getting-started/installation/).
2.  Navigate to the service directory and run the script.

    **Example for Service A:**
    ```sh
    cd Service-A
    k6 run internal/test/k6_http.js
    ```

---

## 🧩 Services Breakdown

### Service A (Go)

- **Role**: Accepts gRPC and HTTP requests, calculates a sum, and writes the result to a PostgreSQL `outbox` table.
- **Technology**: Go, gRPC, PostgreSQL, Kafka producer (via Debezium).
- **Key Feature**: Implements the Outbox Pattern to ensure message durability and at-least-once delivery.

### Service B (Java)

- **Role**: Consumes messages from Kafka, adds the result to a running total, and persists it.
- **Technology**: Java, Spring Boot, Kafka Consumer, PostgreSQL.
- **Key Feature**: Maintains a real-time aggregate of results, providing a consistent view of the data via a REST API. It uses a Leader/Replica database pattern for read/write separation.

---


<img width="3290" height="4270" alt="DMA" src="https://github.com/user-attachments/assets/53f8e206-bd24-49ca-a825-850a449419c6" />


---

## 🙏 Acknowledgements

The initial version of this project was inspired by the concepts and architecture demonstrated in this YouTube video:
- [Build a distributed system at home](https://www.youtube.com/watch?v=Ur6b1NWGbYE)

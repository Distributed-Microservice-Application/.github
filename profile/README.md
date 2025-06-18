This project demonstrates a **microservices-based system** that processes user requests, performs calculations, and communicates asynchronously using **Kafka**, while ensuring observability and reliability using patterns like the **Outbox Pattern**, and tools like **Prometheus**, **Grafana**, and **OpenTelemetry**.

---

### 🚀 **Scenario / Flow Overview**

#### 👤 **User Interaction**

* A **user**, or a simulated user via **K6**, initiates a request to **Service A** through **gRPC**.
* The request contains **two numbers**, and the system's job is to **add them together quickly and reliably**.

#### 🧠 **Service A - The Smart Calculator**

* Listens for incoming gRPC requests.
* Performs the **simple yet essential task** of adding the two numbers.
* But here’s the twist: instead of immediately pushing the result to Kafka, it:

  * **Safely stores the result** in an **"outbox" table** in its database—this follows the **Outbox Pattern**, which ensures messages aren’t lost.
  * A **dedicated background job** then picks up the saved results and sends them to **Kafka**, ensuring **resilience and consistency**.

#### 🔁 **Kafka - The Reliable Messenger**

* Serves as the **message backbone**, connecting services seamlessly.
* **Queues the result messages** so they can be processed asynchronously by **Service B**, enabling a decoupled and scalable system.

#### 🧮 **Service B - The Running Total Keeper**

* Listens to the Kafka topic like a loyal subscriber.
* Retrieves each message, **adds the result to its running total**, and can optionally **log or persist** each value.
* It keeps track of everything, ensuring **nothing is missed**.

#### 📊 **Observability: Know Everything**

* **OpenTelemetry** agents continuously collect **metrics and logs**, like how long messages stay in Kafka.
* **Prometheus** acts as the **metrics timekeeper**, storing rich time-series data.
* **Grafana** gives life to the metrics with **beautiful dashboards**, showing real-time insights and helping engineers make informed decisions.

#### 🔥 **Testing at Scale**

* **K6** simulates **realistic load scenarios**, helping test how the system performs under stress.
* This ensures the architecture is **ready for anything**—from a few users to thousands.

---

### 🧩 **Summary of Components**

| 🌟 Component       | 🌍 Role                                                            |
| ------------------ | ------------------------------------------------------------------ |
| **Kubernetes**     | Orchestrates and scales all services effortlessly.                 |
| **Service A**      | Accepts gRPC requests, calculates sums, and writes to outbox.      |
| **Outbox Pattern** | Ensures **guaranteed delivery** to Kafka—no lost messages.         |
| **Kafka**          | Powers **asynchronous, decoupled communication** between services. |
| **Service B**      | Consumes messages, keeps an accurate running total.                |
| **OpenTelemetry**  | Tracks system behavior with detailed metrics and logs.             |
| **Prometheus**     | **Stores and queries** all metrics data.                           |
| **Grafana**        | Presents metrics on **stunning, real-time dashboards**.            |
| **K6**             | Pushes the system to its limits with **load testing**.             |

---

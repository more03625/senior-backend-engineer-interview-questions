---

## Use the following GPT link

```
https://chatgpt.com/share/67f0d585-d884-8008-a9e9-9d20de62142a
```

---

### 1. With 15+ microservices, how did your team manage inter-service communication?

Was there a preference for **Kafka**, **NATs**, or **REST**?

---

### 2. You mentioned improving delivery timelines by 20%

Curious—what were the most impactful **CI/CD** or **process changes** you introduced?

---

### 3. Leading 35+ developers is impressive

How do you mentor engineers to become strong **tech leads** or **system thinkers**?

---

### 4. What is an API Gateway / BFF (Backend For Frontend)?

1. **API Gateway**: A middleware layer between your frontend & microservices.
2. As a middleware, it enables:
    - 🛡️ **Authentication**: Verifies tokens (e.g., JWT)
    - 🗺️ **Routing**: Sends `/users` → User Service, `/orders` → Order Service
    - ⚙️ **Load Balancing**: Balances load between service replicas
    - ⏱️ **Rate Limiting**: Protects services from being overwhelmed
    - 📜 **Logging & Monitoring**: Central logs and metrics
    - 💥 **Retries/Fallbacks**: Resilience features
    - 🔐 **SSL Termination**: Handles HTTPS certs (so services don’t have to)

3. **Without API Gateway:**

    ```
    React → /user-service/api
    React → /product-service/api
    React → /order-service/api
    ```

4. **With API Gateway:**

    ```
    React → /api → API Gateway → user/product/order services
    ```

---

### 5. What if there are issues with the API Gateway? (Single Point of Failure)

- If your gateway is down, your entire app becomes unreachable.
- **Solutions:**
    1. In Kubernetes, run multiple gateway pod replicas (e.g., 3 pods).
    2. If one crashes, others continue to serve traffic.
    3. Use a **Load Balancer** in front of API gateway:

        ```
        [ Load Balancer ]
                ↓
        [ Gateway Pod 1 ]
        [ Gateway Pod 2 ]
        [ Gateway Pod 3 ]
        ```

    4. Kubernetes automatically checks gateway pod health.
    5. If unhealthy → removes it from service and spins up a new one.

---

### 6. How do you ensure 0 downtime during deployment?

- Use **Readiness Probe**: Ensures the new pod is ready before removing the old pod from Kubernetes.
- **FANG companies deploy 100+ times a day** with no downtime—thanks to smart deployment strategies.

---

### 7. What are logs in an application? Do you mean `console.log`s?

---

### 8. Is it a good idea to call services internally? Don’t you think it will take longer?

- Yes, but do it the **right way**, else it adds extra latency.

#### ✅ Sync Call:

1. One service calls another directly (e.g., `User Service` calls `Order Service`).
2. Easy to understand, but:
    - Adds latency
    - Creates tight coupling—if User Service is down, Order Service fails too.

#### ✅ Async Call:

1. Order is created → Event is published.
2. Inventory & Notification services listen and react.
3. Benefits:
    - Decouples services
    - Faster user response
4. Drawbacks:
    - Harder to debug
    - **Eventually consistent** (not real-time)
5. Best for:
    - Emails
    - Stock updates
    - Analytics
    - Notifications

---

### 9. Why do we use JWT Tokens? What if we create our own token mechanism?

- **Why JWT?**
    1. Contains all auth info (user ID, roles, expiry)
    2. No need to hit DB or session store every request
    3. **Stateless Auth**: Ideal for microservices
    4. Each service just verifies the token with a public key
    5. Standardized & widely supported
    6. Format: `<Header>.<Payload>.<Signature>`

- **Building your own JWT-like system:**
    1. You **can**, but it's complex and risky
    2. More edge cases, lower compatibility
    3. Don’t reinvent the wheel—JWT is mature and battle-tested

---

### 10. What do you mean by Logging & Monitoring? Do you mean console logs?

- Logging = writing detailed information about what’s happening inside your system (services, pods, APIs, etc.).

#### ✅ Example Logs:

- Incoming API requests (POST /login)  
- DB query errors  
- Auth token failures  
- Cron job execution time  

- These logs are usually written to files or cloud log aggregators (like AWS CloudWatch)

- Monitoring = measuring and observing system health/performance over time.
    - Request count / success rate
    - CPU, memory usage
    - Pod/container health
    - Error rates
    - DB connection count

#### Tools

| Purpose     | Tools                                                                 |
|-------------|-----------------------------------------------------------------------|
| Logging     | Winston, Pino, Logstash, Fluentd, Loki                                |
| Monitoring  | Grafana, Datadog                                                      |
| Alerting    | AlertManager, PagerDuty                                               |

---

### 11. What is Retries / Fallbacks (Resilience Features)?

- **Retries** = try the failed request again, automatically.

#### 🤔 Why?

Because many failures are temporary, like:
- Network timeout
- DB is restarting
- API rate-limited

##### Example:

```js
// Retry 3 times with 2s delay
retry(() => callPaymentService(), { retries: 3, delay: 2000 });
```

You can use `axios-retry`.

- **Fallback** = what should we do if the main service fails even after retries?

#### ✅ Examples:

- Show cached data if product service is down  
- Return a default response (like: “service unavailable, try again later”)  
- Use a backup service or replica  

---

### 12. What is a Refresh Token?

- A refresh token is a long-lived token that allows you to get a new access token without asking the user to log in again.

#### ⏳ Token Duration

- **Access Token** → Short (5–30 min) → Used for accessing resources  
- **Refresh Token** → Long (7–30 days) → Used to get new access token  

#### 🔁 How does it work?

1. User logs in → backend returns:
    - ✅ Access Token (valid 15 mins)
    - ✅ Refresh Token (valid 7 days)

2. Client stores both securely (usually in HTTP-only cookies or memory)

3. After 15 mins:
    - Access token expires ❌
    - Client sends refresh token to `/refresh-token` API
    - Backend checks refresh token → issues new access token

4. User continues using the app without logging in again

---

### 13. How you can send the email to Millions of users at the same time

1. **Use a Bulk Email Provider**  
    - Like Amazon SES, SendGrid, MailGun  
    - They handle: Rate limits, IP reputation, DKIM/SPF/DMARC, Bounce tracking

2. **Batch + Queue the Emails**  
    - Use a Message Queue like: Kafka, RabbitMQ, AWS SQS  
    - You produce emails into the queue, and multiple workers consume them in parallel and send emails

3. **Use a Job Worker System**  
    - Create worker services (microservices) that:
        - Read messages from the queue  
        - Call SES/SendGrid API to send mail  
        - Log success/failure  

---

### 14. Primary Key vs Unique Key (SQL / Postgres / MySQL)

#### 🔑 Primary Key vs Unique Key (SQL)

| Feature                  | Primary Key 🔐                          | Unique Key 🔍                          |
|--------------------------|----------------------------------------|----------------------------------------|
| **Enforces Uniqueness**  | ✅ Yes                                  | ✅ Yes                                  |
| **Allows NULL values?**  | ❌ No (NOT NULL by default)             | ✅ Yes (can allow NULLs)                |
| **How many per table?**  | ❗ Only 1 allowed                        | ✅ Multiple allowed                     |
| **Use Case**             | Main identifier for a row (e.g., `id`) | Enforce uniqueness on other columns (e.g., `email`) |
| **Used in Foreign Keys?**| ✅ Commonly used                        | ❌ Rarely used                          |
| **Index Type**           | Clustered Index (DB-dependent)         | Non-clustered Index                    |

---

### 15. What are the ways to Optimize the API?

1. **Caching**  
    - Store frequent data in Redis / CDN (Cloudflare, Fastly)  
    - Reduces DB hits & response time  
    - Eg: product list, user profile

2. **Database Optimization**  
    - Use proper indexing 🗂  
    - Optimize slow queries using `EXPLAIN`  
    - Avoid N+1 queries 🔁  
    - Use joins smartly and avoid over-fetching  

3. **Pagination & Filtering**

4. **Compression**  
    - Enable GZIP or Brotli for API responses  

5. **Asynchronous Processing**  
    - For heavy tasks (sending emails, reports), use queues  
    - Avoid blocking response time  

6. **Return Only What’s Needed**  
    - Do not use `SELECT *`

7. **Use CDN for Static/Media**

---

### 16. What are N + 1 Queries

- It happens when your code makes 1 query to fetch main data & N additional queries to fetch related data for each item individually.

```js
// BAD - N+1 Problem
const users = await db.users.findMany(); // 1 query

for (let user of users) {
    user.orders = await db.orders.findMany({ where: { userId: user.id } }); // N queries
}
```

#### 📉 Why it's Bad?

- Slow performance  
- More DB round-trips  
- Increased latency  

---

### 17. How to optimize the database?

- **Use Proper Indexing**:
  - Index frequently searched/filter columns (`WHERE`, `JOIN`, `ORDER BY`)
  - ⚠️ Don’t over-index — each index slows down `INSERT`/`UPDATE`

- 🚫 **Avoid `SELECT *`**

- 🚫 **Avoid N+1 Queries**

- **Normalize**:
  - Normalize to reduce redundancy
  - Denormalize for performance if needed (read-heavy)

- **Use Caching (Redis)**:
  - Cache frequently accessed data/response to reduce DB hits

- **Analyze Slow Queries**:
  - Use `EXPLAIN ANALYZE` in PostgreSQL
  - Enable MySQL Slow Query Log

- **Batch Inserts/Updates**:
  - Avoid inserting rows one-by-one
  - Use bulk insert/update strategies

- **Use Read Replicas**:
  - Offload read traffic from primary DB

- **Partitioning / Sharding**:
  - For very large tables, partition based on date, region, etc.
  - Zero create new table every time for reports.

---

### 19. what is normalization & denormalization?
- ✅ Denormalization = Combining tables to reduce JOINs and improve read performance (ETL)
    - Use Normalization for write-heavy, transactional systems.
    - Use Denormalization for read-heavy, analytics-heavy systems.
    - Example: You have `customers`, `orders`, `payments` tables — you want a fast dashboard for total amount paid by each customer. You can use ETL here.

---

### 20. EXPLAIN `EXPLAIN ANALYZE` Keword
- This runs the query, and shows: Actual execution time
- Also it gives you COST, based on that you can optmize the query.
---

### 21. How to Host Video Securly? To avoid piracy of Movies, YT Videos, Courses
- **✅ 1. Never expose raw video file URLs (MP4 etc.)**
    - ❌ Bad: Direct link like https://cdn.com/video.mp4
    - Anyone can just "Inspect → Network → Download".

- **✅ 2. Use HLS (HTTP Live Streaming) with encryption**
    - Instead of one video file, split it into small .ts chunks + .m3u8 playlist
        - Each chunk is a few seconds
        - Encrypt chunks with AES-128
        - Keys are rotated or provided via a secure key server

- **✅ 3. Use DRM (Digital Rights Management)**
    - The most secure option, used by Netflix, Amazon Prime, Hotstar.

---

---

### 22. What is middleware.
- Middleware runs between request & response cycle in web servers.
- **✅ Advantages of middleware**
    - ⏱ Modify the request (req)
    - 🛡 Add security/auth checks
    - 🔍 Log requests
    - 🧹 Validate data
    - ⛔ Block or pass the request to the next handler

---

### 23. Node.js Over PHP.

| Feature | **Node.js** | **PHP** |
|--------|-------------|--------|
| **Concurrency Model** | Non-blocking, async (event-driven) ✅ | Blocking, sync (per-request thread) ❌ |
| **Speed** | Very fast for I/O-heavy apps | Slower in comparison |
| **Real-time Apps (e.g., chat, gaming)** | Excellent support via WebSockets | Not ideal |
| **Single Language (JS)** | Frontend + Backend = JS everywhere 🔥 | PHP for backend, JS for frontend |
| **Package Ecosystem** | Huge with NPM | Decent with Composer |
| **Modern APIs / JSON** | Built-in, natural fit | Requires extra libraries sometimes |
| **Microservices & Serverless** | Preferred choice | Less common in modern serverless stacks |

---

## 🧠 When to Choose Node.js

- You want **real-time**, fast I/O applications (chat, streaming, live dashboards).
- You're building a **modern microservices** architecture.
- You already have a frontend team using JavaScript.
- You need better **DevOps compatibility** (Docker, K8s, etc).

---

## ⚖️ When PHP is Okay

- CMS platforms like **WordPress**, small blogs.
- Legacy apps already in PHP.

---

### 24. Node.js vs PHP: Concurrency Model & Speed in Backend

## 🧵 1. Concurrency Model

#### 🔹 PHP:
- **Blocking / Synchronous**
- Each request is handled by a **separate thread or process**.
- If one request is slow (e.g., DB query or API call), that thread is **blocked**.
- High memory usage under heavy load.

```
$data = file_get_contents("http://slow-api.com"); // waits till response
echo $data; // will wait here
```

---

#### 🔹 Node.js:
- **Non-blocking / Asynchronous**
- Uses a **single-threaded event loop** + background workers (libuv).
- Can handle **thousands of requests** concurrently without spawning new threads.
- I/O (DB, file, API) is done **asynchronously** in background.

```
// Node.js
fetch("http://slow-api.com").then(res => res.text()).then(console.log);
console.log("This runs while the above request is pending.");
```

✅ **Result:** Better performance under I/O load (DB, file reads, APIs).

---

## ⚡ 2. Speed

- **Node.js is faster** for I/O-bound operations due to its event-driven model.
- **PHP is slower** for high-concurrency scenarios because:
  - Every request creates a new thread/process.
  - Threads consume memory and CPU.

---

## 📌 Summary

| Feature           | Node.js                          | PHP                              |
|------------------|----------------------------------|----------------------------------|
| Model            | Event-driven, async              | Multi-threaded, blocking         |
| Speed (I/O)      | Fast                             | Slower under heavy load          |
| Scalability      | Very scalable                    | Requires more server resources   |

---

### 22. Explain OOPS Concepts.
- We have following OOPs Concepts
- **Encapsulation**
    - You hide the internal logic & just expose the methods.
    - Real-life Example: A car’s gearbox — you use it, but don’t know the internal logic.

- **Abstraction**
    - Hiding unnecessary implementation details from the user.
    - Real-life Example: You send a message on WhatsApp — don’t need to know the underlying network logic.

- **Inheritance**
    - One class can inherit properties/methods from another.
    - It Promote Code reuse.

- **Polymorphisum**
    - Same method behaves differently based on object/context.
    - Class B can use the same function name from class A & change the behavior
    - Example:
    ```
        class Shape {
            draw() {
                console.log("Drawing shape");
            }
        }

        class Circle extends Shape {
            draw() {
                console.log("Drawing circle");
            }
        }

        function render(shape) {
            shape.draw(); // polymorphic call
        }
    ```

---

### 23. Design Patterns & Architecture?

## ✅ Design Patterns
Reusable solutions to common software design problems at the **code level**.

---

### 🔧 Creational Patterns

- **Singleton**  
  Ensures a class has only one instance and provides a global point of access.  
  _Example: Database connection pool._

- **Factory**  
  Creates objects without exposing the creation logic to the client.  
  _Useful when the exact type of object isn’t known until runtime._

- **Builder**  
  Builds complex objects step-by-step.  
  _Example: Constructing UI components or configuration objects._

---

### ⚙️ Structural Patterns

- **Adapter**  
  Bridges the gap between incompatible interfaces.  
  _Example: Connecting to a third-party API with a different format._

- **Proxy**  
  Provides a surrogate or placeholder to control access.  
  _Use cases: Caching, access control, lazy loading._

- **Decorator**  
  Dynamically adds new functionality to objects without altering their structure.  
  _Example: Adding logging or validation behavior._

---

### 🔁 Behavioral Patterns

- **Observer**  
  Enables objects to be notified of state changes in other objects.  
  _Example: Event listeners, publish/subscribe systems._

- **Strategy**  
  Selects an algorithm at runtime based on context.  
  _Example: Switching between multiple payment gateways._

- **Command**  
  Encapsulates a request as an object, allowing it to be queued, logged, or undone.  
  _Example: Task queues, undo operations._

---

## 🏗️ Architectural Patterns
High-level strategies to structure and organize **entire applications** or **services**.

---

### 1️⃣ Monolithic Architecture

- All features and logic reside in a single deployable unit  
- ✅ Easy to develop & deploy  
- ❌ Hard to scale or maintain over time

---

### 2️⃣ Microservices Architecture

- Each feature is a **separate service**  
- ✅ Independent deployments & scalability  
- ❌ Requires inter-service communication (REST, gRPC, Kafka)

---

### 3️⃣ Event-Driven Architecture

- Services communicate by **publishing and consuming events**  
- ✅ Decoupled, asynchronous  
- _Tools: Kafka, RabbitMQ_

---

### 4️⃣ Serverless Architecture

- Based on **Functions-as-a-Service (FaaS)** like AWS Lambda  
- ✅ Auto-scalable, pay-per-use  
- ❌ Cold starts, limited control over infrastructure

---

### 5️⃣ Service-Oriented Architecture (SOA)

- Predecessor to microservices  
- Uses **Enterprise Service Bus (ESB)** for communication  
- Focused on reusability and integration across large enterprises

---

### 6️⃣ Hexagonal Architecture (Ports & Adapters)

- Core business logic is **isolated** from external concerns  
- External interfaces (DB, APIs) plug in via adapters  
- Improves testability and maintainability

---

### 7️⃣ Layered Architecture

- Common in **MVC frameworks**  
- Separation of concerns via layers:
  - `Controller → Service → Repository`  
- ✅ Easy to understand and structure  
- ❌ Can become rigid in large systems

---



### 24. What was the design pattern i was using?
Based on your structure:

> **Node.js + Microservices** using  
> **Routes → Controller → Service → DB**  
> and **GitHub Submodules** for shared code

Let's break down the **architecture** and **design patterns** you're using:

---

## ✅ 1. Architecture You Were Using

### 🧩 Microservices Architecture

Each service is **independently deployed**, likely handling a single business capability (e.g., User, Orders, Payments).

- **Pros**: Scalable, isolated failures, team ownership  
- **Cons**: Complex to monitor, manage inter-service communication

---

### 🧱 Layered Architecture

Your internal flow — `Routes → Controller → Service → DB` — follows the **Layered Architecture** pattern (also known as MVC-ish structure).

- **Route** = Entry point (like a controller)
- **Controller** = Handles request/response
- **Service** = Business logic
- **DB Layer / Repository** = Data access

✅ Each layer has a **single responsibility**, making it easier to test and maintain.

---

## ✅ 2. Design Patterns You Were Using

### 🧰 Common Patterns in Your Setup

| Layer             | Pattern Used               | Explanation                                         |
|------------------|----------------------------|-----------------------------------------------------|
| Service Layer     | **Service Pattern**         | Business logic abstraction                          |
| DB Access         | **Repository Pattern**      | DB queries are abstracted into a separate module    |
| GitHub Submodules | **Modular Monorepo**        | Shared code across services (DRY principle)         |
| Shared Logic      | **Singleton / Utility**     | e.g., Logger, Config, DB Pool                       |
| Middlewares       | **Chain of Responsibility** | e.g., Auth, Logging, Validation                     |

---

## 🔍 Summary

You're using a combination of:

- ✅ **Microservices Architecture**
- ✅ **Layered Architecture**
- ✅ Design Patterns:
  - **Repository Pattern**
  - **Service Pattern**
  - **Singleton Pattern** (for configs/connections)
  - **Chain of Responsibility** (for middlewares)

---
### 22. Agile Vs Waterfall model.
- **🌀 Agile Model**
    - Work is broken into sprints (usually 1–2 weeks)
    - Each sprint delivers a working piece of the product
    - Emphasis on customer feedback, flexibility, and collaboration
    - Teams adapt quickly to changes
    - Used For
        - StartUps
        - Dynamic environments
        - Projects where requirements change frequently

- **💧 Waterfall Model**
    - Insted of sprint we do phase wise development.
    - Waterfall is a linear and sequential approach.
    - Each phase must be completed before the next starts
    - Requirements are defined upfront
    - Changes are hard to accommodate later
    - Used For
        - Government or defense projects
        - Fixed scope projects with clear, unchanging requirements

---

### 23. If we have multiple service and each service has its own db then how to achive consistency?
- **1. Eventual Consistency**
    - Accept that immediate consistency is not always possible.
    - Services publish events (e.g., "OrderCreated") when data changes.
    - Other services consume these events and update their own state.
    - Example: Order Service updates the order, emits an event → Inventory Service consumes it and updates stock.
    - If Inventory service fails here, then
        - **1. Event Retention + Retry Mechanism**
            - The event is not lost. It's kept in the queue or topic.
            - You can configure dead-letter queues (DLQs) to handle events that fail after multiple retries.

        - **2. Idempotent Consumers.**
            - Ensure Inventory Service is idempotent — processing the same event multiple times won't corrupt data.
            - Helps if the service crashes midway and reprocesses the same event later.

        - **3. DLQ Monitoring & Manual Recovery**
            - If an event consistently fails, send it to a Dead Letter Queue.
            - Engineers get alerted, inspect logs, and manually reprocess if necessary.

        - **4. 🧠 Real-life Example:**
            - Think of Amazon. If your payment succeeds but inventory fails to deduct due to a temporary issue, they don't cancel the whole order. They retry internally and notify support if it fails repeatedly.


- **2. Distributed Transactions (SAGA Pattern)**
    - Break one large transaction into a series of local transactions
    - Two types of SAGA:
        - Choreography: Services communicate by events (no central coordinator)
        - Orchestration: A central service manages the flow

    - Example:
        - Order Service creates order
        - It calls Payment Service
        - If payment fails, a compensating action cancels the order

### 24. What if an event fails to be processed by the consumer (Service B)?
- **🧠 Scenario Recap:**
    - Service A updates its own DB ✅
    - It emits an event (e.g., "OrderCreated") 📤
    - Service B receives the event but fails to process it ❌ (e.g., DB error, network issue).

- **🔥 Problem:**
    - Now Service A and Service B are out of sync
    - This breaks the eventual consistency promise — unless handled carefully

- **✅ How to Handle This**
    - **1. Retry Mechanism (Automatic Retries)**
        - Use a message broker like Kafka, RabbitMQ, or AWS SNS/SQS
        - If Service B fails, the broker automatically retries the message
        - You configure:
            - Retry intervals
            - Max attempts

    - **2. Dead Letter Queue (DLQ)**
        - If retries fail, the message goes to a DLQ (dead-letter queue)
        - You can:
            - Alert the team
            - Replay the message manually later
            - Write scripts to reprocess DLQ messages

    - **3. Idempotency:**
        - You should not process same event 2 times.
        - Even if you do, it should not update the data.
        - Basically, just check weather the event is already processed, before processing any event.
        - Example: 
            ```
                if (!eventAlreadyProcessed(eventId)) {
                    processEvent();
                    markEventAsProcessed(eventId);
                }
            ```

    - **4. Monitoring & Alerting**
        - Track failures from DLQ
        - Alert if a certain threshold of retries or DLQ volume is hit

---

### 25. How to Track User Journey Across Microservices?

- **🧠 Problem Statement:**
    - In a microservices setup, tracking a user’s end-to-end activity is tricky but essential for:
        - Debugging
        - Auditing
        - Analytics
---

- **✅ 1. Use Distributed Tracing**
    - **Tools:**
        - Jaeger
        - Zipkin
        - AWS X-Ray
        - OpenTelemetry (vendor-neutral)
    - **How it works:**
        - Assign a **Trace ID** when request enters system (via API Gateway or first service)
        - Pass Trace ID to all downstream services
        - Each service logs its span using the same Trace ID
        - ✅ You get a timeline-style view across services

---

- **✅ 2. Add a Correlation ID Middleware**
    - Inject a `X-Request-ID` or `X-Correlation-ID` in HTTP headers
    - Each microservice logs this ID with every action
    - Helps group logs from different services

    - **Centralized Logging Systems:**
        - Elasticsearch + Kibana
        - Grafana Loki
        - AWS CloudWatch Logs

---

- **✅ 3. Centralized Logging**
    - Use stacks like:
        - **ELK** (Elasticsearch + Logstash + Kibana)
        - **EFK** (Elasticsearch + Fluentd + Kibana)
        - **Grafana Loki + Promtail**
    - Query logs using `trace_id`, `correlation_id`, or `user_id`

---

- **✅ 4. User Action / Event Tracking**
    - Emit structured events like `UserLoggedIn`, `ItemAddedToCart`
    - Send events to Kafka/NATS/EventBus
    - Store in EventStore or Analytics DB
    - Use to build user activity timeline

---

- **✅ 5. Visual Flow Example**
```
[API Gateway] --TraceID--> [Auth Service] --TraceID--> [Order Service] --TraceID--> [Payment Service]

All services log to --> [Central Logging System] or [Tracing Dashboard]

You search logs using TraceID or userId to view complete flow
```

---

### 26. What is mean by Incident & Tools?
- An incident is any unplanned disruption or degradation of a service that affects users, systems, or business operations.

- Sentry: 🪲 Real-time error tracking (frontend & backend), useful for RCA.

- PagerDuty: 🚨 Sends alerts to on-call engineers. Great for automated escalation and scheduling.

**💬 Real-World Analogy:**
    - Like in a hospital:
        - If a patient (your app) suddenly collapses (service crash), a code blue is called (alert), doctors rush (on-call team), revive the patient (rollback/fix), and later analyze what went wrong (RCA/Postmortem).

---

### 27. What is diff between encapsulation & abstraction?
- **⚙️ Encapsulation → “Hiding internal state & logic from outside access**

- **🧠 Abstraction → “Hiding unnecessary details, showing only essential features**

---

### 28. If data deleted directly from DB Not from Application Level, then how we can rollback?
- **🧯 1. Immediate Reality Check**
    - **If it's already gone from the database and there’s no backup — it’s irreversible.**
    - **If it’s critical data, this is why direct DB access is discouraged in production.**

- **🛡️ 2. Ways to Rollback / Recover Data**
    - **Database Backups:**
        - Restore from full or incremental backups (daily/hourly). This is the most common recovery method.

    - **Database Binlogs / Write-ahead logs (WAL):**
        - If binlogs or WAL are enabled, you can replay logs up to a certain point.

    - **Replication Slave:**
        - If you have a read replica that hasn’t synced yet, you can extract data from there.
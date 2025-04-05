
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

### 8. Is it a good idea to call services internally?
Don’t you think it will take longer?

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

- Why JWT?
    1. Contains all auth info (user ID, roles, expiry)
    2. No need to hit DB or session store every request
    3. **Stateless Auth**: Ideal for microservices
    4. Each service just verifies the token with a public key
    5. Standardized & widely supported
    6. Format: `<Header>.<Payload>.<Signature>`

- Building your own JWT-like system:
    7. You **can**, but it's complex and risky
    8. More edge cases, lower compatibility
    9. Don’t reinvent the wheel—JWT is mature and battle-tested

---

### 10. What do you mean by Logging & Monitoring? Do you mean console logs?

- Logging = writing detailed information about what’s happening inside your system (services, pods, APIs, etc.).
- ✅ Example Logs:
    - Incoming API requests (POST /login)
    - DB query errors
    - Auth token failures
    - Cron job execution time

- These logs are usually written to files, cloud log aggregators (like AWS CloudWatch)
- Monitoring = measuring and observing system health/performance over time.
    - Request count / success rate
    - CPU, memory usage
    - Pod/container health
    - Error rates
    - DB connection count

- Purpose	Tools
    - Logging: Winston, Pino, Logstash, Fluentd, Loki
    - Monitoring: Grafana, Datadog
    - Alerting: AlertManager, PagerDuty
---

### 11. What is Retries / Fallbacks (Resilience Features)?

- Retries = try the failed request again, automatically.
- 🤔 Why?
    - Because many failures are temporary, like:
        - Network timeout
        - DB is restarting
        - API rate-limited
    - Example:
        ```
        // Retry 3 times with 2s delay
        retry(() => callPaymentService(), { retries: 3, delay: 2000 });
        ```
    - You can use `axios-retry`.
 
- Fallback = what should we do if the main service fails even after retries?
- ✅ Examples:
    - Show cached data if product service is down
    - Return a default response (like: “service unavailable, try again later”)
    - Use a backup service or replica

---

### 12. What is a Refresh Token?

- A refresh token is a long-lived token that allows you to get a new access token without asking the user to log in again.
    - Access Token --> Short (5–30 min) --> Used for accessing resources
    - Refresh Token --> Long (7–30 days) --> Used to get new access token
- 🔁 How does it work?
- Step-by-step:
    1. User logs in → backend returns:
        - ✅ Access Token (valid 15 mins)
        - ✅ Refresh Token (valid 7 days)
    2. Client stores both securely (usually in HTTP-only cookies or memory)
    3. After 15 mins:
        - Access token expires ❌
        - Client sends refresh token to /refresh-token API
        - Backend checks refresh token → issues new access token
    4. User continues using the app without logging in again

```

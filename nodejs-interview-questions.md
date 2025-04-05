1. With 15+ microservices, how did your team manage inter-service communication? Was there a preference for Kafka, NATs, or REST?

2. You mentioned improving delivery timelines by 20%. Curious—what were the most impactful CI/CD or process changes you introduced?

3. Leading 35+ developers is impressive—how do you mentor engineers to become strong tech leads or system thinkers?

4. What is API gateway / BFF? (Backend For Frontend)

5. How do you ensure 0 donwtime while deployment.
    - Ans is Readiness Probe -> Check if the new pod is ready.

6. What are logs in Application? Do you mean console.logs?
7. Is it good idea to call service internally? don't you think it will take longer time?
    - Yes, do it the right way, else it will add extra latency

    # Sync Call
        1. You can do HTTP call like `User Service` Call `Order service`.
        2. It is simple and easy to understand, but the issue is 
            1. It add extra latency
            2. If user service is donw, order service also fails.
        3. Don't over do this, or avoid this.

    # ASync Call
        1. Order is created & event is published.
        2. Inventory & Notification services listen and react.
        3. It Completely decouples services. No dependency
        4. Faster response to user, as not wating for multiple services now.
        5. But it is harder to debug & it is eventual consistency no real time.
        6. You can use this for email, stock update, analytics, notifications



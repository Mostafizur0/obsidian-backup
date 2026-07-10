  
1. ==🔍 Service Registry==: This pattern involves managing the locations of services in a distributed system. It maintains a list of all available services and their locations, which can be queried by other services to find and communicate with them.  
  
2. ==⚡️ Circuit Breaker==: This pattern is used to prevent cascading failures in a distributed system. It monitors the availability of a service and, if it detects a failure, it can quickly isolate the problematic service and prevent other services from being affected.  
  
3. ==🚪 API Gateway==: This pattern provides a single entry point to a microservices-based system. It acts as a reverse proxy and routes incoming requests to the appropriate microservice. It can also perform authentication, rate limiting, and other security-related tasks.  
  
4. ==📝 Event Sourcing==: This pattern involves capturing all changes to the state of a system as a series of events. These events can be used to reconstruct the current state of the system at any point in time. This pattern is useful for systems with complex business logic that require auditability, traceability, or compliance.  
  
5. ==🎭 Saga==: This pattern is used to manage long-running transactions that involve multiple microservices in a distributed database. It ensures that all services involved in a transaction are completed successfully or rolled back in case of failures.  https://www.youtube.com/watch?v=DOFflggE_0Q https://www.youtube.com/watch?v=d2z78guUR4g
  
6. ==📚 CQRS==: This pattern separates the read and writes operations of a system. It uses separate models for reads and writes, which allows for the optimization and scalability of each. This pattern is particularly useful for systems with high read-and-write workloads.  
  
7. ==💾 Database per Service==: This pattern involves using a separate database for each microservice. This ensures that each microservice has its own data store, which can be optimized for its specific needs. It also helps to prevent coupling between services.  
  
8. ==🕺 Saga Choreography==: This pattern is similar to the Saga pattern, but instead of having a central orchestrator, each microservice involved in the transaction communicates directly with other services to coordinate the transaction.  
  
9. ==🕸️ Service Mesh==: This pattern provides a dedicated infrastructure layer (Istio) for managing communication between microservices. It adds features such as load balancing, service discovery, and security to the network layer, which can be used by any microservice in the system.  
  
10. ==🧳 Sidecar==: This pattern involves deploying a separate process alongside a microservice, which handles certain tasks such as service discovery, load balancing, or communication with other services. This allows the microservice to focus on its core functionality, while the sidecar handles cross-cutting concerns.
11. ==Throttling pattern==: Controlling the rate of processing to prevent resource exhaustion.
12. ==Rate Limiting pattern==: Restricting the rate of incoming requests to a system.
13. ==Publisher-Subscriber pattern==: Communication pattern where senders notify many receivers.
14. ==Materialized View pattern==: Precomputing and storing query results for faster data retrieval.
15. ==Bulkhead pattern==: Isolating components to prevent cascading failures in a system.
16. ==Backends for Frontends pattern==: Specialized backend services for different user interfaces (frontends).
17. ==Ambassador pattern==: Proxy managing communication between microservices.
18. ==Blue-Green Deployment pattern==: Deploying a new version of an application alongside the old one for easy rollback in case of issues.
19. ==Cache-Aside pattern==: Loading data into a cache on-demand to improve data access times and reduce server load.
20. ==Retry Pattern==: Automatically retry failed operations to improve the chances of success.
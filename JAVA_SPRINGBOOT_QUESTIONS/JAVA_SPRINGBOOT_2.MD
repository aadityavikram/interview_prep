# 1. What is Saga design pattern?
1. The Saga design pattern is a microservices architectural pattern used to manage long-running, distributed business transactions without requiring a distributed transaction manager (like 2-phase commit).  
Instead of a single global transaction, the Saga pattern breaks a transaction into a sequence of smaller local transactions. Each service executes its local transaction and publishes an event (or sends a message) to trigger the next step.  
If one step fails, the pattern ensures data consistency by executing compensating transactions (i.e., undo operations) for the already completed steps.

2. Key Concepts of Saga Pattern:

Local Transactions - Each service executes a transaction that updates its database and then triggers the next service.  
Compensating Transactions - If a failure occurs, previously executed transactions are undone using compensating actions.  
Event/Message Driven - Saga relies on messages/events to coordinate between services.

3. Two Main Saga Implementations: -  
  a. Choreography (Event-based)  
  1. No central controller.  
  2. Each service publishes events and listens for the next event.  
  3. Simpler, but can become hard to manage as complexity grows.  
  4. Example:  
  Order Service → creates order → emits OrderCreated event.  
  Payment Service → listens, processes payment → emits PaymentCompleted.  
  Shipping Service → listens, ships product → emits Shipped.  
  If payment fails → emit PaymentFailed → Order Service compensates by canceling the order.

  b. Orchestration (Command-based)  
  1. A central orchestrator (saga coordinator) tells each service what to do.  
  2. Easier to manage complex workflows.  
  3. Example: An Order Orchestrator service calls Payment → Shipping → Notification in sequence. If payment fails, orchestrator calls the Order Service to cancel.

4. Benefits:-  
  a. Avoids distributed transactions (2PC).  
  b. Ensures eventual consistency in microservices.  
  c. Allows long-running workflows.

5. Challenges:-  
  a. Harder to design compensating transactions (not always possible).  
  b. Choreography may lead to complex event flows.  
  c. Requires careful error handling, retries, and idempotency.

6. In short: -  
The Saga pattern is used in microservices to handle distributed, long-running transactions by splitting them into local transactions with compensating actions for failures, ensuring eventual consistency instead of strong consistency.

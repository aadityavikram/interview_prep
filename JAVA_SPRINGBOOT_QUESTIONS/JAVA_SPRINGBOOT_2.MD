# 1. What is Saga design pattern?
1. The Saga design pattern is a microservices architectural pattern used to manage long-running, distributed business transactions without requiring a distributed transaction manager (like 2-phase commit).  
Instead of a single global transaction, the Saga pattern breaks a transaction into a sequence of smaller local transactions. Each service executes its local transaction and publishes an event (or sends a message) to trigger the next step.  
If one step fails, the pattern ensures data consistency by executing compensating transactions (i.e., undo operations) for the already completed steps.

2. Key Concepts of Saga Pattern:-  
  a. Local Transactions - Each service executes a transaction that updates its database and then triggers the next service.  
  b. Compensating Transactions - If a failure occurs, previously executed transactions are undone using compensating actions.  
  c. Event/Message Driven - Saga relies on messages/events to coordinate between services.

3. Two Main Saga Implementations: -  

  | Implementation | Features | Example |
  |----------------|----------|---------|
  | Choreography (Event-based) | No central controller. | Order Service → creates order → emits OrderCreated event. |
  | | Each service publishes events and listens for the next event. | Payment Service → listens, processes payment → emits PaymentCompleted. |
  | | Simpler, but can become hard to manage as complexity grows. | Shipping Service → listens, ships product → emits Shipped. |
  | | | If payment fails → emit PaymentFailed → Order Service compensates by canceling the order. |
  | Orchestration (Command-based) | A central orchestrator (saga coordinator) tells each service what to do. | Example: An Order Orchestrator service calls Payment → Shipping → Notification in sequence. |
  | | Easier to manage complex workflows. | If payment fails, orchestrator calls the Order Service to cancel. |

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
<br />
<br />
<br />



# 2. What is Idempotency in microservice?
1. In microservices, idempotency means that performing the same operation multiple times has the same effect as performing it once. This is critical in distributed systems because network calls can fail, retry, or get duplicated. Without idempotency, retries could cause incorrect states (e.g., double-charging a customer).

2. Key Idea: -  
  a. Idempotent Operation: Multiple identical requests → no side effects beyond the first execution.  
  b. Non-Idempotent Operation: Multiple identical requests → changes the state each time.  

3. Examples: -  

  | Request | Example |
  |---------|---------|
  | Idempotent Request | GET /order/123 → Always returns the same order details, no matter how many times you call it. |
  | | PUT /order/123/status { "status": "CANCELLED" } → Calling it once or ten times results in the order being cancelled once. |
  | Non-Idempotent Request | POST /order → Creates a new order each time. |
  | | Calling it multiple times creates multiple orders. |

4. Why It’s Important in Microservices?  
  a. Retries: If a request times out, the client may retry → server must not duplicate side effects.  
  b. At-Least-Once Messaging: Messaging systems may deliver the same message more than once.  
  c. Consistency: Helps maintain correct system state in distributed workflows (like Sagas).

5. How to Ensure Idempotency  

  | Method | Example |
  |--------|---------|
  | Idempotency Keys | Client sends a unique ID with the request (X-Idempotency-Key: abc123). |
  | | Server stores processed keys to prevent duplicate processing. |
  | Design APIs Properly | Use PUT and DELETE (naturally idempotent) instead of POST where possible. |
  | Example | PUT /payment/txn123 → Processes payment for transaction txn123 only once. |
  | | Retry won’t create a new payment. |
  | Idempotent Message Handlers | Store message IDs that have already been processed to avoid re-processing duplicates. |
  | Compensating Transactions  | For non-idempotent operations, use compensating logic in case of duplicate execution. |

6. In short, Idempotency in microservices ensures safe retries and duplicate handling by making sure repeated requests don’t cause unintended side effects.
<br />
<br />
<br />



# 3. Super class of all java class
1. Every class in Java implicitly inherits from Object (directly or indirectly).  
```java
class MyClass {
    // even if you don’t write "extends Object",
    // it automatically extends java.lang.Object
}
```
2. If you don’t explicitly extend another class, the compiler automatically makes your class extend Object.  
3. This makes Object the root class of the Java class hierarchy.  
4. Important Methods in Object  
  a. toString() → Returns a string representation.  
  b. equals(Object obj) → Compares objects for equality.  
  c. hashCode() → Returns a hash value for the object.  
  d. getClass() → Returns runtime class information.  
  e. clone() → Creates a copy of the object (if Cloneable).  
  f. finalize() → Called by garbage collector before object is destroyed.  
  g. wait(), notify(), notifyAll() → Used for thread synchronization.  

5. Java Class Hierarchy
           Object
              ↑
         -------------
         |           |
      String      Number
                     ↑
                 Integer, Double, etc.  

6. So no matter what class you create in Java, at the very top it always inherits from java.lang.Object.
<br />
<br />
<br />



# 4. OOPS concept
1. Java is a purely object-oriented language (with some exceptions like primitives), and it is built on 4 main OOP principles:  

2. 4 Pillars of OOP (OOPs Concepts)  

  | Pillar | Description |
  |--------|-------------|
  | Encapsulation (Data Hiding) | Wrapping data (fields) and methods (functions) into a single unit (class). |
  | | Use private variables + public getters/setters to control access. |
  | | Prevents direct access to internal state → provides data security. |

```java
class Account {
    private double balance;  // hidden from outside

    public void deposit(double amount) {
        balance += amount;
    }
    public double getBalance() {
        return balance;
    }
}
```

  | Pillar | Description |
  |--------|-------------|
  | Abstraction (Hiding Implementation Details) | Showing only essential features and hiding internal complexity. |
  | | Achieved using abstract classes and interfaces in Java. |
  | | Focuses on what an object does, not how it does it. |

```java
abstract class Vehicle {
    abstract void start(); // abstract method (no body)
}

class Car extends Vehicle {
    void start() {
        System.out.println("Car starts with key");
    }
}
```

  | Pillar | Description |
  |--------|-------------|
  | Inheritance (Reusability) | One class can inherit properties & methods of another class using extends. |
  | | Promotes code reusability. |
  | | Allows hierarchical classification (is-a relationship). |

```java
class Animal {
    void eat() { System.out.println("Eating..."); }
}

class Dog extends Animal {
    void bark() { System.out.println("Barking..."); }
}
```

  | Pillar | Description |
  |--------|-------------|
  | Polymorphism (Many Forms) | Ability of an object to behave in multiple ways. |
  | | Compile-time (Overloading) → same method name, different parameters. |
  | | Runtime (Overriding) → subclass provides a new implementation of a method. |

Compile-time (Overloading)
```java
class Calculator {

    // add two integers
    int add(int a, int b) {
        return a + b;
    }

    // add three integers
    int add(int a, int b, int c) {
        return a + b + c;
    }

    // add two doubles
    double add(double a, double b) {
        return a + b;
    }
}

public class Main {
    public static void main(String[] args) {
        Calculator calc = new Calculator();

        System.out.println(calc.add(10, 20));        // calls add(int, int)
        System.out.println(calc.add(10, 20, 30));    // calls add(int, int, int)
        System.out.println(calc.add(5.5, 4.5));      // calls add(double, double)
    }
}
```

Runtime (Overriding)
```java
class Animal {
    void sound() { System.out.println("Animal makes sound"); }
}

class Dog extends Animal {
    void sound() { System.out.println("Dog barks"); }
}
```

3. Bonus Concepts in OOP:-  
  a. Class & Object → Blueprint vs. real-world entity.  
  b. Association, Aggregation, Composition → Relationships between objects.  
  c. Constructor & Destructor (GC in Java acts like destructor).  

4. In short:-  
  a. OOP in Java = Encapsulation + Abstraction + Inheritance + Polymorphism.  
  b. It helps in code reusability, security, flexibility, and maintainability.

<br />
<br />
<br />




# 5. 

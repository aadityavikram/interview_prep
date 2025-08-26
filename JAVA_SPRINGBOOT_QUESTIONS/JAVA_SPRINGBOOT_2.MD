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




# 5. ArrayList vs LinkedList

| Feature                                 | **ArrayList**                              | **LinkedList**                                             |
| --------------------------------------- | ------------------------------------------ | ---------------------------------------------------------- |
| **Underlying structure**                | Dynamic array                              | Doubly linked list                                         |
| **Access (get/set by index)**           | Fast (O(1))                                | Slow (O(n)) — must traverse nodes                          |
| **Insertion/Deletion (middle of list)** | Slow (O(n)) — requires shifting elements   | Fast (O(1)) — just change pointers                         |
| **Insertion at end**                    | Amortized O(1)                             | O(1)                                                       |
| **Memory usage**                        | Less (stores only values)                  | More (stores values + two references per node: prev, next) |
| **Iteration**                           | Faster (cache-friendly)                    | Slower (cache misses due to scattered memory)              |
| **Best for**                            | Read-heavy operations, random access       | Write-heavy operations, frequent inserts/deletes in middle |

1. Use ArrayList when:-  
  a. You need fast random access (get/set by index).  
  b. Your application is read-heavy (lots of lookups, fewer insertions/deletions).  
  c. Example: Storing employee records where you mostly search by index.

```java
List<String> list = new ArrayList<>();
list.add("A");
list.add("B");
System.out.println(list.get(1)); // Fast O(1)
```

2. Use LinkedList when:-  
  a. You need fast insertions and deletions in the middle or beginning of the list.  
  b. You don’t need frequent random access.  
  c. Example: Implementing a queue or playlist where elements are frequently added/removed.

```java
List<String> list = new LinkedList<>();
list.add("A");
list.add("B");
list.add(1, "C"); // Insert in middle is fast O(1)
```

3. Rule of Thumb  
a. ArrayList → Better for searching and accessing elements.  
b. LinkedList → Better for frequent insertions/deletions.
<br />
<br />
<br />



# 6. Difference between HashMap and ConcurrentHashMap in Java?

| Feature                      | **HashMap**                                                                                              | **ConcurrentHashMap**                                                                                               |
| ---------------------------- | -------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------- |
| **Thread-safety**            | Not thread-safe (can cause data inconsistency if used by multiple threads)                               | Thread-safe (designed for concurrent access)                                                                        |
| **Null Keys/Values**         | Allows **one `null` key** and multiple `null` values                                                     | Does **not allow null keys or null values**                                                                         |
| **Performance**              | Faster in single-threaded applications                                                                   | Slightly slower due to synchronization overhead, but much faster than using `Collections.synchronizedMap()`         |
| **Fail-fast behavior**       | Iterator is **fail-fast** → throws `ConcurrentModificationException` if map is modified during iteration | Iterator is **fail-safe** → does **not throw exception**, it works on a snapshot of the data                        |
| **Internal Synchronization** | No synchronization, multiple threads can corrupt data                                                    | Uses **segment-level (Java 7)** or **bucket-level (Java 8)** locking → multiple threads can read/write concurrently |
| **Use case**                 | Good for single-threaded or read-mostly scenarios                                                        | Best for multi-threaded environments where many threads read/write simultaneously                                   |

```java
import java.util.*;

public class HashMapExample {
    public static void main(String[] args) {
        Map<Integer, String> map = new HashMap<>();
        map.put(1, "A");
        map.put(2, "B");
        map.put(null, "C"); // allows one null key
        map.put(3, null);   // allows null value
        System.out.println(map);
    }
}
```
```java
import java.util.concurrent.*;

public class ConcurrentHashMapExample {
    public static void main(String[] args) {
        ConcurrentHashMap<Integer, String> cmap = new ConcurrentHashMap<>();
        cmap.put(1, "A");
        cmap.put(2, "B");
        // cmap.put(null, "C"); // Not allowed, throws NullPointerException
        // cmap.put(3, null);   // Not allowed
        System.out.println(cmap);
    }
}
```
Summary: -  
  a. Use HashMap → when working in single-threaded apps (or if external synchronization is added manually).  
  b. Use ConcurrentHashMap → when working in multi-threaded environments, to avoid ConcurrentModificationException and data corruption.  

<br />
<br />
<br />



# 7. Explain internal working of HashMap
1. Data Structure Used. In Java 8+, HashMap is implemented using:-  
  a. Array of Nodes (buckets)  
  b. Each Node stores: key, value, hash, next (link to next node) 
  c. If many collisions happen → LinkedList turns into a Balanced Tree (Red-Black Tree) for faster lookups.  

So structure is:
```java
HashMap = Array[Node]
Node = {hash, key, value, next}
```

2. How Put Works (map.put(key, value))  

  | Step | Col 1 | Col 2 |
  |------|-------|-------|
  | Hashing | Compute hash = hashCode(key) | |
  | | Apply internal hash function to reduce collisions. | |
  | Index Calculation | Index = (n - 1) & hash → where n = array size. | |
  | | This ensures values fall within bucket range. | |
  | Insert Logic | If bucket is empty → | create new Node and insert. |
  | | If bucket already has nodes → | |
  | | Compare key with existing: | |
  | | | If same key → overwrite value. |
  | | | If different key but same bucket → collision! |
  | | | Add new node at end of LinkedList (Java 7) / front (Java 8). |
  | | | If chain length > 8 → convert to Red-Black Tree for O(log n) search. |

3. How Get Works (map.get(key))

  | Step | Col 1 |
  |------|-------|
  | Compute hash of key → | find bucket index. |
  | Traverse bucket: | If only one node → return value. |
  | | If multiple nodes (LinkedList/Tree) → compare keys using equals(). |
  | Return value if found, else null. | |

4. Handling Collisions  
  a. Collisions happen when different keys produce the same bucket index.  
  b. Handled using chaining (LinkedList → Red-Black Tree if too many).  

5. Rehashing (Resize)  
  a. Default capacity = 16, Load Factor = 0.75.  
  b. Threshold = capacity × load factor = 16 × 0.75 = 12.  
  c. When size > threshold → resize array (double the size).  
  d. All existing keys are rehashed into new buckets.  

6. Null Keys/Values  
  a. HashMap allows one null key (always stored at bucket index 0).  
  b. Allows multiple null values.  

7. Example
```java
HashMap<String, Integer> map = new HashMap<>();
map.put("John", 25);
map.put("Alex", 30);
map.put("John", 28); // overwrites value
```

  a. "John".hashCode() → hashed → bucket index → insert Node(John, 25).  
  b. "Alex".hashCode() → different index → insert Node(Alex, 30).  
  c. "John" again → hash matches → same bucket → key equals → overwrite value with 28.  

8. Performance  
  a. Best case: O(1) for put/get.  
  b. Worst case: O(log n) if tree, O(n) if LinkedList.  
  c. With good hash function → near O(1) average.  

9. Summary  
  a. HashMap uses array + linked list/tree.  
  b. Uses hashing to find bucket.  
  c. Handles collisions with chaining (LinkedList → Tree).  
  d. Expands via rehashing when load factor exceeds threshold.  

<br />
<br />
<br />




# 8. Internal Differences: HashMap in Java 7 vs Java 8

| Term | Java 7 | Java 8 |
|------|--------|--------|
| Collision Handling | Collisions handled using a LinkedList in each bucket. | Initially uses LinkedList, but if a bucket’s size > 8 and table size ≥ 64 → converts that bucket into a Red-Black Tree. |
| | Worst-case lookup = O(n) (if all keys fall into one bucket). | This improves worst-case lookup to O(log n) instead of O(n). |
| Insertion Order (When Collisions Occur) | New node is inserted at the head of the LinkedList. | New node is inserted at the end of the list (like a queue). |
| | Problem: During rehashing, it could lead to infinite loops in multithreaded code (because rehashing wasn’t thread-safe).[1] | Prevents reverse order issue and avoids infinite loop risk. |
| Hashing Function | Used simple hash function → higher chance of collisions if hashCodes are poorly distributed. | Uses improved hash function (hash ^ (hash >>> 16)) to spread high bits into low bits. Reduces collision probability. |
| Performance (Worst Case) | Worst case = O(n) (if many collisions form a long linked list). | Worst case = O(log n) (when linked list converts to tree). |

[1] In Java 7 HashMap, rehashing during resize was not thread-safe. If multiple threads resized at the same time, the LinkedList in buckets could become corrupted and form a circular reference, leading to an infinite loop during traversal.
This happened because Java 7 used head insertion during rehashing. In Java 8, this was fixed by using tail insertion and introducing better collision handling (LinkedList → Tree), so infinite loops don’t occur anymore.

<br />
<br />
<br />




# 9. Explain internal working of ConcurrentHashMap

1. Purpose  
  a. Unlike HashMap (not thread-safe) and Hashtable (thread-safe but locks the whole map = slow).  
  b. ConcurrentHashMap provides a thread-safe, high-performance map by allowing concurrent reads and writes with minimal locking.  

2. Internal Design (Java 7 vs Java 8)  

  - a. Java 7 (Segmented Locking)  
    - 1. Internally divided into segments (default = 16).  
    - 2. Each segment is like a smaller HashMap with its own lock.  
    - 3. Multiple threads can write to different segments concurrently.  
    - 4. Reads are mostly lock-free.  
    - 5. Limitation: Fixed 16 segments → scalability issue when very high concurrency.  
  
  b. Java 8 (Bucket-level Locking + CAS)  
    1. Removed Segments → uses a single array of Nodes (like HashMap).  
    2. Uses CAS (Compare-And-Swap) and synchronized blocks on individual buckets instead of locking whole segments.  
    3. Collision handling: LinkedList → Red-Black Tree (just like HashMap 8).  
    4. Reads are almost always lock-free.  
    5. Writes use fine-grained locking at bucket level only.  
    6. Improvement: Much better parallelism and less memory usage than Java 7.  

3. Put Operation (put(key, value))  
  a. Compute hash of the key → find bucket index.  
  b. If bucket empty → insert new Node using CAS (no lock).  
  c. If bucket not empty → acquire lock on that bucket:  
    1. If key exists → update value.  
    2. If collision → add node to list/tree.  
    3. If list > threshold (8) → convert to tree.  
  d. Release lock.  
  e. If map size exceeds threshold → resize (done gradually, unlike HashMap).  

4. Get Operation (get(key))  
  a. Compute hash → find bucket index.  
  b. Traverse bucket (LinkedList or Tree).  
  c. Return value.  
  d. No lock needed for reads (volatile reads ensure visibility).  

5. Key Features  
  a. Thread-safe: Multiple threads can safely access/modify.  
  b. High Performance: Uses fine-grained locking and CAS instead of global lock.  
  c. Fail-safe Iterators: Iterator does not throw ConcurrentModificationException. Instead, it iterates over a snapshot of data (weakly consistent).  
  d. No nulls: Does not allow null keys or null values (to avoid ambiguity in multi-threaded access).  

6. Performance (Time Complexity)  
  a. Average: O(1) for put/get.  
  b. Worst-case: O(log n) (if collisions turn into trees).  

<br />
<br />
<br />




# 10. 

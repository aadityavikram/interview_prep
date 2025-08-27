# 1. Indexes in SQL, how will you create one and how does it work?

1. What is an Index in SQL?  
  a. An index in SQL is like the index in the back of a book. Instead of flipping through every page to find a word, you can jump directly to where it’s located.  
  b. Without an index: SQL scans the entire table (full table scan).  
  c. With an index: SQL uses a sorted data structure (usually a B-Tree or sometimes a Hash) to quickly locate rows.  
  d. Indexes improve read performance (SELECT queries) but can slow down writes (INSERT, UPDATE, DELETE) because the index also needs to be updated.  

2. How to Create an Index  
  a. Single-column index
```sql
CREATE INDEX idx_customer_name
ON Customers (CustomerName);
```

This will make queries filtering by CustomerName much faster, e.g.
```sql
SELECT * FROM Customers WHERE CustomerName = 'Aaditya';
```

  b. Composite (multi-column) index
```sql
CREATE INDEX idx_customer_city_country
ON Customers (City, Country);
```

Useful when queries often filter by City AND Country together.  
Order matters: This index will help queries filtering by (City) or (City, Country) but not by (Country) alone.  

  c. Unique index  
```sql
CREATE UNIQUE INDEX idx_unique_email
ON Users (Email);
```
Ensures all emails are unique, often used for constraints.  

  d. Index on expressions (some databases)  
```sql
CREATE INDEX idx_lower_email
ON Users (LOWER(Email));
```
Useful if queries often use functions:  
```sql
SELECT * FROM Users WHERE LOWER(Email) = 'test@gmail.com';
```

3. How It Works Internally  

  | Index | Feature |
  |-------|---------|
  | B-Tree Index (default in most RDBMS like MySQL, PostgreSQL, Oracle) | Stores data in a balanced tree structure. |
  | | Lookup, insert, and delete take O(log n) time. |
  | | Great for range queries (BETWEEN, <, >, ORDER BY). |
  | Hash Index (used in some cases, e.g., PostgreSQL) | Uses a hash table. |
  | | Very fast for equality lookups (=). |
  | | Not useful for range queries. |
  | Bitmap Index (Oracle, Data Warehousing) | Uses bitmaps for low-cardinality columns (e.g., Gender: Male/Female). |
  | | Very efficient for aggregations. |

4. Pros and Cons

  | Pros | Cons |
  |------|------|
  | Columns frequently used in WHERE, JOIN, ORDER BY, GROUP BY. | Columns with very few unique values (e.g., is_active with only 0/1). |
  | High-selectivity columns (many unique values, e.g., email, customer_id). | Tables that are mostly written to (heavy inserts/updates). |

<br />
<br />
<br />




# 2. Clustered vs Non-Clustered Indexes

1. Clustered Index  
  a. A clustered index determines the physical order of rows in a table (like the main arrangement of data).  
  c. b. Think of it like a dictionary: words are stored in alphabetical order, so the data itself is ordered.  
  d. Only one clustered index per table, because data can only be physically sorted one way.  
  e. In SQL Server, the Primary Key creates a clustered index by default (unless specified otherwise).  
  f. In MySQL (InnoDB engine), the Primary Key is always the clustered index.  
  - g. Best for columns that are:  
    - 1. Primary keys, unique identifiers  
    - 2. Frequently used in range queries (BETWEEN, <, >).  

```sql
CREATE TABLE my_table (
    id INT NOT NULL AUTO_INCREMENT,
    name VARCHAR(100),
    PRIMARY KEY (id)
);
```

2. Non-Clustered Index  
  a. A non-clustered index is like a separate lookup table that has pointers to the actual table rows.  
  b. Think of it like a book index at the end of a textbook: the index has keywords + page numbers.  
  c. You can create multiple non-clustered indexes on a table.  
  d. Doesn’t change the physical order of the data.  
  - e. Best for:  
    - 1. Columns used often in WHERE, JOIN, ORDER BY, GROUP BY.  
    - 2. Queries that don’t align with the clustered index column.

```sql
CREATE INDEX index_name ON table_name (column1, column2, ...);
```

<br />
<br />
<br />



# 3. Immutable classes in Java and how to implement them?

1. What is an Immutable Class?  
  a. An immutable class is a class whose objects cannot be modified after creation.  
  b. Once you create an object, its state (field values) cannot change.  
  c. Any modification results in a new object instead of changing the old one.  
  d. Example you already know: String in Java is immutable.  

2. Why do we need Immutable Classes?  
  a. Thread-safety → Multiple threads can use immutable objects without synchronization.  
  b. Caching → Since values never change, they can be safely cached.  
  c. Security → Prevents outside classes from corrupting internal state.  
  d. Simplicity → Predictable behavior (no accidental side effects).

3. How to Implement an Immutable Class in Java  
  a. Declare the class final → so it can’t be subclassed.  
  b. Make all fields private and final → so they can’t be modified after assignment.  
  c. Initialize fields only in the constructor.  
  d. No setters → provide only getters.  
  e. Return copies (deep copy) of mutable fields → so outside code can’t change them.  

4. Example: Immutable Class  
```java
import java.util.Date;
import java.util.List;
import java.util.ArrayList;

public final class Employee {
    
    private final String name;         // Immutable field (String is immutable)
    private final int id;              // Primitive → immutable
    private final Date joinDate;       // Mutable → must handle carefully
    private final List<String> skills; // Mutable → must handle carefully

    // Constructor initializes all fields
    public Employee(String name, int id, Date joinDate, List<String> skills) {
        this.name = name;
        this.id = id;
        // Defensive copy of mutable objects
        this.joinDate = new Date(joinDate.getTime()); 
        this.skills = new ArrayList<>(skills);
    }

    // Only getters (no setters)
    public String getName() {
        return name;
    }

    public int getId() {
        return id;
    }

    public Date getJoinDate() {
        // Return defensive copy
        return new Date(joinDate.getTime());
    }

    public List<String> getSkills() {
        // Return defensive copy
        return new ArrayList<>(skills);
    }
}
```

If we didn’t make defensive copies, someone could call:
```java
employee.getSkills().add("Hacking");
```
and change the object’s state indirectly — which breaks immutability.

5. Immutable classes in Java  
  a. All wrapper classes in Java (Integer, Double, etc.) are immutable.  
  b. String is the most famous immutable class.  

<br />
<br />
<br />




# 4. Immutability and Thread-Safety

1. Why threads cause problems normally?  
  a. When multiple threads share a mutable object:  
  b. One thread might read while another writes, leading to inconsistent or unpredictable results.  
  c. That’s why in multi-threaded code, we often need synchronization (synchronized, locks, volatile) to avoid race conditions.  
  d. Example (mutable object, unsafe):

```java
class Counter {
    private int count = 0;
    public void increment() {
        count++;   // Not atomic, not thread-safe
    }
    public int getCount() {
        return count;
    }
}
```
If two threads call increment() at the same time, the result may be wrong (lost updates).  

2. Why immutable objects are naturally thread-safe?  
  a. In immutable objects, state never changes after creation.  
  b. No matter how many threads reference the same object, they all see the same consistent state.  
  c. Since no thread can modify it, there’s no need for synchronization.  
  d. Example (immutable object, thread-safe):

```java
final class Point {
    private final int x;
    private final int y;

    public Point(int x, int y) {
        this.x = x;
        this.y = y;
    }

    public int getX() { return x; }
    public int getY() { return y; }
}
```
If 100 threads access the same Point object, they’ll always see the same x and y.  
No locks, no race conditions.  

3. Real-world Example: String in Java  
  a. String is immutable → you can safely share "Hello" across multiple threads.  
  b. If one thread does str.toUpperCase(), a new String is created.  
  c. The original "Hello" is unchanged for everyone else.  

4. But what if I need a "modified" object?  
  a. With immutables, instead of modifying, you create a new object.  
  b. Example:  
```java
Point p1 = new Point(10, 20);
Point p2 = new Point(p1.getX() + 5, p1.getY()); // New object with updated x
```
p1 is untouched, p2 is a new object. Both are thread-safe.  

5. Downsides of immutability (trade-offs)  
  a. Performance overhead: Creating new objects instead of modifying can be costly if done in large loops.  
  b. Memory usage: More objects in heap → more GC pressure.  
  c. That’s why frameworks like StringBuilder exist (mutable, for performance in concatenation).  

6. Summary
Immutable objects are inherently thread-safe because their state never changes after construction. Multiple threads can freely share them without synchronization. That’s why classes like String, Integer, and LocalDate are immutable.

<br />
<br />
<br />





# 5. Purpose of serialization in a PoJo, and in what scenarios is it commonly used?

1. What is Serialization in Java?  
  a. Serialization is the process of converting an object into a byte stream so it can be:  
  b. Stored (e.g., in a file, database, cache)  
  c. Transferred (e.g., over a network, between JVMs, or via APIs)  
  d. Later deserialized back into the same object.  
  e.  A class must implement java.io.Serializable to allow its objects to be serialized.  

2. Purpose of Serialization in a POJO  
- a. A POJO (Plain Old Java Object) typically just has fields + getters/setters.  
- b. When you mark it Serializable, it allows:    
  - 1. Persistence  
    - a. Save POJO state into a file or DB for later retrieval.  
    - b. Example: Saving user session objects.  
  - 2. Network communication  
    - a. Send objects over the network (client ↔ server, microservices, RMI).  
    - b. Example: A User object sent as a serialized stream from one JVM to another.  
  - 3. Caching  
    - a. Frameworks like Redis, Ehcache, Hazelcast store serialized POJOs in memory/disk.  
  - 4. Distributed systems  
    - a. In systems like Hadoop, Kafka, or messaging queues, objects are serialized to transfer between nodes.  
  - 5. Deep copy (sometimes)  
    - a. Serialize + deserialize to create a deep copy of an object.  

3. Example: Serializable POJO  
```java
import java.io.*;

class Employee implements Serializable {
    private int id;
    private String name;

    // constructor
    public Employee(int id, String name) {
        this.id = id;
        this.name = name;
    }

    // getters
    public int getId() { return id; }
    public String getName() { return name; }
}

public class TestSerialization {
    public static void main(String[] args) throws Exception {
        Employee emp = new Employee(101, "Aaditya");

        // Serialize to file
        ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream("employee.ser"));
        oos.writeObject(emp);
        oos.close();

        // Deserialize from file
        ObjectInputStream ois = new ObjectInputStream(new FileInputStream("employee.ser"));
        Employee deserializedEmp = (Employee) ois.readObject();
        ois.close();

        System.out.println("Deserialized Employee: " + deserializedEmp.getId() + " - " + deserializedEmp.getName());
    }
}
```

4. Common Scenarios Where POJO Serialization is Used  
  a. Storing sessions - Web apps (e.g., Tomcat, Spring Boot) serialize session objects when saving session state.  
  b. Messaging & Queues - Sending objects through JMS, Kafka, RabbitMQ, etc.  
  c. APIs & Web Services - POJOs are serialized into JSON/XML when communicating via REST/SOAP.  
  d. Frameworks - Hibernate, JPA, Spring often require POJOs to be serializable to store/retrieve objects across layers.

5. Summary  
Serialization in a POJO allows its state to be converted into a byte stream so it can be persisted or transferred across JVMs. It is commonly used in caching, session storage, network communication, APIs, distributed systems, and messaging.

<br />
<br />
<br />



# 6. Java built-in serialization (Serializable) vs custom serialization (Json, Gson)

1. Java’s Built-in Serialization (Serializable)  
  a. Uses java.io.Serializable interface.  
  b. Converts objects into a binary byte stream.  
  c. Can be written to a file, sent over a socket, or stored in DB.  
  d. Example:  
```java
ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream("emp.ser"));
oos.writeObject(emp);
```

2. Cons  
  a. JVM-specific → serialized objects may not be portable across JVM versions.  
  b. Slow & heavy (stores a lot of metadata about classes).  
  c. Fragile if the class changes (e.g., adding/removing fields breaks old serialized objects).  

3. Good for: Simple use cases like caching in same JVM, or small apps.  
4. Bad for: Large-scale distributed systems, cross-language communication.  

5. Custom Serialization Formats  

  | Format | Features | Pros | Cons |
  |--------|----------|------|------|
  | JSON (Text-based) | POJOs converted to JSON using libraries (Jackson, Gson). | Human-readable, cross-language, widely supported. | Bigger size (verbose), slower than binary formats. |
  | | Human-readable, language-independent. | | |
  | | | Widely used in REST APIs. | |

```java
ObjectMapper mapper = new ObjectMapper();
String json = mapper.writeValueAsString(emp);   // serialize
Employee emp2 = mapper.readValue(json, Employee.class); // deserialize
```

  | Format | Features | Pros | Cons |
  |--------|----------|------|------|
  | XML (Text-based) | Similar to JSON, but heavier and more verbose. | Standardized, human-readable. | Very verbose, slower. |
  | | Used in SOAP services, configuration files. | | |
  | Protocol Buffers (Protobuf, by Google) | Compact, binary format. | Very efficient, great for microservices. | Needs schema definition, not human-readable. |
  | | Requires .proto schema file. | | |
  | | Much faster and smaller than JSON/XML. | | |
  | | Cross-language (Java, C++, Python, Go, etc.). | | |
  | Apache Avro | Popular in Kafka, Hadoop, Big Data systems. | Compact, supports schema evolution (handles changes better than Java Serializable). | Requires schema management. |
  | | Similar to Protobuf, schema-based binary serialization. | | |
  | | Schema is stored separately or alongside data. | | |

| Feature             | Java Serializable   | JSON      | Protobuf / Avro         |
| ------------------- | ------------------- | --------- | ----------------------- |
| Format              | Binary (Java-only)  | Text      | Binary                  |
| Cross-language?     | No                  | Yes       | Yes                     |
| Readable by humans? | No                  | Yes       | No                      |
| Performance         | Slow                | Medium    | Very fast               |
| Size                | Large               | Medium    | Small                   |
| Schema required?    | No                  | No        | Yes                     |
| Common Use          | Small apps, caching | REST APIs | Microservices, Big Data |

6. Summary  
Java’s built-in serialization (Serializable) is easy but slow, JVM-specific, and fragile. In real-world distributed systems, JSON is used for REST APIs, and binary formats like Protobuf or Avro are preferred for high-performance, cross-language communication. That’s why frameworks like Kafka, gRPC, and Hadoop avoid Java’s native serialization.

<br />
<br />
<br />



# 7. Suppose you need to send a Java object over a network in a distributed application. What challenges might arise, and how would you handle them?

1. Java Serialization Issues  
  a. JVM-specific: Java’s default serialization works only between JVMs. If a consumer is in Python or Go, it won’t understand the byte stream.  
  b. Versioning problem: If class definition changes (e.g., new field added), old serialized objects might fail to deserialize (InvalidClassException).  
  c. Performance overhead: Java’s built-in serialization is slow and produces large payloads.  
  d. Security risks: Deserialization can be exploited (injection attacks if untrusted data is deserialized).  

2. Network-related Challenges  
  a. Bandwidth & latency: Large object payloads increase transfer time.  
  b. Endianness / encoding issues: Different platforms may interpret raw bytes differently.  
  c. Partial failure: What if the object isn’t fully transferred (e.g., due to network drop)?  

3. Cross-language Compatibility  
  a. In distributed systems, services might not all be Java. A Python or Node.js service won’t understand raw Serializable objects.  

4. Schema Evolution  
  a. If your object changes (new fields, renamed fields), old clients may not understand the new format unless you use a serialization mechanism that supports schema evolution (like Avro or Protobuf).  

5. How to Handle These Challenges:-  

6. Use a Portable Serialization Format  
  a. Instead of Java’s default Serializable:  
  b. Use JSON (via Jackson/Gson) → easy, human-readable, cross-language.  
  c. Use Protobuf / Avro / Thrift → compact binary, schema-based, great for high-performance distributed systems.  
  d. Example with JSON (Jackson):  
```java
ObjectMapper mapper = new ObjectMapper();
String json = mapper.writeValueAsString(myObject);   // Serialize to JSON
MyClass obj = mapper.readValue(json, MyClass.class); // Deserialize back
```
Example with Protobuf:  
```java
// myobject.proto
message MyObject {
  int32 id = 1;
  string name = 2;
}
```
Compile .proto → generates Java class → send over network as binary.

7. Handle Versioning (Schema Evolution)  
  a. JSON → extra fields can usually be ignored.  
  b. Protobuf / Avro → designed to handle field addition/removal without breaking compatibility.  

8. Optimize for Performance  
  a. Use binary formats (Protobuf, Avro) if performance & bandwidth are critical.  
  b. Compress large payloads (e.g., GZIP JSON).  

9. Ensure Security  
  a. Never deserialize untrusted Java byte streams directly (Serializable → RCE risk).  
  b. Validate payloads before deserialization.  
  c. Prefer safe libraries like Jackson, Protobuf, Avro.  

10. Networking Concerns  
  a. Use reliable protocols (e.g., gRPC on HTTP/2, or REST over HTTP/1.1).  
  b. Add retries and timeout handling.  
  c. Ensure objects are small enough (avoid sending huge graphs → break them down).  

11. Summary  
The main challenges are portability, performance, versioning, and security. Java’s built-in serialization is slow, JVM-specific, and insecure, so avoid it. Instead, use JSON for human-readable APIs or a binary format like Protobuf/Avro for efficiency and cross-language support. Design for schema evolution, use compression if needed, and ensure proper error handling and security when deserializing.

<br />
<br />
<br />



# 8. What is a Singleton pattern and how can you implement it in Java?

1. What is the Singleton Pattern?  
  a. A Singleton ensures that only one instance of a class is created in the JVM.  
  b. It also provides a global access point to that instance.  
  - c. Think of it like:  
    - 1. OS Kernel → only one running kernel.  
    - 2. Configuration Manager / Logger → only one global object used by all parts of the app.  

2. When to Use Singleton  
  a. Logging (one global logger).  
  b. Configuration settings (app-wide config).  
  c. Thread pools / Connection pools (shared resource managers).  
  d. Caching (only one cache instance).  

3.  Ways to Implement Singleton in Java  
a. Eager Initialization  
Instance created at class loading time.  
```java
class Singleton {
    private static final Singleton instance = new Singleton(); // created eagerly

    private Singleton() {}   // private constructor

    public static Singleton getInstance() {
        return instance;
    }
}
```
Simple, thread-safe.  
Instance is created even if never used (wasteful in some cases).  

b. Lazy Initialization (Not Thread-Safe)  
Created only when needed.  
```java
class Singleton {
    private static Singleton instance;

    private Singleton() {}

    public static Singleton getInstance() {
        if (instance == null) {   // first time → create
            instance = new Singleton();
        }
        return instance;
    }
}
```
Saves memory (object created only when needed).  
Not thread-safe (two threads could create two instances).  

c. Thread-Safe Singleton (Synchronized Method)
```java
class Singleton {
    private static Singleton instance;

    private Singleton() {}

    public static synchronized Singleton getInstance() {
        if (instance == null) {
            instance = new Singleton();
        }
        return instance;
    }
}
```
Thread-safe.  
synchronized makes it slow (every call locked).  

d. Double-Checked Locking (Best Practice)  
```java
class Singleton {
    private static volatile Singleton instance;

    private Singleton() {}

    public static Singleton getInstance() {
        if (instance == null) {             // first check
            synchronized (Singleton.class) {
                if (instance == null) {     // second check
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}
```
Efficient and thread-safe.  
volatile ensures changes are visible across threads.  
More complex code.  

e. Bill Pugh Singleton (Inner Static Class) – Best Approach  
```java
class Singleton {
    private Singleton() {}

    private static class Holder {
        private static final Singleton instance = new Singleton();
    }

    public static Singleton getInstance() {
        return Holder.instance;
    }
}
```
Thread-safe without synchronization.  
Lazy-loaded (instance created only when needed).  
Clean & simple.  

f. Enum Singleton (Recommended by Joshua Bloch)
```java
enum Singleton {
    INSTANCE;
    
    public void showMessage() {
        System.out.println("Hello from Singleton");
    }
}
```
Easiest, thread-safe, prevents reflection & serialization issues.  
Slightly less flexible (cannot extend other classes).  

4. Summary  
Singleton ensures only one instance of a class is created. We can implement it using eager initialization, lazy initialization, synchronized methods, double-checked locking, static inner class, or enum. The best approaches are the Bill Pugh inner static class or Enum Singleton, since they are thread-safe, lazy, and handle serialization/reflection issues.

<br />
<br />
<br />



# 9. How Singleton Can Be Broken (and Fixes)

1. Breaking Singleton with Reflection  
Reflection can access the private constructor and create another instance.  
```java
import java.lang.reflect.Constructor;

class Singleton {
    private static final Singleton instance = new Singleton();
    private Singleton() {}
    public static Singleton getInstance() {
        return instance;
    }
}

public class TestReflection {
    public static void main(String[] args) throws Exception {
        Singleton s1 = Singleton.getInstance();

        Constructor<Singleton> constructor = Singleton.class.getDeclaredConstructor();
        constructor.setAccessible(true);  // bypass private constructor
        Singleton s2 = constructor.newInstance();

        System.out.println(s1 == s2); // false → two instances created!
    }
}
```
Fix: Throw exception if constructor is called twice.
```java
private Singleton() {
    if (instance != null) {
        throw new RuntimeException("Use getInstance()");
    }
}
```

2. Breaking Singleton with Serialization  
Serialization followed by deserialization creates a new instance.   
```java
import java.io.*;

class Singleton implements Serializable {
    private static final Singleton instance = new Singleton();
    private Singleton() {}
    public static Singleton getInstance() { return instance; }
}

public class TestSerialization {
    public static void main(String[] args) throws Exception {
        Singleton s1 = Singleton.getInstance();

        // Serialize
        ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream("singleton.ser"));
        oos.writeObject(s1);
        oos.close();

        // Deserialize
        ObjectInputStream ois = new ObjectInputStream(new FileInputStream("singleton.ser"));
        Singleton s2 = (Singleton) ois.readObject();

        System.out.println(s1 == s2); // false → two instances created!
    }
}
```
Fix: Implement readResolve() method.
```java
protected Object readResolve() {
    return instance;
}
```

3. Breaking Singleton with Cloning  
If Singleton implements Cloneable, clone() can create another object.   
```java
class Singleton implements Cloneable {
    private static final Singleton instance = new Singleton();
    private Singleton() {}
    public static Singleton getInstance() { return instance; }

    @Override
    protected Object clone() throws CloneNotSupportedException {
        return super.clone();  // creates new instance
    }
}
```
Fix: Override clone() to prevent it.
```java
@Override
protected Object clone() throws CloneNotSupportedException {
    return instance;  // return same instance
}
```

4. Breaking Singleton with Multiple ClassLoaders   
In enterprise apps, different classloaders may load the same Singleton class separately → multiple instances.   
Fix: Use enum Singleton or carefully manage classloaders.   
```java
enum Singleton {
    INSTANCE;
}
```
5. Automatically prevents:  
  a. Reflection (enum constructors can’t be accessed via reflection).  
  b. Serialization (handled by JVM).  
  c. Cloning (not supported for enums).  
  d. That’s why Joshua Bloch (Effective Java) recommends Enum Singleton.  

6. Summary  
Singleton can be broken by Reflection, Serialization, Cloning, or multiple ClassLoaders. To fix these, we add guards in the constructor, use readResolve() in serialization, override clone(), or use Enum Singleton which naturally handles these issues.

<br />
<br />
<br />





# 10. What is the hashCode-equals contract? Why is it important?

1. hashCode–equals Contract  
  a. Java defines a contract between the equals() and hashCode() methods:  
  b. If two objects are equal (equals() returns true), they must have the same hashCode().  
  c. Otherwise, HashMap / HashSet may put them in different buckets → duplicates appear.  
  d. If two objects have the same hashCode(), they may or may not be equal.  
  e. Collisions can happen (different objects, same hash).  
  f. So equality must always be confirmed with equals().  

2. Why is it Important?  
  a. Collections like HashMap, HashSet, Hashtable:  
  b. Use hashCode() to decide bucket placement (where to store/retrieve).  
  c. Use equals() to check if two keys are the same inside that bucket.  
  d. If you break the contract, collections won’t work correctly.  
  e. Example: Breaking the Contract  

```java
class Person {
    String name;

    Person(String name) {
        this.name = name;
    }

    @Override
    public boolean equals(Object obj) {
        if (this == obj) return true;
        if (!(obj instanceof Person)) return false;
        Person other = (Person) obj;
        return name.equals(other.name);
    }
    // forgot to override hashCode()
}
```
```java
Usage:

Person p1 = new Person("Aaditya");
Person p2 = new Person("Aaditya");

HashSet<Person> set = new HashSet<>();
set.add(p1);
set.add(p2);

System.out.println(set.size()); // 2, Expected 1
```

equals() says they’re equal, but default hashCode() (from Object) is different.  

Correct Implementation  
```java
class Person {
    String name;

    Person(String name) {
        this.name = name;
    }

    @Override
    public boolean equals(Object obj) {
        if (this == obj) return true;
        if (!(obj instanceof Person)) return false;
        Person other = (Person) obj;
        return name.equals(other.name);
    }

    @Override
    public int hashCode() {
        return name.hashCode(); // consistent with equals
    }
}
```
Now:
```java
HashSet<Person> set = new HashSet<>();
set.add(new Person("Aaditya"));
set.add(new Person("Aaditya"));
System.out.println(set.size()); // 1
```

3. Rules of the Contract (as per Java spec)  
  a. If a.equals(b), then a.hashCode() == b.hashCode().  
  b. If a.hashCode() == b.hashCode(), a.equals(b) may or may not be true.  
  c. hashCode() must consistently return the same value during the lifetime of the object (unless fields used in equals/hashCode change).  

4. Interview-Ready Summary  
The hashCode–equals contract ensures correct behavior of hash-based collections. If two objects are equal, they must have the same hashCode. Otherwise, collections like HashSet or HashMap may store duplicates or fail to find keys. That’s why whenever we override equals(), we must also override hashCode() consistently.

<br />
<br />
<br />



# 11. If you override equals() but do not override hashCode(), what could go wrong when storing objects in a HashMap?

`. Suppose you override equals() but forget to override hashCode().  
```java
class Person {
    String name;

    Person(String name) { this.name = name; }

    @Override
    public boolean equals(Object obj) {
        if (this == obj) return true;
        if (!(obj instanceof Person)) return false;
        Person other = (Person) obj;
        return this.name.equals(other.name);
    }
    // hashCode not overridden
}
```

2. What Happens in a HashMap  
When inserting:  

```java
map.put(new Person("Aaditya"), "First");
map.put(new Person("Aaditya"), "Second");
```

3. Both objects are equal (equals() returns true).  
4. But since hashCode() is not overridden, they inherit from Object.  
5. Default hashCode() → based on memory address, so two different objects get different hash codes.  
6. HashMap puts them in different buckets.  
7. Result:  
  a. HashMap thinks they are two different keys.  
  b. You end up with two entries instead of updating the value.  

```java
System.out.println(map.size()); 
// Output: 2, Expected: 1
```

8. Another Problem: Retrieval
```java
Person p1 = new Person("Aaditya");
map.put(p1, "Engineer");

// Later
Person p2 = new Person("Aaditya");
System.out.println(map.get(p2)); // null
```

Even though p1.equals(p2) is true,  
Their hashCode() is different → HashMap looks in the wrong bucket → fails to find the entry.  

9. Summary  
If we override equals() but not hashCode(), objects that are equal may have different hash codes. In a HashMap, such objects end up in different buckets, causing duplicate entries and lookup failures. This breaks the integrity of the collection — you won’t be able to retrieve values reliably. That’s why the hashCode–equals contract requires overriding both methods together.

<br />
<br />
<br />



# 12. If you override hashCode() but do not override equals() , what could go wrong when storing objects in a HashMap?

1. Suppose you override hashCode() but NOT equals().  
```java
class Person {
    String name;

    Person(String name) { this.name = name; }

    @Override
    public int hashCode() {
        return name.hashCode(); // consistent hash
    }
    // equals not overridden → uses Object.equals (reference equality)
}
```

2. What Happens in a HashMap  
When inserting:  
```java
Map<Person, String> map = new HashMap<>();
map.put(new Person("Aaditya"), "First");
map.put(new Person("Aaditya"), "Second");
```
3. Both objects produce the same hashCode (since name is same).  
4. HashMap puts them in the same bucket.  
5. BUT: HashMap then checks equals() to see if keys are equal.  
6. Since we didn’t override equals(), it uses Object.equals() → compares memory addresses.  
7. Result: they are treated as different keys, so both entries exist in the same bucket.  
```java
System.out.println(map.size()); 
// Output: 2, Expected: 1
```

8. Retrieval Problem  
```java
Person p1 = new Person("Aaditya");
map.put(p1, "Engineer");

// Later
Person p2 = new Person("Aaditya");
System.out.println(map.get(p2)); // null
```

9. HashMap finds the correct bucket (hashCode matches).  
10. But inside the bucket, it uses equals() to find the key.  
11. Since equals() is not overridden, p1.equals(p2) is false (different objects).  
12. Result: lookup fails.  

13. Summary  
  - a. If you override equals() but not hashCode() →  
    - 1. Equal objects may end up in different buckets.  
    - 2. Leads to duplicates and lookup failures.  
  - b. If you override hashCode() but not equals() →  
    - 1. Objects with the same hash go to the same bucket.  
    - 2. But equals() fails (default checks reference).  
    - 3. Leads to duplicates in the same bucket and lookup failures.  

<br />
<br />
<br />




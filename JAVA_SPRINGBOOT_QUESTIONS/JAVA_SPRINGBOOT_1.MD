# 1. How to handle multiple beans in Springboot?
1. Use @Primary - Mark one of the beans as the default bean for autowiring

``` java
@Configuration
public class AppConfig {

    @Bean
    @Primary
    public PaymentService creditCardPaymentService() {
        return new CreditCardPaymentService();
    }

    @Bean
    public PaymentService paypalPaymentService() {
        return new PaypalPaymentService();
    }
}
```
2. Use @Qualifier - If you want to choose explicitly which bean to inject
```java
@Service
public class CheckoutService {

    private final PaymentService paymentService;

    public CheckoutService(@Qualifier("paypalPaymentService") PaymentService paymentService) {
        this.paymentService = paymentService;
    }
}
```
3. Inject All Beans as a List or Map - If you need all beans of a type, you can inject them as a collection
```java
@Service
public class PaymentProcessor {

    private final List<PaymentService> paymentServices;

    public PaymentProcessor(List<PaymentService> paymentServices) {
        this.paymentServices = paymentServices;
    }

    public void processAllPayments() {
        paymentServices.forEach(PaymentService::pay);
    }
}
```
4. Use @Profile (Conditional Beans) - If you want to activate a specific bean based on environment
```java
@Configuration
public class PaymentConfig {

    @Bean
    @Profile("dev")
    public PaymentService devPaymentService() {
        return new MockPaymentService();
    }

    @Bean
    @Profile("prod")
    public PaymentService prodPaymentService() {
        return new RealPaymentService();
    }
}
```
5. Use @Conditional (Custom Conditions) - For more complex scenarios, you can use Spring’s @Conditional annotations to create beans only under certain conditions (like property values, class availability, etc.)
```java
@Configuration
    @Conditional(MyCustomCondition.class)
    public class MyConditionalConfig {
        // ... bean definitions
    }
```
```java
public class MyCustomCondition implements Condition {
        @Override
        public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
            // Implement your custom logic here
            return true; // or false based on the condition
        }
    }
```
<br />
<br />
<br />




# 2. How to define custom annotation in Springboot?
1. Basic Custom Annotation - A simple annotation is defined with @interface
```java
import java.lang.annotation.*;

@Target(ElementType.METHOD)   // where this annotation can be applied (method, class, field, etc.)
@Retention(RetentionPolicy.RUNTIME) // available at runtime for reflection
public @interface LogExecutionTime {
}
```
@Target → Defines where you can use it (class, method, field, parameter, etc.).

@Retention → Defines how long it’s kept (runtime means Spring can read it at runtime).

2. Using Custom Annotation with Spring AOP - Let’s say you want to log how long a method takes whenever it’s annotated with @LogExecutionTime.
    a. Step 1: Define the annotation (Already shown above)
    b. Step 2: Create an Aspect
```java
import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.springframework.stereotype.Component;

@Aspect
@Component
public class LoggingAspect {

    @Around("@annotation(LogExecutionTime)")
    public Object logExecutionTime(ProceedingJoinPoint joinPoint) throws Throwable {
        long start = System.currentTimeMillis();

        Object proceed = joinPoint.proceed();

        long executionTime = System.currentTimeMillis() - start;
        System.out.println(joinPoint.getSignature() + " executed in " + executionTime + "ms");
        return proceed;
    }
}
```
c. Step 3: Use the annotation
```java
@Service
public class MyService {

    @LogExecutionTime
    public void serve() throws InterruptedException {
        Thread.sleep(1000);
    }
}
```
    Whenever you call myService.serve(), it will print execution time.
3. Custom Annotation with Attributes - You can add parameters to annotations
```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface RoleCheck {
    String value();   // required attribute
}
```
Usage
```java
@RoleCheck("ADMIN")
public void secureMethod() { ... }
```
Aspect can fetch that value:
```java
@Around("@annotation(roleCheck)")
public Object checkRole(ProceedingJoinPoint joinPoint, RoleCheck roleCheck) throws Throwable {
    String role = roleCheck.value();
    // check if user has this role
    return joinPoint.proceed();
}
```
4. Meta-Annotations (Composed Annotations) - You can combine multiple annotations into one custom annotation.
```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@RestController
@RequestMapping("/api")
public @interface ApiController {
}
```
```java
@ApiController
public class UserController { ... }
```
<br />
<br />
<br />




# 3. Caching in Springboot
1. Enable Caching - First, enable caching in your Spring Boot application
```java
@SpringBootApplication
@EnableCaching
public class MyApplication {
    public static void main(String[] args) {
        SpringApplication.run(MyApplication.class, args);
    }
}
```
2. Use Cache Annotations - @Cacheable - Stores the result of a method call in cache.
```java
@Service
public class BookService {

    @Cacheable("books")   // cache name = "books"
    public Book getBookByIsbn(String isbn) {
        simulateSlowService(); // expensive DB/API call
        return new Book(isbn, "Some Book");
    }

    private void simulateSlowService() {
        try { Thread.sleep(3000L); } catch (InterruptedException e) { e.printStackTrace(); }
    }
}
```
3. @CachePut - Always executes the method and updates the cache with the new value.
```java
@CachePut(value = "books", key = "#book.isbn")
public Book updateBook(Book book) {
    return bookRepository.save(book);
}
```
4. @CacheEvict - Removes an entry from cache.
```java
@CacheEvict(value = "books", key = "#isbn")
public void deleteBook(String isbn) {
    bookRepository.deleteByIsbn(isbn);
}
```
5. Clear all entries
```java
@CacheEvict(value = "books", allEntries = true)
public void clearCache() { }
```
6. @Caching - Combine multiple caching rules.
```java
@Caching(
    put = { @CachePut(value = "books", key = "#book.isbn") },
    evict = { @CacheEvict(value = "bookList", allEntries = true) }
)
public Book saveBook(Book book) {
    return bookRepository.save(book);
}
```
7. Supported Cache Providers: -  
    a. Simple (default) → Uses a ConcurrentHashMap (in-memory, good for testing)  
    b. Ehcache  
    c. Caffeine (modern, high-performance in-memory cache)  
    d. Hazelcast  
    e. Redis (distributed cache, best for scaling)  
    f. JCache (JSR-107)  
```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```
```java
spring.cache.type=redis
spring.redis.host=localhost
spring.redis.port=6379
```
8. Custom Cache Key - By default, Spring uses method parameters as keys. You can define custom keys with SpEL
```java
@Cacheable(value = "books", key = "#isbn.concat('-').concat(#locale)")
public Book getBookByIsbn(String isbn, String locale) { ... }
```
<br />
<br />
<br />



# 4. How does inter microservice communication happen?
1. In a microservices architecture, multiple services run independently, often on different servers/containers. For the system to work, these services need to communicate with each other.  
2. Synchronous Communication (Direct, Real-time) - One service calls another and waits for a response.  
    a. Common ways: -  
    | REST (HTTP) | |
    | gRPC (HTTP/2, binary protocol) | Faster, strongly typed, supports streaming. |
    | | Good for high-performance internal communication. |
        1. REST (HTTP)  
        2. gRPC (HTTP/2, binary protocol)  
            a. Faster, strongly typed, supports streaming.  
            b. Good for high-performance internal communication.  
        3. GraphQL federation  
            a. Services expose GraphQL APIs, aggregated by a gateway.  
    b. Pros: Simple, widely understood, real-time.  
    c. Cons: Tight coupling, network latency, failures propagate (if B is down, A fails).  
3. Asynchronous Communication (Event-driven, Decoupled) - One service sends a message/event and doesn’t wait for a response.
    a. Common ways: -  
        1. Message Brokers (Pub/Sub or Queue) - Kafka, RabbitMQ, ActiveMQ, AWS SQS, etc.  
        2. Event Bus / Event Streaming  
            a. Services publish "domain events" (like OrderCreated) that others can subscribe to.  
            b. Encourages event-driven architecture.  
    b. Pros: Loosely coupled, resilient, scalable.  
    c. Cons: Harder debugging, eventual consistency, added infra (Kafka, RabbitMQ).  
4. Service Discovery + API Gateway - In microservices, service locations change dynamically (containers restart, scaling).  
    a. Service Discovery (Eureka, Consul, Zookeeper, Kubernetes Service Registry) helps services find each other.  
    b. API Gateway (Spring Cloud Gateway, Nginx, Kong, Zuul) acts as a single entry point → routes requests to the right service, adds security, rate limiting, logging, etc.  
<br />
<br />
<br />



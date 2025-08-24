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

    | Col 1 | Col 2 |
    |-----|---------|
    | REST (HTTP) | Most common approach. |
    | | Uses @RestController in Spring Boot. |
    | gRPC (HTTP/2, binary protocol) | Faster, strongly typed, supports streaming. |
    | | Good for high-performance internal communication. |
    | GraphQL federation | Services expose GraphQL APIs, aggregated by a gateway. |
    | Pros | Simple, widely understood, real-time. |
    | Cons | Tight coupling, network latency, failures propagate (if B is down, A fails). |

3. Asynchronous Communication (Event-driven, Decoupled) - One service sends a message/event and doesn’t wait for a response.

    | Col 1 | Col 2 |
    |-----|---------|
    | Message Brokers (Pub/Sub or Queue) | Kafka, RabbitMQ, ActiveMQ, AWS SQS, etc. |
    | | Service A publishes event → Broker → Service B consumes. |
    | Event Bus / Event Streaming | Services publish "domain events" (like OrderCreated) that others can subscribe to. |
    | | Encourages event-driven architecture. |
    | Pros | Loosely coupled, resilient, scalable. |
    | Cons | Harder debugging, eventual consistency, added infra (Kafka, RabbitMQ). |

4. Service Discovery + API Gateway - In microservices, service locations change dynamically (containers restart, scaling).  

    | Col 1 | Col 2 |
    |-----|---------|
    | Service Discovery | Eureka, Consul, Zookeeper, Kubernetes Service Registry |
    | | Helps services find each other. |
    | API Gateway | Spring Cloud Gateway, Nginx, Kong, Zuul |
    | | Acts as a single entry point |
    | | Routes requests to the right service, adds security, rate limiting, logging, etc. |

5. Hybrid Approach

    a. Use synchronous REST/gRPC for query-style operations.  
    b. Use asynchronous messaging (Kafka/RabbitMQ) for events and decoupling.  
    c. Use API Gateway + Service Discovery for routing and load balancing.  

<br />
<br />
<br />


# 5. How can we customize automatic configuration in Springboot?
1. Override Beans with @Bean - If Spring Boot auto-configures a bean, and you define your own bean of the same type/name, Spring will use yours instead.
```java
@Configuration
public class MyDataSourceConfig {

    @Bean
    public DataSource dataSource() {
        return DataSourceBuilder.create()
                .url("jdbc:mysql://localhost:3306/mydb")
                .username("user")
                .password("pass")
                .build();
    }
}
```
2. Use application.properties / application.yml - Most auto-configurations are controlled via Spring Boot properties.
Example: Instead of writing your own DataSource bean, you can set properties:  
```java
spring.datasource.url=jdbc:mysql://localhost:3306/mydb
spring.datasource.username=user
spring.datasource.password=pass
spring.datasource.hikari.maximum-pool-size=10
```
3. Exclude Auto-Configurations - You can disable specific auto-configurations if you don’t want them.
```java
@SpringBootApplication(exclude = { DataSourceAutoConfiguration.class })
public class MyApp { ... }
```
```java
spring.autoconfigure.exclude=org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration
```
4. Conditional Auto-Configuration with @Conditional - You can create your own auto-configuration class that only applies under certain conditions.
```java
@Configuration
@ConditionalOnProperty(name = "featureX.enabled", havingValue = "true")
public class FeatureXConfig {

    @Bean
    public FeatureXService featureXService() {
        return new FeatureXService();
    }
}
```
This bean will only load if featureX.enabled=true in properties.  
Other useful conditionals:  
    a. @ConditionalOnClass → Only if a class is on classpath.  
    b. @ConditionalOnMissingBean → Only if no bean of that type exists.  
    c. @ConditionalOnProperty → Only if a property is set.  

5. Create a Custom Starter (Custom Auto-Configuration) - For reusable libraries or internal frameworks, you can define your own auto-configuration.
```java
@Configuration
@ConditionalOnClass(MyService.class)
@EnableConfigurationProperties(MyProperties.class)
public class MyAutoConfiguration {

    @Bean
    @ConditionalOnMissingBean
    public MyService myService(MyProperties props) {
        return new MyService(props.getName(), props.getTimeout());
    }
}
```
```java
@ConfigurationProperties(prefix = "my.service")
public class MyProperties {
    private String name = "default";
    private int timeout = 30;
    // getters and setters
}
```
```java
com.example.config.MyAutoConfiguration
```
6. Use @EnableAutoConfiguration selectively
```java
@Configuration
@EnableAutoConfiguration(exclude = { DataSourceAutoConfiguration.class })
@ComponentScan
public class MyApp { }
```
<br />
<br />
<br />


# 6. Different scopes available in Springboot
1. Singleton (default)  
    a. One instance per Spring container.  
    b. Same bean is reused everywhere (@Autowired).  
    c. Default scope in Spring Boot.  
    d. Good for stateless services.  
```java
@Component
@Scope("singleton")
public class MySingletonService { }
```
2. Prototype  
    a. New instance every time the bean is requested.  
    b. Useful when you want a fresh object each time.  
    c. Spring does not manage the full lifecycle (e.g., no automatic destroy).  
```java
@Component
@Scope("prototype")
public class MyPrototypeService { }
```
3. Request (Web-aware scope)  
    a. One instance per HTTP request.  
    b. Only works in a web app.  
    c. Useful for storing request-specific data.  
```java
@Component
@Scope("request")
public class MyRequestScopedBean { }
```
4. Session (Web-aware scope)  
    a. One instance per HTTP session.  
    b. Same object reused across multiple requests from the same user session.  
    c. Useful for storing user session data (like a shopping cart).  
```java
@Component
@Scope("session")
public class MySessionScopedBean { }
```
5. Application (Web-aware scope)  
    a. One instance per ServletContext (per web app).  
    b. Shared across all sessions & requests.  
    c. Similar to a global singleton, but tied to the web context.  
```java
@Component
@Scope("application")
public class MyApplicationScopedBean { }
```
6. WebSocket (Web-aware scope)  
    a. One instance per WebSocket session.  
    b. Useful in WebSocket-based apps (chat, live notifications).  
```java
@Component
@Scope("websocket")
public class MyWebSocketBean { }
```
<br />
<br />
<br />




# 7. How to create custom scope in Springboot?
1. Implement Scope interface.  
2. Register it in CustomScopeConfigurer.  
3. Use @Scope("customScopeName").  
```java
import org.springframework.beans.factory.ObjectFactory;
import org.springframework.beans.factory.config.Scope;

import java.util.HashMap;
import java.util.Map;

public class ThreadScope implements Scope {

    private static final ThreadLocal<Map<String, Object>> threadScope =
            ThreadLocal.withInitial(HashMap::new);

    @Override
    public Object get(String name, ObjectFactory<?> objectFactory) {
        Map<String, Object> scope = threadScope.get();
        return scope.computeIfAbsent(name, k -> objectFactory.getObject());
    }

    @Override
    public Object remove(String name) {
        return threadScope.get().remove(name);
    }

    @Override
    public void registerDestructionCallback(String name, Runnable callback) {
        // Not supported in this example (can be implemented if needed)
    }

    @Override
    public Object resolveContextualObject(String key) {
        return null;
    }

    @Override
    public String getConversationId() {
        return Thread.currentThread().getName();
    }
}
```
```java
import org.springframework.beans.factory.config.CustomScopeConfigurer;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import java.util.HashMap;
import java.util.Map;

@Configuration
public class ScopeConfig {

    @Bean
    public static CustomScopeConfigurer customScopeConfigurer() {
        CustomScopeConfigurer configurer = new CustomScopeConfigurer();
        Map<String, Object> scopes = new HashMap<>();
        scopes.put("thread", new ThreadScope());
        configurer.setScopes(scopes);
        return configurer;
    }
}
```
```java
import org.springframework.context.annotation.Scope;
import org.springframework.stereotype.Component;

@Component
@Scope("thread")
public class MyThreadScopedBean {
    private final String id = java.util.UUID.randomUUID().toString();

    public String getId() {
        return id;
    }
}
```
<br />
<br />
<br />




# 8. @SpringBootApplication annotation
1. It’s a meta-annotation (a combination of multiple annotations) that makes bootstrapping and configuration easier.  
2. It is shorthand for three annotations  
    a. @Configuration  
    b. @EnableAutoConfiguration  
    c. @ComponentScan  
3. @Configuration  
    a. Marks the class as a source of bean definitions.  
    b. Equivalent to writing a traditional @Configuration class with @Bean methods.  
```java
@Configuration
public class AppConfig {
    @Bean
    public MyService myService() {
        return new MyService();
    }
}
```
4. @EnableAutoConfiguration  
    a. Tells Spring Boot to automatically configure beans based on dependencies on the classpath.  
    b. Example  

    | Col 1 | Col 2 |
    |-------|-------|
    | If spring-boot-starter-web is present | Sets up Tomcat, DispatcherServlet, Jackson, etc. |
    | If spring-boot-starter-data-jpa is present | Sets up EntityManagerFactory, DataSource, Hibernate. |

    c. You can exclude specific auto-configurations  
```java
@SpringBootApplication(exclude = { DataSourceAutoConfiguration.class })
public class MyApp { ... }
```
5. @ComponentScan  
    a. Tells Spring to scan for components (@Component, @Service, @Repository, @Controller, etc.) in the package of the main class and its subpackages.  
```java
@SpringBootApplication
public class MyApp {
    public static void main(String[] args) {
        SpringApplication.run(MyApp.class, args);
    }
}
```
    b. If MyApp is in com.example, Spring will scan com.example.*
```java
@SpringBootApplication(scanBasePackages = {"com.example.services", "com.example.repositories"})
```
6. Extra Features  
    a. Provides a default main class for Spring Boot apps.  
    b. Works seamlessly with SpringApplication.run() to bootstrap the app.  
    c. Lets you configure auto-configuration exclusions, scan base packages, etc.  
<br />
<br />
<br />





# 9. @AutoConfiguration annotation
1. It’s a specialized form of @Configuration that tells Spring Boot that this is an auto-configuration class, load it automatically if certain conditions are met.  
```java
import org.springframework.boot.autoconfigure.AutoConfiguration;

@AutoConfiguration
public class MyAutoConfig {
    @Bean
    public MyService myService() {
        return new MyService();
    }
}
```
2. This bean will only be registered if the auto-configuration is actually imported (by Boot’s auto-configuration mechanism).  
3. Difference from @Configuration  
    a. @Configuration → You explicitly declare in your app (@Import, @ComponentScan, etc.).  
    b. @AutoConfiguration → Spring Boot discovers it automatically (from META-INF/spring/... files) and applies it conditionally.  
4. Where it’s Used  
    a. In Spring Boot 2.x, auto-configurations were registered via  

```java
META-INF/spring.factories
```

    b. In Spring Boot 3.x, this was replaced by 

```java
META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports
```
5. Conditional Loading - Auto-configurations are almost always combined with conditions so they only apply when needed  
```java
@AutoConfiguration
@ConditionalOnClass(name = "com.example.ExternalLibrary")
@ConditionalOnMissingBean(MyService.class)
public class MyAutoConfig {

    @Bean
    public MyService myService() {
        return new MyService();
    }
}
```

    a. This will only load if ExternalLibrary is on the classpath  
    b. and no other MyService bean is already defined.  
6. Example: Custom Starter  
    a. Create auto-config class

```java
@AutoConfiguration
@EnableConfigurationProperties(MyProperties.class)
public class MyAutoConfiguration {
    @Bean
    @ConditionalOnMissingBean
    public MyService myService(MyProperties props) {
        return new MyService(props.getName());
    }
}
```

    b. Register it in META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports  

```java
com.example.MyAutoConfiguration
```

    c. Add your starter as a dependency → Spring Boot will configure it automatically.  
<br />
<br />
<br />





# 10. What is service registry and how is it implemented in Springboot?
1. A service registry is a database of service instances.  
    a. Each service registers itself (name + host + port).  
    b. Other services discover them dynamically.  
    c. The registry also performs health checks to remove dead instances.  
2. This enables service discovery, so services can find each other without hardcoding URLs.  
3. Popular registries:  
    a. Netflix Eureka (Spring Cloud Netflix)  
    b. Consul  
    c. Zookeeper  
    d. Kubernetes Service Registry  
4. Create Eureka Server (Service Registry)

```java
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
</dependency>
```
```java
@SpringBootApplication
@EnableEurekaServer   // marks this as Eureka server
public class EurekaServerApplication {
    public static void main(String[] args) {
        SpringApplication.run(EurekaServerApplication.class, args);
    }
}
```
```java
server:
  port: 8761

eureka:
  client:
    register-with-eureka: false
    fetch-registry: false
```  

Start this → Eureka dashboard at http://localhost:8761.  

5. Create Eureka Client (Microservice)  
```java
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```
```java
@SpringBootApplication
@EnableDiscoveryClient   // marks as a Eureka client
public class OrderServiceApplication {
    public static void main(String[] args) {
        SpringApplication.run(OrderServiceApplication.class, args);
    }
}
```
```java
spring:
  application:
    name: order-service   # service name to register in Eureka
server:
  port: 8081

eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/
```

This registers order-service in Eureka.

6. Discover Another Service  
    a. Say user-service also registers in Eureka.  
    b. From order-service, you can call it dynamically using Spring Cloud LoadBalancer  
```java
@Service
public class UserClient {

    private final RestTemplate restTemplate;

    public UserClient(RestTemplateBuilder builder) {
        this.restTemplate = builder.build();
    }

    public String getUserData() {
        return restTemplate.getForObject("http://user-service/users/1", String.class);
    }
}
```
Instead of hardcoding http://localhost:8082, we used http://user-service.  
Eureka + LoadBalancer will resolve to an actual instance.  

7. Alternatives  
    a. Consul: spring-cloud-starter-consul-discovery  
    b. Zookeeper: spring-cloud-starter-zookeeper-discovery  
    c. Kubernetes: Service discovery handled by Kubernetes service objects (no need for Eureka).
<br />
<br />
<br />





# 11. What is service discovery and how is it implemented in Springboot?
1. In microservices architecture, service instances  
    a. may run on different hosts/ports  
    b. can scale up/down dynamically  
    c. can fail and restart at different addresses  
2. Hardcoding service URLs (http://localhost:8081) is impossible to maintain.  
3. Service Discovery solves this  
    a. A service (e.g., order-service) queries the Service Registry (e.g., Eureka, Consul) to discover another service (e.g., user-service).  
    b. The registry returns a list of healthy instances.  
    c. A load balancer picks one instance and routes the request.  
4. Types of Service Discovery  

    | Discovery Type | Features |
    |----------------|----------|
    | Client-Side Discovery | The client queries the registry and decides where to send the request. |
    | | Example: Netflix Eureka + Spring Cloud LoadBalancer. |
    | Server-Side Discovery | The client sends a request to a load balancer (gateway). |
    | | The load balancer queries the registry and forwards the request. |
    | | Example: Spring Cloud Gateway, Kubernetes Ingress. |

5. Alternatives to Eureka  
    a. Consul (spring-cloud-starter-consul-discovery)  
    b. Zookeeper (spring-cloud-starter-zookeeper-discovery)  
    c. Kubernetes (no Eureka needed → services are auto-discovered via DNS, e.g., http://user-service:8080)  
6. Code in previous question
<br />
<br />
<br />





# 12. What is the difference between service registry and service discovery?

| Aspect | Service Registry | Service Discovery |
|--------|------------------|-------------------|
| What | Database of available services | Process of finding a service |
| Role | Stores service names, hosts, ports | Uses registry to resolve service endpoints |
| Examples | Eureka, Consul, Zookeeper, Kubernetes DNS | RestTemplate + @LoadBalanced, Feign Client, API Gateway |

Service Registry - Who’s available?
Service Discovery - How do I find and talk to them?

<br />
<br />
<br />

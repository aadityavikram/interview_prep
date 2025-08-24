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





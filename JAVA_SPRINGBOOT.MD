### How to handle multiple beans in Springboot?
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

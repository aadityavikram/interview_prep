### How to handle multiple beans in Springboot?
| S.No. | Answer | Example |
|-------|--------|---------|
| 1 | Use @Primary - Mark one of the beans as the default bean for autowiring | ```java [] @Configuration
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
} ``` |

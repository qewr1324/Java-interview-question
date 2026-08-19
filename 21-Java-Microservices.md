# 📘 فاز ۲۱: میکروسرویس‌ها (Microservices) با Spring Cloud

میکروسرویس‌ها یکی از مهم‌ترین مباحث برای بک‌اندکاران حرفه‌ای هستن. در مصاحبه‌های سطح بالا حتماً ازت می‌پرسن.

---

## ۲۱.۱ Microservices چیست؟

### تعریف

میکروسرویس‌ها یه سبک معماری هستن که در اون یک برنامه بزرگ به چندین سرویس کوچک و مستقل تقسیم میشه. هر سرویس:
- یک Business Capability رو پیاده‌سازی می‌کنه
- مستقل از بقیه Deploy میشه
- با HTTP یا Message Queue ارتباط برقرار می‌کنه
- می‌تونه با زبان‌های مختلف نوشته بشه

### مقایسه Monolith vs Microservices

```text
┌─────────────────────────────────────────────────────────────────────┐
│                                                                     │
│        Monolith (تکنولوژی)          │   Microservices               │
│─────────────────────────────────────┼───────────────────────────────│
│  یک برنامه یکپارچه                 │  چندین سرویس کوچک              │
│  همه چیز توی یک JAR                │  هر سرویس JAR جداگانه          │
│  یک دیتابیس مشترک                  │  هر سرویس دیتابیس خودش رو داره │
│  تغییر = Deploy کل برنامه          │  هر سرویس مستقل Deploy میشه    │
│  خطا = کل برنامه کرش می‌کنه        │  خطا فقط اون سرویس رو می‌گیره  │
│  مقیاس‌پذیری: کل برنامه            │  مقیاس‌پذیری: فقط سرویس مورد نیاز│
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

# مزایا و معایب معماری Microservices

| مزایا | معایب |
|-------|-------|
| مقیاس‌پذیری بالا | پیچیدگی زیاد |
| جدا بودن تیم‌ها (استقلال تیم‌ها) | ارتباطات شبکه (Latency و مسائل شبکه) |
| Deploy مستقل (هر سرویس جداگانه) | مدیریت داده (تراکنش‌های توزیع‌شده) |
| خطا فقط یک سرویس را تحت تأثیر قرار می‌دهد | دیباگ سخت‌تر (ردیابی در چند سرویس) |
| انتخاب تکنولوژی آزاد برای هر سرویس | هزینه بیشتر (زیرساخت، عملیات) |

> **نکات تکمیلی:**
> - **مقیاس‌پذیری**: هر سرویس به‌صورت مستقل مقیاس‌پذیر است و می‌توان فقط سرویس‌های پرترافیک را مقیاس کرد.
> - **استقلال تیم‌ها**: هر تیم می‌تواند روی سرویس خود کار کند بدون وابستگی به تیم‌های دیگر.
> - **مدیریت داده**: چالش اصلی در Microservices، مدیریت تراکنش‌ها و یکپارچگی داده بین سرویس‌هاست.
> - **هزینه بیشتر**: نیاز به مدیریت چندین سرویس، لاگ‌گیری متمرکز، کشف سرویس و زیرساخت پیچیده‌تر دارد.

## ۲۱.۲ معماری میکروسرویس‌ها

```text
┌─────────────────────────────────────────────────────────────────────┐
│                                                                     │
│                    Microservices Architecture                       │
│                                                                     │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │                   API Gateway                               │    │
│  │              (مسیریابی، احراز هویت، Rate Limiting)          │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                              │                                      │
│           ┌──────────────────┼──────────────────┐                   │
│           ▼                  ▼                  ▼                   │
│  ┌───────────────┐  ┌───────────────┐  ┌───────────────┐            │
│  │   User        │  │   Order       │  │   Product     │            │
│  │   Service     │  │   Service     │  │   Service     │            │
│  └───────────────┘  └───────────────┘  └───────────────┘            │
│           │                  │                  │                   │
│           ▼                  ▼                  ▼                   │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │               Service Discovery (Eureka)                    │    │
│  │            ثبت و پیدا کردن سرویس‌ها                          │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### ۲۱.۳ Spring Cloud Netflix

#### وابستگی‌ها

```xml
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-dependencies</artifactId>
            <version>2022.0.0</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>

<dependencies>
    <!-- Eureka Server -->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
    </dependency>
    
    <!-- Eureka Client -->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
    </dependency>
    
    <!-- API Gateway -->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-gateway</artifactId>
    </dependency>
    
    <!-- Circuit Breaker -->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-circuitbreaker-resilience4j</artifactId>
    </dependency>
    
    <!-- Load Balancer -->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-loadbalancer</artifactId>
    </dependency>
</dependencies>
```

### ۲۱.۴ Service Discovery (Eureka)

### Eureka Server

```java
@SpringBootApplication
@EnableEurekaServer
public class DiscoveryApplication {
    public static void main(String[] args) {
        SpringApplication.run(DiscoveryApplication.class, args);
    }
}
```

```prperties
# application.properties (Eureka Server)
server.port=8761
eureka.client.register-with-eureka=false
eureka.client.fetch-registry=false
eureka.server.wait-time-in-ms-when-sync-empty=0
```

### Eureka Client

```java
@SpringBootApplication
@EnableDiscoveryClient
public class UserServiceApplication {
    public static void main(String[] args) {
        SpringApplication.run(UserServiceApplication.class, args);
    }
}
```

```properties
# application.properties (Eureka Client)
spring.application.name=user-service
server.port=8081

eureka.client.serviceUrl.defaultZone=http://localhost:8761/eureka/
eureka.instance.prefer-ip-address=true
eureka.instance.instance-id=${spring.application.name}:${server.port}
```

### استفاده از Service Discovery


```java
@Service
@Slf4j
public class OrderService {
    
    @Autowired
    private LoadBalancerClient loadBalancer;
    
    @Autowired
    private RestTemplate restTemplate;
    
    @Autowired
    private DiscoveryClient discoveryClient;
    
    // ۱. استفاده از LoadBalancerClient
    public User getUserByUsername(String username) {
        ServiceInstance instance = loadBalancer.choose("user-service");
        String url = "http://" + instance.getHost() + ":" + instance.getPort() + "/api/users/" + username;
        return restTemplate.getForObject(url, User.class);
    }
    
    // ۲. استفاده از DiscoveryClient
    public List<User> getAllUsers() {
        List<ServiceInstance> instances = discoveryClient.getInstances("user-service");
        // ...
    }
    
    // ۳. با RestTemplate + @LoadBalanced
    @Bean
    @LoadBalanced
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }
    
    // ۴. استفاده از RestTemplate (با LoadBalanced)
    public User getUserByName(String username) {
        return restTemplate.getForObject(
            "http://user-service/api/users/" + username, 
            User.class
        );
    }
}
```

### ۲۱.۵ API Gateway

### Gateway Service

```java
@SpringBootApplication
@EnableDiscoveryClient
public class GatewayApplication {
    public static void main(String[] args) {
        SpringApplication.run(GatewayApplication.class, args);
    }
}
```

```yaml
# application.yml
server:
  port: 8080

spring:
  application:
    name: api-gateway
  
  cloud:
    gateway:
      routes:
        - id: user-service
          uri: lb://user-service
          predicates:
            - Path=/api/users/**
        
        - id: order-service
          uri: lb://order-service
          predicates:
            - Path=/api/orders/**
        
        - id: product-service
          uri: lb://product-service
          predicates:
            - Path=/api/products/**
      
      # Global filters
      default-filters:
        - name: CircuitBreaker
          args:
            name: default
            fallbackUri: forward:/fallback
        - name: Retry
          args:
            retries: 3
            statuses: SERVER_ERROR

management:
  endpoints:
    web:
      exposure:
        include: health,info,gateway
```

### Gateway Filters

```java
@Configuration
public class GatewayConfig {
    
    // ۱. Custom Filter
    @Bean
    public GlobalFilter customFilter() {
        return (exchange, chain) -> {
            // قبل از route
            ServerHttpRequest request = exchange.getRequest();
            log.info("Request: {} {}", request.getMethod(), request.getURI());
            
            // ادامه
            return chain.filter(exchange).then(Mono.fromRunnable(() -> {
                // بعد از route
                ServerHttpResponse response = exchange.getResponse();
                log.info("Response status: {}", response.getStatusCode());
            }));
        };
    }
    
    // ۲. Rate Limiter Filter
    @Bean
    public RedisRateLimiter redisRateLimiter() {
        return new RedisRateLimiter(10, 20, 1); // 10 requests per second
    }
}
```

### ۲۱.۶ Circuit Breaker (Resilience4j)

### فعال‌سازی

```java
@Configuration
@EnableCircuitBreaker
public class CircuitBreakerConfig { ... }
```

### استفاده

```java
@Service
@Slf4j
public class PaymentService {
    
    @Autowired
    private PaymentClient paymentClient;
    
    // ۱. Circuit Breaker با Fallback
    @CircuitBreaker(name = "paymentService", fallbackMethod = "fallbackPayment")
    public Payment processPayment(PaymentRequest request) {
        log.info("Processing payment...");
        return paymentClient.process(request);
    }
    
    // ۲. Fallback Method
    public Payment fallbackPayment(PaymentRequest request, Throwable t) {
        log.error("Payment fallback: {}", t.getMessage());
        return Payment.builder()
            .status("FAILED")
            .message("Payment service unavailable")
            .build();
    }
    
    // ۳. Retry
    @Retry(name = "paymentService", fallbackMethod = "fallbackPayment")
    public Payment processPaymentWithRetry(PaymentRequest request) {
        return paymentClient.process(request);
    }
    
    // ۴. Rate Limiter
    @RateLimiter(name = "paymentService")
    public Payment processPaymentWithRateLimit(PaymentRequest request) {
        return paymentClient.process(request);
    }
    
    // ۵. Bulkhead (محدودیت همزمان)
    @Bulkhead(name = "paymentService", type = Bulkhead.Type.THREADPOOL)
    public Payment processPaymentWithBulkhead(PaymentRequest request) {
        return paymentClient.process(request);
    }
}
```

### پیکربندی Resilience4j


```yaml
resilience4j:
  circuitbreaker:
    instances:
      paymentService:
        sliding-window-size: 10
        failure-rate-threshold: 50
        wait-duration-in-open-state: 10s
        minimum-number-of-calls: 5
  
  retry:
    instances:
      paymentService:
        max-attempts: 3
        wait-duration: 2s
  
  ratelimiter:
    instances:
      paymentService:
        limit-for-period: 10
        limit-refresh-period: 1s
  
  bulkhead:
    instances:
      paymentService:
        max-concurrent-calls: 5
```

### ۲۱.۷ Distributed Tracing (Sleuth + Zipkin)


```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-sleuth</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-zipkin</artifactId>
</dependency>
```

```properties
spring.sleuth.sampler.probability=1.0
spring.zipkin.base-url=http://localhost:9411
```

### استفاده از Trace ID

```java
@Service
@Slf4j
public class OrderService {
    
    @Autowired
    private Tracer tracer;
    
    public void processOrder(Order order) {
        // Trace ID خودکار در لاگ‌ها ظاهر میشه
        log.info("Processing order: {}", order.getId());
        
        Span span = tracer.nextSpan().name("processOrder");
        try (Tracer.SpanInScope ws = tracer.withSpan(span.start())) {
            // پردازش
            log.info("Order processed");
        } finally {
            span.end();
        }
    }
}
```

## ۲۱.۸ ❗️ سوالات مصاحبه‌ای

### ❓ سوال ۱: میکروسرویس چیه و چرا استفاده میشه؟

**جواب:** Microservices یه معماری هست که برنامه رو به چندین سرویس کوچک و مستقل تقسیم می‌کنه. مزایا: مقیاس‌پذیری بالا، Deploy مستقل، انتخاب تکنولوژی آزاد، خطا فقط یک سرویس.

### ❓ سوال ۲: ``Eureka`` چیه؟

**جواب:** ``Eureka`` یه سرویس ثبت و کشف (Service Discovery) هست. سرویس‌ها خودشون رو در ``Eureka`` ثبت می‌کنن و سرویس‌های دیگه می‌تونن همونجا پیداشون کنن.

### ❓ سوال ۳: ``API Gateway`` چیه و چرا استفاده میشه؟

**جواب:** ``API Gateway`` یه نقطه ورودی واحد برای همه سرویس‌هاست. کارها: مسیریابی درخواست‌ها، احراز هویت، Rate Limiting، Caching، Logging.

### ❓ سوال ۴: ``Circuit Breaker`` چیه؟

**جواب:** ``Circuit Breaker`` یه الگو برای مدیریت خطا در سرویس‌های دیگه هست. وقتی یه سرویس خطا بده، Circuit Breaker باز میشه و درخواست‌ها دیگه به اون سرویس نمی‌رن (تا بازیابی).

### ❓ سوال ۵: فرق ``Monolith`` و ``Microservices`` چیه؟

**جواب:** ``Monolith`` یک برنامه یکپارچه است، ``Microservices`` چندین سرویس کوچک. ``Monolith`` ساده‌تر ولی کمتر مقیاس‌پذیر، ``Microservices`` پیچیده‌تر ولی مقیاس‌پذیرتر.

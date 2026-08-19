# 📘 فاز ۱۳: Spring Boot – خلاصه و کاربردی

این بخش برای مصاحبه‌هایی که ازت Spring Boot می‌پرسن ضروریه. نیازی نیست همه جزئیات رو بدونی، ولی مفاهیم کلیدی رو باید بلد باشی.

## ۱۳.۱ چرخه حیات Beanها در Spring

**Bean:** یه شیء که توسط Spring Container مدیریت میشه.

**مراحل چرخه حیات:**
1. **Instantiation:** سازنده صدا زده میشه
2. **Dependency Injection:** وابستگی‌ها تزریق میشن
3. **Initialization:** متدهای `@PostConstruct` و `afterPropertiesSet` اجرا میشن
4. **Ready for Use:** Bean آماده استفاده است
5. **Destruction:** متدهای `@PreDestroy` اجرا میشن

```java
@Component
class MyBean {
    public MyBean() {
        System.out.println("1. سازنده");
    }
    
    @Autowired
    public void setDependency(SomeService service) {
        System.out.println("2. تزریق وابستگی");
    }
    
    @PostConstruct
    public void init() {
        System.out.println("3. مقداردهی اولیه");
    }
    
    @PreDestroy
    public void destroy() {
        System.out.println("4. پاک‌سازی");
    }
}
```

---

## ۱۳.۲ انواع تزریق وابستگی (Dependency Injection)

### ۱. Constructor Injection (توصیه شده ✅)

```java
@Service
class UserService {
    private final UserRepository repository;
    
    public UserService(UserRepository repository) {
        this.repository = repository;
    }
}
```

### ۲. Setter Injection

```java
@Service
class UserService {
    private UserRepository repository;
    
    @Autowired
    public void setRepository(UserRepository repository) {
        this.repository = repository;
    }
}
```

### ۳. Field Injection (کمتر توصیه میشه)

```java
@Service
class UserService {
    @Autowired
    private UserRepository repository;
}
```

---

## ۱۳.۳ @Transactional چطور کار می‌کنه؟

Spring با Proxy دور کلاس می‌پیچه و قبل و بعد از متد، Transaction رو مدیریت می‌کنه.

```java
@Service
class OrderService {
    @Autowired
    private OrderRepository orderRepository;
    
    @Transactional
    public void createOrder(Order order) {
        // ۱. Transaction شروع میشه
        orderRepository.save(order);
        // ۲. Transaction Commit میشه (اگر خطا نبود)
        // ۳. اگر خطا بود، Rollback میشه
    }
}
```

### نکات مهم:

- @Transactional روی متدهای public کار می‌کنه (به خاطر Proxy)

- روی متدهای private کار نمی‌کنه

- RuntimeException باعث Rollback میشه، Checked Exception نه (مگه اینکه تنظیم کنی)

```java
// Rollback برای همه Exception‌ها
@Transactional(rollbackFor = Exception.class)
public void save() { ... }

// فقط برای RuntimeException
@Transactional
public void save() { ... }
```

---

## ۱۳.۴ @RestController و @Service و @Repository

| Annotation | کاربرد |
|------------|--------|
| `@RestController` | کنترلر REST API (ترکیب `@Controller` + `@ResponseBody`) |
| `@Service` | لایه Business Logic |
| `@Repository` | لایه دسترسی به دیتابیس (با ترجمه خودکار Exception) |
| `@Component` | عمومی‌ترین، برای هر Bean دلخواه |

```java
@RestController
@RequestMapping("/api/users")
class UserController {
    @Autowired
    private UserService userService;
    
    @GetMapping("/{id}")
    public User getUser(@PathVariable Long id) {
        return userService.findById(id);
    }
}

@Service
class UserService {
    @Autowired
    private UserRepository repository;
    
    public User findById(Long id) {
        return repository.findById(id).orElseThrow();
    }
}

@Repository
interface UserRepository extends JpaRepository<User, Long> {
}
```

---

## ۱۳.۵ application.properties مهم‌ترین تنظیمات

```properties
# properties config

# سرور
server.port=8080
server.servlet.context-path=/api

# دیتابیس
spring.datasource.url=jdbc:mysql://localhost:3306/mydb
spring.datasource.username=root
spring.datasource.password=1234

# JPA
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL8Dialect

# لاگ
logging.level.org.springframework=INFO
logging.level.com.myapp=DEBUG
```

---

## ۱۳.۶ Scopeهای Bean در Spring

| Scope | توضیح |
|-------|--------|
| `singleton` (پیش‌فرض) | فقط یک نمونه در کل برنامه |
| `prototype` | هر بار که درخواست بشه، یه نمونه جدید |
| `request` | برای هر درخواست HTTP، یه نمونه (فقط در Web) |
| `session` | برای هر نشست کاربر، یه نمونه (فقط در Web) |

```java
@Component
@Scope("prototype")
class MyPrototypeBean { ... }
```

---

## ۳. Spring Beanها (مفصل‌تر)

## ۱۳.۷ Beanهای Spring – کامل

### ۱۳.۷.۱ چرخه حیات کامل Bean

```text
┌─────────────────────────────────────────────────────────────────────┐
│                                                                     │
│  1. Instantiation (ساخت شیء)                                        │
│     ↓                                                               │
│  2. Populate Properties (تزریق وابستگی‌ها)                           │
│     ↓                                                               │
│  3. BeanNameAware (اگر setBeanName پیاده‌سازی شده)                   │
│     ↓                                                               │
│  4. BeanClassLoaderAware (اگر setBeanClassLoader پیاده‌سازی شده)     │
│     ↓                                                               │
│  5. BeanFactoryAware (اگر setBeanFactory پیاده‌سازی شده)             │
│     ↓                                                               │
│  6. @PostConstruct (متد init)                                       │
│     ↓                                                               │
│  7. InitializingBean (اگر afterPropertiesSet پیاده‌سازی شده)         │
│     ↓                                                               │
│  8. Bean Ready! (آماده استفاده)                                     │
│     ↓                                                               │
│  9. @PreDestroy (متد destroy)                                       │
│     ↓                                                               │
│  10. DisposableBean (اگر destroy پیاده‌سازی شده)                     │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### ۱۳.۷.۲ انواع Scopes

```java
// ۱. Singleton (پیش‌فرض) - یک نمونه برای کل برنامه
@Component
@Scope("singleton")
class SingletonBean { ... }

// ۲. Prototype - هر بار یه نمونه جدید
@Component
@Scope("prototype")
class PrototypeBean { ... }

// ۳. Request - برای هر درخواست HTTP
@Component
@Scope("request")
class RequestBean { ... }

// ۴. Session - برای هر نشست کاربر
@Component
@Scope("session")
class SessionBean { ... }

// ۵. Application - یک نمونه برای کل ServletContext
@Component
@Scope("application")
class ApplicationBean { ... }
```

### ۱۳.۷.۳ Bean Definition (روش‌های تعریف Bean)

```java
// a. با @Component
@Component
class MyComponent { ... }

// b. با @Service
@Service
class MyService { ... }

// c. با @Repository
@Repository
class MyRepository { ... }

// d. با @Controller / @RestController
@RestController
class MyController { ... }

// e. با @Bean در کلاس @Configuration
@Configuration
class AppConfig {
    @Bean
    public MyBean myBean() {
        return new MyBean();
    }
    
    @Bean
    public MyBean myBeanWithParams(AnotherBean another) {
        return new MyBean(another);
    }
}

// f. با @Import
@Import({MyConfig.class, AnotherConfig.class})
class MainConfig { ... }
```

### ۱۳.۷.۴ Bean Wiring (اتصال Beanها)

```java
// ۱. Constructor Injection (توصیه شده)
@Service
class UserService {
    private final UserRepository repository;
    
    // اگه فقط یک سازنده باشه، نیازی به @Autowired نیست
    public UserService(UserRepository repository) {
        this.repository = repository;
    }
}

// ۲. Setter Injection
@Service
class UserService {
    private UserRepository repository;
    
    @Autowired
    public void setRepository(UserRepository repository) {
        this.repository = repository;
    }
}

// ۳. Field Injection
@Service
class UserService {
    @Autowired
    private UserRepository repository;
}

// ۴. @Qualifier (وقتی چند تا Bean از یه نوع هستن)
@Service
class ReportService {
    @Autowired
    @Qualifier("pdfReport")
    private ReportGenerator pdfGenerator;
    
    @Autowired
    @Qualifier("excelReport")
    private ReportGenerator excelGenerator;
}

// ۵. @Primary (اولویت با این Bean)
@Configuration
class ReportConfig {
    @Bean
    @Primary
    public ReportGenerator pdfGenerator() {
        return new PdfGenerator();
    }
    
    @Bean
    public ReportGenerator excelGenerator() {
        return new ExcelGenerator();
    }
}

// ۶. @Lazy (ساخت دیرهنگام)
@Service
class UserService {
    @Autowired
    @Lazy
    private HeavyService heavyService; // فقط وقتی استفاده بشه ساخته میشه
}
```

### ۱۳.۷.۵ Conditional Beans (ساخت شرطی)

```java
@Configuration
class ConditionalConfig {
    
    // ۱. اگه کلاس مشخصی در Classpath بود
    @ConditionalOnClass(Driver.class)
    @Bean
    public DataSource mysqlDataSource() { ... }
    
    // ۲. اگه کلاس مشخصی در Classpath نبود
    @ConditionalOnMissingClass("com.mysql.jdbc.Driver")
    @Bean
    public DataSource h2DataSource() { ... }
    
    // ۳. اگه پراپرتی مشخصی تنظیم شده بود
    @ConditionalOnProperty(name = "app.use.mysql", havingValue = "true")
    @Bean
    public DatabaseService mysqlService() { ... }
    
    // ۴. اگه Bean خاصی موجود بود
    @ConditionalOnBean(MyService.class)
    @Bean
    public AnotherService anotherService() { ... }
    
    // ۵. ترکیب شرط‌ها
    @ConditionalOnProperty(name = "app.cache.enabled")
    @ConditionalOnBean(CacheManager.class)
    @Bean
    public CacheService cacheService() { ... }
}
```

# 📘 فاز ۲۰: مفاهیم پیشرفته Spring – Caching, Async, Scheduling, Validation

این بخش شامل مباحث پیشرفته‌تر Spring هست که توی پروژه‌های واقعی خیلی استفاده میشن.

---

## ۲۰.۱ Caching (کش کردن)

### ۲۰.۱.۱ فعال‌سازی Caching

```java
@Configuration
@EnableCaching
public class CacheConfig {
    
    @Bean
    public CacheManager cacheManager() {
        return new ConcurrentMapCacheManager("users", "orders", "products");
    }
}
```

### ۲۰.۱.۲ @Cacheable (ذخیره در کش)

```java
@Service
@Slf4j
public class UserService {
    
    @Autowired
    private UserRepository repository;
    
    // ۱. ساده
    @Cacheable("users")
    public User findUser(Long id) {
        log.info("Fetching user from database: {}", id);
        return repository.findById(id).orElseThrow();
    }
    
    // ۲. با کلید سفارشی
    @Cacheable(value = "users", key = "#username")
    public User findByUsername(String username) {
        return repository.findByUsername(username);
    }
    
    // ۳. با کلید ترکیبی
    @Cacheable(value = "users", key = "#username + ':' + #password")
    public User findByUsernameAndPassword(String username, String password) {
        return repository.findByUsernameAndPassword(username, password);
    }
    
    // ۴. با شرط (فقط اگه id > 0 باشه)
    @Cacheable(value = "users", condition = "#id > 0")
    public User findUserWithCondition(Long id) {
        return repository.findById(id).orElseThrow();
    }
    
    // ۵. با شرط (نداشتن)
    @Cacheable(value = "users", unless = "#result == null")
    public User findUserUnlessNull(Long id) {
        return repository.findById(id).orElse(null);
    }
}
```

### ۲۰.۱.۳ @CacheEvict (پاک کردن کش)


```java
@Service
public class UserService {
    
    // ۱. پاک کردن یک کلید خاص
    @CacheEvict(value = "users", key = "#id")
    public void deleteUser(Long id) {
        repository.deleteById(id);
    }
    
    // ۲. پاک کردن همه کش‌های مربوط
    @CacheEvict(value = "users", allEntries = true)
    public void clearAllCache() {
        // همه کش‌های users پاک میشن
    }
    
    // ۳. پاک کردن قبل از اجرای متد
    @CacheEvict(value = "users", key = "#id", beforeInvocation = true)
    public void updateUser(User user) {
        // قبل از اجرا کش پاک میشه
        repository.save(user);
    }
}
```

### ۲۰.۱.۴ @CachePut (به‌روزرسانی کش)

```java
@Service
public class UserService {
    
    // همیشه کش رو به‌روز می‌کنه (حتی اگه متد خطا بده)
    @CachePut(value = "users", key = "#user.id")
    public User updateUser(User user) {
        return repository.save(user);
    }
}
```

### ۲۰.۱.۵ Caching با Redis

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

```properties
spring.redis.host=localhost
spring.redis.port=6379
spring.redis.password=
spring.redis.timeout=2000

spring.cache.type=redis
spring.cache.redis.time-to-live=600000  # ۱۰ دقیقه
spring.cache.redis.cache-null-values=false
```

```java
@Configuration
@EnableCaching
public class RedisCacheConfig {
    
    @Bean
    public RedisCacheConfiguration cacheConfiguration() {
        return RedisCacheConfiguration.defaultCacheConfig()
            .entryTtl(Duration.ofMinutes(10))
            .disableCachingNullValues()
            .serializeValuesWith(RedisSerializationContext.SerializationPair
                .fromSerializer(new GenericJackson2JsonRedisSerializer()));
    }
}
```

## ۲۰.۲ Async (پردازش ناهمگام)

### ۲۰.۲.۱ فعال‌سازی Async

```java
@Configuration
@EnableAsync
public class AsyncConfig {
    
    @Bean(name = "taskExecutor")
    public Executor taskExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(5);      // Threadهای اولیه
        executor.setMaxPoolSize(10);      // حداکثر Threadها
        executor.setQueueCapacity(100);   // صف انتظار
        executor.setThreadNamePrefix("Async-");
        executor.initialize();
        return executor;
    }
}
```

### ۲۰.۲.۲ @Async

```java
@Service
@Slf4j
public class EmailService {
    
    // ۱. ساده
    @Async
    public void sendEmail(String to, String body) {
        log.info("Sending email to: {} on thread: {}", to, Thread.currentThread().getName());
        // کد ارسال ایمیل
        try {
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            // ignore
        }
        log.info("Email sent to: {}", to);
    }
    
    // ۲. با Executor خاص
    @Async("taskExecutor")
    public void sendEmailWithCustomExecutor(String to, String body) {
        // ...
    }
    
    // ۳. برگرداندن Future
    @Async
    public CompletableFuture<String> processData(String data) {
        String result = data.toUpperCase();
        return CompletableFuture.completedFuture(result);
    }
}
```

### ۲۰.۲.۳ استفاده از Async

```java
@RestController
@RequestMapping("/api/email")
public class EmailController {
    
    @Autowired
    private EmailService emailService;
    
    @GetMapping("/send")
    public ResponseEntity<String> sendEmail() {
        emailService.sendEmail("test@example.com", "Hello!");
        return ResponseEntity.ok("Email sending started");
    }
    
    @GetMapping("/process")
    public CompletableFuture<ResponseEntity<String>> process() throws ExecutionException, InterruptedException {
        CompletableFuture<String> future = emailService.processData("hello");
        String result = future.get();
        return CompletableFuture.completedFuture(ResponseEntity.ok(result));
    }
}
```

### ۲۰.۲.۴ مدیریت خطا در Async

```java
@Configuration
@EnableAsync
public class AsyncConfig implements AsyncConfigurer {
    
    @Override
    public Executor getAsyncExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(5);
        executor.setMaxPoolSize(10);
        executor.setThreadNamePrefix("Async-");
        executor.initialize();
        return executor;
    }
    
    @Override
    public AsyncUncaughtExceptionHandler getAsyncUncaughtExceptionHandler() {
        return (throwable, method, params) -> {
            log.error("Async error in method: {}", method.getName(), throwable);
        };
    }
}
```

## ۲۰.۳ Scheduling (برنامه‌ریزی)

### ۲۰.۳.۱ فعال‌سازی Scheduling

```java
@Configuration
@EnableScheduling
public class SchedulingConfig { ... }
```

### ۲۰.۳.۲ @Scheduled

```java
@Component
@Slf4j
public class ScheduledTasks {
    
    // ۱. با فاصله ثابت (بعد از اتمام کار قبلی)
    @Scheduled(fixedDelay = 5000)
    public void fixedDelayTask() {
        log.info("Fixed delay task executed");
    }
    
    // ۲. با فاصله ثابت (بدون توجه به اتمام کار قبلی)
    @Scheduled(fixedRate = 10000)
    public void fixedRateTask() {
        log.info("Fixed rate task executed");
    }
    
    // ۳. با تأخیر اولیه
    @Scheduled(initialDelay = 10000, fixedRate = 30000)
    public void delayedTask() {
        log.info("Delayed task executed");
    }
    
    // ۴. با Cron Expression
    @Scheduled(cron = "0 0 2 * * *") // هر روز ساعت ۲ بامداد
    public void cronTask() {
        log.info("Cron task executed - daily backup");
    }
    
    // ۵. با Cron و TimeZone
    @Scheduled(cron = "0 0 0 * * SAT", zone = "Asia/Tehran")
    public void weeklyTask() {
        log.info("Weekly task - every Saturday midnight Tehran time");
    }
}
```

### ۲۰.۳.۳ Cron Expression کامل


```text
┌───────────── ثانیه (0-59)
│ ┌───────────── دقیقه (0-59)
│ │ ┌───────────── ساعت (0-23)
│ │ │ ┌───────────── روز ماه (1-31)
│ │ │ │ ┌───────────── ماه (1-12 یا JAN-DEC)
│ │ │ │ │ ┌───────────── روز هفته (0-7 یا SUN-SAT)
│ │ │ │ │ │
* * * * * * *

مثال‌ها:
"0 0 2 * * *"          → هر روز ساعت ۲:۰۰ بامداد
"0 30 9 * * MON-FRI"   → روزهای کاری ساعت ۹:۳۰ صبح
"0 0 0 1 * *"          → اولین روز هر ماه ساعت ۱۲:۰۰
"0 0 12 * * 6"         → هر جمعه ساعت ۱۲:۰۰
"0 0 0 * * 6"          → هر جمعه نیمه‌شب
"0 0 0 1 1 *"          → اول ژانویه نیمه‌شب
"0 0 0 * * 0"          → هر یکشنبه نیمه‌شب
"0 0 0 15 * *"         → پانزدهم هر ماه نیمه‌شب
```

### ۲۰.۳.۴ Scheduling با پروفایل


```java
@Component
@Slf4j
@Profile("prod")
public class ProdScheduledTasks {
    
    @Scheduled(cron = "0 0 2 * * *")
    public void backupDatabase() {
        log.info("Backing up database...");
    }
}
```

## ۲۰.۴ Validation (اعتبارسنجی)

### ۲۰.۴.۱ فعال‌سازی Validation


```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-validation</artifactId>
</dependency>
```

### ۲۰.۴.۲ Annotationهای پرکاربرد


```java
@Data
public class UserRequest {
    
    @NotNull(message = "ID نمی‌تواند null باشد")
    private Long id;
    
    @NotBlank(message = "نام کاربری اجباری است")
    @Size(min = 3, max = 20, message = "نام کاربری باید بین ۳ تا ۲۰ کاراکتر باشد")
    private String username;
    
    @NotBlank(message = "پسورد اجباری است")
    @Size(min = 8, message = "پسورد باید حداقل ۸ کاراکتر باشد")
    @Pattern(
        regexp = "^(?=.*[0-9])(?=.*[a-z])(?=.*[A-Z]).+$",
        message = "پسورد باید شامل عدد و حروف بزرگ و کوچک باشد"
    )
    private String password;
    
    @Email(message = "ایمیل نامعتبر است")
    @NotBlank(message = "ایمیل اجباری است")
    private String email;
    
    @Min(value = 18, message = "سن باید حداقل ۱۸ باشد")
    @Max(value = 100, message = "سن باید حداکثر ۱۰۰ باشد")
    private Integer age;
    
    @Pattern(
        regexp = "^\\d{10}$",
        message = "شماره تلفن باید ۱۰ رقم باشد"
    )
    private String phoneNumber;
    
    @Size(max = 500, message = "توضیحات حداکثر ۵۰۰ کاراکتر")
    private String description;
}
```

### ۲۰.۴.۳ استفاده در Controller


```java
@RestController
@RequestMapping("/api/users")
public class UserController {
    
    // ۱. @Valid برای اعتبارسنجی
    @PostMapping
    public ResponseEntity<User> createUser(@Valid @RequestBody UserRequest request) {
        return ResponseEntity.ok(userService.save(request));
    }
    
    // ۲. @Validated با گروه‌ها
    @PostMapping("/register")
    public ResponseEntity<User> registerUser(@Validated @RequestBody UserRequest request) {
        return ResponseEntity.ok(userService.register(request));
    }
    
    // ۳. BindingResult برای خطاهای اعتبارسنجی
    @PostMapping("/validate")
    public ResponseEntity<?> validateUser(@Valid @RequestBody UserRequest request, 
                                          BindingResult bindingResult) {
        if (bindingResult.hasErrors()) {
            List<String> errors = bindingResult.getAllErrors()
                .stream()
                .map(DefaultMessageSourceResolvable::getDefaultMessage)
                .collect(Collectors.toList());
            return ResponseEntity.badRequest().body(errors);
        }
        return ResponseEntity.ok("Valid");
    }
}
```

### ۲۰.۴.۴ مدیریت خطاهای Validation (Global)


```java
@ControllerAdvice
public class GlobalExceptionHandler {
    
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<Map<String, List<String>>> handleValidationErrors(
            MethodArgumentNotValidException ex) {
        List<String> errors = ex.getBindingResult()
            .getAllErrors()
            .stream()
            .map(DefaultMessageSourceResolvable::getDefaultMessage)
            .collect(Collectors.toList());
        
        return ResponseEntity.badRequest()
            .body(Map.of("errors", errors));
    }
    
    @ExceptionHandler(ConstraintViolationException.class)
    public ResponseEntity<Map<String, String>> handleConstraintViolation(
            ConstraintViolationException ex) {
        String message = ex.getConstraintViolations()
            .stream()
            .map(violation -> violation.getMessage())
            .collect(Collectors.joining(", "));
        
        return ResponseEntity.badRequest()
            .body(Map.of("error", message));
    }
}
```

### ۲۰.۴.۵ Group Validation (اعتبارسنجی گروهی)


```java
// a. تعریف گروه‌ها
public interface Create {}
public interface Update {}

// b. استفاده در کلاس
@Data
public class UserRequest {
    @NotNull(groups = Update.class)
    private Long id;
    
    @NotBlank(groups = {Create.class, Update.class})
    private String username;
    
    @NotBlank(groups = {Create.class})
    private String password;
}

// c. استفاده در Controller
@PostMapping
public ResponseEntity<User> create(@Validated(Create.class) @RequestBody UserRequest request) {
    // فقط اعتبارسنجی Create گروه
}

@PutMapping
public ResponseEntity<User> update(@Validated(Update.class) @RequestBody UserRequest request) {
    // فقط اعتبارسنجی Update گروه
}
```

### ۲۰.۴.۶ Custom Validation (سفارشی)


```java
// ۱. Annotation
@Target({ElementType.FIELD, ElementType.PARAMETER})
@Retention(RetentionPolicy.RUNTIME)
@Constraint(validatedBy = IranNationalCodeValidator.class)
public @interface IranNationalCode {
    String message() default "کد ملی نامعتبر است";
    Class<?>[] groups() default {};
    Class<? extends Payload>[] payload() default {};
}

// ۲. Validator
public class IranNationalCodeValidator implements ConstraintValidator<IranNationalCode, String> {
    
    @Override
    public boolean isValid(String code, ConstraintValidatorContext context) {
        if (code == null || code.length() != 10) {
            return false;
        }
        // الگوریتم اعتبارسنجی کد ملی ایران
        try {
            // کد الگوریتم...
            return true;
        } catch (Exception e) {
            return false;
        }
    }
}

// ۳. استفاده
@Data
public class PersonRequest {
    @IranNationalCode
    private String nationalCode;
}
```

## ۲۰.۵ ❗️ سوالات مصاحبه‌ای

### ❓ سوال ۱: فرق ``@Cacheable`` و ``@CacheEvict`` چیه؟

**جواب:**

- **@Cacheable:** نتیجه متد رو در کش ذخیره می‌کنه

- **@CacheEvict:** کش رو پاک می‌کنه

### ❓ سوال ۲: ``@Async`` چه کاربردی داره؟

**جواب:** ``@Async`` برای اجرای ناهمگام (Asynchronous) متدهاست. یعنی متد در یک Thread جداگانه اجرا میشه و Thread اصلی منتظر نمی‌مونه.

### ❓ سوال ۳: Cron Expression چیه؟

**جواب:** یه فرمت برای مشخص کردن زمان اجرای کارهای زمان‌بندی شده. شامل ۶ بخش: ثانیه، دقیقه، ساعت، روز ماه، ماه، روز هفته.

### ❓ سوال ۴: ``Validation`` با ``@Valid`` و ``@Validated`` چه فرقی دارن؟

**جواب:**

- **@Valid:** از JPA میاد، برای اعتبارسنجی استاندارد

- **@Validated:** از Spring میاد، پشتیبانی از Group Validation داره

### ❓ سوال ۵: چطور کش رو در ``Spring`` فعال می‌کنیم؟

**جواب:** با اضافه کردن ``@EnableCaching`` به یک ``@Configuration`` کلاس.

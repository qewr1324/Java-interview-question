## 📄 فایل جدید: `19-Java-Logging.md`

# 📘 فاز ۱۹: لاگ‌گیری (Logging) در جاوا

لاگ‌گیری یکی از مهم‌ترین کارها برای Debugging، Monitoring و عیب‌یابی در برنامه‌های واقعیه. توی مصاحبه ازت می‌پرسن با چه ابزاری کار کردی و چرا.

---

## ۱۹.۱ چرا لاگ؟

1. **Debugging:** پیدا کردن باگ‌ها
2. **Monitoring:** بررسی وضعیت برنامه
3. **Audit:** ثبت فعالیت‌های کاربران
4. **Troubleshooting:** عیب‌یابی مشکلات
5. **Performance:** بررسی عملکرد

---

## ۱۹.۲ SLF4J (Simple Logging Facade for Java)

SLF4J یه **Facade** (رابط) برای لاگ‌گیری هست. یعنی خودش لاگ نمی‌نویسه، بلکه به کتابخانه‌های دیگه وصل میشه.

```text
┌─────────────────────────────────────────────────────────────────────┐
│                                                                     │
│                    SLF4J Architecture                               │
│                                                                     │
│  ┌─────────────────────────────────────────────────────────────────┐│
│  │                       Application Code                          ││
│  │                    (SLF4J API استفاده می‌کند)                    ││
│  └─────────────────────────────────────────────────────────────────┘│
│                              │                                      │
│                              ▼                                      │
│  ┌─────────────────────────────────────────────────────────────────┐│
│  │                         SLF4J                                   ││
│  │                        (Facade)                                 ││
│  └─────────────────────────────────────────────────────────────────┘│
│                              │                                      │
│           ┌──────────────────┼──────────────────┐                   │
│           ▼                  ▼                  ▼                   │
│  ┌───────────────┐  ┌───────────────┐  ┌───────────────┐            │
│  │   Logback     │  │   Log4j2      │  │   Java Util   │            │
│  │   (پیش‌فرض)    │  │   (جایگزین)   │  │   Logging     │            │
│  └───────────────┘  └───────────────┘  └───────────────┘            │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

#### مزایای SLF4J

- وابستگی کم: می‌تونی هر کتابخانه‌ای رو استفاده کنی

- ساختار یکسان: فرقی نمی‌کنه از Logback یا Log4j استفاده کنی، کد یکسانه

- پارامتریشدن: با {} راحت‌تر

### ۱۹.۳ Logback (پیش‌فرض Spring Boot)

#### تنظیمات در application.properties

```properties
# a. سطح لاگ برای پکیج‌ها
logging.level.root=INFO
logging.level.com.example=DEBUG
logging.level.org.springframework=INFO
logging.level.org.hibernate=WARN
logging.level.org.hibernate.SQL=DEBUG
logging.level.org.hibernate.type.descriptor.sql.BasicBinder=TRACE

# b. فایل لاگ
logging.file.name=logs/myapp.log
logging.file.path=logs/

# c. فرمت لاگ
logging.pattern.console=%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n
logging.pattern.file=%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n

# d. گروه‌های لاگ (برای گروه‌بندی)
logging.group.db=org.hibernate,org.springframework.jdbc
logging.level.db=DEBUG
```

### فایل logback-spring.xml (تنظیمات پیشرفته)

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    
    <!-- ۱. Console Appender -->
    <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>
    
    <!-- ۲. File Appender -->
    <appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>logs/myapp.log</file>
        
        <!-- چرخش فایل بر اساس زمان -->
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>logs/myapp.%d{yyyy-MM-dd}.log</fileNamePattern>
            <maxHistory>30</maxHistory> <!-- ۳۰ روز نگهداری -->
        </rollingPolicy>
        
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>
    
    <!-- ۳. File Appender با اندازه محدود -->
    <appender name="FILE_SIZE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>logs/myapp.log</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
            <fileNamePattern>logs/myapp.%d{yyyy-MM-dd}.%i.log</fileNamePattern>
            <maxFileSize>10MB</maxFileSize>
            <maxHistory>30</maxHistory>
            <totalSizeCap>1GB</totalSizeCap>
        </rollingPolicy>
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>
    
    <!-- ۴. JSON Formatter (برای ELK) -->
    <appender name="JSON" class="ch.qos.logback.core.ConsoleAppender">
        <encoder class="net.logstash.logback.encoder.LogstashEncoder">
            <includeMdc>true</includeMdc>
        </encoder>
    </appender>
    
    <!-- ۵. تنظیم سطح لاگ برای پکیج‌های مختلف -->
    <logger name="com.example" level="DEBUG"/>
    <logger name="org.springframework" level="INFO"/>
    <logger name="org.hibernate" level="WARN"/>
    <logger name="com.example.security" level="DEBUG"/>
    
    <!-- ۶. Root Logger -->
    <root level="INFO">
        <appender-ref ref="CONSOLE"/>
        <appender-ref ref="FILE"/>
    </root>
    
    <!-- ۷. فعال کردن JSON در محیط خاص -->
    <springProfile name="prod">
        <root level="INFO">
            <appender-ref ref="JSON"/>
        </root>
    </springProfile>
    
</configuration>
```

## ۱۹.۴ استفاده از SLF4J در کد

### روش ۱: با Lombok (توصیه شده)

```java
import lombok.extern.slf4j.Slf4j;

@Service
@Slf4j
public class UserService {
    
    public void processUser(Long userId) {
        // ۱. اطلاعات
        log.info("Processing user: {}", userId);
        
        // ۲. خطا
        try {
            // کد...
        } catch (Exception e) {
            log.error("Error processing user: {}", userId, e);
        }
        
        // ۳. هشدار
        if (userId < 0) {
            log.warn("Invalid user ID: {}", userId);
        }
        
        // ۴. دیباگ
        log.debug("User processing started: {}", userId);
        
        // ۵. trace (جزئی‌ترین)
        log.trace("User processing step 1");
        
        // ۶. بدون پارامتر
        log.info("Method called");
    }
}
```

### روش ۲: بدون Lombok

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

@Service
public class UserService {
    private static final Logger log = LoggerFactory.getLogger(UserService.class);
    
    public void processUser(Long userId) {
        log.info("Processing user: {}", userId);
    }
}
```

### روش ۳: Dynamic Logging (با پارامترهای شرطی)

```java
@Service
@Slf4j
public class UserService {
    
    public void processUser(Long userId) {
        // چک کردن سطح لاگ (برای بهینه‌سازی)
        if (log.isDebugEnabled()) {
            log.debug("Processing user: {}", userId);
        }
        
        // با چند پارامتر
        String username = "ali";
        int age = 25;
        log.info("User: {}, Age: {}", username, age);
        
        // با exception
        try {
            // ...
        } catch (Exception e) {
            log.error("Error occurred", e);
        }
    }
}
```

### ۱۹.۵ سطوح لاگ (Log Levels)

```text
┌─────────────────────────────────────────────────────────────────────┐
│                                                                     │
│                    Log Levels (از کمترین به بیشترین)                │
│                                                                     │
│  ┌─────────────────────────────────────────────────────────────────┐│
│  │  ERROR  ← خطاهای جدی که برنامه رو متوقف می‌کنن                   ││
│  └─────────────────────────────────────────────────────────────────┘│
│  ┌─────────────────────────────────────────────────────────────────┐│
│  │  WARN   ← وضعیت‌های غیرعادی ولی قابل بازیابی                     ││
│  └─────────────────────────────────────────────────────────────────┘│
│  ┌─────────────────────────────────────────────────────────────────┐│
│  │  INFO   ← اطلاعات عمومی (کاربر ثبت شد، سفارش ایجاد شد)           ││
│  └─────────────────────────────────────────────────────────────────┘│
│  ┌─────────────────────────────────────────────────────────────────┐│
│  │  DEBUG  ← اطلاعات برای دیباگ (ورودی متدها، مقادیر)               ││
│  └─────────────────────────────────────────────────────────────────┘│
│  ┌─────────────────────────────────────────────────────────────────┐│
│  │  TRACE  ← جزئی‌ترین اطلاعات (هر قدم کوچک)                         ││
│  └─────────────────────────────────────────────────────────────────┘│
│                                                                     │
│  هر سطح، سطوح بالاتر از خودش رو هم شامل میشه:                        │
│  مثلاً INFO شامل INFO, WARN, ERROR هست (نه DEBUG و TRACE)             │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### مثال: سطوح در عمل

```java
@Slf4j
public class LogExample {
    
    public void divide(int a, int b) {
        log.trace("divide called with a={}, b={}", a, b);
        
        if (b == 0) {
            log.error("Cannot divide by zero! a={}, b={}", a, b);
            throw new ArithmeticException("Division by zero");
        }
        
        if (a < 0) {
            log.warn("Dividing negative number: {}", a);
        }
        
        int result = a / b;
        log.debug("Result: {}", result);
        log.info("Division completed: {} / {} = {}", a, b, result);
    }
}
```

### ۱۹.۶ MDC (Mapped Diagnostic Context)

#### MDC برای ذخیره اطلاعات مرتبط با هر درخواست (مثل User ID, Request ID) مفیده.

```java
import org.slf4j.MDC;

@Service
@Slf4j
public class UserService {
    
    public void processUser(Long userId) {
        // ۱. ذخیره در MDC
        MDC.put("userId", userId.toString());
        MDC.put("requestId", UUID.randomUUID().toString());
        
        try {
            log.info("Processing user");
            // کد پردازش...
            log.info("User processed successfully");
        } finally {
            // ۲. پاک کردن MDC (مهم!)
            MDC.clear();
        }
    }
}
```

### پیکربندی MDC در logback

```xml
<encoder>
    <pattern>%d{yyyy-MM-dd HH:mm:ss} [%thread] %-5level %logger{36} - [%X{userId}] [%X{requestId}] - %msg%n</pattern>
</encoder>
```

### Filter برای پر کردن MDC خودکار

```java
@Component
@Order(Ordered.HIGHEST_PRECEDENCE)
public class MDCFilter implements Filter {
    
    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) {
        try {
            MDC.put("requestId", UUID.randomUUID().toString());
            // اگه کاربر لاگین هست
            Authentication auth = SecurityContextHolder.getContext().getAuthentication();
            if (auth != null && auth.isAuthenticated()) {
                MDC.put("username", auth.getName());
            }
            chain.doFilter(request, response);
        } finally {
            MDC.clear();
        }
    }
}
```

## ۱۹.۷ ❗️ سوالات مصاحبه‌ای

### ❓ سوال ۱: SLF4J چیه و چرا ازش استفاده می‌کنیم؟

#### جواب: SLF4J یه Facade برای کتابخانه‌های لاگ‌گیری هست. مزایا:

- وابستگی به کتابخانه خاصی نداریم

- می‌تونیم کتابخانه لاگ رو عوض کنیم بدون تغییر کد

- API یکسان داره

- پشتیبانی از پارامتریشدن ({})

### ❓ سوال ۲: فرق Logback و Log4j2 چیه؟

**جواب:**

# مقایسه Logback و Log4j2

| ویژگی | Logback | Log4j2 |
|-------|---------|--------|
| **وضعیت در Spring Boot** | پیش‌فرض Spring Boot | جایگزین Logback |
| **سرعت** | سریع | سریع‌تر |
| **سادگی** | ساده‌تر | پیچیده‌تر |
| **قابلیت‌ها** | خوب | قابلیت‌های بیشتر (Async, Lambda) |

> **نکات کلیدی:**
> - **Logback** به‌عنوان جانشین Log4j نسخه 1 طراحی شده و به‌صورت پیش‌فرض در Spring Boot استفاده می‌شود.
> - **Log4j2** نسخه بازطراحی شده Log4j است که از نظر عملکرد (به‌ویژه با قابلیت Async) از Logback سریع‌تر است.
> - Log4j2 از **Lambda Expressions** برای Lazy Logging پشتیبانی می‌کند که می‌تواند بهبود عملکرد داشته باشد.
> - انتخاب بین این دو معمولاً به نیاز پروژه و ترجیح تیم بستگی دارد؛ هر دو کتابخانه قدرتمندی هستند.

### ❓ سوال ۳: سطوح لاگ رو به ترتیب از کمترین به بیشترین بگو

**جواب:**

```text
TRACE < DEBUG < INFO < WARN < ERROR
```

#### هر سطح، سطوح بالاتر رو هم شامل میشه.

### ❓ سوال ۴: چطور لاگ رو در فایل ذخیره کنیم؟

#### جواب: با تنظیمات ``logback-spring.xml`` یا ``application.properties``:

```properties
logging.file.name=logs/myapp.log
```

### ❓ سوال ۵: MDC چیه و چه کاربردی داره؟

**جواب:** MDC (Mapped Diagnostic Context) یه مکانیزم برای ذخیره اطلاعات مرتبط با هر درخواست (مثل User ID, Request ID) در Context لاگ‌گیری هست. این اطلاعات می‌تونن توی همه لاگ‌های اون درخواست ظاهر بشن.

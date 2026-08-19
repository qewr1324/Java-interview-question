# 📘 فاز ۹: امنیت (Security) – کامل و عمیق

امنیت یه لایه نیست، یه اصل هست که باید توی تمام لایه‌های برنامه (از کد تا سرور) رعایت بشه. ما اینجا روی مفاهیمی تمرکز می‌کنیم که یه بک‌اند کار جاوا (مخصوصاً با Spring Security) باید بلد باشه.

---

## 9.1 مفاهیم پایه و حیاتی امنیت

### ❓ سوال مصاحبه‌ای ۱: Authentication و Authorization چه فرقی دارن؟ (سوال تکراری و پایه‌ای)

**جواب:**

- **Authentication (احراز هویت):** پاسخ به سوال "تو کی هستی؟". فرآیند تأیید هویت کاربر (مثلاً با بررسی نام کاربری و رمز عبور). اگر هویت تأیید نشه، خطای 401 Unauthorized برگردونده میشه.
- **Authorization (مجوزدهی/کنترل دسترسی):** پاسخ به سوال "تو چه کاری می‌تونی انجام بدی؟". بعد از احراز هویت، مشخص می‌کنیم که این کاربر به چه منابعی (APIها، صفحات) دسترسی داره. مثلاً کاربر عادی نمیتونه پنل ادمین رو ببینه. اگر دسترسی نداشته باشه، خطای 403 Forbidden برگردونده میشه.

**مثال ساده:**

- پاسپورت تو = **Authentication** (ثابت می‌کنه تو شهروندی).
- ویزای تو = **Authorization** (ثابت می‌کنه اجازه داری وارد یه کشور خاص بشی).

---

### ❓ سوال مصاحبه‌ای ۲: چرا نباید پسورد رو به صورت Plain Text ذخیره کنیم؟ راه درستش چیه؟

**جواب:** اگر دیتابیس به هر دلیلی (هک، خطای انسانی) لو بره، پسورد تمام کاربران فاش میشه. چون خیلی از کاربران از یه پسورد برای سایت‌های مختلف استفاده می‌کنن، این یه فاجعه‌ست.

**راه حل: Hashing (درهم‌سازی)**

- **هش کردن:** یه الگوریتم ریاضی یک‌طرفه (One-way) است. یعنی از روی هش، هرگز نمیشه به پسورد اصلی رسید.
- **Salt (نمک):** یه رشته تصادفی یکتا که به هر پسورد اضافه میشه و بعد ازش هش گرفته میشه. این کار حمله با جدول‌های رنگین‌کمان (Rainbow Table Attacks) رو خنثی می‌کنه.
- **الگوریتم‌های مناسب:** BCrypt، SCrypt یا Argon2. این الگوریتم‌ها عمداً کُند (Slow) طراحی شدن تا حملات Brute-Force رو سخت کنن.
- **در عمل (جاوا/Spring Security):** از `BCryptPasswordEncoder` استفاده می‌کنیم.

---

## 9.2 احراز هویت مدرن: JWT (JSON Web Token)

### ❓ سوال مصاحبه‌ای ۳: JWT چیه، چه ساختاری داره و چطور کار می‌کنه؟ (سوال طلایی مصاحبه‌های Spring)

**جواب:** JWT یه استاندارد باز (RFC 7519) برای انتقال امن اطلاعات (Claims) بین دو طرف به صورت یه JSON فشرده و امضاشده است. بزرگترین مزیتش Stateless (بدون وضعیت) بودنش هست، یعنی سرور نیازی به ذخیره Session ندارد و مقیاس‌پذیری رو بالا می‌بره.

**ساختار یه JWT از سه بخش Base64-encoded تشکیل شده که با نقطه جدا میشن:**
`HEADER.PAYLOAD.SIGNATURE`

1. **Header:** نوع توکن (JWT) و الگوریتم امضا (مثلاً HS256).

```json
   { "alg": "HS256", "typ": "JWT" }
```

```text
{ "sub": "123", "name": "Ali", "role": "ADMIN", "exp": 1680000000 }
```

### Signature (امضا): مهم‌ترین بخش برای امنیت. با این فرمول ساخته میشه

```text
HMACSHA256( base64UrlEncode(header) + "." + base64UrlEncode(payload), SECRET_KEY )
```

### این امضا تضمین می‌کنه که توکن در مسیر بین کلاینت و سرور دستکاری (Tamper) نشده. چون اگه Payload تغییر کنه، امضا دیگر معتبر نخواهد بود

## فرآیند کار

- کاربر با username/password لاگین می‌کنه.

- سرور هویت رو چک می‌کنه و یک JWT امضاءشده برمی‌گردونه.

- کلاینت JWT رو ذخیره می‌کنه (مثلاً توی Local Storage یا Cookie).

- در هر درخواست بعدی، کلاینت توکن رو در هدر Authorization می‌فرسته:

```text
Authorization: Bearer <token>
```

- سرور بدون مراجعه به دیتابیس، با استفاده از SECRET_KEY امضای توکن رو چک می‌کنه. اگر معتبر بود، اطلاعات کاربر رو از Payload استخراج می‌کنه و پردازش انجام میشه.

### نکته حیاتی: Payload فقط Base64 Encode شده، نه Encrypt! بنابراین هرگز اطلاعات حساس مثل پسورد رو توی JWT قرار نده. هر کسی که توکن رو داشته باشه، می‌تونه Payload رو بخونه

---

## 9.3 آسیب‌پذیری‌های رایج (OWASP Top 10)

### ❓ سوال مصاحبه‌ای ۴: SQL Injection چیه و چطور در جاوا ازش جلوگیری می‌کنی؟

**جواب:** وقتی ورودی کاربر مستقیماً (مثلاً از طریق concatenation رشته‌ها) به یک کوئری SQL اضافه بشه، و مهاجم بتونه با تزریق کد SQL ساختار کوئری رو تغییر بده.

```sql
-- a.
-- ورودی کاربر در فیلد username: ' OR '1'='1
-- کوئری نهایی:
SELECT * FROM users WHERE username = '' OR '1'='1' AND password = '...'
```

## راه حل در جاوا

- هرگز و مطلقاً هرگز با + زدن یا String.format() کوئری SQL نساز!

- همیشه از Parameterized Queries (Prepared Statements) استفاده کن.

- در JDBC: استفاده از PreparedStatement:

```java
String query = "SELECT * FROM users WHERE username = ?";
PreparedStatement ps = connection.prepareStatement(query);
ps.setString(1, userInput); // اینجا ورودی امن می‌شود
```

---

### ❓ سوال مصاحبه‌ای ۵: XSS (Cross-Site Scripting) و CSRF رو خیلی خلاصه بگو و راه حلشون چیه؟

- **XSS (حمله با اسکریپت بین سایتی):** مهاجم یه اسکریپت مخرب جاوااسکریپت رو توی وب‌سایت قرار میده که در مرورگر قربانی اجرا میشه.
  - **راه حل:** مهم‌ترین کار همیشه Escape/Encode کردن خروجی‌ها قبل از نمایش در HTML هست. کتابخانه‌هایی مثل Thymeleaf این کار رو خودکار انجام میدن.

- **CSRF (جعل درخواست بین سایتی):** مهاجم قربانی رو فریب میده تا وقتی توی سایت A لاگین هست، ناخواسته یک درخواست مخرب (مثلاً انتقال پول) به سایت A بفرسته.
  - **راه حل:** استفاده از CSRF Token (یه مقدار تصادفی مخفی در فرم‌ها که توسط سرور تولید و بررسی میشه). **نکته برای بک‌اند کار:** اگر API تو Stateless هست و با JWT کار می‌کنی (توکن در هدر میاد، نه Cookie)، به طور پیش‌فرض در برابر CSRF ایمن هستی و معمولاً CSRF protection رو غیرفعال می‌کنن. اما برای برنامه‌های سنتی که با Session و Cookie کار می‌کنن، حتماً باید فعال باشه.

## 9.4 جمع‌بندی بهترین روش‌های امنیتی (Security Best Practices)

- **اصل حداقل دسترسی (Principle of Least Privilege):** به هر کاربر یا سرویس فقط و فقط کمترین سطح دسترسی که برای انجام کارش نیاز داره رو بده.
- **لاگ‌گیری و مانیتورینگ:** تمام تلاش‌های ناموفق برای ورود، خطاهای ۴۰۱ و ۴۰۳، و فعالیت‌های مشکوک رو لاگ کن.
- **به‌روزرسانی وابستگی‌ها:** همیشه کتابخونه‌هات رو آپدیت نگه دار. آسیب‌پذیری‌های معروف (CVEs) دائماً کشف میشن.
- **HTTPS همه جا:** هرگز، تأکید می‌کنم هرگز از HTTP ساده در Production استفاده نکن. همیشه SSL/TLS رو فعال کن.

---

## ۲. Spring Security

## ۹.۵ Spring Security (امنیت در Spring Boot)

Spring Security فریم‌ورک اصلی امنیت در اکوسیستم Spring هست.

### ۹.۵.۱ مفاهیم اصلی Spring Security

```text
┌─────────────────────────────────────────────────────────────────────┐
│                                                                     │
│  ┌─────────────┐    ┌───────────────┐    ┌────────────────────────┐ │
│  │  Username   │    │               │    │     PasswordEncoder    │ │
│  │  Password   │───▶│ Authentication│───▶│     (BCrypt, etc)      │ │
│  │  Filter     │    │    Manager    │    │                        │ │
│  └─────────────┘    └───────────────┘    └────────────────────────┘ │
│         │                  │                         │              │
│         ▼                  ▼                         ▼              │
│  ┌────────────────────────────────────────────────────────────────┐ │
│  │                          SecurityContext                       │ │
│  │                    (ذخیره اطلاعات کاربر لاگین شده)               │ │
│  └────────────────────────────────────────────────────────────────┘ │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### ۹.۵.۲ PasswordEncoder (رمزنگاری پسورد)

```java
@Configuration
class SecurityConfig {
    
    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder(); // قوی و استاندارد
        // یا: return new SCryptPasswordEncoder();
        // یا: return new Argon2PasswordEncoder();
    }
}

// استفاده
@Service
class UserService {
    @Autowired
    private PasswordEncoder passwordEncoder;
    
    public void registerUser(String username, String rawPassword) {
        String encodedPassword = passwordEncoder.encode(rawPassword);
        // ذخیره encodedPassword در دیتابیس
    }
    
    public boolean checkPassword(String rawPassword, String encodedPassword) {
        return passwordEncoder.matches(rawPassword, encodedPassword);
    }
}
```

### ۹.۵.۳ UserDetailsService (بارگذاری کاربر)

```java
@Service
class CustomUserDetailsService implements UserDetailsService {
    
    @Autowired
    private UserRepository repository;
    
    @Override
    public UserDetails loadUserByUsername(String username) 
            throws UsernameNotFoundException {
        
        User user = repository.findByUsername(username)
            .orElseThrow(() -> new UsernameNotFoundException("User not found"));
        
        return User.builder()
            .username(user.getUsername())
            .password(user.getPassword())
            .authorities(user.getRoles().stream()
                .map(role -> new SimpleGrantedAuthority("ROLE_" + role))
                .toArray(String[]::new))
            .build();
    }
}
```

### ۹.۵.۴ JWT در Spring Security

```java
@Component
class JwtService {
    
    @Value("${jwt.secret}")
    private String secretKey;
    
    public String generateToken(String username, List<String> roles) {
        return Jwts.builder()
            .setSubject(username)
            .claim("roles", roles)
            .setIssuedAt(new Date())
            .setExpiration(new Date(System.currentTimeMillis() + 86400000)) // 24h
            .signWith(SignatureAlgorithm.HS256, secretKey)
            .compact();
    }
    
    public String extractUsername(String token) {
        return Jwts.parser()
            .setSigningKey(secretKey)
            .parseClaimsJws(token)
            .getBody()
            .getSubject();
    }
    
    public boolean isTokenValid(String token) {
        try {
            Jwts.parser().setSigningKey(secretKey).parseClaimsJws(token);
            return true;
        } catch (JwtException e) {
            return false;
        }
    }
}

// JWT Filter
@Component
class JwtAuthenticationFilter extends OncePerRequestFilter {
    
    @Autowired
    private JwtService jwtService;
    
    @Autowired
    private CustomUserDetailsService userDetailsService;
    
    @Override
    protected void doFilterInternal(HttpServletRequest request,
                                   HttpServletResponse response,
                                   FilterChain filterChain) {
        String authHeader = request.getHeader("Authorization");
        
        if (authHeader != null && authHeader.startsWith("Bearer ")) {
            String token = authHeader.substring(7);
            String username = jwtService.extractUsername(token);
            
            if (username != null && SecurityContextHolder.getContext().getAuthentication() == null) {
                UserDetails userDetails = userDetailsService.loadUserByUsername(username);
                SecurityContextHolder.getContext().setAuthentication(
                    new UsernamePasswordAuthenticationToken(userDetails, null, userDetails.getAuthorities())
                );
            }
        }
        filterChain.doFilter(request, response);
    }
}
```

### ۹.۵.۵ تنظیمات امنیتی (Security Filter Chain)

```java
@Configuration
@EnableWebSecurity
@EnableMethodSecurity
class SecurityConfig {
    
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        return http
            // ۱. CSRF (برای JWT غیرفعال کن)
            .csrf(csrf -> csrf.disable())
            
            // ۲. تنظیم دسترسی‌ها
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/api/auth/**").permitAll()  // لاگین/ثبت‌نام
                .requestMatchers("/api/admin/**").hasRole("ADMIN")
                .requestMatchers("/api/user/**").hasAnyRole("USER", "ADMIN")
                .anyRequest().authenticated()
            )
            
            // ۳. مدیریت Session (برای JWT Stateless)
            .sessionManagement(session -> session
                .sessionCreationPolicy(SessionCreationPolicy.STATELESS)
            )
            
            // ۴. اضافه کردن JWT Filter
            .addFilterBefore(jwtAuthenticationFilter, UsernamePasswordAuthenticationFilter.class)
            
            .build();
    }
}
```

### ۹.۵.۶ استفاده از @PreAuthorize و @Secured

```java
@RestController
@RequestMapping("/api/users")
class UserController {
    
    // فقط ADMIN می‌تونه
    @PreAuthorize("hasRole('ADMIN')")
    @GetMapping("/all")
    public List<User> getAllUsers() { ... }
    
    // USER یا ADMIN
    @PreAuthorize("hasAnyRole('USER', 'ADMIN')")
    @GetMapping("/{id}")
    public User getUser(@PathVariable Long id) { ... }
    
    // بررسی اینکه کاربر خودش باشه
    @PreAuthorize("#id == authentication.principal.id")
    @PutMapping("/{id}")
    public User updateUser(@PathVariable Long id, @RequestBody User user) { ... }
    
    // فقط USER
    @Secured("ROLE_USER")
    @PostMapping("/profile")
    public User updateProfile() { ... }
}
```

### ۹.۵.۷ Method Security (در Service)

```java
@Service
class OrderService {
    
    // فقط ADMIN
    @PreAuthorize("hasRole('ADMIN')")
    public List<Order> getAllOrders() { ... }
    
    // مدیر شرکت یا صاحب سفارش
    @PreAuthorize("hasRole('ADMIN') or #userId == authentication.principal.id")
    public Order getOrder(Long userId, Long orderId) { ... }
}
```

### ۹.۵.۸ Authentication Entry Point (مدیریت خطاهای امنیتی)

```java
@Component
class CustomAuthenticationEntryPoint implements AuthenticationEntryPoint {
    
    @Override
    public void commence(HttpServletRequest request,
                        HttpServletResponse response,
                        AuthenticationException authException) {
        response.setStatus(HttpServletResponse.SC_UNAUTHORIZED);
        response.setContentType(MediaType.APPLICATION_JSON_VALUE);
        response.getWriter().write("""
            {
                "error": "Unauthorized",
                "message": "Please login first"
            }
            """);
    }
}
```


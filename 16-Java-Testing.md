# 📘 فاز ۱۶: تست‌نویسی (Testing) – کامل و عمیق

تست‌نویسی یکی از مهم‌ترین مهارت‌های یه توسعه‌دهنده حرفه‌ای هست. توی مصاحبه‌های شغلی، ازت انتظار می‌ره که حداقل با مفاهیم پایه تست و ابزارهاش آشنا باشی.

---

## ۱۶.۱ چرا تست‌نویسی؟

**سوال مصاحبه‌ای: چرا باید تست بنویسیم؟**

**جواب:**
1. **اطمینان از صحت کد:** مطمئن میشیم کدمون درست کار می‌کنه
2. **جلوگیری از Regression Bug:** تغییرات جدید، کد قدیمی رو نمی‌شکنه
3. **مستندسازی:** تست‌ها به عنوان مستندات زنده عمل می‌کنن
4. **طراحی بهتر:** کد تست‌پذیر، معمولاً طراحی بهتری داره
5. **اعتمادبه‌نفس:** با خیال راحت می‌تونی کد رو تغییر بدی

---

## ۱۶.۲ انواع تست از نظر سطح (Test Pyramid)

هرم تست (Test Pyramid) به ما می‌گه چه مقدار از هر نوع تست باید داشته باشیم:

```text
                          ╱╲
                         ╱  ╲
                        ╱    ╲
                       ╱ E2E  ╲      ← کمترین تعداد
                      ╱────────╲
                     ╱          ╲
                    ╱ Integration╲    ← تعداد متوسط
                   ╱──────────────╲
                  ╱                ╲
                 ╱    Unit Tests    ╲  ← بیشترین تعداد
                ╱────────────────────╲
```

# جدول انواع تست

| نوع تست | توضیح | مثال | تعداد |
| :--- | :--- | :--- | :---: |
| **Unit Test** | تست یک واحد کوچک (متد یا کلاس) به تنهایی | تست کردن `calculateSalary()` | زیاد ✅ |
| **Integration Test** | تست ارتباط چند جزء با هم | تست `UserService` با `UserRepository` واقعی | متوسط |
| **E2E Test** | تست کل سیستم از اول تا آخر | ثبت‌نام ← لاگین ← خرید ← خروج | کم |


---

## ۱۶.۳ اصول طلایی تست (اصول FIRST)

این اصول پایه‌ای‌ترین چیزهایی هستن که باید در مورد تست بدونی:

| اصل | توضیح کامل | مثال |
|-----|-----------|------|
| Fast (سریع) | تست‌ها باید خیلی سریع اجرا بشن تا بتونی مرتب اجراشون کنی | تست‌های Unit نباید به دیتابیس یا شبکه وصل بشن |
| Independent (مستقل) | هر تست نباید به ترتیب یا نتیجه تست دیگه وابسته باشه | هر تست محیط خودش رو آماده کنه و بعد پاک کنه |
| Repeatable (تکرارپذیر) | هر بار که تست اجرا بشه، نتیجه یکسان باشه | از مقادیر ثابت استفاده کن، نه تاریخ و زمان جاری |
| Self-Validating (خودتأییدگر) | تست باید خودش تشخیص بده قبول شده یا رد شده | استفاده از assert، نه اینکه خروجی رو چشمی چک کنی |
| Timely (به‌موقع) | تست رو قبل یا هم‌زمان با کد بنویس | TDD: اول تست، بعد کد |


## ۱۶.۴ ساختار Given-When-Then (AAA)

این ساختار استاندارد نوشتن تست‌هاست و باعث میشه تست‌ها خواناتر بشن:

```java
@Test
void shouldReturnUserWhenValidUsernameProvided() {
    // GIVEN (یا Arrange) - آماده‌سازی
    UserService service = new UserService();
    String username = "ali";
    
    // WHEN (یا Act) - اجرای عملیات
    User result = service.findUser(username);
    
    // THEN (یا Assert) - تأیید نتیجه
    assertNotNull(result);
    assertEquals("ali", result.getUsername());
}
```

#### نام‌های دیگه این ساختار:

- Arrange-Act-Assert (AAA)

- Given-When-Then (GWT)

---

## ۱۶.۵ تکنیک‌های طراحی تست

### ۱. Boundary Value Analysis (تحلیل مرزها)

**قانون:** خطاها معمولاً در مرزهای ورودی رخ میدن. همیشه مرزها رو تست کن.

```java
// a. متد acceptAge(int age) فقط ۱۸ تا ۶۰ رو قبول می‌کنه

@Test
void testAgeBoundaries() {
    // مرزهای پایین
    assertFalse(validator.acceptAge(17)); // دقیقاً زیر مرز
    assertTrue(validator.acceptAge(18));  // درست روی مرز پایین
    
    // مرزهای بالا
    assertTrue(validator.acceptAge(60));  // درست روی مرز بالا
    assertFalse(validator.acceptAge(61)); // دقیقاً بالای مرز
    
    // داخل بازه
    assertTrue(validator.acceptAge(30));
    assertTrue(validator.acceptAge(45));
}
```

### ۲. Equivalence Partitioning (تقسیم به کلاس‌های هم‌ارز)

**قانون:** ورودی‌ها رو به دسته‌هایی تقسیم کن که رفتار مشابهی دارن. از هر دسته فقط یکی رو تست کن.

```java
// b. متد calculateDiscount(int age) بر اساس سن تخفیف میده

@Test
void testDiscountByAgeGroups() {
    // دسته کودک (کمتر از ۱۲) - یک نمونه کافیه
    assertEquals(0.5, calculator.calculateDiscount(5));
    
    // دسته بزرگسال (۱۲ تا ۶۰)
    assertEquals(0.0, calculator.calculateDiscount(25));
    
    // دسته سالمند (بیشتر از ۶۰)
    assertEquals(0.3, calculator.calculateDiscount(70));
}
```

### ۳. Negative Testing (تست منفی)

**قانون:** فقط مسیر خوش‌بخت رو تست نکن. ورودی‌های غلط رو هم تست کن.

```java
@Test
void testNegativeScenarios() {
    UserService service = new UserService();
    
    // ورودی null
    assertThrows(IllegalArgumentException.class, 
        () -> service.registerUser(null));
    
    // ورودی خالی
    User emptyUser = new User("", "");
    assertThrows(ValidationException.class, 
        () -> service.registerUser(emptyUser));
    
    // پسورد خیلی کوتاه
    User weakUser = new User("ali", "123");
    assertFalse(service.registerUser(weakUser));
    
    // کاربر تکراری
    service.registerUser(new User("ali", "pass123"));
    assertThrows(DuplicateUserException.class,
        () -> service.registerUser(new User("ali", "pass456")));
}
```

### ۴. اصل تک‌مسئولیتی در تست

**قانون:** هر تست فقط یک رفتار رو بررسی کنه.

```java
// a. ❌ بد - چند تا assert با موضوعات مختلف
@Test
void testUserService() {
    User user = service.register("ali", "pass");
    assertNotNull(user);
    assertEquals("ali", user.getUsername());
    assertTrue(user.isActive());
    assertNotNull(user.getCreatedAt());
    // اینجا ۴ تا چیز مختلف رو تست کرده!
}

// b. ✅ خوب - هر تست فقط یه کار
@Test
void shouldCreateUserWithCorrectUsername() {
    User user = service.register("ali", "pass");
    assertEquals("ali", user.getUsername());
}

@Test
void shouldSetUserActiveByDefault() {
    User user = service.register("ali", "pass");
    assertTrue(user.isActive());
}

@Test
void shouldSetCreatedDateOnRegistration() {
    User user = service.register("ali", "pass");
    assertNotNull(user.getCreatedAt());
}
```

### ۵. اصل نام‌گذاری گویا

**۵. اصل نام‌گذاری گویا** اسم تست باید مشخص کنه چه چیزی رو تست میکنه، تحت چه شرایطی، و نتیجه‌ی مورد انتظار چیه.

```java
// a. ✅ نام‌گذاری خوب
@Test
void shouldReturnFalseWhenPasswordIsTooShort() { }

@Test
void shouldThrowExceptionWhenUserNotFound() { }

@Test
void shouldCalculateDiscountCorrectlyForSeniorCitizens() { }

@Test
void shouldNotAllowDuplicateUsernames() { }

// b. ❌ نام‌گذاری بد
@Test
void test1() { }

@Test
void testLogin() { } // معلوم نیست چی رو تست میکنه

@Test
void test() { } // اصلاً چی؟
```

### ۶. اصل جداسازی دغدغه‌ها (Separation of Concerns)

##### قانون: تست‌های واحد باید فقط یک تابع یا کلاس رو تست کنن و بقیه وابستگی‌ها رو با Mock جایگزین کنی.


---

## ۱۶.۶ تست در جاوا با JUnit 5

JUnit 5 محبوب‌ترین فریم‌ورک تست در جاواست.

### ۱۶.۶.۱ مفاهیم پایه JUnit

```java
import org.junit.jupiter.api.*;
import static org.junit.jupiter.api.Assertions.*;

class CalculatorTest {
    
    private Calculator calculator;
    
    @BeforeAll
    static void beforeAll() {
        System.out.println("قبل از همه تست‌ها");
    }
    
    @AfterAll
    static void afterAll() {
        System.out.println("بعد از همه تست‌ها");
    }
    
    @BeforeEach
    void setUp() {
        calculator = new Calculator();
    }
    
    @AfterEach
    void tearDown() {
        System.out.println("بعد از هر تست");
    }
    
    @Test
    @DisplayName("باید جمع دو عدد رو درست محاسبه کنه")
    void shouldAddTwoNumbersCorrectly() {
        int a = 5;
        int b = 3;
        int result = calculator.add(a, b);
        assertEquals(8, result, "جمع ۵ و ۳ باید ۸ بشه");
    }
    
    @Test
    void shouldThrowExceptionWhenDividingByZero() {
        assertThrows(ArithmeticException.class,
            () -> calculator.divide(10, 0));
    }
}
```

### ۱۶.۶.۲ Assertionهای مهم JUnit

```java
// a. بررسی برابری
assertEquals(4, calculator.add(2, 2));
assertEquals(3.14, PI, 0.001);

// b. بررسی نابرابری
assertNotEquals(5, calculator.add(2, 2));

// c. بررسی null بودن
assertNull(user);
assertNotNull(user);

// d. بررسی شرط
assertTrue(validator.isValid("ali"));
assertFalse(validator.isValid(""));

// e. بررسی استثنا
assertThrows(ArithmeticException.class, 
    () -> calculator.divide(10, 0));

// f. بررسی همه شرط‌ها
assertAll("user validation",
    () -> assertNotNull(user),
    () -> assertTrue(user.isActive()),
    () -> assertEquals("ali", user.getUsername())
);

// g. بررسی آرایه‌ها
assertArrayEquals(expectedArray, actualArray);

// h. بررسی timeout
assertTimeout(Duration.ofSeconds(1),
    () -> service.heavyOperation());
```

### ۱۶.۶.۳ تست‌های پارامتری (Parameterized Tests)

```java
import org.junit.jupiter.params.ParameterizedTest;
import org.junit.jupiter.params.provider.*;

class ParameterizedTestExample {
    
    @ParameterizedTest
    @ValueSource(ints = {2, 4, 6, 8, 10})
    void shouldReturnTrueForEvenNumbers(int number) {
        assertTrue(calculator.isEven(number));
    }
    
    @ParameterizedTest
    @CsvSource({
        "5, 3, 8",
        "10, 20, 30",
        "-5, 3, -2"
    })
    void shouldAddNumbersCorrectly(int a, int b, int expected) {
        assertEquals(expected, calculator.add(a, b));
    }
    
    @ParameterizedTest
    @EnumSource(DayOfWeek.class)
    void shouldHaveCorrectDayName(DayOfWeek day) {
        assertNotNull(day.name());
    }
}
```

---

## ۱۶.۷ Mock و Stub (شبیه‌سازی)

**چرا؟** وقتی یه کلاس وابستگی به دیتابیس یا سرویس خارجی داره، برای تست باید اون وابستگی رو شبیه‌سازی کنیم.

### ۱۶.۷.۱ تفاوت Stub و Mock

| مفهوم | توضیح | پیچیدگی | کاربرد |
|-------|-------|---------|--------|
| Stub | یه پیاده‌سازی ساده که همیشه یه مقدار ثابت برمی‌گردونه | ساده | وقتی فقط به یه مقدار خاص نیاز داری |
| Mock | یه شیء شبیه‌سازی شده که می‌تونی روش Expectation تعریف کنی و Verify کنی | پیچیده | وقتی می‌خوای رفتار رو کنترل و بررسی کنی |

### ۱۶.۷.۲ مثال Stub (ساده)

```java
class EmailServiceStub implements EmailService {
    @Override
    public boolean sendEmail(String to, String body) {
        return true;
    }
}

class UserServiceTest {
    private UserService service;
    
    @BeforeEach
    void setUp() {
        EmailService emailService = new EmailServiceStub();
        service = new UserService(emailService);
    }
    
    @Test
    void shouldRegisterUser() {
        assertTrue(service.register("ali", "pass"));
    }
}
```

### ۱۶.۷.۳ مثال Mock با Mockito

```java
import org.mockito.*;
import static org.mockito.Mockito.*;

class UserServiceTest {
    
    @Mock
    private UserRepository repository;
    
    @Mock
    private EmailService emailService;
    
    @InjectMocks
    private UserService service;
    
    @BeforeEach
    void setUp() {
        MockitoAnnotations.openMocks(this);
    }
    
    @Test
    void shouldReturnUserWhenFound() {
        User mockUser = new User("ali", "pass");
        when(repository.findByUsername("ali"))
            .thenReturn(Optional.of(mockUser));
        
        User result = service.findUser("ali");
        
        assertNotNull(result);
        assertEquals("ali", result.getUsername());
        verify(repository).findByUsername("ali");
    }
    
    @Test
    void shouldThrowExceptionWhenUserNotFound() {
        when(repository.findByUsername("unknown"))
            .thenReturn(Optional.empty());
        
        assertThrows(UserNotFoundException.class,
            () -> service.findUser("unknown"));
    }
    
    @Test
    void shouldSendEmailAfterRegistration() {
        User user = new User("ali", "pass");
        when(repository.save(any(User.class))).thenReturn(user);
        
        service.register("ali", "pass");
        
        verify(emailService).sendEmail(eq("ali"), anyString());
        verify(emailService, times(1)).sendEmail(any(), any());
    }
}
```

### ۱۶.۷.۴ متدهای پرکاربرد Mockito

```java
// a. تنظیم رفتار
when(mock.method()).thenReturn(value);
when(mock.method()).thenThrow(exception);
when(mock.method()).thenReturn(value1).thenReturn(value2);
doThrow(exception).when(mock).voidMethod();

// b. Matcherها
when(repository.findById(anyLong())).thenReturn(user);
when(repository.findById(eq(1L))).thenReturn(user);
when(repository.findByUsername(anyString())).thenReturn(user);

// c. Verification
verify(mock).method();
verify(mock, times(2)).method();
verify(mock, never()).method();
verify(mock, atLeast(1)).method();
verify(mock, atMost(3)).method();
verifyNoMoreInteractions(mock);

// d. Spying
@Spy
private UserService realService;
when(realService.expensiveMethod()).thenReturn("mock result");
```

---

## ۱۶.۸ تست در Spring Boot

Spring Boot ابزارهای قدرتمندی برای تست داره.

### ۱۶.۸.۱ تست لایه Controller (با MockMvc)

```java
import org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTest;
import org.springframework.test.web.servlet.MockMvc;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.*;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.*;

@WebMvcTest(UserController.class)
class UserControllerTest {
    
    @Autowired
    private MockMvc mockMvc;
    
    @MockBean
    private UserService userService;
    
    @Test
    void shouldReturnUserWhenExists() throws Exception {
        User user = new User(1L, "ali");
        when(userService.findUser(1L)).thenReturn(user);
        
        mockMvc.perform(get("/api/users/1")
                .contentType(MediaType.APPLICATION_JSON))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.username").value("ali"))
            .andExpect(jsonPath("$.id").value(1));
    }
    
    @Test
    void shouldReturn404WhenUserNotFound() throws Exception {
        when(userService.findUser(999L))
            .thenThrow(new UserNotFoundException());
        
        mockMvc.perform(get("/api/users/999"))
            .andExpect(status().isNotFound());
    }
}
```

### ۱۶.۸.۲ تست لایه Service (با Mock)

```java
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.boot.test.mock.mockito.MockBean;

@SpringBootTest
class UserServiceTest {
    
    @MockBean
    private UserRepository repository;
    
    @Autowired
    private UserService service;
    
    @Test
    void shouldRegisterNewUser() {
        User newUser = new User("ali", "pass123");
        when(repository.save(any(User.class)))
            .thenReturn(newUser);
        
        User result = service.register("ali", "pass123");
        
        assertNotNull(result);
        verify(repository).save(any(User.class));
    }
}
```

### ۱۶.۸.۳ تست لایه Repository (با دیتابیس واقعی)

```java
import org.springframework.boot.test.autoconfigure.orm.jpa.DataJpaTest;

@DataJpaTest
class UserRepositoryTest {
    
    @Autowired
    private UserRepository repository;
    
    @Test
    void shouldFindUserByUsername() {
        User user = new User("ali", "pass");
        repository.save(user);
        
        Optional<User> found = repository.findByUsername("ali");
        
        assertTrue(found.isPresent());
        assertEquals("ali", found.get().getUsername());
    }
}
```

### ۱۶.۸.۴ تست Integration (کل سیستم)

```java
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.boot.test.web.client.TestRestTemplate;
import org.springframework.boot.test.web.server.LocalServerPort;

@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
class IntegrationTest {
    
    @LocalServerPort
    private int port;
    
    @Autowired
    private TestRestTemplate restTemplate;
    
    @Test
    void shouldRegisterAndLoginUser() {
        User user = new User("ali", "pass");
        ResponseEntity<Void> registerResponse = restTemplate
            .postForEntity("http://localhost:" + port + "/api/register", 
                          user, Void.class);
        assertEquals(HttpStatus.CREATED, registerResponse.getStatusCode());
        
        LoginRequest request = new LoginRequest("ali", "pass");
        ResponseEntity<TokenResponse> loginResponse = restTemplate
            .postForEntity("http://localhost:" + port + "/api/login", 
                          request, TokenResponse.class);
        assertEquals(HttpStatus.OK, loginResponse.getStatusCode());
        assertNotNull(loginResponse.getBody().getToken());
    }
}
```

---

## ۱۶.۹ TDD (Test-Driven Development)

**چرخه TDD (Red-Green-Refactor):**

```text
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│   🔴 RED (قرمز)                                            │
│   اول یک تست بنویس که رد میشه (چون کد وجود نداره)            │
│                         ↓                                   │
│   🟢 GREEN (سبز)                                           │
│   حداقل کد رو بنویس تا تست قبول بشه                         │
│                         ↓                                   │
│   🔵 REFACTOR (بازسازی)                                    │
│   کد رو بهینه کن و تمیزش کن (تست‌ها باید سبز بمونن)         │
│                         ↓                                   │
│  (برگرد به مرحله ۱ برای ویژگی بعدی)                         │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### مثال کامل TDD:

```java
// a. مرحله ۱: RED
@Test
void shouldReturnTrueForEvenNumber() {
    assertTrue(validator.isEven(4));
}

// b. مرحله ۲: GREEN
public class NumberValidator {
    public boolean isEven(int number) {
        return number % 2 == 0;
    }
}

// c. مرحله ۳: REFACTOR
// (در صورت نیاز)

// ویژگی بعدی
@Test
void shouldReturnTrueForOddNumber() {
    assertTrue(validator.isOdd(5));
}
```

### ۱۶.۱۰ Coverage (پوشش تست)

#### سوال مصاحبه‌ای: Code Coverage چیست و چقدر باید باشد؟

جواب: درصد کدی که توسط تست‌ها پوشش داده شده. ابزارهایی مثل JaCoCo این رو اندازه می‌گیرن.

##### انواع Coverage:

- **Line Coverage:** چند درصد خطوط کد اجرا شدن

- **Branch Coverage:** چند درصد شرط‌ها (if/else) تست شدن

- **Method Coverage:** چند درصد متدها تست شدن

##### توصیه:

- هدف: ۷۰-۸۰٪ برای پروژه‌های معمولی

- ۱۰۰٪ همیشه ممکن نیست و گاهی هزینه‌بره

- کیفیت تست مهم‌تر از تعدادشه

```xml
<plugin>
    <groupId>org.jacoco</groupId>
    <artifactId>jacoco-maven-plugin</artifactId>
    <configuration>
        <rules>
            <rule>
                <element>CLASS</element>
                <limits>
                    <limit>
                        <counter>LINE</counter>
                        <value>COVEREDRATIO</value>
                        <minimum>0.80</minimum>
                    </limit>
                </limits>
            </rule>
        </rules>
    </configuration>
</plugin>
```

### ۱۶.۱۱ خلاصه یک‌صفحه‌ای برای مصاحبه

# خلاصه مفاهیم کلیدی تست نرم‌افزار

| مفهوم | خلاصه |
|-------|-------|
| **Test Pyramid** | Unit > Integration > E2E (هرچه بالاتر، تعداد تست‌ها کمتر) |
| **FIRST** | Fast, Independent, Repeatable, Self-Validating, Timely |
| **Given-When-Then** | ساختار استاندارد نوشتن تست (زمینه - اقدام - انتظار) |
| **Boundary Values** | همیشه مرزها رو تست کن (۱۸، ۶۰، ۰، ۱۲۰، ...) |
| **Negative Testing** | ورودی‌های غلط و غیرمنتظره رو هم تست کن |
| **Mock vs Stub** | Mock قابل برنامه‌ریزی با انتظار رفتاری، Stub پاسخ‌دهنده ساده |
| **TDD** | Red ← Green ← Refactor (چرخه توسعه تست‌محور) |
| **JUnit** | فریم‌ورک اصلی تست در جاوا |
| **Mockito** | کتابخانه اصلی Mock در جاوا |
| **@SpringBootTest** | تست کامل Spring Boot با بارگذاری کل Context |
| **@DataJpaTest** | تست لایه دیتابیس (تنها Repository‌ها و Hibernate) |
| **@WebMvcTest** | تست لایه Controller (تنها Web Layer) |
| **Coverage** | هدف: ۷۰-۸۰٪ (کیفیت مهم‌تر از درصد) |

# ۱۶.۱۲ سوالات رایج مصاحبه‌ای تست

## ❓ سوال ۱: فرق Unit Test و Integration Test چیه؟

**جواب:**

- **Unit Test:** یک واحد کوچک رو به تنهایی تست میکنه. وابستگی‌ها رو Mock میکنه. سریع و ارزونه.
- **Integration Test:** ارتباط چند جزء رو با هم تست میکنه. از دیتابیس یا سرویس واقعی استفاده میکنه. کندتر و گرون‌تره.

---

## ❓ سوال ۲: چطور از تست‌های کند جلوگیری کنیم؟

**جواب:**

- تست‌های Unit رو از Integration جدا کن
- از `@Profile("test")` برای دیتابیس تستی استفاده کن
- از H2 (دیتابیس In-Memory) برای تست استفاده کن
- Mock کردن وابستگی‌های سنگین (مثل سرویس‌های خارجی)

---

## ❓ سوال ۳: فرق `@Mock` و `@MockBean` چیه؟

**جواب:**

- `@Mock`: از Mockito میاد و فقط یه Mock ساده می‌سازه (بدون Spring)
- `@MockBean`: از Spring Boot میاد و یه Mock رو توی Spring Context قرار میده (برای تست‌های Spring)

---

## ❓ سوال ۴: چطور یه کلاس رو تست‌پذیر کنیم؟

**جواب:**

- وابستگی‌ها رو از طریق Constructor تزریق کن (نه new کردن داخل کلاس)
- از Interfaceها استفاده کن تا بتونی Mock کنی
- متدها رو کوتاه و با یک مسئولیت بنویس
- از Static methodها تا حد امکان کمتر استفاده کن

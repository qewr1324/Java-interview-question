## 1. Thymeleaf و JSP

### 📍 یه فایل جدید درست کن به نام `17-Java-View-Technologies.md`

# 📘 فاز ۱۷: تکنولوژی‌های View (Thymeleaf و JSP)

این بخش برای موقعی‌هایی که می‌خوای یه برنامه وب با Frontend سمت سرور (Server-Side Rendering) بسازی.

---

## ۱۷.۱ Thymeleaf (موتور قالب‌سازی مدرن)

Thymeleaf یه موتور قالب‌سازی (Template Engine) برای برنامه‌های Spring Boot هست که به HTML شباهت داره.

### ۱۷.۱.۱ مزایای Thymeleaf

1. **Natural Templating:** قالب‌ها در مرورگر هم قابل مشاهده هستن
2. **یکپارچگی عالی با Spring Boot**
3. **پشتیبانی از i18n (چندزبانه)**
4. **امنیت بالا (XSS Protection)**

### ۱۷.۱.۲ تنظیمات

```xml
<!-- pom.xml -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
```

```properties
# application.properties
spring.thymeleaf.prefix=classpath:/templates/
spring.thymeleaf.suffix=.html
spring.thymeleaf.cache=false  # برای توسعه
spring.thymeleaf.encoding=UTF-8
```

### ۱۷.۱.۳ Controller

```java
@Controller
@RequestMapping("/users")
class UserController {
    
    @Autowired
    private UserService userService;
    
    @GetMapping("/list")
    public String listUsers(Model model) {
        List<User> users = userService.findAll();
        model.addAttribute("users", users);
        model.addAttribute("pageTitle", "لیست کاربران");
        return "users/list"; // templates/users/list.html
    }
    
    @GetMapping("/{id}")
    public String getUser(@PathVariable Long id, Model model) {
        User user = userService.findById(id);
        model.addAttribute("user", user);
        return "users/profile";
    }
    
    @GetMapping("/new")
    public String newUserForm(Model model) {
        model.addAttribute("user", new User());
        return "users/form";
    }
    
    @PostMapping("/save")
    public String saveUser(@ModelAttribute User user) {
        userService.save(user);
        return "redirect:/users/list";
    }
}
```

### ۱۷.۱.۴ قالب Thymeleaf


```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <title th:text="${pageTitle}">عنوان پیش‌فرض</title>
</head>
<body>
    <!-- ۱. نمایش متغیر -->
    <h1 th:text="${pageTitle}">عنوان</h1>
    
    <!-- ۲. شرط (if) -->
    <div th:if="${user != null}">
        <p>خوش آمدی، <span th:text="${user.username}">کاربر</span></p>
    </div>
    
    <!-- ۳. شرط (unless - برعکس if) -->
    <div th:unless="${user != null}">
        <p>لطفاً وارد شوید</p>
    </div>
    
    <!-- ۴. حلقه (forEach) -->
    <table>
        <tr th:each="user : ${users}">
            <td th:text="${user.id}">1</td>
            <td th:text="${user.username}">علی</td>
            <td th:text="${user.age}">25</td>
            <td>
                <!-- ۵. لینک -->
                <a th:href="@{/users/{id}(id=${user.id})}">مشاهده</a>
            </td>
        </tr>
    </table>
    
    <!-- ۶. فرم -->
    <form th:action="@{/users/save}" th:object="${user}" method="post">
        <input type="text" th:field="*{username}" placeholder="نام کاربری"/>
        <input type="password" th:field="*{password}" placeholder="رمز عبور"/>
        <input type="number" th:field="*{age}" placeholder="سن"/>
        <button type="submit">ذخیره</button>
    </form>
    
    <!-- ۷. نمایش خطاهای اعتبارسنجی -->
    <div th:if="${#fields.hasErrors('username')}">
        <span th:errors="*{username}">خطای اعتبارسنجی</span>
    </div>
    
    <!-- ۸. شامل کردن قالب دیگه -->
    <div th:include="fragments/header :: header"></div>
    
    <!-- ۹. مقادیر پیش‌فرض -->
    <p th:text="${user?.username} ?: 'کاربر مهمان'">کاربر مهمان</p>
    
    <!-- ۱۰. عملیات ریاضی و منطقی -->
    <p th:text="${user.age + 5}">0</p>
    <p th:text="${user.age > 18} ? 'بزرگسال' : 'کودک'">بزرگسال</p>
</body>
</html>
```

### ۱۷.۱.۵ Fragmentها (اجزای قابل استفاده مجدد)

```html
<!-- fragments/header.html -->
<div th:fragment="header">
    <header>
        <h1>سایت من</h1>
        <nav>
            <a th:href="@{/}">خانه</a>
            <a th:href="@{/users/list}">کاربران</a>
        </nav>
    </header>
</div>

<!-- استفاده در صفحه -->
<div th:replace="fragments/header :: header"></div>
```

### ۱۷.۱.۶ i18n (چندزبانه)

```properties
# messages_fa.properties
welcome=خوش آمدید
username=نام کاربری
password=رمز عبور

# messages_en.properties
welcome=Welcome
username=Username
password=Password
```

```html
<!-- in html -->
<h1 th:text="#{welcome}">Welcome</h1>
<label th:text="#{username}">Username</label>
```

### ۱۷.۱.۷ Security با Thymeleaf

```xml
<dependency>
    <groupId>org.thymeleaf.extras</groupId>
    <artifactId>thymeleaf-extras-springsecurity6</artifactId>
</dependency>
```

```html
<!-- Display logged in user information -->
<!-- نمایش اطلاعات کاربر لاگین شده است -->
<div sec:authorize="isAuthenticated()">
    <p>خوش آمدی، <span sec:authentication="name">کاربر</span></p>
    <a sec:authorize="hasRole('ADMIN')" href="/admin">پنل مدیریت</a>
</div>

<div sec:authorize="!isAuthenticated()">
    <a href="/login">ورود</a>
</div>
```

## ۱۷.۲ JSP (JavaServer Pages)

**JSP** یه تکنولوژی قدیمی‌تر ولی هنوز در برخی پروژه‌ها استفاده میشه.

### ۱۷.۲.۱ تنظیمات

```xml
<!-- pom.xml -->
<dependency>
    <groupId>org.apache.tomcat.embed</groupId>
    <artifactId>tomcat-embed-jasper</artifactId>
    <scope>provided</scope>
</dependency>
<dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>jstl</artifactId>
</dependency>
```

```properties
# application.properties
spring.mvc.view.prefix=/WEB-INF/views/
spring.mvc.view.suffix=.jsp
```

### ۱۷.۲.۲ Controller برای JSP

```java
@Controller
@RequestMapping("/users")
class UserController {
    
    @GetMapping("/list")
    public String listUsers(Model model) {
        List<User> users = userService.findAll();
        model.addAttribute("users", users);
        return "users/list"; // /WEB-INF/views/users/list.jsp
    }
}
```

### ۱۷.۲.۳ قالب JSP (با JSTL)

```html
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
<%@ taglib prefix="fmt" uri="http://java.sun.com/jsp/jstl/fmt" %>
<!DOCTYPE html>
<html>
<head>
    <title>لیست کاربران</title>
</head>
<body>
    <h1>${pageTitle}</h1>
    
    <!-- ۱. شرط -->
    <c:if test="${not empty user}">
        <p>خوش آمدی، ${user.username}</p>
    </c:if>
    
    <!-- ۲. حلقه -->
    <table>
        <c:forEach items="${users}" var="user">
            <tr>
                <td>${user.id}</td>
                <td>${user.username}</td>
                <td>${user.age}</td>
                <td>
                    <a href="/users/${user.id}">مشاهده</a>
                </td>
            </tr>
        </c:forEach>
    </table>
    
    <!-- ۳. فرم -->
    <form action="/users/save" method="post">
        <input type="text" name="username" placeholder="نام کاربری"/>
        <input type="password" name="password" placeholder="رمز عبور"/>
        <input type="number" name="age" placeholder="سن"/>
        <button type="submit">ذخیره</button>
    </form>
    
    <!-- ۴. بررسی خالی بودن -->
    <c:if test="${empty users}">
        <p>هیچ کاربری وجود ندارد</p>
    </c:if>
</body>
</html>
```

### ۱۷.۲.۴ مقایسه Thymeleaf و JSP

# مقایسه Thymeleaf و JSP

| ویژگی | Thymeleaf | JSP |
|-------|-----------|-----|
| **تازه بودن** | مدرن (2011+) | قدیمی (1999+) |
| **یکپارچگی با Spring Boot** | عالی ✅ | متوسط |
| **Natural Templating** | بله ✅ |     خیر ❌ |
| **ایمنی در برابر XSS** | خودکار ✅ | دستی ❌ |
| **پشتیبانی از Layout** | عالی ✅ | ضعیف |
| **سرعت** | متوسط | سریع‌تر |
| **یادگیری** | آسان | سخت‌تر |
| **مناسب برای** | پروژه‌های جدید | پروژه‌های قدیمی |


### ۱۷.۳ خلاصه انتخاب View Technology

```text
┌─────────────────────────────────────────────────────────────────────┐
│                                                                     │
│                         پروژه جدید یا ری‌اکتیو؟                      │
│                                 │                                   │
│                         ┌───────┴───────┐                           │
│                         ▼               ▼                           │
│                    پروژه جدید     پروژه قدیمی                       │
│                         │               │                           │
│                         ▼               ▼                           │
│                   Thymeleaf         JSP (نگهداری)                   │
│                         │                                           │
│                         ▼                                           │
│              چرا Thymeleaf؟                                         │
│              ✅ یکپارچگی عالی با Spring                             │
│              ✅ امنیت بیشتر                                         │
│              ✅ خواناتر و تمیزتر                                    │
│              ✅ پشتیبانی از i18n                                    │
│              ✅ Fragments و Layouts                                 │
│              ✅ Community Support قوی                               │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

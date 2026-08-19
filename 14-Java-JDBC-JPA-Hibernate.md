# 📘 فاز ۱۴: JDBC، JPA و Hibernate

## ۱۴.۱ JDBC (Java Database Connectivity)

**چیست؟** پایه‌ای‌ترین راه ارتباط با دیتابیس در جاوا.

**مراحل کار با JDBC:**

```java
// a. بارگذاری Driver
Class.forName("com.mysql.cj.jdbc.Driver");

// b. ایجاد Connection
Connection conn = DriverManager.getConnection(
    "jdbc:mysql://localhost:3306/mydb",
    "root",
    "password"
);

// c. ساخت Statement
Statement stmt = conn.createStatement();

// d. اجرای Query
ResultSet rs = stmt.executeQuery("SELECT * FROM users");

// e. پردازش نتایج
while (rs.next()) {
    String name = rs.getString("name");
    int age = rs.getInt("age");
    System.out.println(name + " - " + age);
}

// f. بستن منابع (مهم!)
rs.close();
stmt.close();
conn.close();
```

## ۱۴.۱ JDBC و PreparedStatement

### ❗️ PreparedStatement (مهم برای جلوگیری از SQL Injection)

```java
// ✅ PreparedStatement
String query = "SELECT * FROM users WHERE username = ? AND password = ?";
PreparedStatement ps = conn.prepareStatement(query);
ps.setString(1, username);
ps.setString(2, password);
ResultSet rs = ps.executeQuery();

// ❌ concatenation (خطر SQL Injection!)
String query = "SELECT * FROM users WHERE username = '" + username + "'";
Statement stmt = conn.createStatement();
ResultSet rs = stmt.executeQuery(query);
```

---

## ۱۴.۲ JPA (Java Persistence API)

**چیست؟** یه استاندارد (Specification) برای Mapping اشیاء جاوا به جداول دیتابیس.

### مفاهیم کلیدی:

| مفهوم | توضیح |
|-------|--------|
| `@Entity` | کلاس رو به یه جدول دیتابیس تبدیل می‌کنه |
| `@Id` | کلید اصلی رو مشخص می‌کنه |
| `@GeneratedValue` | تولید خودکار مقدار کلید اصلی |
| `@Column` | نگاشت فیلد به ستون دیتابیس |
| `@OneToMany`, `@ManyToOne` | روابط بین جداول |

```java
@Entity
@Table(name = "users")
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(nullable = false, unique = true)
    private String username;
    
    @Column(nullable = false)
    private String password;
    
    @OneToMany(mappedBy = "user", cascade = CascadeType.ALL)
    private List<Order> orders;
    
    // getters and setters
}
```

---

## ۱۴.۳ Hibernate (پیاده‌سازی JPA)

**چیست؟** محبوب‌ترین پیاده‌سازی JPA که کارهای زیر رو انجام میده:

- تبدیل خودکار Object به SQL (و برعکس)
- مدیریت Cache (کش)
- مدیریت Transaction
- Lazy Loading (بارگذاری دیر هنگام)

### ❗️ سوال مصاحبه‌ای: N+1 Query Problem چیه و چطور حل میشه؟

**مشکل:** وقتی یه Entity رو می‌خونی و بعد به Property که Lazy هست دسترسی پیدا می‌کنی، Hibernate یه Query جداگانه می‌زنه.

```java
// a. این کوئری ۱ بار اجرا میشه
List<User> users = userRepository.findAll();

// b. اینجا برای هر کاربر یه کوئری جداگانه زده میشه (N بار)
for (User user : users) {
    System.out.println(user.getOrders().size()); // N+1 Query!
}
```

- راه حل‌ها:

#### ۱. JOIN FETCH:

```java
@Query("SELECT u FROM User u JOIN FETCH u.orders")
List<User> findAllWithOrders();
```

#### ۲. EntityGraph:

```java
@EntityGraph(attributePaths = {"orders"})
List<User> findAll();
```

#### ۳. @NamedEntityGraph:

```java
@Entity
@NamedEntityGraph(name = "User.orders", attributeNodes = @NamedAttributeNode("orders"))
public class User { ... }
```

---

### ❗️ سوال مصاحبه‌ای: تفاوت FetchType.EAGER و LAZY

| FetchType | توضیح | مزیت | عیب |
|-----------|--------|------|------|
| `EAGER` | داده‌های مرتبط رو همین الان بارگذاری کن | ساده‌تر | کندتر، مصرف حافظه بیشتر |
| `LAZY` (پیش‌فرض) | فقط وقتی نیاز شد بارگذاری کن | سریع‌تر، کم‌مصرف | ممکنه به N+1 مشکل بخوری |


## ۱۴.۴ مقایسه JDBC vs JPA vs Hibernate

```text
JDBC (پایین‌ترین سطح)
│
▼
JPA (استاندارد - فقط Interface)
│
▼
Hibernate (پیاده‌سازی - محبوب‌ترین)
│
▼
Spring Data JPA (راحت‌تر از Hibernate خام)
```

| ویژگی | JDBC | JPA | Hibernate |
|--------|------|-----|-----------|
| سطح | پایین | متوسط | بالا |
| کنترل | کامل | کمتر | کمترین |
| سرعت | سریع‌ترین | متوسط | کندتر (به خاطر Overhead) |
| راحتی | سخت | راحت | بسیار راحت |
| کد نویسی | زیاد | متوسط | کم |

## ۱۴.۵ Spring Data JPA (لایه راحت‌تر از Hibernate)

Spring Data JPA یه لایه بالاتر از Hibernate هست که نوشتن کدهای دیتابیس رو خیلی راحت‌تر می‌کنه.

### ۱۴.۵.۱ چرا Spring Data JPA؟

```java
// a. بدون Spring Data JPA (با Hibernate خام)
Session session = entityManager.unwrap(Session.class);
Query query = session.createQuery("FROM User WHERE username = :username");
query.setParameter("username", "ali");
User user = (User) query.uniqueResult();

// b. با Spring Data JPA - خیلی ساده‌تر!
User user = userRepository.findByUsername("ali");
```

### ۱۴.۵.۲ Repositoryها در Spring Data

```java
// a. JpaRepository - کامل‌ترین
public interface UserRepository extends JpaRepository<User, Long> {
    // متدهای آماده: save(), findById(), findAll(), delete(), count()
}

// b. CrudRepository - پایه‌ای‌تر
public interface UserRepository extends CrudRepository<User, Long> { }

// c. PagingAndSortingRepository - برای صفحه‌بندی
public interface UserRepository extends PagingAndSortingRepository<User, Long> { }
```

### ۱۴.۵.۳ متدهای Query Method (نوشتن کوئری با اسم متد)

```java
public interface UserRepository extends JpaRepository<User, Long> {
    
    // پیدا کردن با username
    User findByUsername(String username);
    
    // پیدا کردن با username و password
    User findByUsernameAndPassword(String username, String password);
    
    // پیدا کردن همه کاربران با سن بیشتر از
    List<User> findByAgeGreaterThan(int age);
    
    // پیدا کردن کاربرانی که اسمشون با ... شروع میشه
    List<User> findByUsernameStartingWith(String prefix);
    
    // مرتب‌سازی
    List<User> findAllByOrderByUsernameAsc();
    
    // صفحه‌بندی
    Page<User> findAll(Pageable pageable);
    
    // بررسی وجود داشتن
    boolean existsByUsername(String username);
    
    // شمارش
    long countByAgeGreaterThan(int age);
    
    // حذف
    void deleteByUsername(String username);
}
```

### ۱۴.۵.۴ @Query (نوشتن کوئری دستی)

```java
public interface UserRepository extends JpaRepository<User, Long> {
    
    // ۱. JPQL (مخصوص JPA)
    @Query("SELECT u FROM User u WHERE u.username = :username")
    User findUserByUsername(@Param("username") String username);
    
    // ۲. Native Query (SQL خام)
    @Query(value = "SELECT * FROM users WHERE age > :age", nativeQuery = true)
    List<User> findUsersOlderThan(@Param("age") int age);
    
    // ۳. کوئری با JOIN
    @Query("SELECT u FROM User u JOIN u.orders o WHERE o.total > :amount")
    List<User> findUsersWithOrdersAbove(@Param("amount") double amount);
}
```

### ۱۴.۵.۵ @Modifying (برای UPDATE و DELETE)

```java
public interface UserRepository extends JpaRepository<User, Long> {
    
    @Modifying
    @Query("UPDATE User u SET u.active = false WHERE u.lastLogin < :date")
    int deactivateInactiveUsers(@Param("date") LocalDate date);
    
    @Modifying
    @Query("DELETE FROM User u WHERE u.active = false")
    int deleteInactiveUsers();
}
```

### ۱۴.۵.۶ صفحه‌بندی (Pagination)

```java
@Service
class UserService {
    @Autowired
    private UserRepository repository;
    
    public Page<User> getUsers(int page, int size) {
        // صفحه ۰، هر صفحه ۱۰ تا
        Pageable pageable = PageRequest.of(page, size);
        return repository.findAll(pageable);
    }
    
    public Page<User> getSortedUsers(int page, int size) {
        Pageable pageable = PageRequest.of(page, size, Sort.by("username").ascending());
        return repository.findAll(pageable);
    }
}
```

### ۱۴.۵.۷ Specification (کوئری داینامیک)

```java
// a. برای وقتی که فیلترها متغیر هستن
public class UserSpecification {
    public static Specification<User> hasUsername(String username) {
        return (root, query, criteriaBuilder) -> 
            criteriaBuilder.equal(root.get("username"), username);
    }
    
    public static Specification<User> isActive() {
        return (root, query, criteriaBuilder) -> 
            criteriaBuilder.isTrue(root.get("active"));
    }
}

// b. استفاده
@Service
class UserService {
    public List<User> filterUsers(String username, Boolean active) {
        Specification<User> spec = Specification.where(null);
        
        if (username != null) {
            spec = spec.and(UserSpecification.hasUsername(username));
        }
        if (active != null) {
            spec = spec.and(UserSpecification.isActive());
        }
        
        return repository.findAll(spec);
    }
}
```


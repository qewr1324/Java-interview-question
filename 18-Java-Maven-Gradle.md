# 📘 فاز ۱۸: Maven و Gradle – مدیریت وابستگی‌ها و Build

Maven و Gradle دو ابزار اصلی برای مدیریت پروژه‌های جاوا هستن. در مصاحبه‌ها معمولاً ازت می‌پرسن با کدوم کار کردی و فرقشون چیه.

---

## ۱۸.۱ Maven (محبوب‌ترین)

### ساختار پروژه Maven

```text
my-app/
├── pom.xml
├── src/
│   ├── main/
│   │   ├── java/          ← کدهای جاوا
│   │   └── resources/     ← فایل‌های تنظیمات
│   └── test/
│       ├── java/          ← تست‌ها
│       └── resources/     ← منابع تست
└── target/                ← خروجی build
```

### فایل pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0">
    
    <!-- ۱. اطلاعات پروژه -->
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.example</groupId>
    <artifactId>myapp</artifactId>
    <version>1.0.0</version>
    <packaging>jar</packaging>
    <name>My Application</name>
    <description>توضیحات پروژه</description>
    
    <!-- ۲. تنظیمات والد (برای Spring Boot) -->
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.1.0</version>
        <relativePath/>
    </parent>
    
    <!-- ۳. Properties (متغیرهای سراسری) -->
    <properties>
        <java.version>17</java.version>
        <maven.compiler.source>17</maven.compiler.source>
        <maven.compiler.target>17</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>
    
    <!-- ۴. وابستگی‌ها -->
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <scope>runtime</scope>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>
    
    <!-- ۵. Build settings -->
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <excludes>
                        <exclude>
                            <groupId>org.projectlombok</groupId>
                            <artifactId>lombok</artifactId>
                        </exclude>
                    </excludes>
                </configuration>
            </plugin>
            
            <!-- جاوا کامپایلر -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <configuration>
                    <source>17</source>
                    <target>17</target>
                </configuration>
            </plugin>
            
            <!-- JaCoCo (کاورج تست) -->
            <plugin>
                <groupId>org.jacoco</groupId>
                <artifactId>jacoco-maven-plugin</artifactId>
                <version>0.8.10</version>
            </plugin>
        </plugins>
    </build>
    
    <!-- ۶. مخازن (Repository) -->
    <repositories>
        <repository>
            <id>central</id>
            <url>https://repo.maven.apache.org/maven2</url>
        </repository>
    </repositories>
</project>
```

### دستورات مهم Maven

```bash
# a. پاک کردن خروجی‌های قبلی
mvn clean

# b. کامپایل کردن کد
mvn compile

# c. اجرای تست‌ها
mvn test

# d. ساختن JAR/WAR
mvn package

# e. نصب در مخزن محلی
mvn install

# f. اجرای برنامه Spring Boot
mvn spring-boot:run

# g. اجرا با پروفایل خاص
mvn spring-boot:run -Dspring-boot.run.profiles=dev

# h. ساخت با اسکیپ تست
mvn clean install -DskipTests

# i. مشاهده درخت وابستگی‌ها
mvn dependency:tree

# j. بروزرسانی وابستگی‌ها
mvn versions:display-dependency-updates

# k. همه کارها با هم
mvn clean install -DskipTests
```

### Scopeهای Maven

```xml
<!-- scope مشخص می‌کنه وابستگی کی در دسترس باشه -->
<dependency>
    <groupId>...</groupId>
    <artifactId>...</artifactId>
    <scope>provided</scope> <!-- توسط JDK یا Container فراهم میشه -->
</dependency>

<!-- scopeها -->
<!-- compile: پیش‌فرض، در همه جا در دسترس -->
<!-- provided: مثل compile ولی توی JAR نمیاد (مثل servlet-api) -->
<!-- runtime: فقط در runtime در دسترس (مثل JDBC Driver) -->
<!-- test: فقط برای تست -->
<!-- system: از فایل سیستم (کمتر استفاده میشه) -->
```

### ۱۸.۲ Gradle (جایگزین مدرن Maven)

#### ساختار پروژه Gradle

```text
my-app/
├── build.gradle
├── settings.gradle
├── src/
│   ├── main/
│   │   ├── java/
│   │   └── resources/
│   └── test/
│       ├── java/
│       └── resources/
└── build/                ← خروجی build
```

#### فایل build.gradle

```groovy
// a. پلاگین‌ها
plugins {
    id 'java'
    id 'org.springframework.boot' version '3.1.0'
    id 'io.spring.dependency-management' version '1.1.0'
}

// b. اطلاعات پروژه
group = 'com.example'
version = '1.0.0'
sourceCompatibility = '17'

// c. مخازن
repositories {
    mavenCentral()
}

// d. وابستگی‌ها
dependencies {
    // Spring Boot Starters
    implementation 'org.springframework.boot:spring-boot-starter-web'
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
    
    // Database
    runtimeOnly 'mysql:mysql-connector-java'
    
    // Lombok
    compileOnly 'org.projectlombok:lombok'
    annotationProcessor 'org.projectlombok:lombok'
    
    // Test
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
    testRuntimeOnly 'org.junit.platform:junit-platform-launcher'
}

// e. Test configuration
test {
    useJUnitPlatform()
}

// f. Custom tasks
task hello {
    doLast {
        println 'Hello from Gradle!'
    }
}
```

### فایل settings.gradle

```groovy
rootProject.name = 'myapp'
```
### دستورات مهم Gradle

```bash
# a. مشاهده همه tasks
./gradlew tasks

# b. پاک کردن
./gradlew clean

# c. ساختن
./gradlew build

# d. اجرا (با اسکیپ تست)
./gradlew build -x test

# e. اجرای برنامه
./gradlew bootRun

# f. اجرا با پروفایل خاص
./gradlew bootRun --args='--spring.profiles.active=dev'

# i. مشاهده درخت وابستگی‌ها
./gradlew dependencies

# j. بروزرسانی وابستگی‌ها
./gradlew dependencyUpdates
```

### ۱۸.۳ مقایسه Maven و Gradle

# مقایسه Maven و Gradle

| ویژگی | Maven | Gradle |
|-------|-------|--------|
| **زبان** | XML | Groovy/Kotlin (کوتاه‌تر) |
| **سرعت** | کندتر (به خاطر XML) | سریع‌تر (به خاطر Incremental Build) |
| **یادگیری** | آسان‌تر | سخت‌تر |
| **وابستگی‌ها** | خوب | عالی |
| **قابلیت گسترش** | محدود | بسیار زیاد |
| **محبوبیت** | بیشتر | در حال رشد |
| **مناسب برای** | پروژه‌های ساده | پروژه‌های بزرگ |

### مثال مقایسه (همان کار با دو ابزار)

### Maven (pom.xml):

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

### Gradle (build.gradle):

```groovy
implementation 'org.springframework.boot:spring-boot-starter-web'
```

## ۱۸.۴ ❗️ سوالات مصاحبه‌ای

### ❓ سوال ۱: Maven چیه و چه کارایی داره؟

**جواب:** Maven یه ابزار مدیریت و ساخت (Build Tool) برای پروژه‌های جاواست که:

- وابستگی‌ها رو مدیریت می‌کنه

- پروژه رو Build می‌کنه

- چرخه حیات (Lifecycle) استاندارد داره

- با pom.xml پیکربندی میشه

### ❓ سوال ۲: فرق compile و provided و runtime و test چیه؟

**جواب:**

# اسکوپ‌های (Scope) وابستگی در Maven

| Scope | توضیح |
|-------|-------|
| **compile** | پیش‌فرض، در همه جا در دسترس (هم در کامپایل، هم در اجرا و هم در تست) |
| **provided** | مثل compile ولی در JAR نهایی نمیاد (مثل Servlet API که توسط سرور تأمین می‌شود) |
| **runtime** | فقط در زمان اجرا نیاز است (مثل JDBC Driver) - در کامپایل نیاز نیست |
| **test** | فقط برای تست‌ها در دسترس است (مثل JUnit، Mockito) |
| **system** | از فایل سیستم محلی بارگذاری می‌شود (کمتر استفاده میشه و وابستگی به مسیر سیستمی دارد) |

### ❓ سوال ۳: چرخه حیات Maven رو توضیح بده

#### جواب: سه تا چرخه حیات اصلی داره:

- default: validate → compile → test → package → verify → install → deploy

- clean: pre-clean → clean → post-clean

- site: pre-site → site → post-site → site-deploy


### ❓ سوال ۴: فرق Maven و Gradle چیه؟

**جواب:**

- Maven با XML پیکربندی میشه، Gradle با Groovy/Kotlin

- Gradle سریع‌تره (Incremental Build)

- Gradle انعطاف‌پذیرتره

- Maven ساده‌تر برای شروع

- هر دو برای مدیریت وابستگی‌ها و Build هستن


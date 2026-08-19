# 📘 فاز ۴: جاوا پیشرفته (Advanced Java) – برنامه‌نویسی تابعی و هم‌روندی

تسلط به این مباحث نشون میده که با روندهای جدید جاوا همراهی و می‌تونی کد تمیز، خواناتر و بهینه‌تری بنویسی.

این فاز شامل دو بخش بزرگه: برنامه‌نویسی تابعی (Functional Programming) با Lambda و Stream، و برنامه‌نویسی هم‌روند (Concurrent Programming) با Threadها. هر دوتاش توی مصاحبه و توی کار واقعی خیلی مهمن.

## 4.1 برنامه‌نویسی تابعی (Functional Programming) در جاوا

از Java 8 به بعد، جاوا دیگه فقط یه زبان شی‌گرا نیست. مفاهیم برنامه‌نویسی تابعی بهش اضافه شدن. هدف: نوشتن کدی که خواناتر، کوتاه‌تر و کم‌خطاتر باشه و راحت‌تر بتونیم پردازش‌های موازی انجام بدیم.

### 4.1.1 لامبدا (Lambda Expressions)

**تعریف ساده:** لامبدا یه تابع بی‌نام (Anonymous Function) هست. یعنی یه متد بدون اسم که می‌تونیم مثل یه شیء باهاش رفتار کنیم (به عنوان آرگومان پاسش بدیم، توی متغیر ذخیرش کنیم).

**چرا لامبدا اومد؟** قبل از Java 8، برای تعریف یه رفتار ساده (مثلاً یه Thread جدید) مجبور بودیم یه کلاس بی‌نام (Anonymous Inner Class) بنویسیم که پر از کدهای اضافی و زشت بود.

```java
// a. روش قدیمی با Anonymous Inner Class
Runnable oldWay = new Runnable() {
    @Override
    public void run() {
        System.out.println("Running old way");
    }
};

// روش جدید با Lambda (Java 8+)
Runnable newWay = () -> System.out.println("Running lambda way");
```

### ساختار لامبدا

```text
(param1, param2) -> { body }
```

- (پارامترها): ورودی‌های تابع. اگه فقط یه پارامتر باشه و نوعش رو ننویسی، می‌تونی پرانتز رو حذف کنی.

- ->: علامت فلش.

- { بدنه }: کد اجرایی. اگه فقط یه خط باشه، می‌تونی {} و return رو حذف کنی.

```java
// a. مثال‌های معتبر لامبدا
() -> System.out.println("Hello"); // بدون پارامتر
x -> x * 2; // یک پارامتر، یک خط
(x, y) -> { // چند پارامتر، چند خط
    int sum = x + y;
    return sum;
};
```

---

### ❗️ سوال مصاحبه‌ای ۱: Functional Interface (واسط تابعی) چیه و چه ربطی به لامبدا داره؟

**جواب:** یه اینترفیس که فقط و فقط یک متد انتزاعی داشته باشه. به این متد میگن Abstract Method. لامبدا دقیقاً پیاده‌سازی همین یک متد رو فراهم می‌کنه.

- کامپایلر می‌فهمه که لامبدا باید اون یک متد رو پیاده‌سازی کنه.
- برای جلوگیری از خطا، بهتره از Annotation مخصوص `@FunctionalInterface` استفاده کنیم. این Annotation کامپایلر رو مجبور می‌کنه چک کنه که اینترفیس واقعاً فقط یه متد انتزاعی داره.

```java
@FunctionalInterface
interface Calculator {
    int calculate(int x, int y); // فقط یک متد انتزاعی
}

public class Main {
    public static void main(String[] args) {
        // استفاده از لامبدا برای پیاده‌سازی متد calculate
        Calculator adder = (a, b) -> a + b;
        Calculator multiplier = (a, b) -> a * b;

        System.out.println(adder.calculate(5, 3)); // خروجی: 8
    }
}
```

### بسته java.util.function: جاوا یه سری Functional Interface آماده داره که توی Stream API خیلی استفاده میشن. مهم‌ترین‌هاش

- ``Predicate<T>:`` یه T میگیره، boolean برمیگردونه. (مثل filter که تست شرطی انجام میده).

- ``Function<T, R>:`` یه T میگیره، یه R برمیگردونه. (مثل map که تبدیل انجام میده).

- ``Consumer<T>:`` یه T میگیره و هیچی برنمیگردونه (void). (مثل forEach).

- ``Supplier<T>:`` هیچی نمیگیره، یه T جدید میسازه و برمیگردونه. (مثل Collectors.toList()).

### 4.1.2 Stream API

**تعریف ساده:** Stream یه جریان (رودخونه) از داده‌هاست که به صورت اعلامی (Declarative) روشون عملیات انجام میدیم. به جای اینکه بگیم چطور (با حلقه `for`)، میگیم چه کاری (`filter`, `map`, `collect`).

---

### ❗️ سوال مصاحبه‌ای ۲: Stream با Collection چه فرقی داره؟

**جواب حیاتی:**

- **ذخیره‌سازی:** Collection یه ساختمان داده است که داده رو ذخیره می‌کنه. Stream اصلاً داده ذخیره نمی‌کنه، فقط یه مسیر برای انتقال و پردازش داده از یه منبع (مثل Collection) هست.
- **عملیات:** Stream عملیات رو به دو دسته تقسیم می‌کنه: Intermediate و Terminal.
- **Lazy Evaluation:** یه عملیات Intermediate (مثل `filter`) تا زمانی که یه عملیات Terminal (مثل `collect`) صدا زده نشه، اصلاً اجرا نمی‌شه. این بهینه‌سازی خیلی مهمیه.

### عملیات‌های Stream

**ساخت Stream:**

```java
// a. از Collection
List<String> names = Arrays.asList("Ali", "Reza", "Sara");
Stream<String> stream1 = names.stream();

// از آرایه
Stream<String> stream2 = Arrays.stream(new String[]{"X", "Y"});
```

### عملیات میانی (Intermediate): یه Stream جدید برمیگردونن. Lazy هستن

- ``filter(Predicate):`` فیلتر بر اساس شرط.

- ``map(Function):`` تبدیل هر عنصر به یه چیز دیگه.

- ``sorted():`` مرتب‌سازی.

- ``distinct():`` حذف موارد تکراری.

- ``limit(n):`` فقط n عنصر اول.

### عملیات پایانی (Terminal): یه نتیجه (غیر Stream) برمیگردونن و جریان رو می‌بندن

- ``collect(Collectors.toList()):`` جمع‌آوری المان‌ها در یه List.

- ``forEach(Consumer):`` انجام عملی روی هر عنصر.

- ``reduce(BinaryOperator):`` ترکیب کل المان‌ها در یک مقدار واحد (مثلاً جمع کل).

```java
// a. مثال کامل
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);

List<Integer> evenSquares = numbers.stream()
    .filter(n -> n % 2 == 0)       // ۱. Intermediate: فقط زوج‌ها
    .map(n -> n * n)                // ۲. Intermediate: به توان ۲ برسون
    .sorted((a, b) -> b - a)        // ۳. Intermediate: نزولی مرتب کن
    .limit(3)                       // ۴. Intermediate: فقط ۳ تا اول
    .collect(Collectors.toList());  // ۵. Terminal: بریز توی یه لیست

// نتیجه: [100, 64, 36] (بزرگترین مربع‌های زوج: ۱۰، ۸ و ۶)
```

---

### ❗️ سوال مصاحبه‌ای ۳: متد `reduce` چیکار می‌کنه؟

**جواب:** عملیات پایانی (Terminal) هست که عناصر Stream رو یکی یکی با هم طبق یه قاعده ترکیب می‌کنه تا به یه مقدار واحد برسیم. یه جورایی for حلقه‌ای هست که یه accumulator داریم.

```java
// a. جمع اعداد یه لیست
List<Integer> numbers = Arrays.asList(1, 2, 3, 4);
int sum = numbers.stream()
                 .reduce(0, (subtotal, element) -> subtotal + element);
// 0 + 1 = 1
// 1 + 2 = 3
// 3 + 3 = 6
// 6 + 4 = 10 -> sum = 10
```

## 4.2 برنامه‌نویسی هم‌روند و چندریسمانی (Concurrency & Multithreading)

**هدف:** اجرای همزمان چندین بخش از برنامه برای استفاده بهتر از CPU و افزایش سرعت پاسخگویی. (به جای اینکه کاربر منتظر بمونه، یه Thread دیگه کار رو انجام بده).

### 4.2.1 ساختن و اجرای Thread

دو روش اصلی:

1. **`extends Thread`:** کلاس خودت رو از کلاس `Thread` ارث ببر و متد `run()` رو Override کن. (روش قدیمی، محدودیت‌زا چون نمی‌تونی از کلاس دیگه‌ای ارث ببری).
2. **`implements Runnable` (توصیه شده):** کلاس خودت `Runnable` رو پیاده‌سازی کنه، بعد یه شیء `Thread` بسازی و `Runnable` رو بهش پاس بدی.

```java
// a. روش توصیه شده با Runnable
class MyTask implements Runnable {
    @Override
    public void run() {
        System.out.println("اجرا توسط: " + Thread.currentThread().getName());
    }
}
public class Main {
    public static void main(String[] args) {
        Thread t1 = new Thread(new MyTask());
        Thread t2 = new Thread(new MyTask());

        t1.start(); // هیچوقت run() رو مستقیم صدا نزن!
        t2.start(); // start() یه Thread جدید میسازه و run() رو توی اون صدا میزنه
    }
}
```

---

### 4.2.2 مشکلات رایج در هم‌روندی

### ❗️ سوال مصاحبه‌ای ۴: Race Condition (شرایط مسابقه) چیه؟

**جواب:** وقتی دو یا چند Thread به یه منبع مشترک (Shared Resource) دسترسی همزمان دارن و حداقل یکی از اون‌ها منبع رو تغییر میده. نتیجه نهایی به ترتیب زمان‌بندی Threadها بستگی داره و غیرقابل پیش‌بینی و خطرناک میشه.

```java
public class Counter {
    private int count = 0;

    // این متد Thread-Safe نیست! Race Condition داره
    public void increment() {
        count++; // این یه عملیات ۳ مرحله‌ایه: خواندن، افزودن، نوشتن
    }
}
```

---

### ❗️ سوال مصاحبه‌ای ۵: `synchronized` چطور مشکل Race Condition رو حل می‌کنه؟

**جواب:** کلمه کلیدی `synchronized` تضمین می‌کنه که در هر لحظه فقط یک Thread بتونه متد یا بلوک مشخصی از کد رو برای یک شیء خاص اجرا کنه. این کار با قفل کردن (Lock) انجام میشه.

```java
public class SafeCounter {
    private int count = 0;

    public synchronized void increment() { // فقط یک Thread در هر لحظه
        count++;
    }

    // یا synchronized روی یه بلوک
    public void incrementBlock() {
        synchronized (this) {
            count++;
        }
    }
}
```

### نکته: synchronized باعث کاهش Performance میشه، چون Threadها باید منتظر هم بمونن. باید فقط جایی که لازمه استفاده بشه

---

### ❗️ سوال مصاحبه‌ای ۶: Deadlock (بن‌بست) چیه و چطور رخ میده؟

**جواب:** وضعیتی که دو یا چند Thread برای همیشه منتظر یکدیگر می‌مونن. معمولاً زمانی رخ میده که:

- Thread A قفل منبع ۱ رو داره و قفل منبع ۲ رو میخواد.
- Thread B قفل منبع ۲ رو داره و قفل منبع ۱ رو میخواد.
- هیچکدام قفل خودشون رو آزاد نمی‌کنن، پس هیچکدام به قفل بعدی نمیرسن. برنامه هنگ می‌کنه.

**راه‌های جلوگیری:**

1. ترتیب قفل‌گیری رو یکسان کن (همه اول قفل منبع ۱ رو بگیرن، بعد ۲).
2. از `tryLock()` با زمان مشخص استفاده کن تا اگر نتونست قفل کنه، قفل‌های قبلی رو آزاد کنه و دوباره تلاش کنه.

---

### ❗️ سوال مصاحبه‌ای ۷: تفاوت `Runnable` و `Callable`؟

**جواب:**

- **`Runnable`:** متد `run()` داره که مقدار برنمیگردونه و نمی‌تونه Checked Exception پرتاب کنه.
- **`Callable<V>`:** متد `call()` داره که یه مقدار از نوع V برمیگردونه و می‌تونه استثنا پرتاب کنه.

```java
Callable<String> task = () -> {
    Thread.sleep(1000);
    return "نتیجه کار";
};
// برای اجرای Callable باید از ExecutorService استفاده کرد
```

---

### ❗️ سوال مصاحبه‌ای ۸: Thread Pool و `ExecutorService` چیه؟ چرا ازشون استفاده می‌کنیم؟

**جواب:** ساختن و از بین بردن Threadها هزینه‌ی زیادی داره. به جای ساختن یه Thread جدید برای هر کار کوچیک (که ممکنه هزاران کار همزمان باشه)، از یه استخر (Pool) از Threadهای آماده استفاده می‌کنیم.

- `ExecutorService` یه اینترفیس جاوا برای مدیریت این استخرهاست.
- کارها رو به صورت `Runnable` یا `Callable` بهش میدیم (`submit`).
- خودش یه Thread آزاد از استخر برمی‌داره و کار رو اجرا می‌کنه.
- بعد از اتمام، Thread رو نمی‌کشه، برش میگردونه به استخر برای کار بعدی.

```java
ExecutorService executor = Executors.newFixedThreadPool(5); // استخری با ۵ Thread

for (int i = 0; i < 10; i++) {
    executor.submit(() -> {
        System.out.println(Thread.currentThread().getName() + " is running");
    });
}

executor.shutdown(); // بعد از اتمام کارها، استخر رو خاموش کن
```

---

این هم از فاز ۴. با Stream API و Lambda می‌تونی کد خیلی تمیزتری بنویسی، و با Concurrency می‌فهمی برنامه‌های بزرگ چطور سریع و پاسخگو باقی میمونن.

### ❗️ سوال مصاحبه‌ای ۹: CompletableFuture چیه و چه کاربردی داره؟

**جواب:** CompletableFuture یه راه مدرن برای برنامه‌نویسی ناهمگام (Asynchronous) در جاواست. به ما اجازه میده:
- چندین Task رو به صورت زنجیره‌ای اجرا کنیم
- Taskها رو ترکیب کنیم
- خطاها رو مدیریت کنیم

```java
// a. اجرای یه Task به صورت ناهمگام
CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> {
    // کار سنگین
    return "Result";
});

// b. زنجیره‌ای کردن عملیات
CompletableFuture.supplyAsync(() -> "Hello")
    .thenApply(String::toUpperCase)  // HELLO
    .thenApply(s -> s + " World")    // HELLO World
    .thenAccept(System.out::println); // چاپ نتیجه

// c. ترکیب دو Task
CompletableFuture<Integer> f1 = CompletableFuture.supplyAsync(() -> 5);
CompletableFuture<Integer> f2 = CompletableFuture.supplyAsync(() -> 3);
f1.thenCombine(f2, (a, b) -> a + b) // 8
    .thenAccept(System.out::println);
```

### ❗️ سوال مصاحبه‌ای ۱۰: Optional چیه و چرا استفاده میشه؟

**جواب:** Optional یه Container هست که می‌تونه یه مقدار داشته باشه یا خالی باشه (null). هدفش جلوگیری از NullPointerException هست.
```java
// a. روش قدیمی (خطرناک)
String name = getUserName();
if (name != null) {
    System.out.println(name.length()); // ممکنه NPE بده
}

// b. روش جدید با Optional
Optional<String> optionalName = Optional.ofNullable(getUserName());
optionalName.ifPresent(name -> System.out.println(name.length()));

// c. متدهای مفید Optional
optionalName.orElse("Default"); // اگر null بود، مقدار پیش‌فرض
optionalName.orElseThrow(() -> new RuntimeException("Name not found"));
optionalName.map(String::toUpperCase).orElse("EMPTY");
```


### سوال مصاحبه‌ای ۱۱: متدهای Default و Static در Interface (از Java 8) چین؟

**جواب:** قبل از جاوا ۸، Interfaceها فقط می‌تونستن متدهای abstract داشته باشن. از جاوا ۸:

    Default Method: متدی با بدنه که می‌تونه در Interface باشه و کلاس‌های implement کننده می‌تونن override کنن یا نکنن.

    Static Method: متدی که به خود Interface تعلق داره و با اسم Interface صدا زده میشه.
    
```java
    interface Vehicle {
    // Abstract Method
    void start();
    
    // Default Method (بدنه داره)
    default void stop() {
        System.out.println("Vehicle stopped");
    }
    
    // Static Method
    static void service() {
        System.out.println("Vehicle in service");
    }
}

class Car implements Vehicle {
    @Override
    public void start() {
        System.out.println("Car started");
    }
    // stop() رو override نکرده، پس از Interface استفاده می‌کنه
}

// استفاده
Car car = new Car();
car.start();  // Car started
car.stop();   // Vehicle stopped
Vehicle.service(); // Vehicle in service
```

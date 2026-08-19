# 📘 فاز ۳: مباحث پایه‌ای پیشرفته در جاوا – کامل و عمیق

این فاز شامل سه ستون اصلیه: مدیریت استثناها (Exceptions)، جنریک‌ها (Generics) و کالکشن‌ها (Collections). هر سه توی هر پروژه واقعی روزانه باهاشون سر و کار داری.

## 3.1 مدیریت استثناها (Exception Handling)

### سلسله مراتب استثناها در جاوا

همه چیز از کلاس `Throwable` ارث می‌بره که دو تا زیرکلاس اصلی داره:

- **Error:** مشکلات جدی و غیرقابل بازیابی (مثل `OutOfMemoryError`, `StackOverflowError`). برنامه‌نویس نباید و نمی‌تونه کاری براشون بکنه.
- **Exception:** مشکلاتی که برنامه‌نویس می‌تونه مدیریت کنه. خودش دو دسته داره:
  - **Checked Exception (استثناهای کنترل‌شده):** زیرکلاس مستقیم `Exception` هستند (نه `RuntimeException`). کامپایلر مجبورت می‌کنه یا `try-catch` کنی، یا با `throws` پرتاب کنی.
  - **Unchecked Exception (استثناهای کنترل‌نشده):** زیرکلاس `RuntimeException` هستند (مثل `NullPointerException`). کامپایلر اجباری برای مدیریتشون نداره.

```text
Throwable
├── Error (Unchecked)
└── Exception
    ├── Checked Exception (مثل IOException, SQLException)
    └── RuntimeException (Unchecked - مثل NullPointerException)
```

---

### ❗️ سوال مصاحبه‌ای ۱: Checked Exception و Unchecked Exception رو با مثال توضیح بده. فلسفه وجودیشون چیه؟

**جواب:**

- **Checked:** برای شرایطی که قابل پیش‌بینی و خارج از کنترل برنامه هستن، ولی برنامه می‌تونه از پسشون بربیاد. (مثلاً فایل پیدا نشد: `FileNotFoundException`، یا قطع شدن شبکه). طراح جاوا می‌خواسته برنامه‌نویس رو مجبور کنه به این فکر کنه که اگه فایل نبود چی بشه.
- **Unchecked:** معمولاً ناشی از باگ‌های برنامه‌نویسی هست (مثلاً دسترسی به ایندکس اشتباه، فراخوانی متد روی `null`). انتظار میره برنامه‌نویس کد رو درست بنویسه که این اتفاق نیفته، نه اینکه همش `try-catch` بذاره.

```java
// Checked Exception: مجبوری مدیریتش کنی
try {
    FileReader file = new FileReader("file.txt"); // اگه فایل نباشه چی؟
} catch (FileNotFoundException e) {
    // بازیابی از خطا
}

// Unchecked Exception: اجباری نیست
String s = null;
s.length(); // پرتاب NullPointerException - خودت مقصری!
```

---

### ❗️ سوال مصاحبه‌ای ۲: `finally` چه زمانی اجرا می‌شه؟ تنها زمانی که اجرا نمی‌شه کِی هست؟

**جواب:** بلوک `finally` همیشه بعد از بلوک `try` و `catch` اجرا می‌شه، چه استثنا رخ بده، چه نده، چه `catch` بشه، چه نشه. حتی اگه توی `try` دستور `return` داشته باشیم، `finally` درست قبل از خروج از متد اجرا میشه.

**تنها موردی که اجرا نمی‌شه:** اگه `System.exit()` توی `try` یا `catch` صدا بشه، JVM کلاً خاموش میشه و `finally` اجرا نمی‌شه. یا اگه نخ اعدام بشه. کاربرد اصلیش بستن منابع (فایل، کانکشن دیتابیس) هست.

```java
try {
    System.out.println("Try block");
    return; // حتی با return هم finally اجرا میشه
} finally {
    System.out.println("Finally block"); // این پرینت میشه
}
```

---

### ❗️ سوال مصاحبه‌ای ۳: Try-with-Resources چیه و چه مشکلی رو حل می‌کنه؟

**جواب:** از Java 7 اضافه شد. مشکل اصلی `finally` این بود که برای بستن هر منبع، کلی کد تکراری و زشت می‌نوشتیم و خود بستن هم می‌تونست استثنا بده (Suppressed Exceptions). Try-with-Resources اینو اتوماتیک می‌کنه. هر کلاسی که `AutoCloseable` رو پیاده‌سازی کنه، می‌تونه توی پرانتز `try` قرار بگیره و `close()` ش به طور خودکار و امن صدا زده میشه.

```java
// a. روش قدیمی و زشت
FileInputStream fis = null;
try {
    fis = new FileInputStream("file.txt");
} catch (IOException e) { ... }
finally {
    if (fis != null) {
        try { fis.close(); } catch (IOException e) { ... }
    }
}

// روش جدید و ترجیح داده شده (Try-with-Resources)
try (FileInputStream fis = new FileInputStream("file.txt")) {
    // استفاده از fis
} catch (IOException e) {
    // fis خودکار بسته میشه
}
```

---

## 3.2 جنریک‌ها (Generics)

**هدف اصلی:** فراهم کردن Type Safety (امنیت نوع) در زمان کامپایل، و حذف نیاز به Cast کردن دستی.

تصور کن قبل از Java 5 چه فاجعه‌ای بود:

```java
// a. قبل از جنریک (Java 1.4)
List list = new ArrayList();
list.add("Hello");
list.add(123);      // کامپایلر هیچ اعتراضی نداره!

String s = (String) list.get(0); // نیاز به Cast
String s2 = (String) list.get(1); // خطا در زمان اجرا! ClassCastException
```

### اما با جنریک

```java
List<String> list = new ArrayList<>();
list.add("Hello");
// list.add(123); // خطای کامپایل! عالیه.
String s = list.get(0); // نیازی به Cast نیست
```

---

### ❗️ سوال مصاحبه‌ای ۴: Type Erasure (پاک‌سازی نوع) چیه؟

**جواب:** جنریک‌ها یه ویژگی زمان کامپایل هستن! برای حفظ سازگاری با نسخه‌های قدیمی جاوا، کامپایلر:

1. نوع‌های جنریک رو چک می‌کنه.
2. همه اطلاعات نوع رو پاک می‌کنه.
3. هر جا لازم باشه، Cast اضافه می‌کنه.

پس در زمان اجرا (Runtime)، JVM اصلاً خبر نداره که `List<String>` بوده یا `List<Integer>`. هر دو فقط `List` هستن. برای همین نمی‌تونی `instanceof` روی نوع جنریک انجام بدی (`if (list instanceof List<String>)` غلطه).

### ❗️ سوال مصاحبه‌ای ۵: Wildcardها (`?`) در جنریک چیا هستن؟ `? extends` و `? super` رو توضیح بده (PECS)

**جواب:** وقتی یه متد می‌نویسی، نمی‌خوای محدود به یه نوع خاص باشی، اما نمی‌خوای Type Safety رو هم از دست بدی. اینجا Wildcardها میان:

- **`? extends T` (Upper-Bounded Wildcard):** یعنی "هر نوعی که زیرکلاس T باشه (یا خود T)". از این ساختار فقط می‌تونی بخونی (`get`). نمی‌تونی توی لیست چیزی بذاری (`add`)، چون نمیدونی دقیقاً چه نوعیه.
- **`? super T` (Lower-Bounded Wildcard):** یعنی "هر نوعی که اَبَرکلاس T باشه (یا خود T)". توی این ساختار می‌تونی بذاری (`add` از نوع T).

**PECS: Producer Extends, Consumer Super.**

- اگه میخوای از یه کالکشن بخونی (Producer)، از `extends` استفاده کن.
- اگه میخوای توی یه کالکشن بنویسی (Consumer)، از `super` استفاده کن.

```java
// Producer Extends: فقط میتونم اعداد رو بخونم
public void printNumbers(List<? extends Number> list) {
    for (Number n : list) { // فقط خوندن
        System.out.println(n);
    }
    // list.add(10); // خطا!
}

// Consumer Super: میتونم بهش Integer اضافه کنم
public void addNumbers(List<? super Integer> list) {
    list.add(10); // نوشتن Integer
    // Integer x = list.get(0); // خطا! نوع بازگشتی Object هست.
}
```

## 3.3 کالکشن‌ها (Collections Framework)

این چارچوب یه معماری یکپارچه برای کار با گروهی از اشیاء است. نقشه راه:

- **Collection (Interface):** ریشه. زیرمجموعه‌هاش: `List`, `Set`, `Queue`.
- **Map (Interface):** مستقل از `Collection`. برای زوج‌های کلید-مقدار.

### 3.3.1 لیست‌ها (Lists)

#### ArrayList در مقابل LinkedList

### ❗️ سوال مصاحبه‌ای ۶: `ArrayList` و `LinkedList` رو کامل مقایسه کن. کی کدوم رو انتخاب می‌کنی؟

**جواب:**

**ArrayList:** در داخل یه آرایه پویا هست. المان‌ها پشت سر هم توی حافظه‌ان.

- `get(index)`: O(1) — دسترسی تصادفی فوق‌العاده سریع.
- `add(element)`: O(1) amortized — اضافه کردن به آخر سریعه، ولی اگه ظرفیت آرایه پر بشه، یه آرایه بزرگتر ساخته و همه چیز کپی میشه.
- `add(index, element)`: O(n) — اضافه کردن به وسط کند هست چون بقیه المان‌ها باید جابجا بشن.

**LinkedList:** در داخل یه لیست پیوندی دوطرفه هست. هر گره به قبلی و بعدی اشاره داره.

- `get(index)`: O(n) — باید از اول لیست تا اون ایندکس حرکت کنه.
- `add(index, element)`: O(1) — به شرطی که قبلاً به اون گره رسیده باشی. خود عمل درج/حذف سریعه.
- `addFirst/removeFirst`: O(1). برای پیاده‌سازی Stack و Queue خیلی خوبه.

**چه موقع کدوم؟**

- عملیات غالبت `get` و `set` هست: حتماً **ArrayList**.
- عملیات غالبت اضافه/حذف از وسط یا ابتدای یه لیست خیلی بزرگه: **LinkedList**.
- توی عمل و دنیای واقعی، `ArrayList` به خاطر حافظه کش CPU معمولاً انتخاب بهتری هست و `LinkedList` کمتر استفاده میشه.

### 3.3.2 مجموعه‌ها (Sets)

#### HashSet، LinkedHashSet و TreeSet

- **HashSet:** از `HashMap` در داخل استفاده می‌کنه. ترتیب رو تضمین نمی‌کنه. عملیات `add`, `remove`, `contains` با O(1).
- **LinkedHashSet:** مثل `HashSet`، ولی یه لیست پیوندی هم نگه میداره که ترتیب درج (Insertion Order) رو حفظ کنه.
- **TreeSet:** از یه درخت سرخ-سیاه (Red-Black Tree) استفاده می‌کنه. ترتیب طبیعی (Natural Ordering) یا ترتیب یه `Comparator` که خودت میدی. عملیات اصلی با O(log n). برای وقتی خوبه که نیاز به چیزای مرتب داری.

### 3.3.3 نقشه‌ها (Maps)

#### HashMap، LinkedHashMap، TreeMap و Hashtable

---

### ❗️ سوال مصاحبه‌ای ۷: `HashMap` دقیقاً چطور کار می‌کنه؟ (سوال امتیازآور)

**جواب:**

1. `HashMap` یه آرایه از سطل‌ها (Buckets) هست. هر سطل یه `Node` (کلید، مقدار، اشاره‌گر به بعدی).
2. وقتی `put(key, value)` می‌کنیم:
   - متد `hashCode()` روی کلید صدا زده میشه.
   - یه تابع هش ثانویه، این عدد رو به یه ایندکس در آرایه تبدیل می‌کنه.
   - اگه اون سطل خالی بود، گره جدید اونجا ذخیره میشه.
   - اگه پر بود (Collision)، به انتهای لیست پیوندی در اون سطل اضافه میشه. در Java 8، اگه تعداد گره‌های یه سطل از ۸ بیشتر بشه، اون لیست پیوندی به یه درخت سرخ-سیاه (Red-Black Tree) تبدیل میشه تا جستجو از O(n) به O(log n) بهبود پیدا کنه.
3. وقتی `get(key)` می‌کنیم:
   - دوباره `hashCode()` صدا زده میشه و ایندکس سطل پیدا میشه.
   - اول با `hashCode` مقایسه می‌کنه، سپس برای اطمینان کامل، با `equals()` کلید داخل اون سطل رو با کلید ورودی مقایسه می‌کنه.

---

### ❗️ سوال مصاحبه‌ای ۸: چرا باید `hashCode()` و `equals()` رو همزمان با هم Override کنیم؟ (سوال حیاتی)

**جواب:** بخاطر قرارداد (Contract) بین این دو متد:

- اگه `equals()` میگه دو شیء برابر هستن، `hashCode()` آن‌ها باید یک عدد یکسان برگردونه.
- اگه `hashCode()` دو شیء یکسان باشه، `equals` ممکنه برابر باشه یا نباشه.

**عاقبت نقض قرارداد:** اگه کلیدی رو با `put` ذخیره کنی، و بعداً همون کلید منطقی (با `equals` برابر) رو با `get` جستجو کنی، ولی `hashCode` ش فرق کنه، `HashMap` توی سطل اشتباه میگرده و چیزی پیدا نمی‌کنه. انگار کلیدت گم شده!

---

### ❗️ سوال مصاحبه‌ای ۹: `ConcurrentHashMap` و `Hashtable` چه فرقی دارن؟ کدوم بهتره؟

**جواب:**

- **Hashtable:** قدیمی و از Java 1.0 هست. تمام متدهاش با `synchronized` هستن. یعنی در هر لحظه فقط یک نخ می‌تونه به کل Map دسترسی داشته باشه. Performance خیلی پایینی داره. کلید و مقدار `null` رو قبول نمی‌کنه.
- **ConcurrentHashMap:** مدرن و از Java 5. برای محیط‌های هم‌روند طراحی شده. به جای قفل کردن کل ساختار، فقط سطل مورد نظر (bucket-level locking) رو قفل می‌کنه. این کار باعث میشه چندین نخ بتونن همزمان روی سطل‌های مختلف کار کنن. Performance بسیار بالاتری داره. `null` رو هم قبول نمی‌کنه.

---

### ❗️ سوال مصاحبه‌ای ۱۰: متدهای `equals` و `hashCode` برای یه کلاس سفارشی چطور درست بنویسیم؟

```java
public class Student {
    private int id;
    private String name;

    // equals برای مقایسه منطقی
    @Override
    public boolean equals(Object o) {
        if (this == o) return true; // بهینه‌سازی
        if (o == null || getClass() != o.getClass()) return false; // نوع چک
        Student student = (Student) o; // Cast
        return id == student.id; // مقایسه بر اساس کلید اصلی (id)
    }

    // hashCode منطبق با equals
    @Override
    public int hashCode() {
        return Objects.hash(id); // فقط از id استفاده کن
    }
}
```

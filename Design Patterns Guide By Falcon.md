# راهنمای کاربردی دیزاین پترن‌های پرکاربرد در جاوا

این فایل برای فالکون (برنامه‌نویس جاوا و علاقه‌مند به بازی‌سازی) آماده شده تا بداند **کِی** و **چرا** از هر دیزاین پترن استفاده کند.

ساختار:
- برای هر الگو:
  - مشکلِ رایج (نشانه استفاده)
  - راه‌حل (ایده پترن)
  - مثال ساده در جاوا
  - مزایا
  - معایب / هشدارها

---

## 1. Creational Patterns (الگوهای ساخت اشیاء)

### 1.1 Singleton

- **خلاصهٔ کاربرد:** وقتی فقط «یکی» از یه چیز باید وجود داشته باشه

مثل:

```java
    GameManager
    ConfigManager
    InputManager
```

و نمی‌خوای چندتا ازش ساخته بشه.

این‌جا:

- می‌رسیم به Singleton
- (البته باید با دقت و فقط وقتی واقعاً لازمه استفاده بشه)

---

**مشکل:**

- فقط باید **یک** نمونه از یک کلاس در کل برنامه وجود داشته باشد.
- مثال: `GameConfig`, `Logger`, `InputManager`.

**ایدهٔ پترن:**

- سازنده را `private` می‌کنیم.
- یک نمونه‌ی ثابت (static) از کلاس داخل خودش نگه می‌داریم.
- یک متد `getInstance()` برای دسترسی به آن می‌نویسیم.

**کد ساده:**

```java
public class GameConfig {
    private static GameConfig instance;

    private GameConfig() {
        // تنظیمات اولیه
    }

    public static GameConfig getInstance() {
        if (instance == null) {
            instance = new GameConfig();
        }
        return instance;
    }
}
```

**مزایا:**
- کنترل کامل روی تعداد نمونه‌ها (فقط یکی).
- مناسب برای مدیریت تنظیمات سراسری.

**معایب:**
- اگر زیاد استفاده شود، تبدیل به **Global Variable** می‌شود.
- تست‌نویسی (Unit Test) را سخت می‌کند.
- وابستگی پنهان در سیستم ایجاد می‌کند.

---

### 1.2 Factory Method

- **خلاصهٔ کاربرد:** جلوگیری از تکرار `new` کردن کلاس‌های مشخص (مثلاً ساخت دشمن‌ها).

---

**مشکل:**

- کد شما نباید مستقیم با `new` کردن کلاس‌های مشخص، درگیر جزئیات ساخت شود.
- می‌خواهید بسته به شرایط، نوع مشخصی از یک آبجکت ساخته شود.

**ایدهٔ پترن:**

- یک متد کارخانه (factory) می‌سازید که شیء مناسب را برمی‌گرداند.
- کد کلاینت فقط آن متد را صدا می‌زند، نه `new SomeClass()`.

**کد ساده:**

```java
abstract class Enemy {
    abstract void attack();
}

class Orc extends Enemy {
    @Override
    void attack() { System.out.println("Orc attacks!"); }
}

class Troll extends Enemy {
    @Override
    void attack() { System.out.println("Troll attacks!"); }
}

class EnemyFactory {
    public static Enemy createEnemy(String type) {
        switch (type) {
            case "orc": return new Orc();
            case "troll": return new Troll();
            default: throw new IllegalArgumentException("Unknown type");
        }
    }
}
```

**مزایا:**
- کد اصلی از جزئیات ساخت اشیاء جدا می‌شود.
- افزودن نوع جدید را ساده‌تر می‌کند (با کمی تغییر در کارخانه).

**معایب:**
- اگر شرط‌ها زیاد شوند، خود کارخانه می‌تواند شلوغ شود.
- هنوز هم برای هر نوع جدید باید کارخانه را تغییر دهید (نقض جزئی Open/Closed).

---

### 1.3 Abstract Factory

- **خلاصهٔ کاربرد:** ساخت خانواده‌ای از اشیاء مرتبط (مثلاً ست کامل UI ویندوز در مقابل مک).

---

**مشکل:**

- می‌خواهید **خانواده‌ای از اشیاء مرتبط** بسازید که با هم کار می‌کنند.
- مثلاً: `GUI` برای Windows و Mac؛ هر کدام `Button`, `Checkbox` مخصوص خود را دارند.

**ایدهٔ پترن:**

- یک اینترفیس کارخانهٔ کلی می‌سازید.
- هر پیاده‌سازی کارخانه، یک «خانواده کامل» از اشیاء را تولید می‌کند.

**کد ساده:**

```java
interface GUIFactory {
    Button createButton();
    Checkbox createCheckbox();
}

interface Button { void render(); }
interface Checkbox { void render(); }

class WindowsButton implements Button {
    public void render() { System.out.println("Windows Button"); }
}

class MacButton implements Button {
    public void render() { System.out.println("Mac Button"); }
}

class WindowsFactory implements GUIFactory {
    public Button createButton() { return new WindowsButton(); }
    public Checkbox createCheckbox() { return () -> System.out.println("Windows Checkbox"); }
}

class MacFactory implements GUIFactory {
    public Button createButton() { return new MacButton(); }
    public Checkbox createCheckbox() { return () -> System.out.println("Mac Checkbox"); }
}
```

**مزایا:**
- تضمین می‌کند اشیاء مربوط به هم، هماهنگ ساخته شوند.
- وابستگی به اینترفیس‌ها، نه پیاده‌سازی‌ها.

**معایب:**
- برای پروژه‌های کوچک، اضافه‌کاری محسوب می‌شود.
- ساختار کلاس‌ها زیاد می‌شود.

---

### 1.4 Builder

- **خلاصهٔ کاربرد:** ساخت کلاس‌های پیچیده با تنظیمات زیاد (مثل ساخت Character بازی).

وقتی می‌خوای شیء را بسازی، ولی سازنده‌ات شاخ‌دار شده

مثلاً:

```java
new Character("Player1", 100, 20, 5, true, false, true, 0.2, "RANGER", ... );
```
این‌جا:

- خوندن و فهمیدن این سازنده سخت شده
- پارامتر زیاد = اشتباه انسانی زیاد

---

**مشکل:**

- سازنده‌ی کلاس بسیار طولانی و پر از پارامتر است.
- بعضی پارامترها اختیاری‌اند و خواندن کد سخت می‌شود.

**ایدهٔ پترن:**

- یک کلاس `Builder` می‌سازید که مرحله‌به‌مرحله پارامترها را تنظیم می‌کند.
- در نهایت با `build()` شیء آماده را برمی‌گرداند.

**کد ساده:**

```java
class Character {
    private String name;
    private int hp;
    private int damage;

    private Character(Builder builder) {
        this.name = builder.name;
        this.hp = builder.hp;
        this.damage = builder.damage;
    }

    public static class Builder {
        private String name;
        private int hp;
        private int damage;

        public Builder name(String name) {
            this.name = name; return this;
        }
        public Builder hp(int hp) {
            this.hp = hp; return this;
        }
        public Builder damage(int dmg) {
            this.damage = dmg; return this;
        }

        public Character build() {
            return new Character(this);
        }
    }
}

// استفاده:
Character hero = new Character.Builder()
        .name("Hero")
        .hp(100)
        .damage(20)
        .build();
```

**مزایا:**
- خوانایی بالا، مخصوصاً برای اشیاء پیچیده.
- جلوگیری از سازنده‌های طولانی.

**معایب:**
- کلاس اضافی (Builder) نیاز دارد.
- برای اشیاء ساده، ممکن است غیرضروری باشد.

---

### 1.5 Prototype

- **خلاصهٔ کاربرد:** ساخت اشیاء جدید از روی نمونه‌های آماده، وقتی `new` کردن هزینه‌بر است.

---

**مشکل:**

- ساخت یک شیء جدید، هزینه‌بر است (از نظر زمان یا منابع).
- ولی یک شیء مشابه در اختیار دارید و می‌خواهید از روی آن کپی بسازید.

**ایدهٔ پترن:**

- به جای `new`, از `clone()` یا کپی‌کردن شیء موجود استفاده می‌کنید.

**کد ساده:**

```java
class Enemy implements Cloneable {
    private String type;
    private int hp;

    public Enemy(String type, int hp) {
        this.type = type;
        this.hp = hp;
    }

    @Override
    public Enemy clone() {
        try {
            return (Enemy) super.clone();
        } catch (CloneNotSupportedException e) {
            throw new RuntimeException(e);
        }
    }
}

// استفاده:
Enemy orc = new Enemy("orc", 100);
Enemy clonedOrc = orc.clone();
```

**مزایا:**
- ایجاد اشیاء جدید سریع‌تر از ساخت از صفر.
- مناسب برای سیستم‌هایی مانند بازی که دشمن‌های مشابه زیاد دارد.

**معایب:**
- پیاده‌سازی `clone()` در جاوا کمی حساس و پیچیده است.
- برای اشیاء دارای وابستگی عمیق (Deep Copy)، دقت زیادی لازم است.

---

## 2. Structural Patterns (الگوهای ساختاری)

### 2.1 Adapter

- **خلاصهٔ کاربرد:** اتصال دو کلاس با اینترفیس‌های ناسازگار بدون تغییر کد اصلی.

---

**مشکل:**

- دو کلاس/کتابخانه با هم ناسازگار هستند (متدها یا نام‌ها متفاوتند).
- می‌خواهید بدون تغییر کد اصلی، از آن‌ها با هم استفاده کنید.

**ایدهٔ پترن:**

- یک «آداپتر» می‌نویسید که رابط (interface) مورد انتظار شما را پیاده‌سازی کرده و پشت‌صحنه، کلاس خارجی را صدا بزند.

**کد ساده:**

```java
interface GameInput {
    void jump();
}

// کتابخانه خارجی:
class ExternalInputSystem {
    void doJump() {
        System.out.println("External jump");
    }
}

class InputAdapter implements GameInput {
    private ExternalInputSystem external;

    public InputAdapter(ExternalInputSystem external) {
        this.external = external;
    }

    @Override
    public void jump() {
        external.doJump();
    }
}
```

**مزایا:**
- اتصال کد قدیمی به کتابخانه‌های جدید بدون تغییرِ هسته.
- جداسازی وابستگی‌ها.

**معایب:**
- اگر زیاد استفاده شود، تعداد کلاس‌ها زیاد می‌شود.
- می‌تواند ساختار را پیچیده کند.

---

### 2.2 Bridge

- **خلاصهٔ کاربرد:** جدا کردن انتزاع از پیاده‌سازی وقتی کلاس دو بُعد تغییر دارد (مثل شکل و نوع رندر).

---

**مشکل:**

- یک کلاس در دو بُعد تغییر می‌کند (مثلاً نوعِ سلاح و نوعِ رندر).
- اگر برای هر ترکیب یک کلاس بنویسی، تعداد کلاس‌ها منفجر می‌شود.

**ایdeaٔ پترن:**

- یک کلاس را به دو بخش: «Abstraction» و «Implementation» تقسیم می‌کنی.
- این دو از طریق ترکیب (composition) به هم وصل می‌شوند، نه ارث‌بری.

**کد ساده:**

```java
interface Renderer {
    void renderCircle(float x, float y, float radius);
}

class VectorRenderer implements Renderer {
    public void renderCircle(float x, float y, float radius) {
        System.out.println("Drawing circle as vector");
    }
}

class RasterRenderer implements Renderer {
    public void renderCircle(float x, float y, float radius) {
        System.out.println("Drawing circle as pixels");
    }
}

class Circle {
    private Renderer renderer;
    private float x, y, radius;

    public Circle(Renderer renderer, float x, float y, float radius) {
        this.renderer = renderer;
        this.x = x;
        this.y = y;
        this.radius = radius;
    }

    public void draw() {
        renderer.renderCircle(x, y, radius);
    }
}
```

**مزایا:**
- کاهش انفجار تعداد کلاس‌ها.
- امکان تغییر مستقل دو بُعد (مثلاً نوع رندر و نوع شکل).

**معایب:**
- ساختار اولیه کمی پیچیده‌تر است.

---

### 2.3 Composite

- **خلاصهٔ کاربرد:** مدیریت ساختارهای درختی (مثل صحنه بازی: Node, Child, Group) با یک API یکسان.

---

**مشکل:**

- می‌خواهید با ساختارهای درختی (مثل صحنه‌ی بازی: Node, Child Node) کار کنید.
- می‌خواهید "یک شیء" و "مجموعه‌ای از اشیاء" را با یک API مدیریت کنید.

**ایدهٔ پترن:**

- یک اینترفیس مشترک تعریف می‌کنید.
- هم اشیاء منفرد و هم مجموعه‌ها آن را پیاده‌سازی می‌کنند.

**کد ساده:**

```java
interface GameObject {
    void update();
}

class Sprite implements GameObject {
    public void update() {
        System.out.println("Sprite update");
    }
}

class Group implements GameObject {
    private List<GameObject> children = new ArrayList<>();

    public void add(GameObject obj) { children.add(obj); }

    public void update() {
        for (GameObject obj : children) {
            obj.update();
        }
    }
}
```

**مزایا:**
- یکسان شدن کار با تکی و مجموعه.
- مناسب برای ساختارهای صحنه (Scene Graph).

**معایب:**
- اشکال‌زدایی ساختارهای پیچیده ممکن است سخت شود.

---

### 2.4 Decorator

- **خلاصهٔ کاربرد:** افزودن قابلیت‌های جدید به شیء بدون تغییر کلاس اصلی (مانند شمشیر آتشین).

---

**مشکل:**

- می‌خواهید به یک شیء، رفتار جدید اضافه کنید بدون اینکه کلاس اصلی را تغییر دهید.
- مثلاً روی یک سلاح، قابلیت «آتش» یا «یخ» اضافه کنید.

**ایدهٔ پترن:**

- کلاسی می‌سازید که اینترفیس شیء اصلی را پیاده‌سازی کرده و یک شیء از همان نوع را در خود نگه دارد؛
  سپس قبل/بعد از صدا زدن متدهای آن، رفتار اضافه می‌کند.

**کد ساده:**

```java
interface Weapon {
    void attack();
}

class BasicSword implements Weapon {
    public void attack() {
        System.out.println("Slash");
    }
}

class FireDecorator implements Weapon {
    private Weapon base;

    public FireDecorator(Weapon base) {
        this.base = base;
    }

    public void attack() {
        base.attack();
        System.out.println("+ Fire damage");
    }
}
```

**مزایا:**
- افزودن رفتار بدون تغییر کلاس اصلی.
- ترکیب‌پذیری بالا (مثلاً Fire + Ice + Poison).

**معایب:**
- ممکن است زنجیره‌ی دکوراتورها پیچیده شود.
- ردیابی رفتار واقعی سخت می‌شود.

---

### 2.5 Facade

- **خلاصهٔ کاربرد:** ایجاد یک رابط ساده برای یک سیستم پیچیده و شلوغ.

---

**مشکل:**

- سیستم شما از چندین کلاس و زیرسیستم تشکیل شده و استفاده مستقیم از آن‌ها سخت است.

**ایدهٔ پترن:**

- یک کلاس ساده (Facade) می‌سازید که متدهای ساده و سطح‌بالا ارائه می‌کند و پشت‌صحنه بقیهٔ کلاس‌ها را صدا می‌زند.

**کد ساده:**

```java
class AudioSystem {
    void init() {}
    void playSound(String name) {}
}

class PhysicsSystem {
    void init() {}
}

class GameEngineFacade {
    private AudioSystem audio = new AudioSystem();
    private PhysicsSystem physics = new PhysicsSystem();

    public void startGame() {
        audio.init();
        physics.init();
        System.out.println("Game started");
    }
}
```

**مزایا:**
- ساده‌سازی استفاده از سیستم‌های پیچیده.
- مناسب برای APIهای سطح بالا.

**معایب:**
- اگر بیش از حد بزرگ شود، خودش تبدیل به God Object می‌شود.

---

### 2.6 Flyweight

- **خلاصهٔ کاربرد:** صرفه‌جویی در رم با اشتراک‌گذاری داده‌های مشترک بین میلیون‌ها شیء مشابه.

---

**مشکل:**

- تعداد بسیار زیادی از اشیاء مشابه دارید (مثلاً هزاران سرباز در یک بازی)،
  و هر کدام داده‌های تکراری زیادی دارند.

**ایدهٔ پترن:**

- داده‌های مشترک (invariant) را جدا کرده و فقط یک نسخه نگه می‌دارید.
- داده‌های خاص هر شیء را جداگانه ذخیره می‌کنید.

**کد ساده (خیلی ساده‌شده):**

```java
class TreeType {
    String name;
    String texture;

    public TreeType(String name, String texture) {
        this.name = name;
        this.texture = texture;
    }
}

class Tree {
    int x, y;
    TreeType type;
}
```

**مزایا:**
- کاهش مصرف حافظه.

**معایب:**
- پیچیدگی در مدیریت داده‌های مشترک و اختصاصی.

---

### 2.7 Proxy

- **خلاصهٔ کاربرد:** کنترل دسترسی یا مدیریت پیش‌از-اجرا (مانند Lazy Loading تصویر).

---

**مشکل:**

- می‌خواهید قبل از دسترسی به یک شیء، کنترل اضافه انجام دهید
  (مثلاً Lazy Loading، یا کنترل دسترسی، یا کش).

**ایدهٔ پترن:**

- یک کلاس Proxy می‌سازید که اینترفیس همان شیء اصلی را دارد،
  و درخواست‌ها را قبل از رسیدن به شیء اصلی مدیریت می‌کند.

**کد ساده:**

```java
interface Image {
    void display();
}

class RealImage implements Image {
    private String filename;

    public RealImage(String filename) {
        this.filename = filename;
        loadFromDisk();
    }

    private void loadFromDisk() {
        System.out.println("Loading " + filename);
    }

    public void display() {
        System.out.println("Displaying " + filename);
    }
}

class ImageProxy implements Image {
    private String filename;
    private RealImage realImage;

    public ImageProxy(String filename) {
        this.filename = filename;
    }

    public void display() {
        if (realImage == null) {
            realImage = new RealImage(filename);
        }
        realImage.display();
    }
}
```

**مزایا:**
- کنترل دسترسی، Lazy Loading، کش‌کردن.

**معایب:**
- افزودن یک لایهٔ اضافی بین کلاینت و شیء اصلی.

---

## 3. Behavioral Patterns (الگوهای رفتاری)

### 3.1 Strategy

- **خلاصهٔ کاربرد:** جایگزینی `if-else`های طولانی برای رفتارهای مختلف (مثل الگوریتم‌های حرکت دشمن).

مثلاً:

```java
if (type.equals("CREDIT_CARD")) { ... }
else if (type.equals("PAYPAL")) { ... }
else if (type.equals("CRYPTO")) { ... }
```
و هر جا، همین داستان تکرار می‌شه.

این نشونه‌ی اینه که:

- داری به نوع (type) حساسیت نشون می‌دی
- هر “نوع” رفتار متفاوتی داره

---

**مشکل:**

- چندین روش مختلف برای انجام یک کار دارید
  (مثلاً روش‌های مختلف پرداخت، یا روش‌های مختلف حرکت دشمن)،
  و نمی‌خواهید با `if-else` تشخیص دهید.

**ایدهٔ پترن:**

- یک اینترفیس برای رفتار تعریف می‌کنید.
- چندین پیاده‌سازی مختلف برایش می‌نویسید.
- در زمان اجرا، یکی از آن‌ها را به شئ اصلی تزریق می‌کنید.

**کد ساده (مثال Payment):**

```java
interface PaymentStrategy {
    void pay(double amount);
}

class CreditCardPayment implements PaymentStrategy {
    public void pay(double amount) {
        System.out.println("Paying " + amount + " by credit card");
    }
}

class CryptoPayment implements PaymentStrategy {
    public void pay(double amount) {
        System.out.println("Paying " + amount + " by crypto");
    }
}

class PaymentProcessor {
    private PaymentStrategy strategy;

    public PaymentProcessor(PaymentStrategy strategy) {
        this.strategy = strategy;
    }

    public void process(double amount) {
        strategy.pay(amount);
    }
}
```

**مزایا:**
- جلوگیری از `if-else`های طولانی.
- امکان تغییر رفتار در زمان اجرا.

**معایب:**
- افزایش تعداد کلاس‌ها.

---

### 3.2 Observer

- **خلاصهٔ کاربرد:** خبردار کردن سایر اشیاء از تغییرات (مانند آپدیت UI هنگام تغییر جان بازیکن).

---

**مشکل:**

- یک شیء (Subject) دارد تغییر می‌کند و چندین بخش دیگر باید از این تغییر باخبر شوند
  (مثلاً UI، لاگ، سیستم امتیاز).

**ایدهٔ پترن:**

- لیستی از ناظرها (Observers) در Subject نگه می‌دارید.
- وقتی Subject تغییر کرد، همه Observerها را خبر می‌کند.

**کد ساده:**

```java
interface Observer {
    void update(int hp);
}

class Player {
    private int hp;
    private List<Observer> observers = new ArrayList<>();

    public void addObserver(Observer o) { observers.add(o); }

    public void setHp(int hp) {
        this.hp = hp;
        notifyObservers();
    }

    private void notifyObservers() {
        for (Observer o : observers) {
            o.update(hp);
        }
    }
}
```

**مزایا:**
- Loose Coupling بین Subject و Observerها.

**معایب:**
- ممکن است چرخه‌های پیچیده‌ای از اطلاع‌رسانی ایجاد شود.

---

### 3.3 Command

- **خلاصهٔ کاربرد:** پیمایش روی کالکشن‌ها بدون درگیری با جزئیات داخلی آن‌ها.

---

**مشکل:**

- می‌خواهید درخواست (عملیاتی مثل "حمله"، "پریدن") را به شکل یک شیء ذخیره کنید
  (برای Undo، Redo، Queue کردن دستورات).

**ایدهٔ پترن:**

- هر دستور را در یک کلاس جداگانه می‌ریزید که یک متد `execute()` دارد.

**کد ساده:**

```java
interface Command {
    void execute();
}

class JumpCommand implements Command {
    public void execute() {
        System.out.println("Player jumps");
    }
}

class InputHandler {
    private Command jumpCommand;

    public InputHandler(Command jumpCommand) {
        this.jumpCommand = jumpCommand;
    }

    public void onJumpPressed() {
        jumpCommand.execute();
    }
}
```

**مزایا:**
- مناسب برای Undo/Redo و Queue از دستورات.

**معایب:**
- افزایش تعداد کلاس‌های کوچک.

---

### 3.4 Iterator

- **خلاصهٔ کاربرد:** پیمایش روی کالکشن‌ها بدون درگیری با جزئیات داخلی آن‌ها.

---

**مشکل:**

- می‌خواهید روی عناصر یک مجموعه حرکت کنید
  بدون اینکه جزئیات ساختار داخلی آن را بدانید.

**ایدهٔ پترن:**

- یک اینترفیس `Iterator` تعریف می‌کنید که `hasNext()` و `next()` داشته باشد.

**کد:**

در جاوا، `Iterator` از قبل وجود دارد (`java.util.Iterator`).

```java
List<String> list = new ArrayList<>();
Iterator<String> it = list.iterator();
while (it.hasNext()) {
    System.out.println(it.next());
}
```

**مزایا:**
- ساده‌سازی پیمایش روی ساختارهای مختلف.

**معایب:**
- تقریباً همیشه ضمنی در جاوا استفاده می‌شود.

---

### 3.5 Mediator

- **خلاصهٔ کاربرد:** کاهش وابستگی‌های مستقیم بین کلاس‌ها با استفاده از یک واسط مرکزی.

---

**مشکل:**

- چندین کلاس با هم زیاد در ارتباط‌اند و هر کدام دیگری را صدا می‌زنند.
- این باعث وابستگی شدید و کد پیچیده می‌شود.

**ایدهٔ پترn:**

- به جای اینکه مستقیم همدیگر را صدا بزنند، از یک واسط (Mediator) استفاده می‌کنند.

**کد ساده (مفهومی):**

```java
interface Mediator {
    void notify(Component sender, String event);
}

abstract class Component {
    protected Mediator mediator;
    public Component(Mediator mediator) { this.mediator = mediator; }
}
```

**مزایا:**
- کاهش وابستگی مستقیم بین کلاس‌ها.

**معایب:**
- Mediator ممکن است خیلی بزرگ و پیچیده شود.

---

### 3.6 Memento

- **خلاصهٔ کاربرد:** ذخیره وضعیت فعلی برای بازگشت به عقب (Save/Undo).

---

**مشکل:**

- می‌خواهید وضعیت یک شیء را ذخیره کنید تا بعداً به آن برگردید (Undo).

**ایدهٔ پترن:**

- یک کلاس `Memento` می‌سازید که وضعیت را نگه می‌دارد.
- شیء اصلی می‌تواند وضعیتش را به Memento تبدیل و برگردان کند.

**کد ساده:**

```java
class GameState {
    private int level;

    public GameState(int level) {
        this.level = level;
    }

    public int getLevel() { return level; }
}

class Game {
    private int level;

    public GameState save() {
        return new GameState(level);
    }

    public void restore(GameState state) {
        this.level = state.getLevel();
    }
}
```

**مزایا:**
- پیاده‌سازی Undo/Save در بازی‌ها.

**معایب:**
- مصرف حافظه زیاد اگر وضعیت‌ها بزرگ باشند.

---

### 3.7 State

- **خلاصهٔ کاربرد:** مدیریت رفتار شیء بر اساس وضعیتش (مثل Standing, Attacking) به جای `if`.

---

**مشکل:**

- رفتار یک شیء بسته به وضعیت (State) آن تغییر می‌کند.
- کد پر از `if (state == ...)` می‌شود.

**ایدهٔ پترن:**

- برای هر وضعیت، یک کلاس جداگانه می‌سازید که رفتار را تعریف می‌کند.

**کد ساده:**

```java
interface PlayerState {
    void handleInput();
}

class StandingState implements PlayerState {
    public void handleInput() {
        System.out.println("Standing");
    }
}

class RunningState implements PlayerState {
    public void handleInput() {
        System.out.println("Running");
    }
}

class Player {
    private PlayerState state;

    public void setState(PlayerState state) {
        this.state = state;
    }

    public void handleInput() {
        state.handleInput();
    }
}
```

**مزایا:**
- حذف if-elseهای مربوط به وضعیت.

**معایب:**
- افزایش تعداد کلاس‌ها.

---

### 3.8 Template Method

- **خلاصهٔ کاربرد:** تعریف اسکلت کلی الگوریتم در کلاس پایه و سپردن جزئیات به فرزندان.

---

**مشکل:**

- الگوریتم کلی در چند کلاس مشابه است،
  اما بعضی جزئیات فرق می‌کند.

**ایدهٔ پترن:**

- در کلاس پایه، الگوریتم کلی را به صورت متد `final` می‌نویسید.
- بخش‌هایی که باید متفاوت باشند را به صورت متدهای `abstract` یا قابل override تعریف می‌کنید.

**کد ساده:**

```java
abstract class GameLoop {
    public final void run() {
        init();
        update();
        render();
    }

    protected abstract void init();
    protected abstract void update();
    protected abstract void render();
}
```

**مزایا:**
- اشتراک الگوریتم کلی، جلوگیری از تکرار کد.

**معایب:**
- وابستگی به ارث‌بری (Inheritance).

---

### 3.9 Visitor

- **خلاصهٔ کاربرد:** افزودن عملیات جدید به کلاس‌ها بدون تغییر ساختار داخلی آن‌ها.

---

**مشکل:**

- می‌خواهید روی مجموعه‌ای از کلاس‌های مختلف عملیاتی انجام دهید
  بدون اینکه کلاس‌های اصلی را تغییر دهید.

**ایدهٔ پترن:**

- یک Visitor تعریف می‌کنید که متد `visit` برای هر نوع کلاس دارد.
- کلاس‌ها یک متد `accept(Visitor v)` دارند.

**کد ساده (خیلی خلاصه):**

```java
interface Visitor {
    void visitEnemy(Enemy e);
    void visitPlayer(Player p);
}

interface Element {
    void accept(Visitor v);
}
```

**مزایا:**
- افزودن عملیات جدید بدون تغییر کلاس‌های موجود.

**معایب:**
- اگر انواع جدیدی از Element اضافه کنید، باید Visitorها را تغییر دهید.

---

### 3.10 Chain of Responsibility

- **خلاصهٔ کاربرد:** عبور درخواست از زنجیره‌ای از هندلرها تا یکی مسئولیت آن را بپذیرد.

---

**مشکل:**

- می‌خواهید چندین Handler داشته باشید که هر کدام **ممکن است** یک درخواست را پردازش کنند.
- نمی‌خواهید یک Handler خاص به صورت ثابت مشخص باشد.

**ایدهٔ پترن:**

- Handlerها را در یک زنجیره قرار می‌دهید.
- هر Handler درخواست را گرفته، اگر نتوانست رسیدگی کند، به بعدی پاس می‌دهد.

**کد ساده:**

```java
abstract class Handler {
    protected Handler next;

    public Handler setNext(Handler next) {
        this.next = next;
        return next;
    }

    public void handle(String request) {
        if (next != null) next.handle(request);
    }
}

class AuthHandler extends Handler {
    @Override
    public void handle(String request) {
        System.out.println("Auth checked");
        super.handle(request);
    }
}
```

**مزایا:**
- انعطاف بالا در تعیین ترتیب و نوع Handlerها.

**معایب:**
- دیباگ کردن مسیر عبور درخواست گاهی سخت است.

---

## نکتهٔ پایانی برای فالکون

- از **اسم پترن** شروع نکن؛ از **مشکل** شروع کن.
- هر وقت دیدی:
  - `if-else` بر اساس نوع زیاد شده → به Strategy / State فکر کن.
  - ساخت اشیاء پیچیده است → به Builder / Factory فکر کن.
  - رفتار مشترک با تفاوت‌های کوچک داری → به Template Method فکر کن.
  - چندین آبجکت باید از تغییر یک آبجکت باخبر شوند → به Observer فکر کن.

این فایل را می‌توانی به عنوان `design-pattern-cheatsheet.md` در GitHub یا Gist ذخیره کنی و هر وقت وسط پروژه گیر کردی، سریع نگاهی به آن بیندازی.

# 📘 فاز ۱۵: DevOps، Docker، Kubernetes و ابزارهای مرتبط

## ۱۵.۱ DevOps چیست؟

**تعریف ساده:** ترکیب "Development" و "Operations". یه فرهنگ و مجموعه‌ای از روش‌ها برای:
- خودکارسازی فرآیندهای نرم‌افزاری
- کاهش فاصله بین تیم توسعه و تیم عملیات
- تحویل سریع‌تر و مطمئن‌تر نرم‌افزار

**چرخه DevOps:**
```text
Plan → Code → Build → Test → Release → Deploy → Operate → Monitor → (بازگشت به Plan)
```

---

## ۱۵.۲ Docker (ظروف مجازی)

**چیست؟** یه پلتفرم برای بسته‌بندی برنامه‌ها و وابستگی‌هاشون در یه Container (ظرف) که می‌تونه روی هر سیستمی اجرا بشه.

**تشبیه:** کانتینر مثل یه کانتینر حمل و نقل (کشتی) هست که برنامه و همه چیز مورد نیازش داخلش هست.

### فرق Docker با Virtual Machine

| ویژگی | Docker (Container) | Virtual Machine |
|-------|-------------------|-----------------|
| سیستمعامل | مشترک با Host | مجزا (هر VM یه OS داره) |
| اندازه | سبک (MB) | سنگین (GB) |
| سرعت اجرا | سریع | کند |
| مصرف منابع | کم | زیاد |

### Dockerfile (دستور ساخت کانتینر)

```dockerfile
# a. انتخاب Base Image
FROM openjdk:17-jdk-alpine

# b. تنظیم متغیرهای محیطی
ENV APP_HOME=/app

# c. ایجاد دایرکتوری
WORKDIR $APP_HOME

# d. کپی کردن فایل‌ها
COPY target/myapp.jar app.jar

# e. پورت برنامه
EXPOSE 8080

# f. دستور اجرا
ENTRYPOINT ["java", "-jar", "app.jar"]
```

## دستورات مهم Docker

```bash
# a. ساخت Image
docker build -t myapp:latest .

# b. اجرای Container
docker run -p 8080:8080 myapp:latest

# c. مشاهده Containerهای در حال اجرا
docker ps

# d. مشاهده همه Containerها
docker ps -a

# e. توقف Container
docker stop container_id

# f. حذف Container
docker rm container_id

# g. مشاهده Imageها
docker images

# h. ورود به Container
docker exec -it container_id /bin/sh

# i. مشاهده لاگ‌ها
docker logs container_id
```

---

### Docker Compose (اجرای چند کانتینر با هم)

```yaml
# docker-compose.yml
version: '3.8'

services:
  # اپلیکیشن Spring Boot
  app:
    build: .
    ports:
      - "8080:8080"
    depends_on:
      - db
    environment:
      - SPRING_DATASOURCE_URL=jdbc:mysql://db:3306/mydb
  
  # دیتابیس
  db:
    image: mysql:8.0
    environment:
      - MYSQL_ROOT_PASSWORD=root
      - MYSQL_DATABASE=mydb
    ports:
      - "3306:3306"
  
  # Redis برای کش
  redis:
    image: redis:alpine
    ports:
      - "6379:6379"
```

```bash
# اجرای همه سرویس‌ها
docker-compose up -d

# توقف همه سرویس‌ها
docker-compose down
```

---

## ۱۵.۳ Kubernetes (K8s)

**چیست؟** سیستم مدیریت Containerها در مقیاس بزرگ. مثل یک Captin برای کشتی‌های (Containerهای) Docker.

### مفاهیم کلیدی:

| مفهوم | توضیح | تشبیه |
|-------|--------|-------|
| Pod | کوچک‌ترین واحد، شامل یک یا چند Container | یه سرباز |
| Deployment | مدیریت Podها (تعداد، بروزرسانی) | یه گروه سرباز |
| Service | دسترسی به Podها از بیرون | آدرس پادگان |
| Ingress | مدیریت ورودی (مسیریابی) | ورودی اصلی |
| ConfigMap | ذخیره تنظیمات | تابلوی اعلانات |
| Secret | ذخیره اطلاعات حساس (پسورد) | صندوق امانات |

### مثال ساده Kubernetes

```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-deployment
spec:
  replicas: 3  # ۳ تا Pod اجرا کن
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: myapp
        image: myapp:latest
        ports:
        - containerPort: 8080
---
# service.yaml (برای دسترسی از بیرون)
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
  selector:
    app: myapp
  ports:
  - port: 80
    targetPort: 8080
  type: LoadBalancer
```

```bash
# a. اعمال به کلاستر
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml

# b. مشاهده Podها
kubectl get pods

# c. مشاهده Serviceها
kubectl get services

# d. مشاهده Deploymentها
kubectl get deployments

# e. مقیاس‌دهی (تغییر تعداد Podها)
kubectl scale deployment myapp-deployment --replicas=5
```

---

## ۱۵.۴ Apache Kafka

**چیست؟** یه سیستم پیام‌رسانی توزیع‌شده (Distributed Messaging System) برای پردازش جریان داده (Stream Processing).

**تشبیه:** مثل یه سیستم حمل و نقل عمومی (مترو) که پیام‌ها رو بین Producerها (فرستنده) و Consumerها (گیرنده) جابه‌جا می‌کنه.

### مفاهیم کلیدی:

| مفهوم | توضیح |
|-------|--------|
| Producer | فرستنده پیام |
| Consumer | گیرنده پیام |
| Topic | دسته‌بندی پیام‌ها (مثل کانال) |
| Partition | تقسیم Topic به چند بخش برای مقیاس‌پذیری |
| Broker | سرور Kafka |

### چه موقع از Kafka استفاده کنیم؟

- Event Sourcing: ثبت همه رویدادها
- Log Aggregation: جمع‌آوری لاگ‌ها از چندین سرویس
- Stream Processing: پردازش لحظه‌ای داده (مثل تحلیل لحظه‌ای)
- Message Queue: ارتباط بین میکروسرویس‌ها

### مثال ساده در جاوا (با Spring Boot)

```java
// Producer - فرستنده
@Service
class KafkaProducerService {
    @Autowired
    private KafkaTemplate<String, String> kafkaTemplate;
    
    public void sendMessage(String message) {
        kafkaTemplate.send("my-topic", message);
    }
}

// Consumer - گیرنده
@Service
class KafkaConsumerService {
    @KafkaListener(topics = "my-topic", groupId = "my-group")
    public void listen(String message) {
        System.out.println("Received: " + message);
    }
}
```

---

## ۱۵.۵ Linux (لینوکس) – دستورات مهم برای بک‌اندکار

### دستورات حیاتی که باید بلد باشی:

#### ۱. مدیریت فایل‌ها و دایرکتوری‌ها

```bash
ls -la          # نمایش همه فایل‌ها با جزئیات
cd /path        # تغییر دایرکتوری
mkdir newdir    # ساخت دایرکتوری جدید
rm -rf dir      # حذف دایرکتوری (با احتیاط!)
cp file1 file2  # کپی فایل
mv file1 file2  # جابجایی یا تغییر نام
```

#### ۲. مشاهده محتوای فایل

```bash
cat file.txt    # نمایش کامل فایل
less file.txt   # نمایش صفحه‌به‌صفحه
tail -f log.txt # دنبال کردن لاگ (Live)
head -n 10 file # ۱۰ خط اول
```

#### ۳. مدیریت فرآیندها (Process)

```bash
ps aux          # مشاهده همه فرآیندها
top             # مشاهده مصرف منابع (لایو)
kill -9 PID     # کشتن یک فرآیند
nohup java -jar app.jar &  # اجرا در پس‌زمینه
```

#### ۴. مدیریت شبکه

```bash
ping google.com # تست اتصال
netstat -tulpn  # مشاهده پورت‌های باز
curl http://api.test # درخواست HTTP
ss -tuln        # مشاهده سوکت‌ها
```

#### ۵. مدیریت دیسک و حافظه

```bash
df -h           # فضای دیسک
du -sh *        # حجم پوشه‌ها
free -h         # حافظه RAM
```

# ❗️ سوال مصاحبه‌ای: فرق apt-get و yum چیه؟

| Feature | apt-get | yum |
|-------|--------|--------|
| توزیع | Debian/Ubuntu | RedHat/CentOS |
| مخزن | APT | RPM |
| دستور نصب | apt-get install | yum install |

---

## ۱۵.۶ CI/CD (Continuous Integration / Continuous Deployment)

**چیست؟** خودکارسازی فرآیند Build، Test و Deploy کد.

### مراحل:

1. **Commit:** توسعه‌دهنده کد رو Push می‌کنه
2. **Build:** سرور خودکار کد رو Build می‌کنه
3. **Test:** تست‌ها اجرا میشن
4. **Deploy:** برنامه روی سرور Deploy میشه

### ابزارهای معروف:

- Jenkins (محبوب‌ترین)
- GitHub Actions
- GitLab CI
- CircleCI

### GitHub Actions مثال

```yaml
name: Java CI/CD

on:
  push:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest
    
    steps:
    - uses: actions/checkout@v2
    
    - name: Set up JDK 17
      uses: actions/setup-java@v2
      with:
        java-version: '17'
    
    - name: Build with Maven
      run: mvn clean package
    
    - name: Build Docker Image
      run: docker build -t myapp .
    
    - name: Deploy to Server
      run: docker push myapp
```


---

## ۱۵.۷ خلاصه یک‌صفحه‌ای

| ابزار | کاربرد | سطح یادگیری |
|-------|--------|-------------|
| Docker | بسته‌بندی برنامه | ضروری ★★★★★ |
| Docker Compose | اجرای چند کانتینر | ضروری ★★★★☆ |
| Kubernetes | مدیریت کانتینرها در مقیاس بزرگ | متوسط ★★★☆☆ |
| Kafka | پیام‌رسانی بین سرویس‌ها | متوسط ★★★☆☆ |
| Linux | سیستمعامل سرور | ضروری ★★★★★ |
| Jenkins/GitHub Actions | CI/CD | مهم ★★★★☆ |

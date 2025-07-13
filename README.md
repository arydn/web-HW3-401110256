# web-HW3-401110256
# پروژه‌ی ترسیم اشکال با React، TypeScript، Spring Boot و PostgreSQL

این پروژه یک اپلیکیشن وب برای ترسیم و مدیریت اشکال (دایره، مربع، مثلث) است که از یک frontend در React و یک backend در Spring Boot استفاده می‌کند. داده‌ها در پایگاه‌داده PostgreSQL ذخیره می‌شوند.

---

## 🖼️ تغییرات و توسعه‌ی بخش Frontend برای اتصال به Backend

### 📁 ساختار پوشه `frontend/src`

- `components/`: شامل کامپوننت‌هایی مثل `Canvas`, `Sidebar`, `Header`, `ShapeCounter`
- `context/`: فایل `ShapesContext.tsx` برای مدیریت global لیست اشکال
- `types/`: تعریف نوع داده‌ها مثل `Shape` و `ShapeType`
- `styles/`: استایل‌های ماژولار مخصوص هر کامپوننت
- `api/`: 📌 **پوشه‌ی جدید اضافه‌شده** برای نگهداری توابع ارتباط با Backend (`api.ts`)

---

### ✏️ تغییرات اصلی برای پشتیبانی از Backend

#### ✅ 1. افزودن فیلد ورود نام کاربر (`username`)

در فایل `Header.tsx`، یک input جدید اضافه شده تا کاربر بتواند نام خود را وارد کند. این مقدار در state ذخیره شده و برای توابع ذخیره و بارگذاری به سرور ارسال می‌شود.

```tsx
<input
  type="text"
  className={styles.userInput}
  value={username}
  onChange={(e) => setUsername(e.target.value)}
  placeholder="Username (e.g., user1)"
/>
```

🔸 دلیل:
- به ازای هر کاربر، یک نقاشی جداگانه در پایگاه داده ذخیره می‌شود.
- این طراحی ساده نیاز به احراز هویت ندارد ولی چندکاربره بودن را پشتیبانی می‌کند.

---

#### 💾 2. افزودن دکمه‌های ذخیره و بارگذاری به سرور

در همان فایل `Header.tsx`، دو دکمه جدید برای ذخیره و بارگذاری ترسیم‌ها از سرور اضافه شده‌اند. این دکمه‌ها توابع `exportToBackend` و `importFromBackend` را صدا می‌زنند.

```tsx
<button onClick={exportToBackend}>Save to Server</button>
<button onClick={importFromBackend}>Load from Server</button>
```

🔸 عملکرد:
- دکمه‌ی Save با استفاده از `fetch` یک درخواست POST به سرور ارسال می‌کند و داده‌ها را ذخیره می‌کند.
- دکمه‌ی Load با استفاده از `fetch` یک درخواست GET می‌فرستد و لیست اشکال را بازیابی می‌کند.

---

#### 🔁 3. ایجاد فایل جدید `api/api.ts` برای ارتباط با سرور

```ts
export const saveDrawing = async (username, drawingName, shapes) => { ... }
export const fetchDrawing = async (username) => { ... }
```

🔸 ساختار کلی:
- `saveDrawing`: فقط `type`, `x`, `y` هر شکل را به سرور ارسال می‌کند و از ارسال `id` محلی جلوگیری می‌کند (زیرا سمت سرور، `id` خودکار تولید می‌شود).
- `fetchDrawing`: داده‌ها را از سرور دریافت کرده و به ساختار سازگار با context در frontend تبدیل می‌کند.

🔸 مزایا:
- جدا کردن منطق API از منطق رابط کاربری.
- در آینده امکان گسترش ساده‌تر (مثل احراز هویت، مسیرهای بیشتر یا توابع delete/update) فراهم می‌شود.

---

#### 🎨 4. هماهنگی با Canvas برای پشتیبانی از اشکال دریافت‌شده از سرور

در فایل `Canvas.tsx` تغییراتی ایجاد شد تا اشکال بارگذاری‌شده از سرور (که فاقد `id` یکتا هستند) برای عملکرد drag/delete قابل استفاده باشند. این کار با افزودن `uuid` جدید برای هر شکل انجام شده است:

```tsx
const loadedShapes = fetched.shapes.map(s => ({
  ...s,
  id: uuidv4() // ایجاد شناسه یکتا در frontend
}));
```

🔸 دلیل:
- چون اشکال سمت سرور فقط مختصات و نوع دارند، و `id` لازم برای عملیات داخلی frontend مثل جابجایی و حذف است، یک شناسه در سمت frontend اضافه می‌شود.

---

## ☕ Backend با Spring Boot و PostgreSQL

### 📦 ساختار کلی

```
src/main/java/com/example/drawing/
├── controller/       → کنترلر REST برای ارتباط API
├── service/          → منطق تجاری برنامه (Business Logic)
├── model/            → مدل‌های JPA برای تعریف جداول دیتابیس
├── repository/       → تعریف متدهای ارتباطی با دیتابیس
├── init/             → مقداردهی اولیه کاربران پیش‌فرض
└── DrawingApplication.java → نقطه شروع برنامه
```

### 📄 مدل‌ها (Entities)

#### `User.java`
```java
@Entity
public class User {
    @Id
    private String username;
}
```
- `@Entity`: این کلاس به یک جدول واقعی در دیتابیس نگاشت می‌شود.

- `@Id`: مشخص می‌کند که username کلید اصلی (Primary Key) جدول است.

- بدون @GeneratedValue، چون کلید دستی تعریف شده (نام کاربر)، نیازی به auto-increment نیست.

#### `Shape.java`
```java
@Entity
@Table(name = "shapes")
public class Shape {
    @Id @GeneratedValue
    private Long id;
    private String username, drawingName, type;
    private int x, y;
}
```
- `@Entity`: این کلاس به جدول shapes متصل است.

- `@Table`(name = "shapes"): نام جدول در دیتابیس را صراحتاً مشخص می‌کند.

- `@Id` + `@GeneratedValue`: ستون id به عنوان کلید اصلی با مقدار auto-increment تعریف شده.

- `username و drawingName`: برای ارتباط بین شکل‌ها و کاربران/ترسیم‌ها استفاده می‌شوند.

### 📁 `repository/`

```java
public interface ShapeRepository extends JpaRepository<Shape, Long> {
    List<Shape> findByUsername(String username);
    void deleteByUsername(String username);
}
```
- ارث‌بری از JpaRepository: متدهای CRUD مانند save, findAll, delete به‌صورت خودکار فراهم می‌شوند.
- findByUsername و deleteByUsername: Spring Data JPA این توابع را با استفاده از نام‌گذاری خودکار queryها پیاده‌سازی می‌کند.

### 🧠 `DrawingService.java`

```java
@Service
public class DrawingService {
    public Drawing load(String username) { ... }
    @Transactional
    public void save(Drawing drawing) { ... }
}
```
- `@Service`: این کلاس یک جزء منطقی (logic layer) است، نه کنترلر و نه دیتابیس.

- `load(...)`: اگر کاربر وجود نداشته باشد null برمی‌گرداند. در غیر این‌صورت، تمام اشکال مربوط به کاربر را از دیتابیس می‌گیرد.

- `save(...)`:

- اگر کاربر جدید است، ساخته می‌شود.

- همه‌ی اشکال قبلی حذف می‌شوند.

- اشکال جدید به نام کاربر و ترسیم اضافه می‌شوند.

- `@Transactional`: تضمین می‌کند که عملیات حذف و افزودن به صورت یکجا (اتمی) انجام شود. اگر خطایی پیش بیاید، همه چیز به عقب برمی‌گردد.

### 🌐 `DrawingController.java`

```java
@RestController
@RequestMapping("/api/drawings")
@CrossOrigin(origins = "http://localhost:3000")
public class DrawingController {
    @GetMapping("/{username}") ...
    @PostMapping("/{username}") ...
}
```
- @RestController: ترکیبی از @Controller و @ResponseBody، برای ایجاد API که JSON برمی‌گرداند.

- @RequestMapping("/api/drawings"): مسیر پایه‌ی تمام endpointها.

- @CrossOrigin(...): اجازه‌ی درخواست از frontend (پورت ۳۰۰۰) را می‌دهد (CORS policy).

- @GetMapping("/{username}"): بارگذاری ترسیم‌ها از سرور.

- @PostMapping("/{username}"): ذخیره‌سازی ترسیم جدید در سرور.

- @PathVariable: نام کاربر از URL گرفته می‌شود.

- @RequestBody: داده‌ی JSON ورودی را به یک شی Drawing تبدیل می‌کند.

- ResponseEntity: امکان بازگشت دقیق پاسخ (مثل 404 یا 200) را فراهم می‌کند.

### 🧪 `UserDataInitializer.java`

```java
@Component
public class UserDataInitializer implements CommandLineRunner {
    public void run(...) {
        List.of("user1", "user2", "user3").forEach(...);
    }
}
```


- @Component: این کلاس توسط Spring در زمان اجرا اسکن می‌شود.

- `CommandLineRunner`: اجرای متد run() بلافاصله بعد از شروع برنامه.

- هدف: اطمینان از اینکه user1, user2, user3 همیشه در دیتابیس موجودند.
---

## 🧪 پایگاه‌داده

- از PostgreSQL برای ذخیره‌ی اطلاعات استفاده شده.
- با کمک Spring Data JPA عملیات ذخیره و بارگذاری انجام می‌شود.

---


## 🧾 جمع‌بندی

پروژه نشان می‌دهد چطور می‌توان با React و Spring Boot یک برنامه‌ی تعاملی با قابلیت ذخیره‌سازی داده سمت سرور ایجاد کرد. معماری ماژولار frontend و backend باعث توسعه‌پذیری ساده‌تر می‌شود.

در روند توسعه، از مدل ChatGPT برای راهنمایی در اتصال frontend به backend، طراحی API، استفاده‌ی درست از JPA  استفاده شد

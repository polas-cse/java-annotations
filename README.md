# ☕ Java Annotations — সম্পূর্ণ গাইড (A to Z)

> **লেখার উদ্দেশ্য:** এই ফাইলটা একবার পড়লেই যেন Java-র সব annotation সম্পর্কে crystal clear ধারণা হয়।  
> **ভাষা:** Bangla + English mixed (technical term গুলো English-এই রাখা হয়েছে)  
> **কভারেজ:** Core Java · Spring Core · Spring Boot · Spring MVC · Spring Security · JPA/Hibernate · Spring Data Auditing · AOP · Bean Validation · Lombok · Jackson · JUnit 5 · Mockito · OpenAPI

---

## 📌 Annotation কী এবং কেন ব্যবহার করি?

Annotation হলো Java code-এর উপর একধরনের **metadata label** — এটা নিজে কোনো কাজ করে না, কিন্তু Framework (Spring, Hibernate), Compiler, অথবা Reflection API এই label দেখে সিদ্ধান্ত নেয় কী করবে।

```
তোমার Code → @Service লাগানো আছে
         ↓
Spring Context Scan করে → "ওহ, এটা Service Bean!"
         ↓
Automatically register করে Application Context-এ
```

> **মনে রাখো:** Annotation = চিরকুট। চিরকুট নিজে কাজ করে না — যে পড়ে সে কাজ করে।

---

## 📚 Table of Contents

1. [Core Java Annotations](#1-core-java-annotations)
2. [Meta Annotations](#2-meta-annotations-annotation-এর-উপর-annotation)
3. [Spring Core Annotations](#3-spring-core-annotations)
4. [Spring Boot Annotations](#4-spring-boot-annotations)
5. [Spring MVC / REST Annotations](#5-spring-mvc--rest-annotations)
6. [Spring Security Annotations](#6-spring-security-annotations)
7. [JPA / Jakarta Persistence Annotations](#7-jpa--jakarta-persistence-annotations)
8. [Hibernate-Specific Annotations](#8-hibernate-specific-annotations)
9. [Spring Data Auditing Annotations](#9-spring-data-auditing-annotations)
10. [AOP Annotations](#10-aop-annotations)
11. [Bean Validation Annotations](#11-bean-validation-annotations)
12. [Lombok Annotations](#12-lombok-annotations)
13. [Jackson JSON Annotations](#13-jackson-json-annotations)
14. [JUnit 5 Annotations](#14-junit-5-annotations)
15. [Mockito Annotations](#15-mockito-annotations)
16. [OpenAPI / Swagger Annotations](#16-openapi--swagger-annotations)
17. [Custom Annotation তৈরি](#17-custom-annotation-তৈরি)
18. [Quick Cheat Sheet](#18-quick-cheat-sheet)

---

## 1. Core Java Annotations

> `java.lang` package-এ থাকে। কোনো extra dependency লাগে না।

---

### `@Override`

**কী করে:**  
Parent class বা interface-এর method override হচ্ছে কিনা compile-time-এ verify করে। যদি annotated method আসলে কোনো method override না করে (নাম ভুল হলে বা parameter আলাদা হলে) — compiler সাথে সাথে error দেয়।

**কেন ব্যবহার করবে:**  
এটা না দিলে, তুমি ভাবছো override করছো কিন্তু আসলে নতুন একটা method বানিয়ে ফেলছো — এই silent bug ধরা অনেক কঠিন।

**কখন দেবে না:**  
`@Override` দেওয়া কখনো ক্ষতিকর না। সবসময় দাও।

```java
class Animal {
    public void sound() {
        System.out.println("...");
    }
}

class Dog extends Animal {
    @Override
    public void sound() {
        System.out.println("Bark!");
    }

    // @Override ছাড়া যদি লিখতাম "Sound()" — নতুন method হতো, override হতো না!
    // @Override দিলে compile error → ভুল ধরা পড়তো
}
```

---

### `@Deprecated`

**কী করে:**  
কোনো class, method, field বা constructor-কে "পুরনো এবং ভবিষ্যতে remove হবে" হিসেবে mark করে। যে কেউ এটা ব্যবহার করলে compiler warning দেয়।

**Java 9+ এ নতুন attributes:**
- `since = "2.0"` → কোন version থেকে deprecated
- `forRemoval = true` → ভবিষ্যতে delete হবে

**কেন ব্যবহার করবে:**  
পুরনো API বাদ দিতে চাইলে হঠাৎ delete না করে @Deprecated দাও — existing user-দের migrate করার সময় দেয়।

**Pro Tip:** Javadoc-এ `@deprecated` tag দিয়ে বলো কোন method ব্যবহার করতে হবে।

```java
/**
 * @deprecated Use {@link #getFullName()} instead.
 */
@Deprecated(since = "2.0", forRemoval = true)
public String getName() {
    return getFullName(); // নতুন method-এ delegate করো
}

// নতুন method
public String getFullName() {
    return this.fullName;
}
```

---

### `@SuppressWarnings`

**কী করে:**  
Specific compiler warning চুপ করায়। Warning গুলো মিথ্যা না, কিন্তু কখনো কখনো intentionally এভাবে লেখা হয় — তখন warning suppress করা যায়।

**Common Values:**
| Value | কোন Warning suppress হয় |
|-------|--------------------------|
| `"unchecked"` | Raw generic type casting |
| `"deprecation"` | Deprecated API ব্যবহার |
| `"unused"` | Unused variable/method |
| `"rawtypes"` | Raw type (no generic) |
| `"serial"` | Missing serialVersionUID |
| `"all"` | সব warning (avoid করো!) |

**সতর্কতা:** `"all"` দিলে real bug-এর warning-ও চলে যায়। সবচেয়ে ছোট scope-এ সবচেয়ে specific warning suppress করো।

```java
@SuppressWarnings("unchecked")
public List<User> getUsersFromLegacyDao() {
    // পুরনো code raw List return করে, আমরা জানি এটা List<User>
    return (List<User>) legacyDao.findAll();
}

@SuppressWarnings({"deprecation", "unchecked"})
public void legacyBridge() {
    // একাধিক suppress
}
```

---

### `@FunctionalInterface`

**কী করে:**  
Interface-টিতে exactly **একটি abstract method** থাকতে হবে — এটা compile-time-এ enforce করে। এরকম interface-এ Lambda expression ব্যবহার করা যায়।

**কেন দরকার:**  
কেউ ভুলে second abstract method add করলে compile error হয় — Lambda compatibility নষ্ট হওয়া থেকে বাঁচায়।

**মনে রাখো:** Default ও static method থাকলে সমস্যা নেই — সেগুলো count হয় না।

```java
@FunctionalInterface
public interface Transformer<T, R> {
    R transform(T input); // একমাত্র abstract method

    // Default method OK
    default Transformer<T, R> withLogging() {
        return input -> {
            System.out.println("Input: " + input);
            return transform(input);
        };
    }
}

// Lambda দিয়ে ব্যবহার
Transformer<String, Integer> lengthOf = s -> s.length();
System.out.println(lengthOf.transform("Hello")); // 5

// Method reference দিয়েও
Transformer<String, String> upper = String::toUpperCase;
```

---

### `@SafeVarargs`

**কী করে:**  
Varargs + Generic type ব্যবহারে "heap pollution" warning suppress করে। তুমি assert করছো: "আমি verify করেছি, এই code safe।"

**কোথায় ব্যবহার করা যায়:** `final`, `static`, বা `private` method ও constructor-এ।

**কখন safe:** Varargs array শুধু read করলে। কখন unsafe: array-এ store করলে বা type mix করলে।

```java
@SafeVarargs
public final <T> List<T> flatten(List<T>... lists) {
    List<T> result = new ArrayList<>();
    for (List<T> list : lists) {
        result.addAll(list); // শুধু read করছি, safe
    }
    return result;
}

List<String> names = flatten(
    Arrays.asList("Polas", "Jakariya"),
    Arrays.asList("Sakib", "Rifat")
);
```

---

### `@Native`

**কী করে:**  
Field টি native (C/C++) code থেকে reference করা হতে পারে — এটা indicate করে। সাধারণত JDK internals-এ দেখা যায়।

```java
public class NativeConstants {
    @Native
    public static final int MAX_BUFFER = 8192;
}
```

---

## 2. Meta Annotations (Annotation-এর উপর Annotation)

> নিজের custom annotation বানাতে গেলে এগুলো লাগবেই।

---

### `@Target`

**কী করে:**  
Custom annotation কোথায় apply করা যাবে তা সীমাবদ্ধ করে। না দিলে যেকোনো জায়গায় লাগানো যায় — এটা confusion তৈরি করে।

```java
// METHOD এবং TYPE (class/interface) এ apply করা যাবে
@Target({ElementType.METHOD, ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
public @interface AuditLog {
    String action() default "";
}
```

| ElementType | কোথায় apply হয় |
|-------------|-----------------|
| `TYPE` | Class, Interface, Enum, Record |
| `METHOD` | Method |
| `FIELD` | Field/Attribute |
| `PARAMETER` | Method parameter |
| `CONSTRUCTOR` | Constructor |
| `LOCAL_VARIABLE` | Local variable |
| `ANNOTATION_TYPE` | অন্য annotation |
| `PACKAGE` | Package |
| `TYPE_USE` | যেকোনো type ব্যবহারে |
| `TYPE_PARAMETER` | Generic type parameter `<T>` |
| `MODULE` | Module (Java 9+) |
| `RECORD_COMPONENT` | Record component (Java 14+) |

---

### `@Retention`

**কী করে:**  
Annotation কতক্ষণ জীবিত থাকবে তা নির্ধারণ করে।

| RetentionPolicy | মানে | কারা ব্যবহার করে |
|-----------------|------|------------------|
| `SOURCE` | Compile হলে হারিয়ে যায় | Lombok |
| `CLASS` | .class file-এ থাকে, runtime-এ না | Bytecode tools |
| `RUNTIME` | Runtime-এ Reflection দিয়ে পড়া যায় | Spring, Hibernate, Jackson |

**সবচেয়ে গুরুত্বপূর্ণ নিয়ম:** Spring বা যেকোনো framework-এ custom annotation কাজ করাতে হলে `RetentionPolicy.RUNTIME` দিতেই হবে।

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME) // ← এটা না দিলে Spring পড়তে পারবে না!
public @interface RateLimit {
    int requestsPerMinute() default 60;
}

// Runtime-এ Reflection দিয়ে পড়া
Method method = MyController.class.getMethod("getData");
RateLimit rl = method.getAnnotation(RateLimit.class);
System.out.println(rl.requestsPerMinute()); // 60
```

---

### `@Documented`

**কী করে:**  
এই annotation দিয়ে annotate করা element-গুলো Javadoc-এ দেখাবে।

**কেন দরকার:** Public library বানালে তোমার custom annotation গুলো API docs-এ দেখা যাবে — users জানতে পারবে।

```java
@Documented
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface ApiVersion {
    int value();
}
```

---

### `@Inherited`

**কী করে:**  
Parent class-এ class-level annotation থাকলে child class automatically সেটা পাবে।

**সতর্কতা:** শুধু **class-level** annotation-এ কাজ করে। Method বা field-এ কাজ করে না। Interface-এর annotation inherit হয় না।

```java
@Inherited
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
public @interface Secured {}

@Secured
public class BaseService {}

public class UserService extends BaseService {}
// UserService.class.isAnnotationPresent(Secured.class) → true (inherited!)
```

---

### `@Repeatable`

**কী করে:**  
একই annotation একই জায়গায় একাধিকবার ব্যবহার করতে দেয়। Container annotation বানাতে হয়।

```java
// Repeatable annotation
@Repeatable(Schedules.class)
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface Schedule {
    String cron();
    String zone() default "Asia/Dhaka";
}

// Container annotation
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface Schedules {
    Schedule[] value();
}

// একই method-এ দুটো @Schedule!
@Schedule(cron = "0 0 8 * * MON-FRI")
@Schedule(cron = "0 0 20 * * MON-FRI", zone = "UTC")
public void generateReport() { ... }

// Reflection দিয়ে পড়া — getAnnotationsByType() ব্যবহার করো
Schedule[] schedules = method.getAnnotationsByType(Schedule.class);
```

---

## 3. Spring Core Annotations

> Dependency: `spring-context`

---

### `@Component`

**কী করে:**  
Class-টিকে Spring-managed Bean হিসেবে register করে। Classpath scanning (`@ComponentScan`) চালু থাকলে Spring নিজেই খুঁজে নেয় এবং Application Context-এ রাখে।

**কখন ব্যবহার করবে:** Utility class, helper, adapter — যেগুলো @Service/@Repository/@Controller-এর কোনোটাই না কিন্তু Spring Bean হওয়া দরকার।

**Bean name:** Default হলো uncapitalized class name। Custom: `@Component("myValidator")`

```java
@Component
public class BdPhoneValidator {

    public boolean isValid(String phone) {
        return phone != null && phone.matches("^\\+8801[3-9]\\d{8}$");
    }
}

// যেকোনো জায়গায় inject করো
@Service
public class UserService {
    private final BdPhoneValidator phoneValidator;

    public UserService(BdPhoneValidator phoneValidator) {
        this.phoneValidator = phoneValidator;
    }
}
```

---

### `@Service`

**কী করে:**  
`@Component`-এর specialized version — Business Logic layer-এর জন্য। Spring-এ technically `@Component`-এর মতোই কাজ করে, কিন্তু semantic অর্থ আলাদা: এই class-এ business rule থাকে।

**কেন আলাদা:** Code readability, layered architecture clarity, AOP দিয়ে specific layer target করা সহজ হয়।

**Best Practice:** `@Transactional` annotation সবসময় Service layer-এ দাও।

```java
@Service
public class BookingService {

    private final BookingRepository bookingRepo;
    private final EmailService emailService;
    private final PhoneValidator phoneValidator;

    // Constructor injection (recommended)
    public BookingService(BookingRepository bookingRepo,
                          EmailService emailService,
                          PhoneValidator phoneValidator) {
        this.bookingRepo = bookingRepo;
        this.emailService = emailService;
        this.phoneValidator = phoneValidator;
    }

    @Transactional
    public Booking createBooking(BookingRequest req) {
        // Business logic এখানে
        if (!phoneValidator.isValid(req.getPhone())) {
            throw new InvalidPhoneException("Invalid BD phone number");
        }
        Booking saved = bookingRepo.save(map(req));
        emailService.sendConfirmation(saved);
        return saved;
    }
}
```

---

### `@Repository`

**কী করে:**  
Data Access Layer (DAO) এর জন্য। `@Component`-এর মতো bean register করে, **plus** একটা বিশেষ কাজ করে: Database-specific exception (HibernateException, JDBCException) গুলো Spring-এর unified `DataAccessException`-এ translate করে।

**এই translation কেন দরকার:** Service layer-এ Hibernate বা JDBC-specific exception catch করতে হয় না — শুধু Spring-এর exception hierarchy দিয়ে কাজ চলে। ORM বদলালেও service layer change হয় না।

**Spring Data JPA ব্যবহার করলে:** JpaRepository extend করলে @Repository automatically আসে।

```java
@Repository
public class ProductDao {

    @PersistenceContext
    private EntityManager em;

    public Optional<Product> findBySku(String sku) {
        try {
            return em.createQuery(
                "SELECT p FROM Product p WHERE p.sku = :sku", Product.class)
                .setParameter("sku", sku)
                .getResultStream()
                .findFirst();
        } catch (HibernateException e) {
            // Spring automatically convert করে DataAccessException-এ
            // তুমি কিছু করতে হবে না
            throw e;
        }
    }
}
```

---

### `@Controller` এবং `@RestController`

**`@Controller`:** Spring MVC Controller। Method গুলো View name (JSP/Thymeleaf template) return করে।

**`@RestController`:** `@Controller` + `@ResponseBody` একসাথে। সব method-এর return value সরাসরি HTTP response body-তে JSON/XML হিসেবে যায়।

```java
// Traditional MVC — View render করে
@Controller
public class HomeController {

    @GetMapping("/dashboard")
    public String dashboard(Model model) {
        model.addAttribute("stats", statsService.get());
        return "dashboard"; // templates/dashboard.html render হবে
    }
}

// REST API — JSON return করে
@RestController
@RequestMapping("/api/v1/products")
public class ProductController {

    private final ProductService productService;

    public ProductController(ProductService productService) {
        this.productService = productService;
    }

    @GetMapping("/{id}")
    public ResponseEntity<ProductDto> getProduct(@PathVariable Long id) {
        return ResponseEntity.ok(productService.findById(id));
    }
}
```

---

### `@Autowired`

**কী করে:**  
Spring-এর Dependency Injection mechanism। Spring Application Context থেকে matching type-এর Bean খুঁজে inject করে।

**তিন ধরনের injection:**

| Type | কীভাবে | Recommended? |
|------|--------|--------------|
| Constructor | Constructor-এ | ✅ হ্যাঁ |
| Setter | Setter method-এ | ⚠️ Optional dependency-তে |
| Field | Field-এ directly | ❌ না |

**কেন Field Injection avoid করবে:**
- Test করতে গেলে Reflection লাগে (complicated)
- `final` field হয় না → immutability নষ্ট
- Dependency graph লুকিয়ে যায়

**Spring 4.3+ shortcut:** Single constructor থাকলে `@Autowired` লিখতে হয় না।

```java
@Service
public class OrderService {

    // ✅ Constructor Injection — BEST PRACTICE
    private final OrderRepository orderRepo;
    private final PaymentGateway paymentGateway;
    private final EmailService emailService;

    // @Autowired optional since Spring 4.3 (single constructor)
    public OrderService(OrderRepository orderRepo,
                        PaymentGateway paymentGateway,
                        EmailService emailService) {
        this.orderRepo = orderRepo;
        this.paymentGateway = paymentGateway;
        this.emailService = emailService;
    }

    // ❌ Avoid this — Field Injection
    // @Autowired
    // private PaymentGateway paymentGateway;
}
```

---

### `@Qualifier`

**কী করে:**  
একই type-এর একাধিক Bean থাকলে কোনটা inject হবে তা specify করে। `NoUniqueBeanDefinitionException` থেকে বাঁচায়।

```java
public interface StorageService {
    void upload(byte[] data, String filename);
}

@Service("s3Storage")
public class S3StorageService implements StorageService { ... }

@Service("localDiskStorage")
public class LocalDiskStorageService implements StorageService { ... }

@Service
public class FileUploadService {

    private final StorageService storageService;

    public FileUploadService(@Qualifier("s3Storage") StorageService storageService) {
        this.storageService = storageService;
    }
}
```

---

### `@Primary`

**কী করে:**  
একাধিক Bean-এর মধ্যে default Bean হিসেবে mark করে। `@Qualifier` না দিলে এটাই inject হবে।

```java
@Service
@Primary // Default implementation
public class SmtpEmailService implements EmailService { ... }

@Service
@Profile("dev") // Dev-এ override করা
public class FakeEmailService implements EmailService { ... }
```

---

### `@Bean`

**কী করে:**  
`@Configuration` class-এর method থেকে Bean তৈরি করে। Third-party library-র object গুলোকে Spring Bean বানাতে কাজে লাগে (যেগুলোতে @Component দেওয়া যায় না)।

**Method name = Bean name** (default)।

**গুরুত্বপূর্ণ:** `@Configuration` class Spring proxy করে — একই `@Bean` method বারবার call করলেও একটাই instance তৈরি হয় (singleton)।

```java
@Configuration
public class InfrastructureConfig {

    @Bean
    public RestTemplate restTemplate() {
        RestTemplate rt = new RestTemplate();
        rt.setConnectTimeout(Duration.ofSeconds(5));
        return rt;
    }

    @Bean
    public ObjectMapper objectMapper() {
        return new ObjectMapper()
            .findAndRegisterModules()
            .disable(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS)
            .enable(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES);
    }

    @Bean
    @Profile("dev")
    public DataSource h2DataSource() {
        return new EmbeddedDatabaseBuilder()
            .setType(EmbeddedDatabaseType.H2)
            .build();
    }
}
```

---

### `@Configuration`

**কী করে:**  
Class-টি Spring Configuration source — এখানে `@Bean` define করা হয়। `@Component`-এর special version যেটা Spring proxy করে singleton semantics নিশ্চিত করে।

```java
@Configuration
public class SecurityConfig {

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder(12);
    }

    @Bean
    public AuthenticationManager authManager(
            UserDetailsService userDetailsService) {
        DaoAuthenticationProvider provider = new DaoAuthenticationProvider();
        provider.setUserDetailsService(userDetailsService);
        provider.setPasswordEncoder(passwordEncoder()); // এই call singleton-ই return করে
        return new ProviderManager(provider);
    }
}
```

---

### `@Value`

**কী করে:**  
`application.properties` বা `application.yml` থেকে value inject করে। SpEL (Spring Expression Language) ও support করে।

**Syntax:**
- `${property.key}` → Property file থেকে
- `${property.key:defaultValue}` → Default value সহ
- `#{expression}` → SpEL expression

```java
@Component
public class ExternalApiClient {

    @Value("${payment.api.base-url}")
    private String baseUrl;

    @Value("${payment.api.timeout:5000}") // default 5000ms
    private int timeoutMs;

    @Value("${feature.beta.enabled:false}")
    private boolean betaEnabled;

    // SpEL: computation
    @Value("#{${payment.api.timeout:5000} * 2}")
    private int readTimeout; // timeout-এর দ্বিগুণ

    // SpEL: List inject
    @Value("#{'${allowed.origins}'.split(',')}")
    private List<String> allowedOrigins;
}
```

```properties
# application.properties
payment.api.base-url=https://api.payment.com
payment.api.timeout=3000
feature.beta.enabled=true
allowed.origins=http://localhost:3000,https://app.example.com
```

---

### `@Scope`

**কী করে:**  
Bean-এর lifecycle নির্ধারণ করে।

| Scope | মানে | কখন ব্যবহার |
|-------|------|-------------|
| `singleton` | একটাই instance (Default) | Service, Repository |
| `prototype` | প্রতিবার নতুন instance | Stateful component |
| `request` | প্রতি HTTP request-এ নতুন | Web app |
| `session` | প্রতি HTTP session-এ নতুন | User-specific state |
| `application` | পুরো app-এ একটা (ServletContext) | App-wide config |

```java
@Component
@Scope("prototype") // প্রতিবার নতুন instance
public class CsvReportGenerator {
    private final List<String> rows = new ArrayList<>();

    public void addRow(String row) { rows.add(row); }
    public String build() { return String.join("\n", rows); }
}
```

---

### `@Lazy`

**কী করে:**  
Bean startup-এ তৈরি না হয়ে প্রথমবার ব্যবহারের সময় তৈরি হবে।

**কেন দরকার:** Heavy initialization আছে এমন Bean গুলোর জন্য — startup time কমায়।

```java
@Component
@Lazy
public class HeavyReportEngine {
    public HeavyReportEngine() {
        // ধরো এখানে ৩ সেকেন্ড initialization লাগে
        loadTemplates();
        warmUpCache();
    }
}
```

---

### `@Profile`

**কী করে:**  
নির্দিষ্ট environment-এ (dev/test/prod) Bean active হবে।

```java
@Service
@Profile("dev")
public class MockEmailService implements EmailService {
    @Override
    public void send(String to, String body) {
        System.out.println("DEV: Would send to " + to); // Actually send করে না
    }
}

@Service
@Profile("prod")
public class SmtpEmailService implements EmailService {
    @Override
    public void send(String to, String body) {
        // Real email send করে
        smtpClient.send(to, body);
    }
}
```

```properties
# application.properties
spring.profiles.active=dev
```

---

### `@Transactional`

**কী করে:**  
Database transaction manage করে। Method শুরুতে transaction start, শেষে commit। `RuntimeException` throw হলে rollback।

**Key Attributes:**

| Attribute | মানে | Default |
|-----------|------|---------|
| `propagation` | Existing transaction handle | `REQUIRED` |
| `isolation` | Read isolation level | Database default |
| `readOnly` | SELECT-only hint | `false` |
| `rollbackFor` | কোন exception-এ rollback | `RuntimeException` |
| `noRollbackFor` | কোন exception-এ rollback না | - |
| `timeout` | Max seconds | `-1` (no timeout) |

**Propagation Types:**

| Type | মানে |
|------|------|
| `REQUIRED` | Existing transaction use করো, না থাকলে নতুন (Default) |
| `REQUIRES_NEW` | সবসময় নতুন transaction, existing suspend করো |
| `NESTED` | Existing-এর ভেতরে nested transaction |
| `SUPPORTS` | Existing থাকলে use করো, না থাকলে without transaction |
| `NOT_SUPPORTED` | Transaction ছাড়া চালাও |
| `MANDATORY` | Existing transaction থাকতেই হবে |
| `NEVER` | Transaction থাকলে exception |

```java
@Service
public class MoneyTransferService {

    @Transactional
    public void transfer(Long fromId, Long toId, BigDecimal amount) {
        accountRepo.debit(fromId, amount);   // এটা success হলো
        accountRepo.credit(toId, amount);    // এটা fail হলে → দুটোই rollback!
    }

    @Transactional(readOnly = true) // SELECT-only, faster
    public List<Transaction> getHistory(Long accountId) {
        return transactionRepo.findByAccountId(accountId);
    }

    // REQUIRES_NEW: outer transaction fail করলেও এটা commit হবে
    @Transactional(propagation = Propagation.REQUIRES_NEW,
                   rollbackFor = Exception.class)
    public void saveAuditLog(AuditEntry entry) {
        auditRepo.save(entry);
    }
}
```

**সতর্কতা:**
- Private method-এ কাজ করে না (AOP proxy bypass হয়)
- Same class-এর ভেতর থেকে call করলে proxy bypass হয়
- Exception catch করে না throw করলে rollback হয় না

---

### `@Cacheable`, `@CachePut`, `@CacheEvict`

**কী করে:**
- `@Cacheable` → প্রথমবার DB hit, পরে cache থেকে দেয়
- `@CachePut` → সবসময় DB-তে যায় এবং cache update করে
- `@CacheEvict` → Cache থেকে entry delete করে

```java
@Service
public class ProductService {

    @Cacheable(value = "products", key = "#id")
    public Product findById(Long id) {
        // প্রথমবার DB-তে যাবে, পরে cache থেকে আসবে
        return productRepo.findById(id).orElseThrow();
    }

    @CachePut(value = "products", key = "#product.id")
    public Product update(Product product) {
        // DB update করে এবং cache-ও update করে
        return productRepo.save(product);
    }

    @CacheEvict(value = "products", key = "#id")
    public void delete(Long id) {
        // DB থেকে delete করে, cache-ও clear করে
        productRepo.deleteById(id);
    }

    @CacheEvict(value = "products", allEntries = true)
    public void clearAllProductCache() {
        // পুরো products cache clear
    }
}
```

> Enable করতে main class বা config-এ `@EnableCaching` লাগবে।

---

### `@Async`

**কী করে:**  
Method-টি আলাদা thread-এ asynchronously চলবে। Caller immediately return পাবে, method background-এ চলতে থাকবে।

**সতর্কতা:**
- Same class-এর ভেতর থেকে call করলে async কাজ করে না (proxy bypass)
- `void` return হলে exception silently হারিয়ে যায় — `CompletableFuture` use করো
- Production-এ `ThreadPoolTaskExecutor` configure করো

```java
@Service
public class NotificationService {

    @Async
    public CompletableFuture<Boolean> sendEmailAsync(String to, String subject, String body) {
        try {
            emailClient.send(to, subject, body); // এটা 2 সেকেন্ড নিতে পারে
            return CompletableFuture.completedFuture(true);
        } catch (Exception e) {
            log.error("Email failed to {}: {}", to, e.getMessage());
            return CompletableFuture.failedFuture(e);
        }
    }
}

// Config
@Configuration
@EnableAsync
public class AsyncConfig {

    @Bean
    public Executor taskExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(5);
        executor.setMaxPoolSize(20);
        executor.setQueueCapacity(100);
        executor.setThreadNamePrefix("async-");
        executor.initialize();
        return executor;
    }
}
```

---

### `@Scheduled`

**কী করে:**  
Method-টি নির্দিষ্ট সময় পর পর automatically execute হবে।

**তিনটি mode:**
- `cron` → Cron expression দিয়ে (সবচেয়ে flexible)
- `fixedDelay` → Last execution শেষ হওয়ার N ms পরে
- `fixedRate` → প্রতি N ms-এ (execution time নির্বিশেষে)

**Cron Format:** `second minute hour day-of-month month day-of-week`

```java
@Component
public class MaintenanceScheduler {

    // প্রতিদিন রাত ১২টায়
    @Scheduled(cron = "0 0 0 * * *")
    public void cleanExpiredSessions() { ... }

    // প্রতি সোম-শুক্র সকাল ৮টায় ঢাকা timezone
    @Scheduled(cron = "0 0 8 * * MON-FRI", zone = "Asia/Dhaka")
    public void sendDailyReport() { ... }

    // প্রতি ৩০ সেকেন্ড পরে (last run শেষ হওয়ার পর)
    @Scheduled(fixedDelay = 30_000)
    public void syncInventory() { ... }

    // ঠিক প্রতি ১ মিনিটে (৫ সেকেন্ড পরে শুরু)
    @Scheduled(fixedRate = 60_000, initialDelay = 5_000)
    public void heartbeat() { ... }
}
```

> Enable করতে `@EnableScheduling` লাগবে।

---

### `@EventListener`

**কী করে:**  
Application event-এ react করে। Spring-এর event system দিয়ে loosely coupled communication।

```java
// Custom Event
public class BookingCreatedEvent {
    private final Booking booking;
    public BookingCreatedEvent(Booking booking) { this.booking = booking; }
    public Booking getBooking() { return booking; }
}

// Event Publish করো
@Service
public class BookingService {
    @Autowired
    private ApplicationEventPublisher publisher;

    @Transactional
    public Booking create(BookingRequest req) {
        Booking saved = bookingRepo.save(map(req));
        publisher.publishEvent(new BookingCreatedEvent(saved));
        return saved;
    }
}

// Event Listen করো
@Component
public class BookingEventHandler {

    @EventListener
    public void onBookingCreated(BookingCreatedEvent event) {
        // Email পাঠাও, notification দাও etc.
        emailService.sendConfirmation(event.getBooking());
    }

    @EventListener
    @Async // Async-এ process করো
    public void onBookingCreatedNotifyAnalytics(BookingCreatedEvent event) {
        analyticsService.track(event.getBooking());
    }
}
```

---

## 4. Spring Boot Annotations

---

### `@SpringBootApplication`

**কী করে:**  
তিনটি annotation-এর combination:
- `@Configuration` → Configuration source
- `@EnableAutoConfiguration` → Classpath দেখে auto-configure
- `@ComponentScan` → Current package + sub-packages scan করে

**এটাই সবকিছুর শুরু।**

```java
// Minimum setup
@SpringBootApplication
public class ProggaBdApplication {
    public static void main(String[] args) {
        SpringApplication.run(ProggaBdApplication.class, args);
    }
}

// Customized
@SpringBootApplication(
    exclude = {DataSourceAutoConfiguration.class}, // কিছু auto-config বাদ দাও
    scanBasePackages = {"com.proggabd", "com.shared"} // Custom scan path
)
public class App {
    public static void main(String[] args) {
        SpringApplication.run(App.class, args);
    }
}
```

---

### `@ConfigurationProperties`

**কী করে:**  
`application.properties`-এর grouped properties একটা Java class-এ bind করে। অনেক `@Value` এর বদলে একটা clean class।

**কেন ভালো:**
- IDE auto-completion পাওয়া যায়
- Validation annotation দেওয়া যায়
- Refactoring সহজ

```java
@ConfigurationProperties(prefix = "app.camera-rental")
@Component
@Validated
public class CameraRentalProperties {

    @NotBlank
    private String apiKey;

    @NotBlank
    private String baseUrl;

    @Min(1000)
    @Max(30000)
    private int timeoutMs = 5000;

    private boolean sandboxMode = false;

    private List<String> allowedEquipmentTypes = new ArrayList<>();

    // getters + setters required
}
```

```yaml
# application.yml
app:
  camera-rental:
    api-key: "your-api-key-here"
    base-url: "https://api.camerarental.com"
    timeout-ms: 8000
    sandbox-mode: false
    allowed-equipment-types:
      - CAMERA
      - LENS
      - TRIPOD
```

---

### `@ConditionalOnProperty`

**কী করে:**  
Property value-এর উপর ভিত্তি করে Bean conditionally তৈরি হবে। Feature toggle-এর জন্য perfect।

```java
@Configuration
public class FeatureConfig {

    @Bean
    @ConditionalOnProperty(name = "feature.sms.enabled", havingValue = "true")
    public SmsGateway smsGateway() {
        return new BdSmsGateway();
    }

    // Property না থাকলেও create করো (matchIfMissing=true)
    @Bean
    @ConditionalOnProperty(name = "cache.enabled", matchIfMissing = true)
    public CacheManager cacheManager() {
        return new ConcurrentMapCacheManager();
    }
}
```

---

### `@ConditionalOnMissingBean`, `@ConditionalOnClass`, `@ConditionalOnBean`

**কী করে:**  
বিভিন্ন condition-এ Bean তৈরি করা বা না করার জন্য।

```java
@Configuration
public class ConditionalBeans {

    // ওই type-এর Bean না থাকলে তৈরি করো
    @Bean
    @ConditionalOnMissingBean(DataSource.class)
    public DataSource fallbackDataSource() {
        return new EmbeddedDatabaseBuilder().build();
    }

    // Classpath-এ class থাকলে তৈরি করো
    @Bean
    @ConditionalOnClass(name = "com.amazonaws.services.s3.AmazonS3")
    public S3StorageService s3StorageService() {
        return new S3StorageService();
    }

    // ওই Bean থাকলে এটাও তৈরি করো
    @Bean
    @ConditionalOnBean(S3StorageService.class)
    public S3BackupService s3BackupService(S3StorageService s3) {
        return new S3BackupService(s3);
    }
}
```

---

### `@SpringBootTest`

**কী করে:**  
Full Spring Application Context load করে integration test চালায়।

```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
class ApplicationIntegrationTest {

    @Autowired
    private TestRestTemplate restTemplate;

    @LocalServerPort
    private int port;

    @Test
    void contextLoads() {
        // Context load হলো মানে সব bean ঠিকঠাক
    }

    @Test
    void apiReturnsCorrectData() {
        ResponseEntity<String> response = restTemplate
            .getForEntity("http://localhost:" + port + "/api/health", String.class);
        assertEquals(HttpStatus.OK, response.getStatusCode());
    }
}
```

---

## 5. Spring MVC / REST Annotations

---

### HTTP Method Annotations

```java
@RestController
@RequestMapping("/api/v1/bookings")
public class BookingController {

    @GetMapping                    // GET /api/v1/bookings
    public List<BookingDto> list() { ... }

    @GetMapping("/{id}")           // GET /api/v1/bookings/42
    public BookingDto get(@PathVariable Long id) { ... }

    @PostMapping                   // POST /api/v1/bookings
    public ResponseEntity<BookingDto> create(@RequestBody @Valid BookingRequest req) { ... }

    @PutMapping("/{id}")           // PUT /api/v1/bookings/42
    public BookingDto update(@PathVariable Long id, @RequestBody @Valid BookingUpdateRequest req) { ... }

    @PatchMapping("/{id}/status")  // PATCH /api/v1/bookings/42/status
    public BookingDto updateStatus(@PathVariable Long id, @RequestBody StatusRequest req) { ... }

    @DeleteMapping("/{id}")        // DELETE /api/v1/bookings/42
    public ResponseEntity<Void> delete(@PathVariable Long id) { ... }
}
```

---

### `@PathVariable`

**কী করে:** URL path-এর `{variable}` part থেকে value নেয়।

```java
// GET /users/42/orders/99
@GetMapping("/users/{userId}/orders/{orderId}")
public Order getOrder(
    @PathVariable Long userId,
    @PathVariable("orderId") Long orderId // name mismatch হলে specify করো
) { ... }
```

---

### `@RequestParam`

**কী করে:** URL query string থেকে value নেয়।

```java
// GET /products?page=0&size=10&category=CAMERA&sort=price
@GetMapping("/products")
public Page<Product> search(
    @RequestParam(defaultValue = "0") int page,
    @RequestParam(defaultValue = "10") int size,
    @RequestParam(required = false) String category,
    @RequestParam(required = false) String sort
) { ... }
```

---

### `@RequestBody`

**কী করে:** HTTP request body (JSON/XML) কে Java object-এ deserialize করে।

```java
@PostMapping("/users")
public ResponseEntity<UserDto> createUser(
    @RequestBody @Valid UserCreateRequest request
) {
    // request-এর JSON → UserCreateRequest object
    UserDto created = userService.create(request);
    return ResponseEntity
        .status(HttpStatus.CREATED)
        .body(created);
}
```

---

### `@RequestHeader`

**কী করে:** HTTP header থেকে value নেয়।

```java
@GetMapping("/secure-data")
public DataResponse getData(
    @RequestHeader("Authorization") String authToken,
    @RequestHeader(value = "X-Request-ID", required = false) String requestId,
    @RequestHeader(value = "Accept-Language", defaultValue = "en") String lang
) { ... }
```

---

### `@CookieValue`

**কী করে:** HTTP cookie থেকে value নেয়।

```java
@GetMapping("/profile")
public ProfileDto getProfile(
    @CookieValue("sessionId") String sessionId,
    @CookieValue(value = "preferredLang", defaultValue = "en") String lang
) { ... }
```

---

### `@ResponseStatus`

**কী করে:** HTTP response status code set করে।

```java
@ResponseStatus(HttpStatus.CREATED) // 201
@PostMapping
public Product create(@RequestBody ProductRequest req) { ... }

// Exception class-এ
@ResponseStatus(HttpStatus.NOT_FOUND) // 404 হবে throw করলে
public class ProductNotFoundException extends RuntimeException {
    public ProductNotFoundException(Long id) {
        super("Product not found: " + id);
    }
}
```

---

### `@ExceptionHandler`

**কী করে:** Controller বা Global level-এ exception handle করে meaningful HTTP response return করে।

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    // 404
    @ExceptionHandler(ResourceNotFoundException.class)
    @ResponseStatus(HttpStatus.NOT_FOUND)
    public ErrorResponse handleNotFound(ResourceNotFoundException ex,
                                         HttpServletRequest request) {
        return new ErrorResponse("NOT_FOUND", ex.getMessage(),
                                 request.getRequestURI());
    }

    // 400 — Validation error
    @ExceptionHandler(MethodArgumentNotValidException.class)
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    public ValidationErrorResponse handleValidation(
            MethodArgumentNotValidException ex) {
        Map<String, String> errors = new LinkedHashMap<>();
        ex.getBindingResult().getFieldErrors()
            .forEach(e -> errors.put(e.getField(), e.getDefaultMessage()));
        return new ValidationErrorResponse(errors);
    }

    // 409 — Duplicate
    @ExceptionHandler(DuplicateEmailException.class)
    @ResponseStatus(HttpStatus.CONFLICT)
    public ErrorResponse handleDuplicate(DuplicateEmailException ex) {
        return new ErrorResponse("DUPLICATE_EMAIL", ex.getMessage(), null);
    }

    // 500 — Fallback
    @ExceptionHandler(Exception.class)
    @ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)
    public ErrorResponse handleAll(Exception ex) {
        log.error("Unexpected error", ex);
        return new ErrorResponse("INTERNAL_ERROR", "Something went wrong", null);
    }
}
```

---

### `@CrossOrigin`

**কী করে:** CORS (Cross-Origin Resource Sharing) enable করে — different domain থেকে request allow করে।

```java
@CrossOrigin(
    origins = {"http://localhost:3000", "https://proggabd.com"},
    methods = {RequestMethod.GET, RequestMethod.POST},
    allowedHeaders = "*",
    maxAge = 3600
)
@RestController
public class ProductController { ... }
```

---

## 6. Spring Security Annotations

---

### `@EnableWebSecurity` / `@EnableMethodSecurity`

```java
@Configuration
@EnableWebSecurity
@EnableMethodSecurity(prePostEnabled = true, securedEnabled = true)
public class SecurityConfig {
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        return http
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/api/public/**").permitAll()
                .anyRequest().authenticated())
            .build();
    }
}
```

---

### `@PreAuthorize`

**কী করে:** Method execute হওয়ার **আগে** SpEL expression দিয়ে permission check করে।

```java
@Service
public class DocumentService {

    // Simple role check
    @PreAuthorize("hasRole('ADMIN')")
    public void deleteAll() { ... }

    // Multiple roles
    @PreAuthorize("hasAnyRole('ADMIN', 'MANAGER')")
    public List<Document> getAll() { ... }

    // Method parameter দিয়ে ownership check
    @PreAuthorize("#userId == authentication.principal.id or hasRole('ADMIN')")
    public Document getUserDocument(Long userId, Long docId) { ... }

    // Permission-based (granular)
    @PreAuthorize("hasAuthority('DOCUMENT:WRITE')")
    public Document save(Document doc) { ... }

    // Custom SpEL with bean reference
    @PreAuthorize("@permissionService.canEdit(authentication, #docId)")
    public Document edit(Long docId, DocumentEditRequest req) { ... }
}
```

---

### `@PostAuthorize`

**কী করে:** Method execute হওয়ার **পরে** return value check করে।

```java
// Return করা object-এর owner-ই শুধু দেখতে পাবে
@PostAuthorize("returnObject.ownerId == authentication.principal.id")
public Report getReport(Long reportId) { ... }
```

---

### `@Secured`

**কী করে:** Role-based access control — `@PreAuthorize`-এর simple version, SpEL support নেই।

```java
@Secured("ROLE_ADMIN")
public void adminOnlyTask() { ... }

@Secured({"ROLE_USER", "ROLE_ADMIN"})
public List<Order> getOrders() { ... }
```

---

## 7. JPA / Jakarta Persistence Annotations

---

### `@Entity` এবং `@Table`

**কী করে:** Class-টি database table-এর সাথে map হবে।

```java
@Entity
@Table(
    name = "equipment_bookings",
    schema = "proggabd",
    indexes = {
        @Index(name = "idx_booking_customer", columnList = "customer_id"),
        @Index(name = "idx_booking_status", columnList = "status,created_at")
    },
    uniqueConstraints = {
        @UniqueConstraint(name = "uq_booking_ref", columnNames = {"booking_reference"})
    }
)
public class EquipmentBooking {
    // ...
}
```

---

### `@Id` এবং `@GeneratedValue`

**Primary Key generation strategies:**

```java
@Entity
public class User {

    // Auto increment (MySQL, PostgreSQL)
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
}

@Entity
public class Product {

    // Database sequence
    @Id
    @GeneratedValue(strategy = GenerationType.SEQUENCE,
                    generator = "product_seq")
    @SequenceGenerator(name = "product_seq",
                       sequenceName = "product_id_seq",
                       allocationSize = 50)
    private Long id;
}

@Entity
public class AuditLog {

    // UUID primary key
    @Id
    @GeneratedValue(strategy = GenerationType.UUID)
    private UUID id;
}
```

---

### `@Column`

**কী করে:** Column-এর name, constraint, size নির্ধারণ করে।

```java
@Entity
public class UserProfile {

    @Column(name = "full_name",
            nullable = false,
            length = 100)
    private String fullName;

    @Column(name = "email",
            unique = true,
            nullable = false,
            length = 255)
    private String email;

    @Column(name = "bio",
            columnDefinition = "TEXT") // Large text
    private String bio;

    @Column(name = "is_active",
            nullable = false)
    private Boolean active = true;

    @Column(name = "credit_balance",
            precision = 10,
            scale = 2)
    private BigDecimal creditBalance;
}
```

---

### `@OneToMany`, `@ManyToOne`, `@OneToOne`, `@ManyToMany`

```java
@Entity
public class Customer {

    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    // একজন Customer → অনেক Booking
    @OneToMany(
        mappedBy = "customer",        // Booking entity-র field name
        cascade = CascadeType.ALL,    // Customer delete হলে booking-ও delete
        fetch = FetchType.LAZY,       // Default: প্রয়োজনে load করো
        orphanRemoval = true          // Collection থেকে remove হলে delete
    )
    private List<Booking> bookings = new ArrayList<>();
}

@Entity
public class Booking {

    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    // অনেক Booking → একজন Customer
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "customer_id", nullable = false)
    private Customer customer;

    // অনেক Booking → অনেক Equipment (ManyToMany)
    @ManyToMany(fetch = FetchType.LAZY)
    @JoinTable(
        name = "booking_equipment",
        joinColumns = @JoinColumn(name = "booking_id"),
        inverseJoinColumns = @JoinColumn(name = "equipment_id")
    )
    private Set<Equipment> equipments = new HashSet<>();
}
```

---

### `@Embedded` এবং `@Embeddable`

**কী করে:** আলাদা table না বানিয়ে object-কে same table-এ embed করে।

```java
@Embeddable
public class Address {
    @Column(name = "street", length = 200)
    private String street;

    @Column(name = "city", length = 100)
    private String city;

    @Column(name = "postal_code", length = 10)
    private String postalCode;

    @Column(name = "country", length = 2)
    private String countryCode = "BD";
}

@Entity
public class Customer {
    @Id @GeneratedValue
    private Long id;

    private String fullName;

    @Embedded
    private Address address; // customer table-এই street, city, postal_code columns থাকবে

    @Embedded
    @AttributeOverrides({ // Same class-টা ২ বার use করলে column name override করো
        @AttributeOverride(name = "street", column = @Column(name = "billing_street")),
        @AttributeOverride(name = "city", column = @Column(name = "billing_city"))
    })
    private Address billingAddress;
}
```

---

### `@Enumerated`

**কী করে:** Enum value DB-তে কীভাবে store হবে।

```java
public enum BookingStatus {
    PENDING, CONFIRMED, CANCELLED, COMPLETED
}

@Entity
public class Booking {

    @Enumerated(EnumType.STRING) // ✅ "PENDING", "CONFIRMED" — readable
    private BookingStatus status;

    // @Enumerated(EnumType.ORDINAL) // ❌ 0, 1, 2 — enum order change হলে data corrupt!
}
```

---

### `@Transient`

**কী করে:** Field-টি database-এ save হবে না।

```java
@Entity
public class User {
    @Id private Long id;
    private String firstName;
    private String lastName;

    @Transient // DB column নেই — runtime-এ compute হয়
    private String fullName;

    @PostLoad
    private void computeFullName() {
        this.fullName = firstName + " " + lastName;
    }
}
```

---

### `@Lob`

**কী করে:** Large Object — TEXT বা BLOB store করে।

```java
@Entity
public class Equipment {

    @Lob
    @Column(columnDefinition = "TEXT")
    private String detailedDescription; // বড় text

    @Lob
    private byte[] manualPdf; // Binary file
}
```

---

### `@Version`

**কী করে:** Optimistic Locking — concurrent update ধরে। দুইজন একসাথে same record update করার চেষ্টা করলে দ্বিতীয়জন `OptimisticLockException` পাবে।

```java
@Entity
public class InventoryItem {

    @Id @GeneratedValue
    private Long id;

    private String equipmentName;
    private int availableQuantity;

    @Version // JPA automatically manages করে
    private Long version;
}

// Service-এ handle করো
try {
    inventoryRepo.save(item);
} catch (OptimisticLockException e) {
    // কেউ already update করে ফেলেছে — refresh করে retry করো
    InventoryItem fresh = inventoryRepo.findById(item.getId()).orElseThrow();
    // ... retry logic
}
```

---

### Spring Data Repository Annotations

```java
@Repository
public interface BookingRepository extends JpaRepository<Booking, Long> {

    // Method name থেকে automatic query
    List<Booking> findByCustomerIdAndStatus(Long customerId, BookingStatus status);
    Optional<Booking> findByBookingReference(String ref);
    long countByStatus(BookingStatus status);

    // Custom JPQL
    @Query("SELECT b FROM Booking b WHERE b.customer.id = :customerId " +
           "AND b.startDate >= :from AND b.endDate <= :to")
    List<Booking> findByDateRange(
        @Param("customerId") Long customerId,
        @Param("from") LocalDate from,
        @Param("to") LocalDate to
    );

    // Native SQL
    @Query(value = "SELECT * FROM bookings WHERE DATEDIFF(end_date, NOW()) <= 7",
           nativeQuery = true)
    List<Booking> findEndingSoon();

    // Update query
    @Modifying
    @Transactional
    @Query("UPDATE Booking b SET b.status = :status WHERE b.id = :id")
    int updateStatus(@Param("id") Long id, @Param("status") BookingStatus status);

    // Projection
    @Query("SELECT b.id as id, b.bookingReference as reference FROM Booking b")
    List<BookingSummary> findAllSummaries();
}
```

---

### `@EntityGraph`

**কী করে:** N+1 problem solve করে — related entity গুলো একটা query-তে join করে load করে।

```java
@Repository
public interface BookingRepository extends JpaRepository<Booking, Long> {

    @EntityGraph(attributePaths = {"customer", "equipments", "customer.address"})
    @Query("SELECT b FROM Booking b WHERE b.id = :id")
    Optional<Booking> findByIdWithDetails(@Param("id") Long id);
}
```

---

## 8. Hibernate-Specific Annotations

---

### `@DynamicInsert` এবং `@DynamicUpdate`

**`@DynamicInsert`:** INSERT statement-এ শুধু non-null column গুলো include করে → DB DEFAULT values কাজ করে।

**`@DynamicUpdate`:** UPDATE statement-এ শুধু **পরিবর্তিত** column গুলো include করে → পুরো entity update হয় না।

```java
@Entity
@DynamicInsert
@DynamicUpdate
public class UserProfile {

    @Id @GeneratedValue
    private Long id;

    private String bio;
    private String avatarUrl;
    private String websiteUrl;
    // ... আরো ২০টা field

    // DynamicUpdate: শুধু bio change করলে
    // UPDATE user_profile SET bio=? WHERE id=?
    // বাকি ১৯টা field update হয় না — efficient!
}
```

---

### `@SQLDelete` এবং `@Where` (Soft Delete)

**কী করে:**
- `@SQLDelete` → JPA delete কে custom SQL দিয়ে replace করে (soft delete implement করতে)
- `@Where` → সব query-তে automatic WHERE clause add করে (soft deleted record filter করতে)

```java
@Entity
@Table(name = "products")
@SQLDelete(sql = "UPDATE products SET deleted_at = NOW(), is_deleted = TRUE WHERE id = ?")
@Where(clause = "is_deleted = FALSE")
public class Product {

    @Id @GeneratedValue
    private Long id;

    private String name;
    private BigDecimal price;

    @Column(name = "is_deleted")
    private Boolean deleted = false;

    @Column(name = "deleted_at")
    private LocalDateTime deletedAt;
}

// productRepo.delete(product)
// → UPDATE products SET deleted_at=NOW(), is_deleted=TRUE WHERE id=?

// productRepo.findAll()
// → SELECT * FROM products WHERE is_deleted = FALSE (auto!)

// productRepo.findById(id)
// → SELECT * FROM products WHERE id=? AND is_deleted = FALSE (auto!)
```

---

### `@NaturalId`

**কী করে:**  
Entity-র natural business identifier mark করে (email, username, SKU) — surrogate @Id এর বিপরীতে। Hibernate first-level cache দিয়ে optimize করে।

```java
@Entity
public class User {

    @Id @GeneratedValue
    private Long id; // Surrogate key (internal)

    @NaturalId
    @Column(nullable = false, unique = true)
    private String username; // Natural business key (external)

    @NaturalId
    @Column(nullable = false, unique = true)
    private String email;
}

// Hibernate-optimized lookup (L1 cache use করে):
User user = session.byNaturalId(User.class)
    .using("username", "polas-cse")
    .load();
```

---

### `@CreationTimestamp` এবং `@UpdateTimestamp` (Hibernate)

**Hibernate-specific** (Spring Data-র @CreatedDate-এর alternative)।

```java
@Entity
public class AuditableEntity {

    @CreationTimestamp
    @Column(updatable = false)
    private LocalDateTime createdAt;

    @UpdateTimestamp
    private LocalDateTime updatedAt;
}
```

---

## 9. Spring Data Auditing Annotations

> এই annotation গুলো মিলিয়ে automatic audit trail তৈরি হয়। কে কখন তৈরি বা পরিবর্তন করলো — automatically track।

---

### Setup

```java
// 1. Enable করো
@Configuration
@EnableJpaAuditing(auditorAwareRef = "auditorProvider")
public class JpaAuditConfig {}

// 2. Current user বলে দাও
@Component("auditorProvider")
public class SpringSecurityAuditorAware implements AuditorAware<String> {

    @Override
    public Optional<String> getCurrentAuditor() {
        return Optional.ofNullable(
            SecurityContextHolder.getContext().getAuthentication()
        )
        .filter(Authentication::isAuthenticated)
        .filter(auth -> !"anonymousUser".equals(auth.getPrincipal()))
        .map(Authentication::getName)
        .or(() -> Optional.of("SYSTEM")); // Batch job বা startup-এ
    }
}
```

---

### `@MappedSuperclass` + Audit Fields

```java
// 3. Base entity বানাও
@MappedSuperclass
@EntityListeners(AuditingEntityListener.class) // ← এটা MUST দিতে হবে!
public abstract class BaseAuditableEntity {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @CreatedDate
    @Column(name = "created_at", updatable = false, nullable = false)
    private LocalDateTime createdAt; // insert-এ set হয়, কখনো update হয় না

    @LastModifiedDate
    @Column(name = "updated_at", nullable = false)
    private LocalDateTime updatedAt; // প্রতিটা save-এ update হয়

    @CreatedBy
    @Column(name = "created_by", updatable = false, length = 100)
    private String createdBy; // "polas@example.com"

    @LastModifiedBy
    @Column(name = "last_modified_by", length = 100)
    private String lastModifiedBy; // "admin@example.com"

    // getters...
}

// 4. Entity গুলো extend করো
@Entity
@Table(name = "bookings")
public class Booking extends BaseAuditableEntity {
    // id, createdAt, updatedAt, createdBy, lastModifiedBy — সব auto!
    private String bookingReference;
    private LocalDate startDate;
    // ...
}
```

---

### `@EntityListeners`

**কী করে:**  
Entity-র JPA lifecycle event-এ callback method trigger করে। `AuditingEntityListener.class` ছাড়া @CreatedDate, @CreatedBy কাজ করে না।

**#1 সবচেয়ে common ভুল:** `@EntityListeners` দিতে ভুলে যাওয়া।

```java
@Entity
@EntityListeners({
    AuditingEntityListener.class,   // Spring Data Auditing
    DomainEventPublishingAuditor.class // Custom listener
})
public class Order { ... }

// Custom Listener
public class DomainEventPublishingAuditor {

    @PrePersist
    public void onSave(Object entity) {
        System.out.println("About to save: " + entity.getClass().getSimpleName());
    }

    @PostPersist
    public void afterSave(Object entity) {
        System.out.println("Saved: " + entity);
    }

    @PreUpdate
    public void onUpdate(Object entity) { ... }

    @PostUpdate
    public void afterUpdate(Object entity) { ... }

    @PreRemove
    public void onDelete(Object entity) { ... }

    @PostRemove
    public void afterDelete(Object entity) { ... }

    @PostLoad
    public void onLoad(Object entity) { ... }
}
```

---

### `@Audited` (Hibernate Envers)

**কী করে:**  
Entity-র পরিপূর্ণ revision history রাখে — কখন কী পরিবর্তন হলো। `_AUD` suffix দিয়ে shadow table তৈরি হয়।

```java
// Dependency: hibernate-envers

@Entity
@Audited // products_AUD table তৈরি হবে full history সহ
public class Product {

    @Id @GeneratedValue
    private Long id;

    private String name;
    private BigDecimal price;

    @NotAudited // এই field audit হবে না
    private String internalNote;
}

// History query
AuditReader reader = AuditReaderFactory.get(entityManager);

// কোন কোন revision আছে
List<Number> revisions = reader.getRevisions(Product.class, productId);

// নির্দিষ্ট revision-এর snapshot
Product snapshot = reader.find(Product.class, productId, revisions.get(0));

// Revision details
RevisionType type = reader.findRevision(DefaultRevisionEntity.class, revisions.get(0))
    .getTimestamp(); // ADD, MOD, DEL
```

---

## 10. AOP Annotations

> Cross-cutting concern (logging, security, metrics) গুলো business logic থেকে আলাদা করতে।

---

### `@Aspect`, `@Pointcut`, `@Before`, `@After`, `@Around`, `@AfterReturning`, `@AfterThrowing`

```java
@Aspect
@Component
@Slf4j
public class AuditLoggingAspect {

    // Pointcut reuse করার জন্য define করো
    @Pointcut("@annotation(com.example.AuditLog)")
    public void auditLogMethods() {}

    @Pointcut("within(@org.springframework.stereotype.Service *)")
    public void serviceMethods() {}

    // Method-এর আগে
    @Before("serviceMethods()")
    public void logBefore(JoinPoint jp) {
        log.debug("→ Calling: {}.{}",
            jp.getTarget().getClass().getSimpleName(),
            jp.getSignature().getName());
    }

    // Method-এর পরে (success এবং failure উভয়তে)
    @After("serviceMethods()")
    public void logAfter(JoinPoint jp) {
        log.debug("← Done: {}", jp.getSignature().getName());
    }

    // শুধু success-এ
    @AfterReturning(pointcut = "serviceMethods()", returning = "result")
    public void logSuccess(JoinPoint jp, Object result) {
        log.debug("✓ {} returned: {}", jp.getSignature().getName(), result);
    }

    // শুধু exception-এ
    @AfterThrowing(pointcut = "serviceMethods()", throwing = "ex")
    public void logException(JoinPoint jp, Exception ex) {
        log.error("✗ {} threw: {}", jp.getSignature().getName(), ex.getMessage());
    }

    // সবচেয়ে powerful — সব control তোমার হাতে
    @Around("auditLogMethods() && @annotation(auditLog)")
    public Object auditAround(ProceedingJoinPoint pjp, AuditLog auditLog) throws Throwable {
        String user = SecurityContextHolder.getContext()
            .getAuthentication().getName();
        long start = System.currentTimeMillis();

        Object result;
        try {
            result = pjp.proceed(); // Actual method call করো
        } catch (Exception e) {
            auditService.logFailure(auditLog.action(), user, e.getMessage());
            throw e; // Exception re-throw করো
        }

        long elapsed = System.currentTimeMillis() - start;
        auditService.logSuccess(auditLog.action(), user, elapsed);
        return result;
    }
}

// Custom Annotation
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface AuditLog {
    String action();
}

// Usage
@Service
public class ProductService {

    @AuditLog(action = "CREATE_PRODUCT")
    public Product create(ProductRequest req) { ... }

    @AuditLog(action = "DELETE_PRODUCT")
    public void delete(Long id) { ... }
}
```

---

## 11. Bean Validation Annotations

> Dependency: `spring-boot-starter-validation`  
> Trigger: `@Valid` বা `@Validated` দিয়ে

---

### সব Validation Annotation

```java
public class UserRegistrationRequest {

    // Null check
    @NotNull(message = "Name cannot be null")
    @NotBlank(message = "Name cannot be empty or whitespace") // ← String-এর জন্য সেরা
    @Size(min = 2, max = 50, message = "Name must be 2-50 characters")
    private String fullName;

    // Email format
    @Email(message = "Please enter a valid email address")
    @NotEmpty
    private String email;

    // Password
    @NotBlank
    @Size(min = 8, max = 128, message = "Password must be 8-128 characters")
    @Pattern(
        regexp = "^(?=.*[a-z])(?=.*[A-Z])(?=.*\\d).+$",
        message = "Password must contain uppercase, lowercase, and digit"
    )
    private String password;

    // BD Phone
    @Pattern(
        regexp = "^\\+880[0-9]{10}$",
        message = "Phone must be valid BD number: +880XXXXXXXXXX"
    )
    private String phone;

    // Number range
    @Min(value = 18, message = "Must be at least 18 years old")
    @Max(value = 120)
    private Integer age;

    // Positive/Negative
    @Positive(message = "Price must be positive")
    private BigDecimal price;

    @PositiveOrZero
    private Integer stock;

    // Decimal range
    @DecimalMin("0.00")
    @DecimalMax("999999.99")
    @Digits(integer = 6, fraction = 2)
    private BigDecimal creditLimit;

    // Date validation
    @Past(message = "Birth date must be in the past")
    private LocalDate birthDate;

    @Future(message = "Appointment must be in the future")
    private LocalDate appointmentDate;

    @FutureOrPresent
    private LocalDateTime scheduledAt;

    @PastOrPresent
    private LocalDateTime lastLoginAt;

    // Boolean
    @AssertTrue(message = "You must accept the terms and conditions")
    private Boolean termsAccepted;

    @AssertFalse(message = "Account cannot be banned")
    private Boolean banned;

    // Nested object — cascade validation
    @NotNull
    @Valid
    private AddressRequest address;

    // Collection items validate
    @NotEmpty(message = "At least one role required")
    private List<@NotBlank String> roles;
}
```

---

### Custom Constraint তৈরি

```java
// 1. Annotation define করো
@Target({ElementType.FIELD, ElementType.PARAMETER})
@Retention(RetentionPolicy.RUNTIME)
@Constraint(validatedBy = BdNidValidator.class)
public @interface ValidBdNid {
    String message() default "Invalid Bangladesh NID number";
    Class<?>[] groups() default {};
    Class<? extends Payload>[] payload() default {};
}

// 2. Validator implement করো
public class BdNidValidator implements ConstraintValidator<ValidBdNid, String> {

    @Override
    public boolean isValid(String nid, ConstraintValidatorContext ctx) {
        if (nid == null) return true; // @NotNull আলাদা handle করবে
        return nid.matches("^\\d{10}$") || nid.matches("^\\d{13}$")
               || nid.matches("^\\d{17}$");
    }
}

// 3. Use করো
public class KycRequest {
    @NotBlank
    @ValidBdNid
    private String nidNumber;
}
```

---

## 12. Lombok Annotations

> Compile-time code generation — zero runtime overhead।  
> Dependency: `lombok` (annotationProcessor হিসেবে add করো)

---

| Annotation | কী generate করে |
|---|---|
| `@Getter` | সব field-এর getter |
| `@Setter` | সব non-final field-এর setter |
| `@ToString` | `toString()` |
| `@EqualsAndHashCode` | `equals()` + `hashCode()` |
| `@NoArgsConstructor` | No-arg constructor |
| `@AllArgsConstructor` | All-arg constructor |
| `@RequiredArgsConstructor` | `final` এবং `@NonNull` field-এর constructor |
| `@Data` | @Getter + @Setter + @ToString + @EqualsAndHashCode + @RequiredArgsConstructor |
| `@Value` | Immutable class (all final, no setter) |
| `@Builder` | Builder pattern |
| `@SuperBuilder` | Inheritance-সহ Builder |
| `@Slf4j` | SLF4J Logger inject |
| `@Log4j2` | Log4j2 Logger inject |
| `@NonNull` | Null check করে NPE throw |
| `@Cleanup` | AutoCloseable auto-close |
| `@SneakyThrows` | Checked exception unchecked হিসেবে throw |
| `@Synchronized` | Thread-safe method |
| `@With` | Immutable object-এর "copy with one field changed" |
| `@Delegate` | Delegation pattern |

```java
@Data           // Getter + Setter + ToString + EqualsHashCode + RequiredArgsConstructor
@Builder        // Builder pattern
@Slf4j          // log variable inject
public class BookingRequest {

    @NonNull    // Null হলে NullPointerException automatically
    private final String customerName;

    @NonNull
    private final String equipmentId;

    @Builder.Default    // Builder use করলেও default value রাখো
    private LocalDate startDate = LocalDate.now();

    private LocalDate endDate;

    @Builder.Default
    private BookingStatus status = BookingStatus.PENDING;

    @Singular   // .tag("CAMERA").tag("LENS") এভাবে add করা যাবে
    private List<String> tags;

    @ToString.Exclude   // Log-এ show হবে না
    @EqualsAndHashCode.Exclude
    private String nidImageBase64; // Large field, log-এ দেখানো ঠিক না

    public void validate() {
        log.info("Validating booking for customer: {}", customerName);
        // log variable already inject হয়ে আছে @Slf4j-এর কারণে
    }
}

// Usage
BookingRequest req = BookingRequest.builder()
    .customerName("Polas Hossain")
    .equipmentId("CAM-001")
    .endDate(LocalDate.now().plusDays(3))
    .tag("CAMERA")
    .tag("LENS")
    .build();

System.out.println(req); // toString() auto-generated (nidImageBase64 excluded)
```

---

### JPA Entity-তে Lombok

```java
@Entity
@Table(name = "users")
@Getter           // Getter দাও
@Setter           // Setter দাও
@NoArgsConstructor // JPA-র জন্য required
@AllArgsConstructor
// @Data ❌ — JPA Entity-তে avoid করো। equals/hashCode সব field দিয়ে হলে Hibernate-এ সমস্যা
@ToString(exclude = {"password", "bookings"}) // Lazy-loaded collection exclude করো
public class User {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false, length = 100)
    private String fullName;

    @Column(unique = true, nullable = false)
    private String email;

    @OneToMany(mappedBy = "user", fetch = FetchType.LAZY)
    @ToString.Exclude // Circular reference avoid
    private List<Booking> bookings;
}
```

---

## 13. Jackson JSON Annotations

> JSON serialize (Java → JSON) এবং deserialize (JSON → Java) control করার জন্য।

---

### `@JsonProperty`

```java
public class UserDto {

    @JsonProperty("user_id")         // Java: id → JSON: "user_id"
    private Long id;

    @JsonProperty("full_name")
    private String fullName;

    // Write-only: JSON input-এ accept, কিন্তু response-এ দেয় না
    @JsonProperty(value = "password", access = JsonProperty.Access.WRITE_ONLY)
    private String password;

    // Read-only: response-এ আসে, input-এ ignore হয়
    @JsonProperty(access = JsonProperty.Access.READ_ONLY)
    private LocalDateTime createdAt;
}
```

---

### `@JsonIgnore` এবং `@JsonIgnoreProperties`

```java
// Class level
@JsonIgnoreProperties(ignoreUnknown = true) // Extra JSON field-এ error দেবে না
@JsonIgnoreProperties({"internalNote", "tempData"}) // Specific field ignore
public class ProductDto { ... }

// Field level
public class UserEntity {

    @JsonIgnore // কখনো JSON-এ আসবে না এবং JSON input থেকেও ignore হবে
    private String passwordHash;

    // শুধু output ignore করতে হলে:
    @JsonProperty(access = JsonProperty.Access.WRITE_ONLY)
    private String pin; // Input accept, output ignore
}
```

---

### `@JsonFormat`

```java
public class EventDto {

    @JsonFormat(pattern = "dd-MM-yyyy HH:mm:ss", timezone = "Asia/Dhaka")
    private LocalDateTime eventTime;

    @JsonFormat(pattern = "yyyy-MM-dd")
    private LocalDate eventDate;

    @JsonFormat(shape = JsonFormat.Shape.STRING) // Enum → String
    private EventType type; // "WORKSHOP" not 0

    @JsonFormat(shape = JsonFormat.Shape.STRING) // Long → String (JS precision loss avoid)
    private Long bigId;
}
```

---

### `@JsonAlias`

```java
public class UserDto {
    // JSON-এ "username", "user_name", বা "uname" — যেকোনোটা accept করবে
    @JsonAlias({"user_name", "uname", "login"})
    private String username;
}
```

---

### `@JsonInclude`

```java
@JsonInclude(JsonInclude.Include.NON_NULL)  // null field JSON-এ আসবে না
public class ApiResponse<T> {
    private T data;
    private String message;
    private String error; // null হলে response-এ থাকবে না
}

// Field level
public class UserDto {
    @JsonInclude(JsonInclude.Include.NON_EMPTY) // Empty list/string হলে include না
    private List<String> tags;
}
```

---

### `@JsonManagedReference` এবং `@JsonBackReference`

```java
// Bidirectional relationship-এ infinite loop prevent করে
public class Category {
    private Long id;
    private String name;

    @JsonManagedReference // এই side serialize হবে
    private List<Product> products;
}

public class Product {
    private Long id;
    private String name;

    @JsonBackReference // এই side serialize হবে না (loop prevent)
    private Category category;
}
```

---

### `@JsonTypeInfo` এবং `@JsonSubTypes`

```java
// Polymorphism — JSON থেকে correct subclass identify করা
@JsonTypeInfo(use = JsonTypeInfo.Id.NAME, property = "type")
@JsonSubTypes({
    @JsonSubTypes.Type(value = CameraEquipment.class, name = "CAMERA"),
    @JsonSubTypes.Type(value = LensEquipment.class, name = "LENS"),
    @JsonSubTypes.Type(value = TripodEquipment.class, name = "TRIPOD")
})
public abstract class Equipment {
    private Long id;
    private String name;
}

// { "type": "CAMERA", "id": 1, "sensorSize": "Full Frame" }
// → CameraEquipment object

// { "type": "LENS", "id": 2, "focalLength": "50mm" }
// → LensEquipment object
```

---

### `@JsonSerialize` এবং `@JsonDeserialize`

```java
public class Transaction {

    @JsonSerialize(using = MoneySerializer.class)
    @JsonDeserialize(using = MoneyDeserializer.class)
    private BigDecimal amount;
}

// Custom Serializer
public class MoneySerializer extends JsonSerializer<BigDecimal> {
    @Override
    public void serialize(BigDecimal value, JsonGenerator gen,
                          SerializerProvider provider) throws IOException {
        gen.writeString(value.setScale(2).toString() + " BDT");
    }
}
```

---

## 14. JUnit 5 Annotations

---

```java
@ExtendWith(MockitoExtension.class) // Mockito integration
class BookingServiceTest {

    @Mock
    private BookingRepository bookingRepo;

    @InjectMocks
    private BookingService bookingService;

    @BeforeAll
    static void initAll() {
        // সব test-এর আগে একবার — static method
        System.out.println("Test class starting...");
    }

    @AfterAll
    static void cleanupAll() {
        // সব test-এর পরে একবার
    }

    @BeforeEach
    void setUp() {
        // প্রতিটা test-এর আগে
        MockitoAnnotations.openMocks(this);
    }

    @AfterEach
    void tearDown() {
        // প্রতিটা test-এর পরে
    }

    @Test
    @DisplayName("✅ Valid booking request should be saved successfully")
    void createBooking_withValidRequest_shouldSave() {
        // Arrange
        BookingRequest req = new BookingRequest("Polas", "CAM-001",
            LocalDate.now(), LocalDate.now().plusDays(3));
        when(bookingRepo.save(any())).thenReturn(new Booking(1L, req));

        // Act
        Booking result = bookingService.create(req);

        // Assert
        assertNotNull(result.getId());
        verify(bookingRepo, times(1)).save(any());
    }

    @Test
    @DisplayName("❌ Null customer name should throw exception")
    void createBooking_withNullCustomer_shouldThrow() {
        BookingRequest req = new BookingRequest(null, "CAM-001",
            LocalDate.now(), LocalDate.now().plusDays(3));

        assertThrows(NullPointerException.class,
            () -> bookingService.create(req));
    }

    @Test
    @Disabled("BUG-#456: Date validation broken — fix in next sprint")
    void dateValidation_knownBrokenTest() { ... }

    @Test
    @Timeout(2) // 2 সেকেন্ডের বেশি নিলে fail
    void performanceTest_shouldCompleteWithin2Seconds() {
        bookingService.generateReport(); // Must be fast!
    }

    @RepeatedTest(5) // ৫ বার run করো
    @DisplayName("Concurrent booking test (Rep {currentRepetition}/{totalRepetitions})")
    void concurrentBookingTest() { ... }

    @ParameterizedTest
    @ValueSource(strings = {"CAM-001", "LENS-002", "TRIPOD-003"})
    @DisplayName("Should find equipment by ID: {0}")
    void findEquipment_shouldReturnCorrectItem(String equipmentId) {
        Optional<Equipment> eq = equipmentService.findById(equipmentId);
        assertTrue(eq.isPresent());
    }

    @ParameterizedTest
    @CsvSource({
        "CAM-001, CAMERA, 45000",
        "LENS-002, LENS, 35000",
        "TRIPOD-003, TRIPOD, 2500"
    })
    void equipmentPricing_shouldMatchExpected(
            String id, String type, int expectedPrice) {
        Equipment eq = equipmentService.findById(id).orElseThrow();
        assertEquals(expectedPrice, eq.getPrice().intValue());
    }

    @ParameterizedTest
    @EnumSource(BookingStatus.class) // সব enum value test করো
    void allStatuses_shouldHaveDisplayLabel(BookingStatus status) {
        assertNotNull(status.getDisplayLabel());
    }

    @ParameterizedTest
    @MethodSource("provideBookingRequests")
    void bulkBookingTest(BookingRequest req, boolean expectedResult) {
        assertEquals(expectedResult, bookingService.isValid(req));
    }

    static Stream<Arguments> provideBookingRequests() {
        return Stream.of(
            Arguments.of(new BookingRequest("Polas", "CAM-001",
                LocalDate.now(), LocalDate.now().plusDays(3)), true),
            Arguments.of(new BookingRequest("", "CAM-001",
                LocalDate.now(), LocalDate.now().plusDays(3)), false)
        );
    }

    @Nested
    @DisplayName("When customer is VIP")
    class VipCustomerTests {

        @BeforeEach
        void setUpVip() { /* VIP customer setup */ }

        @Test
        @DisplayName("Should get 20% discount")
        void vipCustomerGetsDiscount() { ... }

        @Test
        @DisplayName("Should have priority queue")
        void vipCustomerHasPriorityQueue() { ... }
    }

    @Tag("integration") // Group করার জন্য
    @Test
    void integrationTest_withRealDatabase() { ... }
}
```

---

## 15. Mockito Annotations

---

```java
@ExtendWith(MockitoExtension.class)
class OrderServiceTest {

    @Mock                          // Fake — সব method stub হয়
    private OrderRepository orderRepo;

    @Mock
    private EmailService emailService;

    @Mock
    private PaymentGateway paymentGateway;

    @InjectMocks                   // Real object — @Mock গুলো inject হয়
    private OrderService orderService;

    @Spy                           // Real object — কিন্তু specific method stub করা যায়
    private PriceCalculator priceCalculator;

    @Captor                        // Argument capture করে assert করার জন্য
    private ArgumentCaptor<Order> orderCaptor;

    @Captor
    private ArgumentCaptor<String> emailCaptor;

    @Test
    void placeOrder_shouldSaveAndSendEmail() {
        // Arrange
        Order mockOrder = new Order(99L, "ORD-2024-001");
        when(orderRepo.save(any(Order.class))).thenReturn(mockOrder);
        when(paymentGateway.charge(any(), any())).thenReturn(PaymentResult.SUCCESS);
        doNothing().when(emailService).send(anyString(), anyString());

        // Act
        Order result = orderService.placeOrder(new OrderRequest("Polas", 1000.0));

        // Assert
        assertNotNull(result);
        assertEquals(99L, result.getId());

        // Verify interactions
        verify(orderRepo, times(1)).save(orderCaptor.capture());
        verify(emailService, times(1)).send(emailCaptor.capture(), anyString());

        // Captured values assert করো
        assertEquals("Polas", orderCaptor.getValue().getCustomerName());
        assertEquals("polas@example.com", emailCaptor.getValue());

        // Never called
        verify(paymentGateway, never()).refund(any());
    }

    @Test
    void placeOrder_whenPaymentFails_shouldNotSaveOrder() {
        when(paymentGateway.charge(any(), any())).thenReturn(PaymentResult.FAILED);

        assertThrows(PaymentFailedException.class,
            () -> orderService.placeOrder(new OrderRequest("Polas", 1000.0)));

        // Order save হওয়া উচিত না
        verify(orderRepo, never()).save(any());
        // Email যাওয়া উচিত না
        verifyNoInteractions(emailService);
    }

    @Test
    void withSpy_realMethodRunsButCanBeOverridden() {
        // Spy: real method চলে
        double realPrice = priceCalculator.calculate(100.0, 18); // Real: 118.0

        // Spy: specific method stub করো
        doReturn(100.0).when(priceCalculator).calculate(anyDouble(), anyInt());
        double stubbedPrice = priceCalculator.calculate(100.0, 18); // Stubbed: 100.0

        assertEquals(118.0, realPrice);
        assertEquals(100.0, stubbedPrice);
    }
}
```

---

### `@MockBean` এবং `@SpyBean` (Spring Boot Test)

```java
@WebMvcTest(BookingController.class) // শুধু Web layer load করে
class BookingControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @Autowired
    private ObjectMapper objectMapper;

    @MockBean // Spring Context-এ real bean replace করে mock দিয়ে
    private BookingService bookingService;

    @Test
    void createBooking_shouldReturn201() throws Exception {
        BookingRequest req = new BookingRequest("Polas", "CAM-001",
            LocalDate.now(), LocalDate.now().plusDays(3));
        BookingDto expected = new BookingDto(1L, "BK-2024-001", "PENDING");

        when(bookingService.create(any())).thenReturn(expected);

        mockMvc.perform(post("/api/v1/bookings")
                .contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(req)))
            .andExpect(status().isCreated())
            .andExpect(jsonPath("$.bookingReference").value("BK-2024-001"))
            .andExpect(jsonPath("$.status").value("PENDING"))
            .andDo(print());
    }

    @Test
    void getBooking_whenNotFound_shouldReturn404() throws Exception {
        when(bookingService.findById(999L))
            .thenThrow(new ResourceNotFoundException("Booking not found: 999"));

        mockMvc.perform(get("/api/v1/bookings/999"))
            .andExpect(status().isNotFound())
            .andExpect(jsonPath("$.error").value("NOT_FOUND"));
    }
}
```

---

## 16. OpenAPI / Swagger Annotations

> Dependency: `springdoc-openapi-starter-webmvc-ui`  
> UI: `/swagger-ui.html`

---

```java
@Tag(name = "Booking Management",
     description = "APIs for managing equipment booking lifecycle")
@RestController
@RequestMapping("/api/v1/bookings")
public class BookingController {

    @Operation(
        summary = "Create a new booking",
        description = "Creates a new equipment booking request. " +
                      "Customer must pass KYC verification before booking is confirmed."
    )
    @ApiResponses({
        @ApiResponse(responseCode = "201", description = "Booking created",
            content = @Content(schema = @Schema(implementation = BookingDto.class))),
        @ApiResponse(responseCode = "400", description = "Invalid request data"),
        @ApiResponse(responseCode = "409", description = "Equipment not available"),
        @ApiResponse(responseCode = "500", description = "Internal server error")
    })
    @PostMapping
    public ResponseEntity<BookingDto> create(
        @io.swagger.v3.oas.annotations.parameters.RequestBody(
            description = "Booking details",
            required = true
        )
        @RequestBody @Valid BookingRequest req
    ) { ... }

    @Operation(summary = "Get booking by ID")
    @ApiResponse(responseCode = "200", description = "Booking found")
    @ApiResponse(responseCode = "404", description = "Booking not found")
    @GetMapping("/{id}")
    public BookingDto getById(
        @Parameter(description = "Booking ID", example = "42", required = true)
        @PathVariable Long id,

        @Parameter(description = "Include equipment details", example = "true")
        @RequestParam(defaultValue = "false") boolean includeDetails
    ) { ... }
}

// DTO Documentation
@Schema(description = "Booking creation request")
public class BookingRequest {

    @Schema(description = "Customer full name", example = "Polas Hossain", required = true)
    @NotBlank
    private String customerName;

    @Schema(description = "Equipment ID", example = "CAM-001", required = true)
    @NotBlank
    private String equipmentId;

    @Schema(description = "Rental start date", example = "2024-12-01")
    @NotNull
    private LocalDate startDate;

    @Schema(description = "Rental end date", example = "2024-12-07")
    @NotNull
    private LocalDate endDate;
}
```

---

## 17. Custom Annotation তৈরি

নিজের annotation বানানোর সম্পূর্ণ উদাহরণ:

---

### উদাহরণ ১: Rate Limiting

```java
// 1. Annotation define
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface RateLimit {
    int requestsPerMinute() default 60;
    String keyPrefix() default "";
}

// 2. AOP দিয়ে handle
@Aspect
@Component
@Slf4j
public class RateLimitAspect {

    private final Map<String, List<Long>> requestLog = new ConcurrentHashMap<>();

    @Around("@annotation(rateLimit)")
    public Object checkRateLimit(ProceedingJoinPoint pjp,
                                  RateLimit rateLimit) throws Throwable {
        String key = rateLimit.keyPrefix() + "_" +
                     pjp.getSignature().toShortString();
        long now = System.currentTimeMillis();
        long oneMinuteAgo = now - 60_000;

        requestLog.computeIfAbsent(key, k -> new ArrayList<>())
                  .removeIf(t -> t < oneMinuteAgo);

        List<Long> requests = requestLog.get(key);
        if (requests.size() >= rateLimit.requestsPerMinute()) {
            log.warn("Rate limit exceeded for: {}", key);
            throw new TooManyRequestsException("Rate limit exceeded");
        }

        requests.add(now);
        return pjp.proceed();
    }
}

// 3. Use করো
@Service
public class SmsService {

    @RateLimit(requestsPerMinute = 10, keyPrefix = "sms")
    public void sendOtp(String phone) {
        // প্রতি মিনিটে ১০টার বেশি OTP send হবে না
    }
}
```

---

### উদাহরণ ২: Execution Time Logging

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface LogExecutionTime {
    String name() default "";
    boolean logArgs() default false;
    long warnThresholdMs() default 1000;
}

@Aspect
@Component
@Slf4j
public class ExecutionTimeAspect {

    @Around("@annotation(logExecutionTime)")
    public Object logTime(ProceedingJoinPoint pjp,
                           LogExecutionTime logExecutionTime) throws Throwable {
        String name = logExecutionTime.name().isEmpty()
            ? pjp.getSignature().toShortString()
            : logExecutionTime.name();

        if (logExecutionTime.logArgs()) {
            log.debug("→ {} called with: {}", name,
                Arrays.toString(pjp.getArgs()));
        }

        long start = System.currentTimeMillis();
        Object result = pjp.proceed();
        long elapsed = System.currentTimeMillis() - start;

        if (elapsed > logExecutionTime.warnThresholdMs()) {
            log.warn("⚠️ SLOW: {} took {}ms (threshold: {}ms)",
                name, elapsed, logExecutionTime.warnThresholdMs());
        } else {
            log.debug("✓ {} completed in {}ms", name, elapsed);
        }

        return result;
    }
}

// Use
@Service
public class ReportService {

    @LogExecutionTime(name = "PDF Report", warnThresholdMs = 2000, logArgs = true)
    public byte[] generatePdfReport(ReportRequest req) {
        // ২ সেকেন্ডের বেশি লাগলে WARN log
        ...
    }
}
```

---

## 18. Quick Cheat Sheet

```
☕ CORE JAVA
├── @Override              → Parent method override verify করো
├── @Deprecated            → পুরনো, avoid করো (since + forRemoval)
├── @SuppressWarnings      → Specific compiler warning suppress
├── @FunctionalInterface   → Lambda-compatible SAM interface
└── @SafeVarargs           → Generic varargs heap pollution suppress

🏷️ META ANNOTATIONS
├── @Target                → কোথায় apply হবে (ElementType)
├── @Retention             → কতক্ষণ থাকবে (SOURCE/CLASS/RUNTIME)
├── @Documented            → Javadoc-এ দেখাবে
├── @Inherited             → Child class inherit করবে
└── @Repeatable            → একই জায়গায় বারবার use

🌱 SPRING STEREOTYPE
├── @Component             → Generic Spring Bean
├── @Service               → Business Logic layer
├── @Repository            → Data Access layer + exception translation
└── @Controller / @RestController → Web Layer

💉 DEPENDENCY INJECTION
├── @Autowired             → Dependency inject (constructor preferred)
├── @Qualifier             → কোন Bean inject হবে specify করো
├── @Primary               → Default Bean
├── @Bean                  → Manual Bean creation (@Configuration-এ)
├── @Value                 → Property inject (${key:default})
└── @Scope                 → Bean lifecycle (singleton/prototype/request/session)

⚙️ SPRING BOOT
├── @SpringBootApplication → @Config + @EnableAutoConfig + @ComponentScan
├── @ConfigurationProperties → Grouped property bind করো
├── @ConditionalOnProperty → Property-based conditional Bean
├── @ConditionalOnMissingBean → Bean absent হলে create করো
└── @SpringBootTest        → Full context integration test

🌐 SPRING MVC / REST
├── @GetMapping            → HTTP GET
├── @PostMapping           → HTTP POST
├── @PutMapping            → HTTP PUT
├── @PatchMapping          → HTTP PATCH
├── @DeleteMapping         → HTTP DELETE
├── @PathVariable          → URL path {variable}
├── @RequestParam          → Query string ?key=value
├── @RequestBody           → JSON body → Object
├── @RequestHeader         → HTTP header value
├── @ResponseStatus        → HTTP status code set
├── @ExceptionHandler      → Exception → HTTP response
└── @CrossOrigin           → CORS enable

🔐 SPRING SECURITY
├── @EnableWebSecurity     → Security activate
├── @EnableMethodSecurity  → Method-level security
├── @PreAuthorize          → Method-এর আগে permission check (SpEL)
├── @PostAuthorize         → Return value-এ permission check
└── @Secured               → Simple role-based access

🗄️ JPA / PERSISTENCE
├── @Entity                → DB table mapping
├── @Table                 → Table name, index, constraint
├── @Id                    → Primary key
├── @GeneratedValue        → PK generation strategy
├── @Column                → Column definition
├── @OneToMany             → One-to-many relationship
├── @ManyToOne             → Many-to-one relationship
├── @ManyToMany            → Many-to-many + @JoinTable
├── @OneToOne              → One-to-one
├── @JoinColumn            → Foreign key column
├── @Embedded / @Embeddable → Value object embed করো
├── @Enumerated            → Enum mapping (STRING recommended)
├── @Transient             → DB-তে save হবে না
├── @Lob                   → Large object (TEXT/BLOB)
├── @Version               → Optimistic locking
├── @Query                 → Custom JPQL/SQL query
└── @Modifying             → UPDATE/DELETE query

🦎 HIBERNATE SPECIFIC
├── @DynamicInsert         → Non-null column-only INSERT
├── @DynamicUpdate         → Changed column-only UPDATE
├── @SQLDelete             → Custom DELETE SQL (soft delete)
├── @Where                 → Global WHERE filter (soft delete query)
├── @NaturalId             → Business identifier
└── @Audited               → Full revision history (Envers)

📋 SPRING DATA AUDITING
├── @EnableJpaAuditing     → Auditing activate + AuditorAware config
├── @MappedSuperclass      → Common field inheritance (no table)
├── @EntityListeners       → JPA lifecycle callback (REQUIRED for auditing!)
├── @CreatedDate           → Insert timestamp (auto)
├── @LastModifiedDate      → Update timestamp (auto)
├── @CreatedBy             → Creator username (auto via AuditorAware)
└── @LastModifiedBy        → Last modifier username (auto via AuditorAware)

🎯 AOP
├── @Aspect                → AOP class mark করো
├── @Pointcut              → Reusable pointcut expression
├── @Before                → Method-এর আগে
├── @After                 → Method-এর পরে (always)
├── @AfterReturning        → Success-এ, return value capture করা যায়
├── @AfterThrowing         → Exception-এ, exception capture করা যায়
└── @Around                → সব control — pjp.proceed() দিয়ে call করো

✅ BEAN VALIDATION
├── @Valid / @Validated    → Validation trigger করো
├── @NotNull               → null হলে fail
├── @NotBlank              → null/blank/whitespace হলে fail (String-এর জন্য)
├── @NotEmpty              → null/empty হলে fail
├── @Size(min,max)         → Length/size range
├── @Email                 → Email format
├── @Pattern               → Regex match
├── @Min / @Max            → Number range (integer)
├── @DecimalMin/Max        → Decimal range
├── @Positive/Negative     → > 0 / < 0
├── @Past / @Future        → Date validation
└── @AssertTrue/False      → Boolean validation

🔨 LOMBOK
├── @Data                  → Getter+Setter+ToString+Equals+RequiredArgsConstructor
├── @Builder               → Builder pattern (+ @Builder.Default, @Singular)
├── @Value                 → Immutable class
├── @Getter / @Setter      → Individual getter/setter
├── @NoArgsConstructor     → No-arg constructor
├── @AllArgsConstructor    → All-arg constructor
├── @RequiredArgsConstructor → final/@NonNull field constructor
├── @Slf4j                 → SLF4J logger inject
├── @NonNull               → Null check + NPE
├── @ToString.Exclude      → toString()-এ field বাদ দাও
└── @EqualsAndHashCode.Exclude → equals/hashCode-এ field বাদ দাও

📦 JACKSON
├── @JsonProperty          → JSON field name customize
├── @JsonIgnore            → Serialize/Deserialize-এ ignore
├── @JsonIgnoreProperties  → Class-level ignore + ignoreUnknown
├── @JsonFormat            → Date format, timezone, enum shape
├── @JsonInclude           → Null/empty-তে include না
├── @JsonAlias             → Multiple JSON name accept
├── @JsonManagedReference  → Bidirectional loop prevent (parent side)
├── @JsonBackReference     → Bidirectional loop prevent (child side)
├── @JsonSerialize         → Custom serializer
├── @JsonDeserialize       → Custom deserializer
└── @JsonTypeInfo          → Polymorphism type info

🧪 JUNIT 5
├── @Test                  → Test method
├── @DisplayName           → Human-readable test name
├── @Disabled              → Skip test (with reason!)
├── @BeforeEach/AfterEach  → প্রতি test-এর আগে/পরে
├── @BeforeAll/AfterAll    → Class-এর আগে/পরে (একবার)
├── @ParameterizedTest     → Multiple input test
├── @ValueSource           → Simple array input
├── @CsvSource             → Tabular CSV input
├── @MethodSource          → Stream/method input
├── @EnumSource            → Enum values input
├── @Nested                → Nested test class
├── @Tag                   → Test grouping
├── @Timeout               → Max execution time
└── @RepeatedTest          → N বার run

🎭 MOCKITO
├── @Mock                  → Fake object (all stubbed)
├── @InjectMocks           → Real object + mocks inject
├── @Spy                   → Real object + specific stub
├── @Captor                → Argument capture for assert
├── @MockBean              → Spring Context-এ mock replace
└── @SpyBean               → Spring Context-এ spy wrap

📖 OPENAPI
├── @Tag                   → API group/category
├── @Operation             → Endpoint document (summary + description)
├── @ApiResponse           → HTTP response document
├── @Parameter             → Request parameter document
└── @Schema                → DTO field document
```

---

## 💡 সবচেয়ে Common Mistakes

| ভুল | সঠিক |
|-----|-------|
| Field injection use করা | Constructor injection use করো |
| `@Entity`-তে `@Data` দেওয়া | `@Getter` + `@Setter` আলাদা দাও |
| `@EntityListeners` ভুলে যাওয়া | `@CreatedDate` কাজ করে না এটা ছাড়া |
| `@Transactional` private method-এ | Proxy bypass হয়, কাজ করে না |
| `@Enumerated(ORDINAL)` ব্যবহার | সবসময় `ORDINAL` নয়, `STRING` use করো |
| `@Where` + native query | Native query-তে filter manually দিতে হয় |
| `@Async` same class-এ call | Proxy bypass — আলাদা Bean থেকে call করো |
| `@SuppressWarnings("all")` | Specific warning suppress করো |

---

> **মনে রাখো:** Annotation = শুধু চিরকুট।  
> কাজ করে **Compiler**, **Framework (Spring/Hibernate)**, **AOP Proxy**, এবং **Reflection API**।  
> Framework ছাড়া annotation নিজে কিছুই করে না।

---

*📅 Last Updated: 2024 | ☕ Java + Spring Ecosystem*

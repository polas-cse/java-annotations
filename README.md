# ☕ Java Annotations — Complete Reference Guide

> **সব Java Annotation এক জায়গায় | নাম + কাজ + Example সহ**  
> Covers: Core Java · Spring Boot · JPA/Hibernate · Lombok · Jackson · JUnit/Mockito

---

## 📚 Table of Contents

1. [Core Java Annotations](#1-core-java-annotations)
2. [Meta Annotations](#2-meta-annotations)
3. [Spring Core Annotations](#3-spring-core-annotations)
4. [Spring Boot Annotations](#4-spring-boot-annotations)
5. [Spring MVC / REST Annotations](#5-spring-mvc--rest-annotations)
6. [Spring Security Annotations](#6-spring-security-annotations)
7. [Spring Data / JPA Annotations](#7-spring-data--jpa-annotations)
8. [Hibernate Validator Annotations](#8-hibernate-validator-annotations)
9. [Lombok Annotations](#9-lombok-annotations)
10. [Jackson Annotations](#10-jackson-annotations)
11. [JUnit 5 Annotations](#11-junit-5-annotations)
12. [Mockito Annotations](#12-mockito-annotations)
13. [Jakarta EE / Servlet Annotations](#13-jakarta-ee--servlet-annotations)
14. [Concurrency Annotations](#14-concurrency-annotations)
15. [Quick Cheat Sheet](#15-quick-cheat-sheet)

---

## 1. Core Java Annotations

> Built-in, no dependency লাগে না। `java.lang` এবং `java.lang.annotation` package এ থাকে।

---

### `@Override`
**কাজ:** Parent class বা interface এর method override হচ্ছে কিনা compile-time এ check করে।

```java
class Animal {
    public void sound() { System.out.println("..."); }
}

class Dog extends Animal {
    @Override
    public void sound() { System.out.println("Bark!"); }
}
```

---

### `@Deprecated`
**কাজ:** Method/class/field টি পুরনো এবং ভবিষ্যতে remove হবে — compiler warning দেয়।

```java
@Deprecated(since = "1.5", forRemoval = true)
public void oldMethod() {
    // use newMethod() instead
}
```

---

### `@SuppressWarnings`
**কাজ:** Specific compiler warning suppress করে।

```java
@SuppressWarnings("unchecked")
public void riskyMethod() {
    List list = new ArrayList(); // raw type warning suppress হবে
    list.add("item");
}
```
> Common values: `"unchecked"`, `"deprecation"`, `"unused"`, `"all"`

---

### `@FunctionalInterface`
**কাজ:** Interface টি exactly একটি abstract method রাখবে — compiler নিশ্চিত করে। Lambda এর জন্য ব্যবহার হয়।

```java
@FunctionalInterface
public interface Greeting {
    String greet(String name); // একটিই abstract method
}

// Usage
Greeting g = name -> "Hello, " + name;
System.out.println(g.greet("Polas")); // Hello, Polas
```

---

### `@SafeVarargs`
**কাজ:** Varargs + generics ব্যবহারে heap pollution warning suppress করে।

```java
@SafeVarargs
public final <T> List<T> asList(T... elements) {
    return Arrays.asList(elements);
}
```

---

### `@Native`
**কাজ:** Field টি native code এ refer হতে পারে, এটি indicate করে। (`java.lang.annotation.Native`)

```java
public class Constants {
    @Native public static final int MAX_SIZE = 100;
}
```

---

## 2. Meta Annotations

> Annotation এর উপর লাগানো annotation — অন্য annotation define করতে ব্যবহার হয়।

---

### `@Target`
**কাজ:** Custom annotation কোথায় apply করা যাবে তা নির্ধারণ করে।

```java
@Target({ElementType.METHOD, ElementType.TYPE})
public @interface MyAnnotation {}
```
| Value | মানে |
|-------|------|
| `TYPE` | Class, Interface, Enum |
| `METHOD` | Method |
| `FIELD` | Field/Variable |
| `PARAMETER` | Method parameter |
| `CONSTRUCTOR` | Constructor |
| `LOCAL_VARIABLE` | Local variable |
| `ANNOTATION_TYPE` | অন্য annotation |
| `PACKAGE` | Package |
| `TYPE_USE` | যেকোনো type ব্যবহারে |

---

### `@Retention`
**কাজ:** Annotation কতক্ষণ থাকবে তা নির্ধারণ করে।

```java
@Retention(RetentionPolicy.RUNTIME) // Runtime এ reflection দিয়ে পড়া যাবে
public @interface Auditable {}
```
| Policy | মানে |
|--------|------|
| `SOURCE` | শুধু source code এ, compile হলে হারায় |
| `CLASS` | .class file এ থাকে, runtime এ না |
| `RUNTIME` | Runtime এ reflection দিয়ে পড়া যায় |

---

### `@Documented`
**কাজ:** Annotation টি JavaDoc এ দেখাবে।

```java
@Documented
@Target(ElementType.METHOD)
public @interface ApiEndpoint {}
```

---

### `@Inherited`
**কাজ:** Parent class এ annotation থাকলে child class automatically সেটি inherit করবে।

```java
@Inherited
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
public @interface Auditable {}

@Auditable
class Parent {}

class Child extends Parent {} // Child ও @Auditable পাবে
```

---

### `@Repeatable`
**কাজ:** একই annotation একই জায়গায় একাধিকবার ব্যবহার করা যাবে।

```java
@Repeatable(Schedules.class)
public @interface Schedule {
    String day();
}

@Retention(RetentionPolicy.RUNTIME)
public @interface Schedules {
    Schedule[] value();
}

// Usage
@Schedule(day = "Monday")
@Schedule(day = "Friday")
public void run() {}
```

---

## 3. Spring Core Annotations

> `spring-context` dependency লাগবে।

---

### `@Component`
**কাজ:** Class টিকে Spring Bean হিসেবে register করে। Generic stereotype।

```java
@Component
public class EmailValidator {
    public boolean isValid(String email) {
        return email.contains("@");
    }
}
```

---

### `@Service`
**কাজ:** Business logic layer এ ব্যবহার। `@Component` এরই specialized version।

```java
@Service
public class UserService {
    public User findById(Long id) { ... }
}
```

---

### `@Repository`
**কাজ:** Data access layer (DAO) এ ব্যবহার। Database exception গুলো Spring exception এ translate করে।

```java
@Repository
public class UserRepository {
    public User findByEmail(String email) { ... }
}
```

---

### `@Controller`
**কাজ:** Spring MVC controller। View (JSP/Thymeleaf) return করে।

```java
@Controller
public class HomeController {
    @GetMapping("/")
    public String home(Model model) {
        model.addAttribute("msg", "Welcome");
        return "index"; // view name
    }
}
```

---

### `@RestController`
**কাজ:** `@Controller` + `@ResponseBody` এর combination। JSON/XML response return করে।

```java
@RestController
@RequestMapping("/api/users")
public class UserController {
    @GetMapping("/{id}")
    public User getUser(@PathVariable Long id) {
        return userService.findById(id);
    }
}
```

---

### `@Autowired`
**কাজ:** Dependency Injection — Spring নিজেই dependency inject করে দেয়।

```java
@Service
public class OrderService {

    private final UserService userService;

    @Autowired // Constructor injection (recommended)
    public OrderService(UserService userService) {
        this.userService = userService;
    }
}
```
> ✅ Constructor injection সবচেয়ে ভালো practice।

---

### `@Qualifier`
**কাজ:** একই type এর একাধিক Bean থাকলে কোনটা inject করবে তা specify করে।

```java
@Component("smsNotifier")
public class SmsNotifier implements Notifier {}

@Component("emailNotifier")
public class EmailNotifier implements Notifier {}

@Service
public class AlertService {
    @Autowired
    @Qualifier("emailNotifier")
    private Notifier notifier;
}
```

---

### `@Primary`
**কাজ:** একাধিক Bean এর মধ্যে default Bean হিসেবে mark করে।

```java
@Component
@Primary
public class EmailNotifier implements Notifier {}
```

---

### `@Bean`
**কাজ:** `@Configuration` class এর method থেকে Bean তৈরি করে। Third-party class গুলো Bean বানাতে কাজে লাগে।

```java
@Configuration
public class AppConfig {
    @Bean
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }

    @Bean
    public ObjectMapper objectMapper() {
        return new ObjectMapper();
    }
}
```

---

### `@Configuration`
**কাজ:** Class টি Spring configuration source — এখানে `@Bean` define করা হয়।

```java
@Configuration
public class SecurityConfig {
    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
}
```

---

### `@Value`
**কাজ:** `application.properties` বা environment variable থেকে value inject করে।

```java
@Component
public class PaymentService {
    @Value("${payment.api.key}")
    private String apiKey;

    @Value("${payment.timeout:5000}") // default 5000ms
    private int timeout;
}
```

---

### `@Scope`
**কাজ:** Bean এর lifecycle নির্ধারণ করে।

```java
@Component
@Scope("prototype") // প্রতিবার নতুন instance
public class ReportGenerator {}
```
| Scope | মানে |
|-------|------|
| `singleton` | একটি instance (default) |
| `prototype` | প্রতিবার নতুন |
| `request` | প্রতি HTTP request |
| `session` | প্রতি HTTP session |

---

### `@Lazy`
**কাজ:** Bean startup এ তৈরি না হয়ে প্রথমবার ব্যবহারের সময় তৈরি হবে।

```java
@Component
@Lazy
public class HeavyService {}
```

---

### `@Profile`
**কাজ:** নির্দিষ্ট environment (dev/prod) এ Bean active হবে।

```java
@Component
@Profile("dev")
public class DevEmailService implements EmailService {}

@Component
@Profile("prod")
public class SmtpEmailService implements EmailService {}
```

---

### `@EventListener`
**কাজ:** Application event শুনে reaction করে।

```java
@Component
public class UserEventHandler {
    @EventListener
    public void onUserCreated(UserCreatedEvent event) {
        System.out.println("New user: " + event.getEmail());
    }
}
```

---

### `@Async`
**কাজ:** Method টি আলাদা thread এ async execute হবে।

```java
@Service
public class EmailService {
    @Async
    public CompletableFuture<Void> sendEmail(String to) {
        // runs in separate thread
        return CompletableFuture.completedFuture(null);
    }
}
```
> ⚠️ Main class এ `@EnableAsync` লাগবে।

---

### `@Scheduled`
**কাজ:** Method টি নির্দিষ্ট সময় পর পর automatically execute হবে।

```java
@Component
public class ReportJob {
    @Scheduled(cron = "0 0 8 * * MON-FRI") // প্রতিদিন সকাল ৮টায়
    public void generateDailyReport() { ... }

    @Scheduled(fixedDelay = 5000) // প্রতি ৫ সেকেন্ড পর পর
    public void healthCheck() { ... }
}
```
> ⚠️ Main class এ `@EnableScheduling` লাগবে।

---

### `@Transactional`
**কাজ:** Database transaction manage করে। Error হলে rollback, সফল হলে commit।

```java
@Service
public class TransferService {
    @Transactional
    public void transfer(Long fromId, Long toId, double amount) {
        accountRepo.debit(fromId, amount);
        accountRepo.credit(toId, amount); // কোনো error হলে উভয়ই rollback
    }
}
```

---

### `@Cacheable`, `@CachePut`, `@CacheEvict`
**কাজ:** Method result cache করে performance বাড়ায়।

```java
@Service
public class ProductService {

    @Cacheable(value = "products", key = "#id")
    public Product findById(Long id) { ... } // প্রথমবার DB, পরে cache

    @CachePut(value = "products", key = "#product.id")
    public Product update(Product product) { ... } // সবসময় DB + cache update

    @CacheEvict(value = "products", key = "#id")
    public void delete(Long id) { ... } // cache থেকে remove
}
```

---

## 4. Spring Boot Annotations

---

### `@SpringBootApplication`
**কাজ:** `@Configuration` + `@EnableAutoConfiguration` + `@ComponentScan` এক সাথে।

```java
@SpringBootApplication
public class MyApp {
    public static void main(String[] args) {
        SpringApplication.run(MyApp.class, args);
    }
}
```

---

### `@EnableAutoConfiguration`
**কাজ:** Classpath দেখে Spring Boot নিজেই configuration করে নেয়।

```java
@EnableAutoConfiguration(exclude = {DataSourceAutoConfiguration.class})
public class AppConfig {}
```

---

### `@ConfigurationProperties`
**কাజ:** `application.properties` এর group property গুলো এক class এ bind করে।

```java
@ConfigurationProperties(prefix = "app.mail")
@Component
public class MailProperties {
    private String host;
    private int port;
    private String username;
    // getters + setters
}
```
```properties
# application.properties
app.mail.host=smtp.gmail.com
app.mail.port=587
app.mail.username=noreply@example.com
```

---

### `@ConditionalOnProperty`
**কাজ:** Property value এর উপর ভিত্তি করে Bean conditionally তৈরি হবে।

```java
@Bean
@ConditionalOnProperty(name = "feature.sms.enabled", havingValue = "true")
public SmsService smsService() {
    return new SmsService();
}
```

---

### `@ConditionalOnMissingBean`
**কাজ:** ওই type এর Bean না থাকলে এই Bean তৈরি হবে।

```java
@Bean
@ConditionalOnMissingBean(DataSource.class)
public DataSource defaultDataSource() { ... }
```

---

### `@SpringBootTest`
**কাজ:** Full Spring context load করে integration test করে।

```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
class UserControllerTest {
    @Autowired
    private TestRestTemplate restTemplate;
    // ...
}
```

---

## 5. Spring MVC / REST Annotations

---

### Request Mapping Annotations

```java
@RestController
@RequestMapping("/api/v1/products")
public class ProductController {

    @GetMapping               // GET /api/v1/products
    @GetMapping("/{id}")      // GET /api/v1/products/1

    @PostMapping              // POST /api/v1/products

    @PutMapping("/{id}")      // PUT /api/v1/products/1

    @PatchMapping("/{id}")    // PATCH /api/v1/products/1

    @DeleteMapping("/{id}")   // DELETE /api/v1/products/1
}
```

---

### `@PathVariable`
**কাজ:** URL path থেকে value নেয়।

```java
@GetMapping("/users/{id}/orders/{orderId}")
public Order getOrder(@PathVariable Long id, @PathVariable Long orderId) { ... }
```

---

### `@RequestParam`
**কাজ:** URL query parameter থেকে value নেয়।

```java
// GET /products?page=1&size=10&sort=name
@GetMapping("/products")
public List<Product> getProducts(
    @RequestParam(defaultValue = "0") int page,
    @RequestParam(defaultValue = "10") int size,
    @RequestParam(required = false) String sort
) { ... }
```

---

### `@RequestBody`
**কাজ:** HTTP request body (JSON) কে Java object এ convert করে।

```java
@PostMapping("/users")
public ResponseEntity<User> createUser(@RequestBody @Valid UserRequest request) {
    User user = userService.create(request);
    return ResponseEntity.status(HttpStatus.CREATED).body(user);
}
```

---

### `@ResponseBody`
**কাজ:** Return value কে HTTP response body তে লেখে (JSON)। `@RestController` এ আলাদাভাবে লাগে না।

```java
@ResponseBody
@GetMapping("/hello")
public String hello() {
    return "Hello World"; // JSON response
}
```

---

### `@RequestHeader`
**কাজ:** HTTP request header থেকে value নেয়।

```java
@GetMapping("/secure")
public String secure(@RequestHeader("Authorization") String token) { ... }
```

---

### `@CookieValue`
**কাজ:** HTTP cookie থেকে value নেয়।

```java
@GetMapping("/dashboard")
public String dashboard(@CookieValue("sessionId") String sessionId) { ... }
```

---

### `@ResponseStatus`
**কাজ:** Handler method বা exception class এ HTTP status code set করে।

```java
@ResponseStatus(HttpStatus.CREATED) // 201
@PostMapping("/items")
public Item create(@RequestBody Item item) { ... }

@ResponseStatus(HttpStatus.NOT_FOUND) // 404
public class ResourceNotFoundException extends RuntimeException {}
```

---

### `@ExceptionHandler`
**কাজ:** Controller এর মধ্যে exception handle করে।

```java
@RestController
public class UserController {

    @ExceptionHandler(UserNotFoundException.class)
    public ResponseEntity<String> handleNotFound(UserNotFoundException ex) {
        return ResponseEntity.status(404).body(ex.getMessage());
    }
}
```

---

### `@ControllerAdvice` / `@RestControllerAdvice`
**কাজ:** Global exception handler — সব controller এর exception এক জায়গায় handle করে।

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorResponse> handleAll(Exception ex) {
        return ResponseEntity.internalServerError()
            .body(new ErrorResponse(ex.getMessage()));
    }

    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<Map<String, String>> handleValidation(
            MethodArgumentNotValidException ex) {
        Map<String, String> errors = new HashMap<>();
        ex.getBindingResult().getFieldErrors()
            .forEach(e -> errors.put(e.getField(), e.getDefaultMessage()));
        return ResponseEntity.badRequest().body(errors);
    }
}
```

---

### `@CrossOrigin`
**কাজ:** CORS enable করে — অন্য domain থেকে request allow করে।

```java
@CrossOrigin(origins = "http://localhost:3000")
@RestController
public class ApiController {}
```

---

### `@ModelAttribute`
**কাজ:** Form data কে model object এ bind করে।

```java
@PostMapping("/submit")
public String submit(@ModelAttribute UserForm form) { ... }
```

---

## 6. Spring Security Annotations

---

### `@EnableWebSecurity`
**কাজ:** Spring Security activate করে।

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig { ... }
```

---

### `@EnableMethodSecurity`
**কাজ:** Method-level security annotation enable করে।

```java
@Configuration
@EnableMethodSecurity
public class MethodSecurityConfig {}
```

---

### `@PreAuthorize`
**কাজ:** Method execute হওয়ার আগে permission check করে।

```java
@PreAuthorize("hasRole('ADMIN')")
public void deleteUser(Long id) { ... }

@PreAuthorize("hasAnyRole('ADMIN', 'MANAGER') and #id == authentication.principal.id")
public User getUser(Long id) { ... }
```

---

### `@PostAuthorize`
**কাজ:** Method execute হওয়ার পরে return value এর উপর permission check করে।

```java
@PostAuthorize("returnObject.ownerId == authentication.principal.id")
public Document getDocument(Long id) { ... }
```

---

### `@Secured`
**কাজ:** Role-based access control। `@PreAuthorize` এর সরল version।

```java
@Secured("ROLE_ADMIN")
public void adminTask() { ... }
```

---

### `@RolesAllowed`
**কাজ:** JSR-250 standard। `@Secured` এর মতো।

```java
@RolesAllowed({"ROLE_USER", "ROLE_ADMIN"})
public List<Order> getOrders() { ... }
```

---

## 7. Spring Data / JPA Annotations

---

### `@Entity`
**কাজ:** Class টি database table এর সাথে map হবে।

```java
@Entity
@Table(name = "users")
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(name = "full_name", nullable = false, length = 100)
    private String fullName;

    @Column(unique = true)
    private String email;
}
```

---

### `@Table`
**কাজ:** Table name, schema, index নির্ধারণ করে।

```java
@Entity
@Table(
    name = "products",
    schema = "inventory",
    indexes = {@Index(name = "idx_sku", columnList = "sku")}
)
public class Product {}
```

---

### `@Id`
**কাজ:** Primary key field নির্দেশ করে।

---

### `@GeneratedValue`
**কাজ:** Primary key generation strategy নির্ধারণ করে।

```java
@Id
@GeneratedValue(strategy = GenerationType.IDENTITY) // Auto increment
private Long id;

// অন্য strategies:
// GenerationType.AUTO — JPA নিজে decide করে
// GenerationType.SEQUENCE — DB sequence use করে
// GenerationType.UUID — UUID generate করে
```

---

### `@Column`
**কাজ:** Column এর name, constraint, size নির্ধারণ করে।

```java
@Column(name = "phone_number", nullable = false, length = 15, unique = true)
private String phone;
```

---

### `@OneToMany`, `@ManyToOne`, `@OneToOne`, `@ManyToMany`
**কাজ:** Table এর মধ্যে relationship নির্ধারণ করে।

```java
@Entity
public class Department {
    @Id private Long id;

    @OneToMany(mappedBy = "department", cascade = CascadeType.ALL, fetch = FetchType.LAZY)
    private List<Employee> employees;
}

@Entity
public class Employee {
    @Id private Long id;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "department_id")
    private Department department;
}
```

---

### `@JoinColumn`
**কাজ:** Foreign key column নির্ধারণ করে।

```java
@ManyToOne
@JoinColumn(name = "category_id", nullable = false)
private Category category;
```

---

### `@JoinTable`
**কাজ:** ManyToMany relationship এর join table নির্ধারণ করে।

```java
@ManyToMany
@JoinTable(
    name = "user_roles",
    joinColumns = @JoinColumn(name = "user_id"),
    inverseJoinColumns = @JoinColumn(name = "role_id")
)
private Set<Role> roles;
```

---

### `@Embedded` / `@Embeddable`
**কাজ:** আলাদা table না বানিয়ে object কে same table এ embed করে।

```java
@Embeddable
public class Address {
    private String street;
    private String city;
    private String zipCode;
}

@Entity
public class User {
    @Embedded
    private Address address; // user table এর ভেতরেই address columns থাকবে
}
```

---

### `@Transient`
**কাজ:** Field টি database এ save হবে না।

```java
@Transient
private String temporaryToken; // শুধু runtime এ ব্যবহার
```

---

### `@Enumerated`
**কাজ:** Enum value কে কীভাবে DB তে store করবে।

```java
@Enumerated(EnumType.STRING) // "ACTIVE" হিসেবে store হবে
private UserStatus status;

// EnumType.ORDINAL — 0, 1, 2 হিসেবে store হয় (avoid করা ভালো)
```

---

### `@CreationTimestamp` / `@UpdateTimestamp`
**কাজ:** Create এবং update time automatically set করে। (Hibernate)

```java
@CreationTimestamp
@Column(updatable = false)
private LocalDateTime createdAt;

@UpdateTimestamp
private LocalDateTime updatedAt;
```

---

### `@Lob`
**কাজ:** Large Object (text/blob) store করতে।

```java
@Lob
@Column(columnDefinition = "TEXT")
private String description;
```

---

### `@Version`
**কাজ:** Optimistic locking — concurrent update handle করে।

```java
@Version
private Long version;
```

---

### Spring Data Repository Annotations

```java
@Repository
public interface UserRepository extends JpaRepository<User, Long> {

    // Method name থেকে query তৈরি
    Optional<User> findByEmail(String email);
    List<User> findByStatusAndCreatedAtAfter(UserStatus status, LocalDateTime date);

    // Custom JPQL query
    @Query("SELECT u FROM User u WHERE u.email = :email AND u.active = true")
    Optional<User> findActiveByEmail(@Param("email") String email);

    // Native SQL query
    @Query(value = "SELECT * FROM users WHERE phone = ?1", nativeQuery = true)
    Optional<User> findByPhone(String phone);

    // Modifying query
    @Modifying
    @Transactional
    @Query("UPDATE User u SET u.status = :status WHERE u.id = :id")
    int updateStatus(@Param("id") Long id, @Param("status") UserStatus status);
}
```

---

### `@EntityGraph`
**কাজ:** N+1 query problem solve করতে fetch strategy define করে।

```java
@EntityGraph(attributePaths = {"roles", "address"})
@Query("SELECT u FROM User u WHERE u.id = :id")
Optional<User> findByIdWithDetails(@Param("id") Long id);
```

---

## 8. Hibernate Validator Annotations

> `spring-boot-starter-validation` dependency লাগবে।  
> `@Valid` বা `@Validated` দিয়ে trigger করতে হয়।

```java
public class UserRequest {

    @NotNull(message = "Name cannot be null")
    @NotBlank(message = "Name cannot be blank")
    @Size(min = 2, max = 50, message = "Name must be between 2-50 characters")
    private String name;

    @Email(message = "Invalid email format")
    @NotEmpty
    private String email;

    @Min(value = 18, message = "Age must be at least 18")
    @Max(value = 120)
    private Integer age;

    @Pattern(regexp = "^\\+880[0-9]{10}$", message = "Invalid BD phone number")
    private String phone;

    @NotNull
    @Positive(message = "Price must be positive")
    private Double price;

    @PositiveOrZero
    private Integer stock;

    @Negative
    private Double discount; // must be negative

    @Past(message = "Birth date must be in the past")
    private LocalDate birthDate;

    @Future(message = "Expiry must be in future")
    private LocalDate expiryDate;

    @PastOrPresent
    private LocalDateTime lastLogin;

    @FutureOrPresent
    private LocalDateTime appointmentDate;

    @AssertTrue(message = "Terms must be accepted")
    private Boolean termsAccepted;

    @AssertFalse
    private Boolean banned;

    @DecimalMin("0.0")
    @DecimalMax("100.0")
    @Digits(integer = 3, fraction = 2)
    private BigDecimal taxRate;

    @NotEmpty
    @Valid // nested object validation
    private List<@NotBlank String> tags;
}
```

---

## 9. Lombok Annotations

> `lombok` dependency লাগবে।

---

| Annotation | কাজ |
|---|---|
| `@Getter` | সব field এর getter generate করে |
| `@Setter` | সব field এর setter generate করে |
| `@ToString` | `toString()` generate করে |
| `@EqualsAndHashCode` | `equals()` এবং `hashCode()` generate করে |
| `@NoArgsConstructor` | No-arg constructor generate করে |
| `@AllArgsConstructor` | All-arg constructor generate করে |
| `@RequiredArgsConstructor` | `final` field গুলোর constructor generate করে |
| `@Data` | `@Getter` + `@Setter` + `@ToString` + `@EqualsAndHashCode` + `@RequiredArgsConstructor` |
| `@Builder` | Builder pattern implement করে |
| `@Value` | Immutable class বানায় |
| `@Slf4j` | `log` variable inject করে (SLF4J) |
| `@Log4j2` | `log` variable inject করে (Log4j2) |
| `@NonNull` | Null check করে NullPointerException throw করে |
| `@Cleanup` | AutoCloseable resource auto-close করে |
| `@SneakyThrows` | Checked exception কে unchecked হিসেবে throw করে |
| `@Synchronized` | Thread-safe synchronized method বানায় |

```java
@Data
@Builder
@NoArgsConstructor
@AllArgsConstructor
@Slf4j
public class Product {

    @NonNull
    private String name;
    private Double price;
    private Integer stock;

    public void processOrder(int qty) {
        log.info("Processing order for {} units of {}", qty, name);
        // ...
    }
}

// Builder usage
Product p = Product.builder()
    .name("Camera")
    .price(45000.0)
    .stock(10)
    .build();
```

---

## 10. Jackson Annotations

> JSON serialize/deserialize এর জন্য। `jackson-databind` দরকার।

---

### `@JsonProperty`
**কাজ:** JSON field এর name customize করে।

```java
public class User {
    @JsonProperty("full_name")
    private String fullName; // JSON: { "full_name": "Polas" }
}
```

---

### `@JsonIgnore` / `@JsonIgnoreProperties`
**কাজ:** Serialize/deserialize এ field ignore করে।

```java
public class User {
    @JsonIgnore
    private String password; // JSON response এ আসবে না
}

@JsonIgnoreProperties({"createdAt", "updatedAt"})
public class ProductDto {}

@JsonIgnoreProperties(ignoreUnknown = true) // Unknown field এ error দেবে না
public class ApiResponse {}
```

---

### `@JsonFormat`
**কাজ:** Date/time format নির্ধারণ করে।

```java
@JsonFormat(pattern = "dd-MM-yyyy HH:mm:ss", timezone = "Asia/Dhaka")
private LocalDateTime createdAt;
```

---

### `@JsonInclude`
**কাজ:** কোন condition এ field serialize হবে।

```java
@JsonInclude(JsonInclude.Include.NON_NULL) // null হলে JSON এ আসবে না
private String optionalField;

@JsonInclude(JsonInclude.Include.NON_EMPTY) // empty/null হলে আসবে না
private List<String> tags;
```

---

### `@JsonAlias`
**কাজ:** Deserialize এ একাধিক JSON field name accept করে।

```java
@JsonAlias({"user_name", "userName", "uname"})
private String username;
```

---

### `@JsonSerialize` / `@JsonDeserialize`
**কাজ:** Custom serializer/deserializer use করে।

```java
@JsonSerialize(using = MoneySerializer.class)
@JsonDeserialize(using = MoneyDeserializer.class)
private BigDecimal amount;
```

---

### `@JsonManagedReference` / `@JsonBackReference`
**কাজ:** Bidirectional relationship এ infinite loop prevent করে।

```java
public class Department {
    @JsonManagedReference
    private List<Employee> employees;
}

public class Employee {
    @JsonBackReference
    private Department department; // serialize হবে না
}
```

---

### `@JsonTypeInfo` / `@JsonSubTypes`
**কাজ:** Polymorphism — parent class থেকে সঠিক subclass deserialize করে।

```java
@JsonTypeInfo(use = JsonTypeInfo.Id.NAME, property = "type")
@JsonSubTypes({
    @JsonSubTypes.Type(value = Cat.class, name = "cat"),
    @JsonSubTypes.Type(value = Dog.class, name = "dog")
})
public abstract class Animal {}
```

---

## 11. JUnit 5 Annotations

---

```java
@ExtendWith(MockitoExtension.class)
class UserServiceTest {

    @Test
    @DisplayName("User should be found by email")
    void testFindByEmail() {
        // test body
    }

    @Test
    @Disabled("Bug #123 fix pending")
    void skippedTest() {}

    @BeforeEach
    void setUp() {
        // প্রতিটি test এর আগে run হবে
    }

    @AfterEach
    void tearDown() {
        // প্রতিটি test এর পরে run হবে
    }

    @BeforeAll
    static void initAll() {
        // সব test এর আগে একবার run হবে
    }

    @AfterAll
    static void cleanupAll() {
        // সব test এর পরে একবার run হবে
    }

    @Test
    @Timeout(2) // 2 সেকেন্ডের বেশি লাগলে fail
    void performanceTest() {}

    @RepeatedTest(5) // ৫ বার run হবে
    void repeatedTest() {}

    @ParameterizedTest
    @ValueSource(strings = {"valid@email.com", "another@test.org"})
    void testEmailValidation(String email) {
        assertTrue(isValid(email));
    }

    @ParameterizedTest
    @CsvSource({"1, Apple, 100", "2, Banana, 50"})
    void testProduct(Long id, String name, int price) { ... }

    @ParameterizedTest
    @EnumSource(UserStatus.class)
    void testAllStatuses(UserStatus status) { ... }

    @Test
    @Tag("integration")
    void integrationTest() {}

    @Nested
    @DisplayName("When user is admin")
    class AdminTests {
        @Test
        void canDeleteOtherUsers() { ... }
    }
}
```

---

## 12. Mockito Annotations

---

```java
@ExtendWith(MockitoExtension.class)
class OrderServiceTest {

    @Mock
    private UserRepository userRepository; // fake implementation

    @Mock
    private EmailService emailService;

    @InjectMocks
    private OrderService orderService; // @Mock গুলো এখানে inject হবে

    @Spy
    private OrderValidator orderValidator; // real object, কিন্তু spy করা যায়

    @Captor
    private ArgumentCaptor<Order> orderCaptor; // argument capture করে

    @Test
    void shouldCreateOrder() {
        // Arrange
        User user = new User(1L, "Polas");
        when(userRepository.findById(1L)).thenReturn(Optional.of(user));

        // Act
        orderService.createOrder(1L, new OrderRequest());

        // Assert
        verify(emailService, times(1)).sendConfirmation(orderCaptor.capture());
        assertEquals(1L, orderCaptor.getValue().getUserId());
    }
}
```

---

## 13. Jakarta EE / Servlet Annotations

---

### `@WebServlet`
```java
@WebServlet(urlPatterns = "/hello", name = "HelloServlet")
public class HelloServlet extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws IOException {
        resp.getWriter().write("Hello!");
    }
}
```

---

### `@WebFilter`
```java
@WebFilter("/*")
public class AuthFilter implements Filter {
    @Override
    public void doFilter(ServletRequest req, ServletResponse res, FilterChain chain)
            throws IOException, ServletException {
        // auth check
        chain.doFilter(req, res);
    }
}
```

---

### `@WebListener`
```java
@WebListener
public class AppListener implements ServletContextListener {
    @Override
    public void contextInitialized(ServletContextEvent sce) {
        System.out.println("App started!");
    }
}
```

---

### `@Inject` (CDI)
```java
public class PaymentService {
    @Inject
    private TaxCalculator taxCalculator; // Spring @Autowired এর মতো
}
```

---

## 14. Concurrency Annotations

> `javax.annotation.concurrent` — documentation purpose এ ব্যবহার হয়।

```java
@ThreadSafe
public class SafeCounter {
    private final AtomicInteger count = new AtomicInteger(0);
    public int increment() { return count.incrementAndGet(); }
}

@NotThreadSafe
public class UnsafeCounter {
    private int count = 0;
    public int increment() { return ++count; }
}

@Immutable
public final class Money {
    private final BigDecimal amount;
    private final String currency;
    // no setters
}

@GuardedBy("this")
private List<Order> pendingOrders;
```

---

## 15. Quick Cheat Sheet

```
📦 CORE JAVA
├── @Override            → Method override check
├── @Deprecated          → পুরনো, use করো না
├── @SuppressWarnings    → Warning চুপ করাও
└── @FunctionalInterface → Lambda-friendly interface

🏷️ META
├── @Target              → কোথায় apply হবে
├── @Retention           → কতক্ষণ থাকবে
├── @Inherited           → Child class inherit করবে
└── @Repeatable          → একাধিকবার use করা যাবে

🌱 SPRING STEREOTYPE
├── @Component           → Generic bean
├── @Service             → Business logic
├── @Repository          → Data access
└── @Controller / @RestController → Web layer

💉 SPRING DI
├── @Autowired           → Inject dependency
├── @Qualifier           → কোন bean inject হবে
├── @Primary             → Default bean
├── @Bean                → Manual bean creation
└── @Value               → Property inject

🌐 SPRING MVC
├── @GetMapping          → HTTP GET
├── @PostMapping         → HTTP POST
├── @PutMapping          → HTTP PUT
├── @DeleteMapping       → HTTP DELETE
├── @PathVariable        → URL path value
├── @RequestParam        → Query string value
├── @RequestBody         → JSON body → Object
└── @ResponseStatus      → HTTP status code

🗄️ JPA
├── @Entity              → DB table
├── @Id / @GeneratedValue → Primary key
├── @Column              → Column mapping
├── @OneToMany           → Relationship
├── @Transient           → Not persisted
└── @Transactional       → Transaction management

✅ VALIDATION
├── @NotNull / @NotBlank → Null/blank check
├── @Email               → Email format
├── @Size                → Length check
├── @Min / @Max          → Range check
└── @Pattern             → Regex match

🔨 LOMBOK
├── @Data                → Getter+Setter+etc
├── @Builder             → Builder pattern
├── @Slf4j               → Logger inject
└── @RequiredArgsConstructor → Final field constructor

🧪 TESTING
├── @Test                → Test method
├── @Mock                → Fake object
├── @InjectMocks         → Inject mocks
└── @ParameterizedTest   → Multiple input test
```

---

## 📌 Custom Annotation Example

নিজের annotation বানানোর উদাহরণ:

```java
// 1. Annotation define করো
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface AuditLog {
    String action() default "UNKNOWN";
    boolean logArgs() default true;
}

// 2. AOP দিয়ে handle করো
@Aspect
@Component
public class AuditLogAspect {

    @Around("@annotation(auditLog)")
    public Object log(ProceedingJoinPoint pjp, AuditLog auditLog) throws Throwable {
        System.out.println("Action: " + auditLog.action());
        Object result = pjp.proceed();
        System.out.println("Completed!");
        return result;
    }
}

// 3. ব্যবহার করো
@AuditLog(action = "DELETE_USER", logArgs = true)
public void deleteUser(Long id) {
    userRepository.deleteById(id);
}
```

---

> 💡 **Pro Tip:** Annotation শুধু metadata — নিজে কিছু করে না।  
> কাজ করে **Framework** (Spring, Hibernate) অথবা **AOP** বা **Reflection** দিয়ে।

---

*Made with ❤️ | Java Annotations Complete Reference*

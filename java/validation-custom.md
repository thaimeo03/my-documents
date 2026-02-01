Ok, Cancer trả lời gọn – chuẩn bài enterprise nhưng không khô 😌

Trong **Spring Boot**, khi bạn dùng **Bean Validation (Jakarta Validation / Hibernate Validator)** với annotation (`@NotNull`, `@Email`, …), có **2 level custom** phổ biến:

---

## 1️⃣ Custom message cho validator có sẵn (quick win)

Ví dụ validate email bằng annotation có sẵn:

```java
public class RegisterRequest {

    @NotBlank(message = "Email không được để trống")
    @Email(message = "Email không đúng định dạng")
    private String email;
}
```

👉 Cách này **đủ dùng trong 70% case**.

Nếu muốn **đa ngôn ngữ / message tập trung**, dùng `messages.properties`:

**application.yml**

```yaml
spring:
  messages:
    basename: messages
```

**messages.properties**

```properties
email.invalid=Email không đúng định dạng
```

```java
@Email(message = "{email.invalid}")
private String email;
```

---

## 2️⃣ Custom annotation validator (chuẩn enterprise)

Dùng khi:

* Validate logic phức tạp
* Regex đặc thù
* Check DB (email đã tồn tại, domain nội bộ, …)

---

### Bước 1: Tạo annotation custom

```java
@Documented
@Constraint(validatedBy = EmailCustomValidator.class)
@Target({ ElementType.FIELD, ElementType.PARAMETER })
@Retention(RetentionPolicy.RUNTIME)
public @interface EmailCustom {

    String message() default "Email không hợp lệ";

    Class<?>[] groups() default {};

    Class<? extends Payload>[] payload() default {};
}
```

---

### Bước 2: Implement `ConstraintValidator`

```java
public class EmailCustomValidator
        implements ConstraintValidator<EmailCustom, String> {

    private static final Pattern EMAIL_PATTERN =
            Pattern.compile("^[A-Za-z0-9+_.-]+@[A-Za-z0-9.-]+$");

    @Override
    public boolean isValid(String value, ConstraintValidatorContext context) {
        if (value == null || value.isBlank()) {
            return true; // để @NotBlank xử lý
        }
        return EMAIL_PATTERN.matcher(value).matches();
    }
}
```

📌 **Best practice**:

* Validator **không xử lý null**
* Tách rõ `@NotNull`, `@NotBlank`

---

### Bước 3: Sử dụng

```java
public class RegisterRequest {

    @NotBlank(message = "Email không được để trống")
    @EmailCustom
    private String email;
}
```

---

## 3️⃣ Custom message động (advanced)

Ví dụ message phụ thuộc logic:

```java
@Override
public boolean isValid(String value, ConstraintValidatorContext context) {
    if (!value.endsWith("@company.com")) {
        context.disableDefaultConstraintViolation();
        context.buildConstraintViolationWithTemplate(
            "Email phải thuộc domain @company.com"
        ).addConstraintViolation();
        return false;
    }
    return true;
}
```

---

## 4️⃣ Validate được trigger khi nào?

```java
@PostMapping("/register")
public ResponseEntity<?> register(
        @Valid @RequestBody RegisterRequest request
) {
    ...
}
```

📌 **Không có `@Valid` hoặc `@Validated` → validator không chạy**

---

## 5️⃣ Global handle lỗi validate (chuẩn chỉnh)

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<?> handleValidation(
            MethodArgumentNotValidException ex) {

        Map<String, String> errors = new HashMap<>();

        ex.getBindingResult().getFieldErrors()
          .forEach(e -> errors.put(e.getField(), e.getDefaultMessage()));

        return ResponseEntity.badRequest().body(errors);
    }
}
```

---

## Khi nào dùng cách nào?

| Case            | Cách                         |
| --------------- | ---------------------------- |
| Format đơn giản | `@Email`                     |
| Regex đặc thù   | Custom `ConstraintValidator` |
| Check DB        | Custom validator + service   |
| Multi-field     | Class-level validator        |

---

Nếu bạn muốn, Cancer có thể:

* Viết **validator check email tồn tại trong DB**
* Viết **class-level validator** (vd: password + confirmPassword)
* So sánh **@Validated vs @Valid** (hay bị hỏi phỏng vấn 😏)

Bạn chọn nhánh nào?

Ok, Cancer triển khai **đủ combo 3 nhánh** cho bạn theo style **chuẩn enterprise – dễ mở rộng – đúng thứ hay bị hỏi phỏng vấn**. Đi từ dễ → khó để dễ ngấm 👌

---

# 1️⃣ Validate **email tồn tại trong DB** (Custom Validator + Service)

## Use case

* Đăng ký user
* Không cho dùng email đã tồn tại

---

## Bước 1: Repository

```java
public interface UserRepository extends JpaRepository<User, Long> {

    boolean existsByEmail(String email);
}
```

---

## Bước 2: Custom annotation

```java
@Documented
@Constraint(validatedBy = EmailNotExistsValidator.class)
@Target({ ElementType.FIELD })
@Retention(RetentionPolicy.RUNTIME)
public @interface EmailNotExists {

    String message() default "Email đã tồn tại";

    Class<?>[] groups() default {};

    Class<? extends Payload>[] payload() default {};
}
```

---

## Bước 3: Validator (inject service)

```java
@Component
public class EmailNotExistsValidator
        implements ConstraintValidator<EmailNotExists, String> {

    private final UserRepository userRepository;

    public EmailNotExistsValidator(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    @Override
    public boolean isValid(String email, ConstraintValidatorContext context) {
        if (email == null || email.isBlank()) {
            return true; // @NotBlank xử lý
        }
        return !userRepository.existsByEmail(email);
    }
}
```

📌 **Note quan trọng (phỏng vấn hay hỏi)**
✔ Validator **được inject Spring Bean** bình thường
✔ Không nên viết logic phức tạp trong controller

---

## Bước 4: Sử dụng

```java
public class RegisterRequest {

    @NotBlank
    @Email
    @EmailNotExists
    private String email;
}
```

---

# 2️⃣ Class-level Validator

### (password + confirmPassword)

## Use case

* 2 field phụ thuộc nhau
* Không thể validate bằng field-level annotation

---

## Bước 1: Annotation cho class

```java
@Documented
@Constraint(validatedBy = PasswordMatchValidator.class)
@Target({ ElementType.TYPE })
@Retention(RetentionPolicy.RUNTIME)
public @interface PasswordMatch {

    String message() default "Password không khớp";

    Class<?>[] groups() default {};

    Class<? extends Payload>[] payload() default {};
}
```

---

## Bước 2: DTO

```java
@PasswordMatch
public class RegisterRequest {

    @NotBlank
    private String password;

    @NotBlank
    private String confirmPassword;
}
```

---

## Bước 3: Validator

```java
public class PasswordMatchValidator
        implements ConstraintValidator<PasswordMatch, RegisterRequest> {

    @Override
    public boolean isValid(RegisterRequest value,
                           ConstraintValidatorContext context) {

        if (value == null) {
            return true;
        }

        boolean match =
            Objects.equals(value.getPassword(), value.getConfirmPassword());

        if (!match) {
            context.disableDefaultConstraintViolation();
            context.buildConstraintViolationWithTemplate(
                    "confirmPassword không khớp với password"
            )
            .addPropertyNode("confirmPassword")
            .addConstraintViolation();
        }

        return match;
    }
}
```

📌 **Điểm ăn tiền**

* `addPropertyNode()` → lỗi gắn đúng field
* Frontend bắt lỗi dễ hơn (Angular / React rất thích)

---

# 3️⃣ `@Validated` vs `@Valid` (câu hỏi phỏng vấn classic)

## 3.1 `@Valid`

* Chuẩn Jakarta
* Dùng cho:

  * `@RequestBody`
  * Entity
* **Không hỗ trợ validation group ở method-level**

```java
@PostMapping("/register")
public void register(@Valid @RequestBody RegisterRequest request) {
}
```

---

## 3.2 `@Validated` (Spring)

### Dùng khi:

* Validate ở **method param**
* Dùng **Validation Group**

```java
@RestController
@Validated
public class UserController {

    @GetMapping("/users/{id}")
    public User getUser(@Min(1) @PathVariable Long id) {
        return ...
    }
}
```

📌 Không có `@Validated` → `@Min` **không chạy**

---

## 3.3 Validation Group (real enterprise)

### Define group

```java
public interface Create {}
public interface Update {}
```

### DTO

```java
public class UserRequest {

    @NotNull(groups = Update.class)
    private Long id;

    @NotBlank(groups = {Create.class, Update.class})
    private String email;
}
```

### Controller

```java
@PostMapping
public void create(@Validated(Create.class) @RequestBody UserRequest req) {
}

@PutMapping
public void update(@Validated(Update.class) @RequestBody UserRequest req) {
}
```

---

# Tổng kết nhanh cho não đỡ overload 🧠

| Tình huống                | Cách làm                     |
| ------------------------- | ---------------------------- |
| Validate format           | `@Email`, `@Pattern`         |
| Validate DB               | Custom `ConstraintValidator` |
| Multi-field               | Class-level validator        |
| Method param              | `@Validated`                 |
| Create / Update khác rule | Validation Group             |

---

Nếu bạn muốn next level, Cancer có thể:

* Áp validator vào **Angular Reactive Form**
* So sánh **validate ở DTO vs validate ở Service**
* Chỉ ra **anti-pattern khi dùng validator trong dự án ngân hàng**

Bạn muốn đào sâu nhánh nào tiếp? 🚀

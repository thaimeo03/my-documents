Nice, đúng “combo enterprise” luôn 😄
Cancer triển khai **2 ví dụ hoàn chỉnh – sát Spring Boot thực tế**, code vừa đủ clean để bạn áp dụng ngay.

---

# I. Chain of Responsibility – Validate Order

## 1. Bài toán

Khi tạo **Order**, cần validate theo nhiều bước:

1. Order không rỗng
2. User còn hoạt động
3. Hàng còn tồn kho
4. Tổng tiền hợp lệ

👉 Mỗi rule:

* Độc lập
* Có thể bật / tắt
* Có thể đổi thứ tự

→ **Chain of Responsibility** là chân ái.

---

## 2. Thiết kế

```
OrderController
    ↓
OrderService
    ↓
OrderValidationChain
    ↓
[BasicValidation]
    ↓
[UserValidation]
    ↓
[StockValidation]
    ↓
[AmountValidation]
```

---

## 3. Định nghĩa Validation Handler

```java
public abstract class OrderValidationHandler {

    private OrderValidationHandler next;

    public OrderValidationHandler linkWith(OrderValidationHandler next) {
        this.next = next;
        return next;
    }

    public void validate(Order order) {
        doValidate(order);
        if (next != null) {
            next.validate(order);
        }
    }

    protected abstract void doValidate(Order order);
}
```

---

## 4. Các validation cụ thể

### 4.1. Basic validation

```java
@Component
public class BasicOrderValidation extends OrderValidationHandler {

    @Override
    protected void doValidate(Order order) {
        if (order == null || order.getItems().isEmpty()) {
            throw new IllegalArgumentException("Order is empty");
        }
    }
}
```

---

### 4.2. User validation

```java
@Component
public class UserOrderValidation extends OrderValidationHandler {

    @Override
    protected void doValidate(Order order) {
        if (!order.getUser().isActive()) {
            throw new IllegalStateException("User is inactive");
        }
    }
}
```

---

### 4.3. Stock validation

```java
@Component
public class StockOrderValidation extends OrderValidationHandler {

    @Override
    protected void doValidate(Order order) {
        for (OrderItem item : order.getItems()) {
            if (item.getQuantity() > item.getProduct().getStock()) {
                throw new IllegalStateException("Out of stock: " + item.getProduct().getName());
            }
        }
    }
}
```

---

### 4.4. Amount validation

```java
@Component
public class AmountOrderValidation extends OrderValidationHandler {

    @Override
    protected void doValidate(Order order) {
        if (order.getTotalAmount().compareTo(BigDecimal.ZERO) <= 0) {
            throw new IllegalStateException("Invalid order amount");
        }
    }
}
```

---

## 5. Build chain trong Spring (rất quan trọng)

```java
@Configuration
public class OrderValidationConfig {

    @Bean
    public OrderValidationHandler orderValidationChain(
            BasicOrderValidation basic,
            UserOrderValidation user,
            StockOrderValidation stock,
            AmountOrderValidation amount) {

        basic.linkWith(user)
             .linkWith(stock)
             .linkWith(amount);

        return basic; // root chain
    }
}
```

👉 Chain được control **bằng config**, không hard-code trong service.

---

## 6. Sử dụng trong Service

```java
@Service
public class OrderService {

    private final OrderValidationHandler validationChain;

    public OrderService(OrderValidationHandler validationChain) {
        this.validationChain = validationChain;
    }

    public void createOrder(Order order) {
        validationChain.validate(order);
        // save order
    }
}
```

---

## 7. Khi nào dùng Chain?

✔ Validate nhiều bước
✔ Rule thay đổi thường xuyên
✔ Có thể reuse / bật tắt từng rule

---

# II. Template Method – Xử lý Import File

## 1. Bài toán

Import nhiều loại file:

* CSV
* Excel
* XML

Luồng xử lý **giống nhau**, chỉ khác chi tiết:

1. Validate file
2. Parse data
3. Validate data
4. Save DB
5. Log / report

👉 **Template Method Pattern**

---

## 2. Abstract Template

```java
public abstract class AbstractFileImportTemplate<T> {

    public final void importFile(MultipartFile file) {
        validateFile(file);
        List<T> data = parse(file);
        validateData(data);
        save(data);
        afterImport(data);
    }

    protected abstract void validateFile(MultipartFile file);
    protected abstract List<T> parse(MultipartFile file);
    protected abstract void validateData(List<T> data);
    protected abstract void save(List<T> data);

    protected void afterImport(List<T> data) {
        // optional hook
    }
}
```

👉 `final` để **khóa flow**, không cho override bừa.

---

## 3. Import CSV

```java
@Service
public class CsvOrderImportService extends AbstractFileImportTemplate<Order> {

    @Override
    protected void validateFile(MultipartFile file) {
        if (!file.getOriginalFilename().endsWith(".csv")) {
            throw new IllegalArgumentException("Invalid CSV file");
        }
    }

    @Override
    protected List<Order> parse(MultipartFile file) {
        // parse csv
        return List.of();
    }

    @Override
    protected void validateData(List<Order> data) {
        // validate business
    }

    @Override
    protected void save(List<Order> data) {
        // save batch
    }
}
```

---

## 4. Import Excel

```java
@Service
public class ExcelOrderImportService extends AbstractFileImportTemplate<Order> {

    @Override
    protected void validateFile(MultipartFile file) {
        if (!file.getOriginalFilename().endsWith(".xlsx")) {
            throw new IllegalArgumentException("Invalid Excel file");
        }
    }

    @Override
    protected List<Order> parse(MultipartFile file) {
        // Apache POI parse
        return List.of();
    }

    @Override
    protected void validateData(List<Order> data) {
        // validate business
    }

    @Override
    protected void save(List<Order> data) {
        // save batch
    }
}
```

---

## 5. Controller chọn đúng Template

```java
@RestController
@RequestMapping("/import")
public class ImportController {

    private final Map<String, AbstractFileImportTemplate<?>> importers;

    public ImportController(List<AbstractFileImportTemplate<?>> services) {
        this.importers = services.stream()
            .collect(Collectors.toMap(
                s -> s.getClass().getSimpleName(),
                Function.identity()
            ));
    }

    @PostMapping("/order")
    public void importOrder(
        @RequestParam("type") String type,
        @RequestParam("file") MultipartFile file) {

        AbstractFileImportTemplate<?> importer = importers.get(type);
        importer.importFile(file);
    }
}
```

---

## 6. Kết hợp Pattern trong thực tế

🔥 Import enterprise thường dùng combo:

* **Template Method** → fixed flow
* **Strategy** → chọn parser
* **Chain** → validate từng dòng
* **Observer** → log / notify async

---

## 7. Tổng kết nhanh

| Pattern                 | Dùng khi                       |
| ----------------------- | ------------------------------ |
| Chain of Responsibility | Validate nhiều rule            |
| Template Method         | Luồng cố định, khác chi tiết   |
| Strategy                | Logic thay đổi thường xuyên    |
| Factory                 | Khởi tạo object theo condition |

---

Nếu bạn muốn, Cancer có thể:

* Refactor lại ví dụ theo **Clean Architecture**
* Vẽ sơ đồ sequence / class diagram
* So sánh **Chain vs Validator annotation**
* Áp dụng vào **Banking use-case thực tế**

Cứ chọn hướng bạn muốn đào sâu 👌

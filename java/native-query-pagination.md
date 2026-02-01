Ok Cancer đóng gói lại thành tài liệu markdown gọn – rõ – mang tính hướng dẫn chuẩn enterprise. Mình sẽ:

✔ trình bày lại toàn bộ code dạng tài liệu
✔ có giải thích
✔ có sequence
✔ có thêm **trường hợp xử lý JOIN** khi cần filter hoặc enrich dữ liệu từ bảng khác

---

# 📘 **TÀI LIỆU – NativeQuery + Pagination + One-to-Many Mapping trong Spring Boot**

## 🧱 1. **Mục tiêu**

Triển khai API phân trang danh sách User và trả về list Orders của từng User theo dạng:

```json
[
  {
    "id": 1,
    "username": "alice",
    "orders": [...]
  }
]
```

---

## 📚 2. **Entities**

```java
@Entity
@Table(name = "users")
public class User {
    @Id
    private Long id;
    private String username;
    private String email;
}
```

```java
@Entity
@Table(name = "orders")
public class Order {
    @Id
    private Long id;

    @Column(name = "user_id")
    private Long userId;

    private Double amount;

    @Column(name = "created_at")
    private LocalDateTime createdAt;
}
```

> **Không khai báo `@OneToMany`** để tránh pagination issue.

---

## 🧾 3. **DTO Models**

```java
public record OrderDTO(Long id, Double amount, LocalDateTime createdAt) {}

public record UserWithOrdersDTO(Long id, String username, String email, List<OrderDTO> orders) {}
```

---

## 🗂 4. **Repositories**

### 4.1 UserRepository – Native + Pageable

```java
public interface UserRepository extends JpaRepository<User, Long> {

    @Query(value = """
        SELECT u.id, u.username, u.email
        FROM users u
        ORDER BY u.id
        """,
        countQuery = "SELECT COUNT(*) FROM users",
        nativeQuery = true)
    Page<Object[]> findUsers(Pageable pageable);
}
```

---

### 4.2 OrderRepository – Load Orders theo batch

```java
public interface OrderRepository extends JpaRepository<Order, Long> {

    @Query(value = """
        SELECT o.id, o.user_id, o.amount, o.created_at
        FROM orders o
        WHERE o.user_id IN :userIds
        ORDER BY o.created_at DESC
        """,
        nativeQuery = true)
    List<Object[]> findByUserIds(@Param("userIds") List<Long> userIds);
}
```

---

## 🧩 5. **Service Layer – Mapping Logic**

```java
@Service
public class UserService {

    private final UserRepository userRepository;
    private final OrderRepository orderRepository;

    public UserService(UserRepository userRepository,
                       OrderRepository orderRepository) {
        this.userRepository = userRepository;
        this.orderRepository = orderRepository;
    }

    public Page<UserWithOrdersDTO> listUsersWithOrders(Pageable pageable) {

        Page<Object[]> userPage = userRepository.findUsers(pageable);

        List<Long> userIds = userPage.getContent()
                .stream()
                .map(row -> ((Number) row[0]).longValue())
                .toList();

        Map<Long, List<OrderDTO>> orderMap = new HashMap<>();

        if (!userIds.isEmpty()) {
            List<Object[]> orders = orderRepository.findByUserIds(userIds);

            for (Object[] row : orders) {
                Long userId = ((Number) row[1]).longValue();
                orderMap.computeIfAbsent(userId, x -> new ArrayList<>())
                        .add(new OrderDTO(
                                ((Number) row[0]).longValue(),
                                ((Number) row[2]).doubleValue(),
                                ((Timestamp) row[3]).toLocalDateTime()
                        ));
            }
        }

        List<UserWithOrdersDTO> result = userPage.getContent()
                .stream()
                .map(row -> new UserWithOrdersDTO(
                        ((Number) row[0]).longValue(),
                        (String) row[1],
                        (String) row[2],
                        orderMap.getOrDefault(
                                ((Number) row[0]).longValue(),
                                Collections.emptyList()
                        )
                ))
                .toList();

        return new PageImpl<>(result, pageable, userPage.getTotalElements());
    }
}
```

---

## 🌐 6. **REST Controller**

```java
@RestController
@RequestMapping("/users")
public class UserController {

    private final UserService userService;

    public UserController(UserService userService) {
        this.userService = userService;
    }

    @GetMapping
    public Page<UserWithOrdersDTO> list(Pageable pageable) {
        return userService.listUsersWithOrders(pageable);
    }
}
```

---

# ➕ **7. Trường hợp mở rộng – JOIN enrich dữ liệu**

Giả sử có thêm bảng:

```sql
order_status(id, code, description)
```

và cần trả về:

```json
{
  "id": 10,
  "amount": 200.0,
  "status": "COMPLETED"
}
```

### 7.1 Update OrderRepository:

```java
@Query(value = """
    SELECT o.id, o.user_id, o.amount, o.created_at, s.code AS status
    FROM orders o
    JOIN order_status s ON s.id = o.status_id
    WHERE o.user_id IN :userIds
    ORDER BY o.created_at DESC
    """,
    nativeQuery = true)
List<Object[]> findOrdersWithStatus(@Param("userIds") List<Long> userIds);
```

### 7.2 Update DTO:

```java
public record OrderDTO(Long id, Double amount, LocalDateTime createdAt, String status) {}
```

### 7.3 Mapping:

```java
new OrderDTO(
    ((Number) row[0]).longValue(),
    ((Number) row[2]).doubleValue(),
    ((Timestamp) row[3]).toLocalDateTime(),
    (String) row[4]
)
```

---

# ⚙ **8. Ưu điểm kiến trúc này**

| Ưu điểm             | Giải thích                    |
| ------------------- | ----------------------------- |
| Predictable query   | Không bị Hibernate magic      |
| Tránh N+1           | Chỉ 2 queries                 |
| Index friendly      | Đẩy work về DB                |
| Không duplicate row | Do không fetch collection     |
| Tách logic clean    | Repository không ô nhiễm      |
| Dễ mở rộng          | JOIN / filter / sort / search |

---

# 📉 **9. Khi nào cần cân nhắc hiệu suất?**

Chỉ khi:

* Orders quá lớn (100k+ / user)
* Không giới hạn filter orders

Lúc đó nên:

✔ paginate nested Orders
hoặc
✔ chuyển sang summary mode (aggregation)

---

# 📦 **10. Kết luận**

Pattern:

> **Pagination Users → Batch fetch Orders → Group tại Java → DTO**

là pattern phổ biến, đã được dùng trong các hệ thống banking & e-commerce lớn vì:

* hiệu suất tốt
* dễ maintain
* dễ port sang Oracle / PG / MySQL

---

Nếu bạn muốn Cancer bổ sung thêm phần:

* filter + sort + search dynamic
* version cho Oracle (vì Oracle syntax hơi khác)
* version JSON aggregate (Postgres only)
* version QueryDSL
* version Spring Data Projection

chỉ cần nói:

> "Bổ sung filter" hoặc "Oracle version" hoặc "JSON Postgres" 😎

Ok, Cancer triển khai **chuẩn enterprise** cho bạn:
👉 **Spring Security (Session-based) + Spring Session + Redis**
Phong cách đúng core banking / internal system, scale ngang vô tư.

---

# 1. Kiến trúc tổng thể

```
Client (Browser)
   ↓ Cookie: JSESSIONID
Spring Security
   ↓ HttpSession
Spring Session
   ↓
Redis (store session)
```

✔ Session **không nằm trong JVM**
✔ Restart app không logout
✔ Scale nhiều instance ok

---

# 2. Dependencies (Spring Boot 3.x)

### Maven

```xml
<!-- Spring Security -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>

<!-- Spring Session + Redis -->
<dependency>
    <groupId>org.springframework.session</groupId>
    <artifactId>spring-session-data-redis</artifactId>
</dependency>

<!-- Redis client -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>

<!-- JPA -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
```

---

# 3. Redis Config

### application.yml

```yaml
spring:
  data:
    redis:
      host: localhost
      port: 6379

  session:
    store-type: redis
    timeout: 30m

server:
  servlet:
    session:
      cookie:
        name: JSESSIONID
        http-only: true
        same-site: lax
        secure: false
```

📌 Chỉ cần thế → Spring auto lưu session vào Redis.

---

# 4. Entity & Repository

```java
@Entity
@Table(name = "users")
public class User {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(unique = true)
    private String username;

    private String password;

    private String role;
}
```

```java
public interface UserRepository extends JpaRepository<User, Long> {
    Optional<User> findByUsername(String username);
}
```

---

# 5. UserDetailsService (Core của Spring Security)

```java
@Service
public class CustomUserDetailsService implements UserDetailsService {

    private final UserRepository userRepository;

    public CustomUserDetailsService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    @Override
    public UserDetails loadUserByUsername(String username)
            throws UsernameNotFoundException {

        User user = userRepository.findByUsername(username)
                .orElseThrow(() -> new UsernameNotFoundException("User not found"));

        return org.springframework.security.core.userdetails.User
                .withUsername(user.getUsername())
                .password(user.getPassword())
                .roles(user.getRole())
                .build();
    }
}
```

---

# 6. Password Encoder

```java
@Configuration
public class PasswordConfig {

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
}
```

---

# 7. Spring Security Config (Session-based)

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {

        http
            .csrf(csrf -> csrf.disable())
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/auth/**").permitAll()
                .anyRequest().authenticated()
            )
            .formLogin(form -> form.disable()) // dùng REST login
            .sessionManagement(session -> session
                .sessionCreationPolicy(SessionCreationPolicy.IF_REQUIRED)
                .maximumSessions(1) // 1 user = 1 session
            );

        return http.build();
    }
}
```

📌 Đây là **Session-based**, KHÔNG JWT.

---

# 8. REST Login Controller (không dùng formLogin)

### Login Request

```java
public record LoginRequest(
    String username,
    String password
) {}
```

### Controller

```java
@RestController
@RequestMapping("/auth")
public class AuthController {

    private final AuthenticationManager authenticationManager;

    public AuthController(AuthenticationManager authenticationManager) {
        this.authenticationManager = authenticationManager;
    }

    @PostMapping("/login")
    public ResponseEntity<?> login(
            @RequestBody LoginRequest request,
            HttpServletRequest httpRequest
    ) {
        Authentication authentication = authenticationManager.authenticate(
                new UsernamePasswordAuthenticationToken(
                        request.username(),
                        request.password()
                )
        );

        SecurityContext context = SecurityContextHolder.createEmptyContext();
        context.setAuthentication(authentication);

        HttpSession session = httpRequest.getSession(true);
        session.setAttribute(
            HttpSessionSecurityContextRepository.SPRING_SECURITY_CONTEXT_KEY,
            context
        );

        return ResponseEntity.ok("Login success");
    }

    @PostMapping("/logout")
    public ResponseEntity<?> logout(HttpServletRequest request) {
        request.getSession().invalidate();
        return ResponseEntity.ok("Logout success");
    }
}
```

📌 Sau login:

* Spring Security lưu `SecurityContext`
* Spring Session lưu session vào Redis
* Cookie `JSESSIONID` trả về browser

---

# 9. API Protected

```java
@RestController
@RequestMapping("/profile")
public class ProfileController {

    @GetMapping
    public ResponseEntity<?> profile(Authentication authentication) {
        return ResponseEntity.ok(
            "Hello " + authentication.getName()
        );
    }
}
```

✔ Không cần check session thủ công
✔ Spring Security lo hết

---

# 10. Redis Session Structure (hiểu để debug)

Trong Redis:

```
spring:session:sessions:<sessionId>
  ├── sessionAttr:SPRING_SECURITY_CONTEXT
  ├── sessionAttr:creationTime
  ├── sessionAttr:lastAccessedTime
```

📌 Debug cực dễ khi production issue.

---

# 11. Docker Redis (dev nhanh)

```yaml
version: "3.8"
services:
  redis:
    image: redis:7
    ports:
      - "6379:6379"
```

---

# 12. Best Practice (chuẩn enterprise)

✅ NÊN

* Session + Redis cho web nội bộ
* `maximumSessions(1)` tránh login trùng
* Timeout ngắn (15–30m)
* Monitor Redis memory

❌ KHÔNG NÊN

* Session cho mobile app
* Disable CSRF nếu public web (ở đây REST internal ok)

---

# 13. Khi nào chọn combo này?

✔ Core Banking
✔ Internal Portal
✔ Admin System
✔ Compliance cao (revoke session ngay)

---

Nếu bạn muốn tiếp:

* 🔥 **CSRF chuẩn khi dùng cookie**
* 🔥 **Concurrent login handling**
* 🔥 **Remember-me bằng session**
* 🔥 **So sánh Session-Security vs JWT-Security**
* 🔥 **Diagram luồng auth để làm tài liệu**

Cứ nói, Cancer triển tiếp đúng thứ bạn cần 💼🚀

Chuẩn câu hỏi **đi vào “chất lượng hệ thống”** luôn 👌
Handle **message lỗi authentication** đúng cách giúp:

* UX rõ ràng
* Không leak security info
* Dễ log, dễ audit

Cancer chia ra **đúng 4 lớp cần handle** trong **Spring Security (session-based)**.

---

# 1. Phân biệt 2 loại lỗi auth (rất quan trọng)

| Loại               | Khi nào xảy ra                | HTTP               |
| ------------------ | ----------------------------- | ------------------ |
| **Authentication** | Chưa login / login sai        | `401 Unauthorized` |
| **Authorization**  | Đã login nhưng không đủ quyền | `403 Forbidden`    |

👉 Mỗi loại **handle khác nhau**, đừng gộp.

---

# 2. Handle lỗi **CHƯA LOGIN / SESSION HẾT HẠN**

👉 `AuthenticationEntryPoint`

### Custom AuthenticationEntryPoint

```java
@Component
public class RestAuthenticationEntryPoint implements AuthenticationEntryPoint {

    @Override
    public void commence(
            HttpServletRequest request,
            HttpServletResponse response,
            AuthenticationException authException
    ) throws IOException {

        response.setStatus(HttpServletResponse.SC_UNAUTHORIZED);
        response.setContentType("application/json");

        response.getWriter().write("""
        {
          "code": "AUTH_001",
          "message": "Authentication required or session expired"
        }
        """);
    }
}
```

📌 Dùng cho:

* Session timeout
* Chưa login
* Cookie không hợp lệ

---

# 3. Handle lỗi **KHÔNG ĐỦ QUYỀN**

👉 `AccessDeniedHandler`

```java
@Component
public class RestAccessDeniedHandler implements AccessDeniedHandler {

    @Override
    public void handle(
            HttpServletRequest request,
            HttpServletResponse response,
            AccessDeniedException accessDeniedException
    ) throws IOException {

        response.setStatus(HttpServletResponse.SC_FORBIDDEN);
        response.setContentType("application/json");

        response.getWriter().write("""
        {
          "code": "AUTH_002",
          "message": "Access denied"
        }
        """);
    }
}
```

---

# 4. Register vào Spring Security

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    public SecurityFilterChain filterChain(
            HttpSecurity http,
            RestAuthenticationEntryPoint authenticationEntryPoint,
            RestAccessDeniedHandler accessDeniedHandler
    ) throws Exception {

        http
            .csrf(csrf -> csrf.disable())
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/auth/**").permitAll()
                .anyRequest().authenticated()
            )
            .exceptionHandling(ex -> ex
                .authenticationEntryPoint(authenticationEntryPoint)
                .accessDeniedHandler(accessDeniedHandler)
            )
            .sessionManagement(session -> session
                .sessionFixation().migrateSession()
            );

        return http.build();
    }
}
```

✔ Từ đây:

* 401 → message custom
* 403 → message custom

---

# 5. Handle lỗi **LOGIN SAI (email/password)**

👉 Lỗi này xảy ra **TRONG quá trình authenticate**

### Controller login (best practice)

```java
@PostMapping("/login")
public ResponseEntity<?> login(
        @RequestBody LoginRequest request,
        HttpServletRequest httpRequest
) {
    try {
        Authentication authentication =
            authenticationManager.authenticate(
                new UsernamePasswordAuthenticationToken(
                    request.email(),
                    request.password()
                )
            );

        SecurityContext context = SecurityContextHolder.createEmptyContext();
        context.setAuthentication(authentication);

        HttpSession session = httpRequest.getSession(true);
        session.setAttribute(
            HttpSessionSecurityContextRepository.SPRING_SECURITY_CONTEXT_KEY,
            context
        );

        return ResponseEntity.ok("Login success");

    } catch (BadCredentialsException ex) {
        return ResponseEntity
                .status(HttpStatus.UNAUTHORIZED)
                .body(Map.of(
                    "code", "AUTH_003",
                    "message", "Invalid credentials"
                ));
    }
}
```

❗ **Không bao giờ trả**:

* “Email not found”
* “Password wrong”

👉 Tránh user enumeration.

---

# 6. Handle **SESSION TIMEOUT** (rất hay bị bỏ sót)

### application.yml

```yaml
spring:
  session:
    timeout: 15m
```

Khi session expire:

* Request sau → **401**
* Đi vào `AuthenticationEntryPoint`

✔ Không cần code thêm

---

# 7. Chuẩn format error response (enterprise-style)

👉 NÊN thống nhất:

```json
{
  "timestamp": "2026-02-03T10:15:30",
  "code": "AUTH_001",
  "message": "Authentication required",
  "path": "/api/profile"
}
```

📌 Frontend xử lý cực dễ:

* 401 → redirect login
* 403 → show permission dialog

---

# 8. Mapping lỗi auth → UX (rất thực tế)

| Code     | FE xử lý                      |
| -------- | ----------------------------- |
| AUTH_001 | Redirect login                |
| AUTH_002 | Show “no permission”          |
| AUTH_003 | Show “email/password invalid” |

---

# 9. Anti-pattern cần tránh ❌

❌ Throw RuntimeException trong controller
❌ Global `@ControllerAdvice` bắt auth lỗi
❌ Trả message quá chi tiết
❌ HTTP 200 nhưng body báo lỗi

---

# 10. Cancer’s final take 🧠

> **Authentication error handling là contract giữa BE & FE**
>
> Không phải chỉ để “bắt lỗi”, mà để **kiểm soát hành vi hệ thống**

Nếu làm đúng:

* Security tăng
* UX rõ
* Debug prod dễ

---

Nếu bạn muốn, mình có thể:

* 🔐 Handle **CSRF error riêng**
* 🔄 Custom logout success message
* 🧪 Test auth error bằng MockMvc
* 🧠 Design error code chuẩn cho ngân hàng

Bạn muốn đào sâu phần nào tiếp?


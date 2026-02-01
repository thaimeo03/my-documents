Alright bro — đây là cheat sheet Mockito trong Spring Boot gọn – đủ – đúng bài cho dev backend test API, service, repository. Cancer soạn kiểu dev cho dev, không hàn lâm lý thuyết thừa.

---

# 🧩 **1. Các Annotation Quan trọng**

| Annotation     | Ý nghĩa                                                                                  |
| -------------- | ---------------------------------------------------------------------------------------- |
| `@Mock`        | Tạo mock object                                                                          |
| `@InjectMocks` | Inject mock vào class cần test (constructor hoặc setter)                                 |
| `@Spy`         | Một phần real — một phần mock                                                            |
| `@Captor`      | Capture argument của mock                                                                |
| `@MockBean`    | Spring Boot mock bean trong ApplicationContext (thường dùng khi test controller/service) |

---

# 🧱 **2. Setup Class Test**

```java
@ExtendWith(MockitoExtension.class)
class UserServiceTest {
}
```

Nếu chạy với Spring context:

```java
@SpringBootTest
@AutoConfigureMockMvc
class UserControllerTest {
}
```

---

# 🎯 **3. Các Kiểu Fake | Mock | Spy | Stub**

* **Mock** → hành vi do mình định nghĩa (`when().thenReturn()`)
* **Spy** → real + override 1 phần
* **Stub** → trả về cố định, ít logic

---

# 🧪 **4. Khi nào dùng `@Mock` vs `@MockBean`**

| Case                                   | Dùng gì                  |
| -------------------------------------- | ------------------------ |
| Test service logic thuần               | `@Mock`                  |
| Test controller & muốn DI mock service | `@MockBean`              |
| Muốn skip repository                   | `@Mock` hoặc `@MockBean` |

Example:

```java
@MockBean
private UserService userService; // Controller test
```

---

# 📌 **5. Basic Stubbing**

### Trả về giá trị:

```java
when(userRepository.findById(1L)).thenReturn(Optional.of(user));
```

### Throw exception:

```java
when(userService.getUser(1L)).thenThrow(new NotFoundException());
```

### Void method:

```java
doNothing().when(emailService).sendEmail(any());
```

---

# 🕵️ **6. Verify hành vi**

```java
verify(userRepository, times(1)).save(any());
verify(emailService, never()).sendEmail(any());
verify(tokenService, atLeastOnce()).generate();
```

Delay với timeout:

```java
verify(cacheService, timeout(200)).put(any(), any());
```

---

# 🎣 **7. ArgumentMatcher & ArgumentCaptor**

### Match:

```java
when(userRepository.findByEmail(eq("abc@gmail.com"))).thenReturn(user);
when(orderService.create(any(), anyInt())).thenReturn(order);
```

### Capture:

```java
@Captor
ArgumentCaptor<User> captor;

verify(userRepository).save(captor.capture());
assertEquals("Thai", captor.getValue().getName());
```

---

# 🧬 **8. Using Spy**

Example spy List:

```java
List<String> list = spy(new ArrayList<>());
doReturn(3).when(list).size();
```

---

# 🧱 **9. Mocking với `@InjectMocks`**

```java
@Mock private UserRepository userRepository;
@InjectMocks private UserService userService;
```

Spring sẽ inject vào constructor hoặc setter.

---

# 🔁 **10. Mocking Chain Method**

Ví dụ repository có chain:

```java
when(query.getResultList()).thenReturn(result);
```

Hoặc service:

```java
when(userService.getById(1L).getName()).thenReturn("Thai"); // anti pattern — tránh chain kiểu này
```

Nên mock từng step:

---

# 💡 **11. Mocking Static (Mockito 3.4+)**

```java
try (MockedStatic<UUID> mocked = mockStatic(UUID.class)) {
    mocked.when(UUID::randomUUID).thenReturn(UUID.fromString("0000..."));
}
```

---

# 🧰 **12. Mocking RestTemplate / WebClient**

### RestTemplate

```java
when(restTemplate.getForObject(anyString(), eq(String.class)))
    .thenReturn("OK");
```

### WebClient

Dùng `WebClient.Builder` + `MockWebServer` hoặc `@WebFluxTest`

---

# 🎮 **13. Test Controller với MockMvc + MockBean**

```java
@WebMvcTest(UserController.class)
class UserControllerTest {

    @Autowired private MockMvc mockMvc;

    @MockBean private UserService userService;

    @Test
    void shouldReturnUser() throws Exception {
        when(userService.getUser(1L)).thenReturn(userDto);

        mockMvc.perform(get("/users/1"))
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.name").value("Thai"));
    }
}
```

---

# 🧩 **14. Real Test Sample — Service**

```java
@ExtendWith(MockitoExtension.class)
class UserServiceTest {

    @Mock private UserRepository userRepository;
    @Mock private EmailService emailService;
    @InjectMocks private UserService userService;

    @Test
    void createUserShouldSendEmail() {
        User user = new User("Thai");

        when(userRepository.save(any())).thenReturn(user);

        userService.createUser(user);

        verify(emailService).sendEmail(user);
    }
}
```

---

# 🧠 **15. Best Practices dành cho Spring Boot**

✔ Tập trung test logic → không spin context nếu không cần
✔ Tránh mock entity → mock repo/service là đủ
✔ Mỗi test nên test 1 case
✔ Tránh verify quá nhiều → noise
✔ Đặt tên test kiểu BDD:

```
shouldReturnUser_WhenExists()
shouldThrowException_WhenNotFound()
```

---

# 📦 Bonus: Tools hay đi kèm

* `@DataJpaTest` → test repository
* `MockMvc` → controller test
* `Testcontainers` → real DB integration
* `AssertJ` → assertion dẻo
* `WireMock` → mock dịch vụ ngoài

---

Nếu muốn Cancer build cho bạn:

✅ full template test project
hoặc
✅ template test cho service + controller + repository
hoặc
✅ thêm phần Mockito + Spring Boot + Testcontainers

Bạn muốn tiếp ở hướng nào?

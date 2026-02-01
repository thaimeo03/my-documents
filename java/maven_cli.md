Chắc chắn rồi Cancer 😎
Dưới đây là **tổng hợp các câu lệnh Maven CLI cơ bản và cực kỳ hữu dụng** khi làm việc với Spring Boot hoặc Java project nói chung.

---
## 🔧 Câu lệnh Maven cơ bản

| Lệnh          | Mục đích                                                            |
| ------------- | ------------------------------------------------------------------- |
| `mvn clean`   | Xoá thư mục `target/` (build cũ)                                    |
| `mvn compile` | Biên dịch mã nguồn Java                                             |
| `mvn test`    | Chạy toàn bộ test                                                   |
| `mvn package` | Build project ra file `.jar` hoặc `.war` trong `target/`            |
| `mvn install` | Build và install vào local repository (\~ `~/.m2/repository`)       |
| `mvn verify`  | Kiểm tra toàn bộ phase: compile → test → package → integration test |

---

## 🚀 Câu lệnh chạy Spring Boot App

Nếu project dùng plugin `spring-boot-maven-plugin`, bạn có thể chạy app trực tiếp bằng:

```bash
mvn spring-boot:run
```

### ✅ Lợi ích:

* Không cần build `.jar`
* Tự động detect thay đổi

---

## 🧪 Một số lệnh hữu ích khác

### 🔍 Kiểm tra dependency:

```bash
mvn dependency:tree
```

> Hiển thị cây dependency, rất hữu ích để debug xung đột version.

---

### 🧼 Làm sạch & build từ đầu:

```bash
mvn clean install
```

Nếu có proxy, cấu hình file settings.xml (Tại root project để chỉ định cấu hình):
```xml
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0
                              https://maven.apache.org/xsd/settings-1.0.0.xsd">
  <proxies>
    <proxy>
      <id>corp-proxy</id>
      <active>true</active>
      <protocol>http</protocol>
      <host>10.53.120.9</host>
      <port>8080</port>
      <nonProxyHosts>www.google.com|*.example.com</nonProxyHosts>
    </proxy>
  </proxies>
</settings>
```

Rồi chạy lệnh
```bash
mvn clean install --settings settings.xml
```

> Câu lệnh "đa năng" thường dùng nhất.

---

### ⚙️ Chạy với profile khác:

```bash
mvn spring-boot:run -Dspring-boot.run.profiles=dev
```

---

### 🧱 Build nhưng không test:

```bash
mvn install -DskipTests
```

---

## 📁 Maven Lifecycle (hiểu rõ giúp bạn debug tốt hơn)

```txt
validate → compile → test → package → verify → install → deploy
```

* `validate`: kiểm tra project hợp lệ
* `compile`: biên dịch
* `test`: chạy unit test
* `package`: tạo JAR/WAR
* `verify`: xác thực kết quả
* `install`: copy JAR vào local repo
* `deploy`: deploy đến remote repo (dùng CI/CD)

---

## 💡 Tips dành cho dev Spring Boot

* Luôn `clean` trước khi `install` nếu project có vấn đề về build
* Dùng `spring-boot:run` trong môi trường dev cho tiện debug
* Dùng `mvn dependency:tree` khi gặp lỗi xung đột thư viện
* Với multi-module project: dùng `-pl module-name` để build 1 module riêng

---

Nếu bạn đang dùng Maven với Spring Boot và muốn mình tạo file `pom.xml` chuẩn cấu hình luôn cho dự án (có Lombok, Web, JPA, MySQL...), mình gửi template sẵn dùng luôn nha 💪

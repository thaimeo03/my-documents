Ok Thai, mình đi theo **chuẩn enterprise + Docker-friendly + multi-profile**, dùng được lâu dài trong môi trường bank / core system. Giữ gọn nhưng đúng bài 👌

---

# 1. Nguyên tắc thiết kế (rất quan trọng)

**Chuẩn cần nhớ:**

1. ❌ Không hard-code config theo environment
2. ❌ Không phụ thuộc file nằm trong image
3. ✅ Config theo **profile**
4. ✅ Secrets lấy từ **Environment Variables**
5. ✅ Docker image **giống nhau cho mọi môi trường**
6. ✅ Runtime quyết định profile & config

(12-Factor App mindset)

---

# 2. Cấu trúc config khuyến nghị

```
src/main/resources/
├── application.yaml              # Common config
├── application-dev.yaml          # Dev
├── application-staging.yaml      # Staging
├── application-prod.yaml         # Production
```

❌ **Không** dùng:

```
.env.dev
.env.prod
```

---

# 3. application.yaml (base – dùng chung)

```yaml
spring:
  application:
    name: user-service

  profiles:
    active: ${SPRING_PROFILES_ACTIVE:dev}

server:
  port: 8080

logging:
  level:
    root: INFO
```

📌 Giải thích:

* Mặc định `dev`
* Docker / k8s chỉ cần set env là override được

---

# 4. application-dev.yaml

```yaml
spring:
  datasource:
    url: ${DB_URL:jdbc:postgresql://localhost:5432/dev_db}
    username: ${DB_USER:dev_user}
    password: ${DB_PASSWORD:dev_pass}

  jpa:
    hibernate:
      ddl-auto: update
    show-sql: true
```

📌 Dev:

* Có default value
* Chạy local không cần export env

---

# 5. application-staging.yaml

```yaml
spring:
  datasource:
    url: ${DB_URL}
    username: ${DB_USER}
    password: ${DB_PASSWORD}

  jpa:
    hibernate:
      ddl-auto: validate
    show-sql: false
```

📌 Staging:

* Bắt buộc truyền env
* Gần production

---

# 6. application-prod.yaml (chuẩn bank-grade)

```yaml
spring:
  datasource:
    url: ${DB_URL}
    username: ${DB_USER}
    password: ${DB_PASSWORD}

  jpa:
    hibernate:
      ddl-auto: none
    open-in-view: false

logging:
  level:
    root: WARN
```

📌 Production:

* ❌ Không auto schema
* ❌ Không log SQL

---

# 7. Dockerfile (chuẩn, không đụng config)

```dockerfile
FROM eclipse-temurin:21-jdk-jammy

ARG JAR_FILE=target/*.jar
COPY ${JAR_FILE} app.jar

ENTRYPOINT ["java", "-jar", "/app.jar"]
```

✔️ Một image chạy mọi môi trường

---

# 8. Chạy container theo từng môi trường

### Dev

```bash
docker run \
  -e SPRING_PROFILES_ACTIVE=dev \
  your-image
```

### Staging

```bash
docker run \
  -e SPRING_PROFILES_ACTIVE=staging \
  -e DB_URL=jdbc:postgresql://stg-db:5432/app \
  -e DB_USER=app_user \
  -e DB_PASSWORD=secret \
  your-image
```

### Production

```bash
docker run \
  -e SPRING_PROFILES_ACTIVE=prod \
  -e DB_URL=jdbc:oracle:thin:@//prod-db:1521/ORCL \
  -e DB_USER=core_user \
  -e DB_PASSWORD=*** \
  your-image
```

---

# 9. docker-compose.yml (dev/staging)

```yaml
version: "3.9"

services:
  app:
    image: user-service
    ports:
      - "8080:8080"
    environment:
      SPRING_PROFILES_ACTIVE: dev
      DB_URL: jdbc:postgresql://db:5432/dev_db
      DB_USER: dev_user
      DB_PASSWORD: dev_pass

  db:
    image: postgres:16
    environment:
      POSTGRES_DB: dev_db
      POSTGRES_USER: dev_user
      POSTGRES_PASSWORD: dev_pass
```

---

# 10. Những thứ **TUYỆT ĐỐI KHÔNG** nên làm

❌ Copy file `.env` vào image
❌ Dùng `spring.config.import=file:.env.xxx` trong production
❌ Hardcode password trong YAML
❌ Build nhiều image cho nhiều môi trường

---

# 11. Bonus: config nâng cao (khi project to)

```
config/
├── common/
│   └── datasource.yaml
├── dev/
│   └── datasource.yaml
├── prod/
│   └── datasource.yaml
```

Dùng khi:

* Microservices
* Multi-module
* External config server (Spring Cloud Config)

---

## Tổng kết nhanh

👉 **Profile quyết định hành vi**
👉 **Environment variables quyết định data**
👉 **Docker image không chứa config**

Nếu bạn muốn, Cancer có thể:

* Refactor lại config hiện tại của bạn theo chuẩn này
* Gợi ý cấu trúc cho **multi-module Spring Boot**
* Map sang **K8s ConfigMap + Secret**
* Hoặc giải thích sâu `spring.config.import` & thứ tự load config

Cứ nói tiếp, mình đi tới cùng 🤝

1. Core & Web (gần như bắt buộc)

| Thư viện                         | Use case              | Ghi chú                        |
| -------------------------------- | --------------------- | ------------------------------ |
| `spring-boot-starter-web`        | REST API              | MVC, Jackson, validation       |
| `spring-boot-starter-validation` | Validate request      | `@NotNull`, `@Size`, `@Email`  |
| `spring-boot-starter-actuator`   | Health check, metrics | Bắt buộc khi deploy production |

2. Data & Persistence (rất quan trọng)

| Thư viện                           | Use case     | Ghi chú                |
| ---------------------------------- | ------------ | ---------------------- |
| `spring-boot-starter-data-jpa`     | ORM          | Hibernate + JPA        |
| `postgresql` / `mysql-connector-j` | JDBC driver  | Tùy DB                 |
| `spring-boot-starter-jdbc`         | SQL thuần    | Dùng `JdbcTemplate`    |
| `flyway-core`                      | DB migration | Rất khuyến nghị        |
| `liquibase-core`                   | DB migration | Alternative của Flyway |

3. Security & Auth (enterprise standard)

| Thư viện                                 | Use case       | Ghi chú                     |
| ---------------------------------------- | -------------- | --------------------------- |
| `spring-boot-starter-security`           | Authentication | Base security               |
| `spring-security-oauth2-resource-server` | JWT            | API secured                 |
| `spring-security-oauth2-client`          | Login OAuth2   | Keycloak, Google            |
| `jjwt` / `nimbus-jose-jwt`               | JWT utils      | Thường dùng cho custom auth |

4. Logging & Observability

| Thư viện                         | Use case       | Ghi chú          |
| -------------------------------- | -------------- | ---------------- |
| `spring-boot-starter-logging`    | Logging        | Logback mặc định |
| `logstash-logback-encoder`       | JSON log       | ELK stack        |
| `micrometer-registry-prometheus` | Metrics        | Monitoring       |
| `spring-boot-starter-aop`        | Logging, audit | Aspect-based     |

5. API Documentation & Contract

| Thư viện                               | Use case | Ghi chú             |
| -------------------------------------- | -------- | ------------------- |
| `springdoc-openapi-starter-webmvc-ui`  | Swagger  | Chuẩn thay Swagger2 |
| `springdoc-openapi-starter-webmvc-api` | OpenAPI  | Generate spec       |

6. Mapper & Boilerplate killer

| Thư viện                  | Use case         | Ghi chú          |
| ------------------------- | ---------------- | ---------------- |
| `mapstruct`               | DTO ↔ Entity     | Nhanh, type-safe |
| `lombok`                  | Giảm boilerplate | Getter/Setter    |
| `jackson-datatype-jsr310` | DateTime         | Java Time API    |

7. Testing

| Thư viện                   | Use case              | Ghi chú       |
| -------------------------- | --------------------- | ------------- |
| `spring-boot-starter-test` | Unit/Integration test | JUnit 5       |
| `testcontainers`           | Test với DB thật      | Rất mạnh      |
| `mockito-core`             | Mock                  | Test service  |
| `rest-assured`             | Test API              | Clean, dễ đọc |

8. Async, Messaging, Cache

| Thư viện                         | Use case | Ghi chú           |
| -------------------------------- | -------- | ----------------- |
| `spring-boot-starter-cache`      | Cache    | Redis, Caffeine   |
| `spring-boot-starter-data-redis` | Redis    | Session, cache    |
| `spring-kafka`                   | Kafka    | Event-driven      |
| `spring-boot-starter-amqp`       | RabbitMQ | Queue             |
| `spring-boot-starter-webflux`    | Reactive | Khi cần scale lớn |

9. Utility & Productivity

| Thư viện                      | Use case         | Ghi chú              |
| ----------------------------- | ---------------- | -------------------- |
| `apache-commons-lang3`        | Utils            | String, Date, Object |
| `apache-commons-collections4` | Collection utils | Optional             |
| `hibernate-types-60`          | JSONB, ARRAY     | PostgreSQL           |
| `p6spy`                       | Log SQL          | Debug query          |

10. Starter stack khuyến nghị
- spring-boot-starter-web
- spring-boot-starter-data-jpa
- spring-boot-starter-security
- spring-boot-starter-validation
- flyway-core
- postgresql
- lombok
- mapstruct
- springdoc-openapi
- spring-boot-starter-test

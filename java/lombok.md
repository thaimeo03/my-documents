| Annotation                 | Tác dụng                                                                                    |
| -------------------------- | ------------------------------------------------------------------------------------------- |
| `@Getter` / `@Setter`      | Tạo getter/setter cho fields                                                                |
| `@ToString`                | Tạo phương thức `toString()`                                                                |
| `@NoArgsConstructor`       | Constructor không có tham số                                                                |
| `@AllArgsConstructor`      | Constructor với tất cả field                                                                |
| `@RequiredArgsConstructor` | Constructor cho các final field                                                             |
| `@Data`                    | Bao gồm `@Getter`, `@Setter`, `@ToString`, `@EqualsAndHashCode`, `@RequiredArgsConstructor` |
| `@Builder`                 | Tạo builder pattern (`User.builder().name("A").age(20).build()`)                            |
| `@Value`                   | Giống `@Data` nhưng immutable (dùng cho DTO)                                                |
| `@Slf4j`                   | Inject sẵn logger `log.info(...)`                                                           |

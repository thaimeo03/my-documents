| Use case                       | Java type        | DB column                  | Mô tả ngắn gọn                                     |
| ------------------------------ | ---------------- | -------------------------- | -------------------------------------------------- |
| Ngày sinh, ngày hiệu lực       | `LocalDate`      | `DATE`                     | Chỉ lưu ngày, không có giờ, không timezone         |
| Giờ trong ngày                 | `LocalTime`      | `TIME`                     | Chỉ lưu giờ/phút/giây, không gắn với ngày          |
| Thời điểm tạo/cập nhật dữ liệu | `LocalDateTime`  | `TIMESTAMP`                | Ngày + giờ, **không timezone**, dùng phổ biến nhất |
| Audit log, event time          | `Instant`        | `TIMESTAMP`                | Thời điểm tuyệt đối (UTC), an toàn cho hệ phân tán |
| API trả thời gian theo vùng    | `OffsetDateTime` | `TIMESTAMP WITH TIME ZONE` | Ngày giờ kèm offset (+07:00, +00:00)               |
| Booking, lịch đa quốc gia      | `ZonedDateTime`  | `TIMESTAMP WITH TIME ZONE` | Ngày giờ + timezone đầy đủ                         |
| Hệ thống cũ (legacy)           | `Timestamp`      | `TIMESTAMP`                | API cũ, chỉ dùng khi bắt buộc                      |
| Code cũ rất lâu đời            | `Date`           | `TIMESTAMP`                | Tránh dùng trong dự án mới                         |

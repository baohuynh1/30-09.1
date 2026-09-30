# identity-service

Identity Service ("Phòng quản lý nhân khẩu") — lưu thông tin đăng nhập người dùng. Spring Boot 3.5, Java 21, Gradle, PostgreSQL.

## Chạy nhanh

```bash
# 1. Khởi động PostgreSQL (tự tạo database identity_db)
docker compose up -d

# Hoặc nếu đã có PostgreSQL sẵn:
# psql -U postgres -c "CREATE DATABASE identity_db;"

# 2. Chạy service (port 8081)
./gradlew bootRun
```

Cấu hình DB trong `src/main/resources/application.yml` (override bằng biến môi trường `DB_USERNAME`, `DB_PASSWORD`).

## API

### POST /api/auth/register

```bash
curl -X POST http://localhost:8081/api/auth/register \
  -H "Content-Type: application/json" \
  -d '{"username":"bao","password":"123456"}'
```

- `201 Created` → `{"id":"<uuid>","username":"bao","role":"USER"}`
- `409 Conflict` → username đã tồn tại
- `400 Bad Request` → dữ liệu không hợp lệ

Kiểm tra mật khẩu đã được băm:

```sql
SELECT id, username, password, role FROM users;
-- password dạng $2a$10$...
```

## Cấu trúc

```
src/main/java/com/example/identityservice
├── IdentityServiceApplication.java
├── config/SecurityConfig.java        # SecurityFilterChain, csrf off, permitAll register, BCrypt bean
├── controller/AuthController.java    # POST /api/auth/register
├── dto/RegisterRequest.java, UserResponse.java
├── entity/User.java, Role.java       # id UUID, username, password, role
├── exception/...                     # 409 / 400 handlers
├── repository/UserRepository.java
└── service/AuthService.java          # băm BCrypt rồi lưu DB
```

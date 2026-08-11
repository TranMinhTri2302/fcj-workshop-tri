---
title : "Kiểm thử API"
date : 2024-01-01
weight : 1
chapter : false
pre : " <b> 5.10.1. </b> "
---

#### 5.10.1. Kiểm thử API qua Swagger UI và Postman

Bước kiểm thử đầu tiên là xác nhận rằng API Gateway đã kết nối đúng với Lambda và có thể nhận/xử lý request. Chúng ta sẽ dùng **Swagger UI** (giao diện tự sinh từ FastAPI) và **Postman**.

---

**Bước 1: Kiểm tra Swagger UI**

1. Lấy **Invoke URL** của API Gateway (đã lưu ở bước 5.5.3), ví dụ:
   `https://abc123.execute-api.ap-southeast-1.amazonaws.com/`
2. Mở trình duyệt, dán URL và thêm `/docs` vào cuối:
   `https://abc123.execute-api.ap-southeast-1.amazonaws.com/docs`
3. Nếu giao diện **Swagger UI** của FastAPI hiện ra với đầy đủ danh sách API Endpoint là API Gateway đã kết nối Lambda thành công.

> **Kết quả mong đợi:** Trang Swagger UI hiển thị đầy đủ các nhóm API: `/api/auth`, `/api/users`, `/api/attendance`, `/api/faces`...
> ![Giao diện Swagger UI](/aws-image/setupTestapi/testapi1.png)

---

**Bước 2: Tạo người dùng mới (POST /api/users)**

1. Mở **Postman** (hoặc dùng trực tiếp Swagger UI), tạo một Request mới với:
   - **Method:** `POST`
   - **URL:** `https://<invoke-url>/api/users`
   - **Headers:** `Content-Type: application/json`
   - **Body (raw JSON):**
   ```json
   {
     "email": "hohane8316@mrworlds.com",
     "name": "Nguyen Van Test",
     "role": "STAFF",
     "department": "IT",
     "phone": "0901234567",
     "employee_id": "EMP-12345"
   }
   ```
   > ![Body tạo user](/aws-image/setupTestapi/testapi2.png)
2. Bấm **Send** (hoặc **Execute** trên Swagger).
3. (Lưu ý: Mật khẩu tạm thời sẽ được AWS Cognito tự động gửi về email bạn cung cấp ở trên).

> **Kết quả mong đợi:** HTTP `201 Created` với body trả về thông tin user vừa tạo và `user_id`.
> ![Tạo user thành công](/aws-image/setupTestapi/testapi3.png)

---

**Bước 3: Đăng nhập và lấy Access Token (POST /api/auth/login)**

1. Kiểm tra Email của bạn để lấy mật khẩu tạm thời do Cognito gửi.
   > ![Email mật khẩu tạm thời](/aws-image/setupTestapi/testapi4.png)
2. Tạo Request mới:
   - **Method:** `POST`
   - **URL:** `https://<invoke-url>/api/auth/login`
   - **Body (raw JSON):**
   ```json
   {
     "email": "test.user@example.com",
     "password": "<Mật_khẩu_tạm_thời_trong_email>"
   }
   ```
   > ![Body login](/aws-image/setupTestapi/testapi5.png)
3. Bấm **Send** (hoặc **Execute**).
4. (Nếu hệ thống yêu cầu đổi mật khẩu lần đầu, bạn có thể gọi API `POST /api/auth/respond-challenge` với `new_password`).
5. Sau khi login thành công, sao chép giá trị `access_token` từ response body. Token này sẽ dùng ở các bước sau.

> **Kết quả mong đợi:** HTTP `200 OK` với response chứa `access_token` (JWT token).
> ![Đăng nhập thành công](/aws-image/setupTestapi/testapi6.png)

---

**Bước 4: Gọi API có xác thực (GET /api/users)**

1. Tạo Request mới:
   - **Method:** `GET`
   - **URL:** `https://<invoke-url>/api/users`
   - **Headers:** Thêm `Authorization: Bearer <access_token_vừa_lấy>`
   > ![Gọi API danh sách user](/aws-image/setupTestapi/testapi7.png)
2. Bấm **Send** (hoặc **Execute**).

> **Kết quả mong đợi:** HTTP `200 OK` trả về danh sách người dùng trong hệ thống (bao gồm user bạn vừa tạo).
> ![Kết quả danh sách user](/aws-image/setupTestapi/testapi8.png)



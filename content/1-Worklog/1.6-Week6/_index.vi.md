---
title: "Worklog Tuần 6"
date: 2026-07-27
weight: 6
chapter: false
pre: " <b> 1.6. </b> "
---

# Tuần 6: Dashboard Analytics & RBAC – Sinh trắc học nâng cao – Well-Architected Review

## 1. Mục tiêu tuần

* Tái thiết kế Dashboard theo hướng trực quan hóa dữ liệu tốt hơn, đưa Analytics thành trang chủ và phân tầng hiển thị theo role.
* Nâng cấp luồng xác thực Cognito, đăng ký khuôn mặt cá nhân, đăng nhập bằng khuôn mặt và khôi phục mật khẩu bằng sinh trắc học.
* Bổ sung màn hình Đăng nhập, cơ chế bảo vệ route và tăng tính nhất quán giữa vai trò người dùng với phần hiển thị dữ liệu.
* Rà soát hệ thống theo AWS Well-Architected Framework và cân đối lại thiết kế trong bối cảnh giới hạn credit AWS.

## 2. Nhật ký công việc chi tiết

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | - Thiết kế **luồng dữ liệu kép**: luồng cập nhật báo cáo hàng ngày qua EventBridge và luồng truy vấn trực tiếp qua Athena <br> - Áp dụng 6 trụ cột **AWS Well-Architected Framework** để rà soát lại kiến trúc ở các góc độ vận hành, bảo mật, độ tin cậy, hiệu năng, chi phí và tính bền vững <br> - Phân chia lại vai trò nhóm: FE, BE, CI/CD, xử lý dữ liệu, giám sát; đồng thời cập nhật sơ đồ kiến trúc phân lớp <br> - Lên phương án ứng phó với giới hạn credit AWS khi bước sang giai đoạn hoàn thiện | 27/07/2026 | 27/07/2026 | <https://docs.aws.amazon.com/wellarchitected/> |
| 3 | - **Frontend:** Thiết kế lại các KPI Summary Card cho trang Analytics; xây dựng Circular Progress Ring (SVG) và Donut Chart (SVG thuần) để hiển thị tỷ lệ công việc/điểm danh <br> - **Backend:** Hoàn thiện WF1 – Cognito ở phía quản trị: dùng `admin_create_user` trong endpoint tạo tài khoản của Admin để Cognito tự sinh Temporary Password và gửi email cho nhân sự mới | 28/07/2026 | 28/07/2026 | <https://docs.aws.amazon.com/cognito/> |
| 4 | - **Frontend:** Xây dựng Area Chart gradient cho xu hướng điểm danh theo thời gian, danh sách Top nhân viên vắng mặt với thanh tiến độ phân ngưỡng màu; cập nhật dữ liệu động từ endpoint Analytics/Reports <br> - **Backend:** Hoàn thiện luồng sinh trắc học: trang My Profile cho phép nhân viên tự đăng ký khuôn mặt bằng Webcam + Upload; chống trùng lặp bằng SearchFacesByImage trước IndexFaces | 29/07/2026 | 29/07/2026 | <https://docs.aws.amazon.com/rekognition/> |
| 5 | - **Frontend:** Xây dựng màn hình Login tích hợp xác thực với Cognito, xử lý luồng đổi mật khẩu tạm và chuyển hướng sau đăng nhập <br> - Viết cơ chế Private Route/HOC để chặn người dùng chưa xác thực truy cập vào các trang nội bộ <br> - **Backend:** Hoàn thiện đăng nhập bằng khuôn mặt trả về JWT token; xây dựng tính năng khôi phục mật khẩu bằng khuôn mặt: nhận email + ảnh base64, xác minh qua Rekognition, reset password trên Cognito; xử lý trường hợp trùng `face_id` | 30/07/2026 | 30/07/2026 | <https://docs.aws.amazon.com/cognito/> <br> <https://docs.aws.amazon.com/rekognition/> |
| 6 | - **Frontend:** Tích hợp RBAC 3 tầng hoàn toàn ở frontend: ADMIN/DIRECTOR xem toàn hệ thống, MANAGER bị khóa theo phòng ban phụ trách, STAFF chuyển thành “My Analytics” <br> - Hoàn thiện giao diện Analytics.jsx làm trang chủ, tối ưu lại bố cục và khả năng responsive <br> - Fix lỗi rò rỉ bộ nhớ camera khi chuyển tab bằng hàm `stopFaceCamera()` <br> - Rà soát tổng thể giao diện mới, bảo đảm thống nhất giữa JWT role, route protection và dữ liệu hiển thị | 31/07/2026 | 01/08/2026 | — |

## 3. Đóng góp cá nhân

* Phụ trách chính phần frontend của tuần này:
  * đưa Analytics.jsx làm trang chủ;
  * xây dựng KPI card;
  * tự code biểu đồ SVG như Donut Chart, Progress Ring, Area Chart, Progress Bar mà không phụ thuộc thư viện ngoài;
  * hoàn thiện màn hình Login và Private Route;
  * triển khai RBAC 3 tầng ở giao diện;
  * xử lý lỗi camera khi chuyển tab.
* Cập nhật cách hiển thị dữ liệu theo role để phần giao diện phản ánh đúng quyền xem của từng nhóm người dùng.
* Rà soát luồng xác thực và sinh trắc học từ góc nhìn người dùng cuối, bảo đảm logic đăng nhập, đổi mật khẩu tạm và khôi phục mật khẩu nối được với nhau.
* Ghi chú lại các điểm cần cân đối giữa chi phí, hiệu năng và phạm vi dịch vụ sau khi rà soát theo Well-Architected.

## 4. Kết quả đạt được

* **Frontend:**
  * Dashboard được tái thiết kế thành trang Analytics trực quan hơn, có phân tầng hiển thị theo role và không phụ thuộc thư viện chart bên ngoài.
  * Login, Private Route và RBAC frontend hoạt động đồng bộ với JWT.

* **Backend và xác thực:**
  * Hoàn thiện hơn luồng Cognito cho quản trị tài khoản.
  * Luồng đăng ký khuôn mặt cá nhân, đăng nhập bằng khuôn mặt và khôi phục mật khẩu bằng sinh trắc học đã được nối vào hệ thống.

* **Tối ưu và rà soát:**
  * Xử lý được lỗi camera lifecycle khi chuyển tab.
  * Hoàn thành bước rà soát kiến trúc theo Well-Architected, làm rõ thêm các trade-off liên quan đến chi phí và khả năng vận hành.
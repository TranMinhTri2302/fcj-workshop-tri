---
title: "Worklog Tuần 6"
date: 2026-07-27
weight: 6
chapter: false
pre: " <b> 1.6. </b> "
---

# Tuần 6: Dashboard Analytics & RBAC – Sinh trắc học nâng cao – Rà soát Well-Architected

## 1. Mục tiêu tuần

* Tái thiết kế Dashboard theo hướng trực quan hóa dữ liệu tốt hơn, đưa Analytics thành trang chủ và phân tầng hiển thị theo role.
* Hoàn thiện các màn hình liên quan đến xác thực như Login, Private Route và các phần hiển thị phụ thuộc quyền người dùng.
* Bổ sung giao diện cho các luồng sinh trắc học nâng cao như đăng ký khuôn mặt cá nhân và các bước xác thực liên quan.
* Rà soát lại kiến trúc hệ thống theo AWS Well-Architected Framework và cập nhật các điểm ảnh hưởng đến phần hiển thị/frontend integration.

## 2. Nhật ký công việc chi tiết

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | - Rà soát lại luồng dữ liệu kép của hệ thống: luồng cập nhật báo cáo định kỳ và luồng truy vấn trực tiếp theo yêu cầu người dùng <br> - Cập nhật sơ đồ kiến trúc phân lớp, role matrix và các điểm ảnh hưởng đến phần hiển thị dữ liệu trên giao diện <br> - Ghi chú lại các yếu tố cần cân đối về chi phí, hiệu năng và phạm vi dịch vụ sau khi nhóm rà soát theo AWS Well-Architected Framework | 27/07/2026 | 27/07/2026 | <https://docs.aws.amazon.com/wellarchitected/> |
| 3 | - Thiết kế lại các KPI Summary Card cho trang Analytics <br> - Xây dựng Circular Progress Ring (SVG) và Donut Chart (SVG thuần) để hiển thị tỷ lệ công việc/điểm danh mà không phụ thuộc thư viện ngoài <br> - Kiểm tra lại các nhóm dữ liệu hiển thị và một số response liên quan đến quản trị tài khoản, role và dashboard | 28/07/2026 | 28/07/2026 | <https://react.dev/> |
| 4 | - Xây dựng Area Chart gradient cho xu hướng điểm danh theo thời gian và danh sách Top nhân viên vắng mặt với thanh tiến độ phân ngưỡng màu <br> - Cập nhật dữ liệu động từ endpoint Analytics/Reports <br> - Bổ sung giao diện My Profile cho phép người dùng tự đăng ký khuôn mặt bằng Webcam + Upload và kiểm tra lại các trạng thái đầu-cuối của luồng này | 29/07/2026 | 29/07/2026 | <https://docs.aws.amazon.com/rekognition/> |
| 5 | - Xây dựng màn hình Login tích hợp với Cognito, xử lý luồng đổi mật khẩu tạm và chuyển hướng sau đăng nhập <br> - Viết cơ chế Private Route/HOC để chặn người dùng chưa xác thực truy cập vào các trang nội bộ <br> - Rà soát lại trải nghiệm người dùng cho các luồng xác thực, đăng ký khuôn mặt, đăng nhập bằng khuôn mặt và khôi phục truy cập từ góc nhìn giao diện | 30/07/2026 | 30/07/2026 | <https://docs.aws.amazon.com/cognito/> |
| 6 | - Tích hợp RBAC 3 tầng hoàn toàn ở frontend: ADMIN/DIRECTOR xem toàn hệ thống, MANAGER bị khóa theo phòng ban phụ trách, STAFF chuyển thành “My Analytics” <br> - Hoàn thiện giao diện `Analytics.jsx` làm trang chủ, tối ưu lại bố cục và khả năng responsive <br> - Fix lỗi rò rỉ bộ nhớ camera khi chuyển tab bằng hàm `stopFaceCamera()` <br> - Hỗ trợ rà soát lại các trường role, token và cách hiển thị dữ liệu để bảo đảm thống nhất giữa JWT, route protection và phần quyền trong hệ thống | 31/07/2026 | 01/08/2026 | — |

## 3. Đóng góp cá nhân

* Phụ trách chính phần giao diện của tuần này:
  * đưa `Analytics.jsx` làm trang chủ;
  * xây dựng KPI card;
  * tự code biểu đồ SVG như Donut Chart, Progress Ring, Area Chart, Progress Bar;
  * hoàn thiện Login và Private Route;
  * triển khai RBAC 3 tầng ở giao diện;
  * xử lý lỗi camera khi chuyển tab.
* Cập nhật cách hiển thị dữ liệu theo role để phần giao diện phản ánh đúng quyền xem của từng nhóm người dùng.
* Rà soát các luồng xác thực và sinh trắc học từ góc nhìn người dùng cuối để bảo đảm các bước đăng nhập, đổi mật khẩu tạm, đăng ký khuôn mặt và các trạng thái lỗi được nối liền mạch.
* Ghi chú lại các điểm cần cân đối giữa chi phí, hiệu năng và phạm vi dịch vụ sau khi rà soát theo Well-Architected.

## 4. Kết quả đạt được

* Dashboard được tái thiết kế thành trang Analytics trực quan hơn, có phân tầng hiển thị theo role và không phụ thuộc thư viện chart bên ngoài.
* Login, Private Route và RBAC frontend hoạt động đồng bộ với JWT.
* Giao diện cho các luồng đăng ký khuôn mặt cá nhân, đăng nhập bằng khuôn mặt và các bước xác thực liên quan đã được nối vào hệ thống.
* Xử lý được lỗi camera lifecycle khi chuyển tab và hoàn tất bước rà soát kiến trúc theo Well-Architected.
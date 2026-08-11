---
title: "Worklog Tuần 4"
date: 2026-07-13
weight: 4
chapter: false
pre: " <b> 1.4. </b> "
---

# Tuần 4: UI Nhân sự & Điểm danh – Tích hợp webcam – Đồng bộ luồng nhận diện khuôn mặt

## 1. Mục tiêu tuần

* Xây dựng giao diện Quản lý nhân sự và trang Điểm danh với webcam trên trình duyệt.
* Hoàn thiện luồng đăng ký khuôn mặt và điểm danh ở phía frontend theo hướng end-to-end.
* Rà soát lại payload, naming convention và trạng thái nghiệp vụ giữa giao diện với API nhận diện khuôn mặt/điểm danh.
* Hỗ trợ kiểm tra các thay đổi liên quan đến dữ liệu, rule nghiệp vụ và module công việc trong giai đoạn baseline.

## 2. Nhật ký công việc chi tiết

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | - Họp nhóm chốt mục tiêu hoàn thành baseline trong tuần, ưu tiên các workflow có khả năng trình diễn rõ <br> - Xây dựng trang Quản lý nhân sự (Users Page), gọi API lấy danh sách nhân viên và hiển thị bằng DataTable đã chuẩn bị từ tuần 3 <br> - Rà soát lại luồng đăng ký khuôn mặt ở mức giao diện để bảo đảm các trường dữ liệu đầu vào phù hợp với phía xử lý ảnh | 13/07/2026 | 13/07/2026 | <https://docs.aws.amazon.com/rekognition/> |
| 3 | - Phát triển form thêm mới/chỉnh sửa nhân viên kèm validation đầu vào <br> - Xây dựng modal “Đăng ký khuôn mặt” trong `Users.jsx`, hỗ trợ hai phương thức Upload file hoặc bật Webcam chụp trực tiếp qua `navigator.mediaDevices` <br> - Kiểm tra lại cấu trúc payload gửi ảnh và các trường phản hồi như `face_id`, `confidence`, `BoundingBox` để bảo đảm giao diện đọc đúng dữ liệu | 14/07/2026 | 14/07/2026 | <https://developer.mozilla.org/en-US/docs/Web/API/MediaDevices/getUserMedia> |
| 4 | - Nghiên cứu WebRTC API (`getUserMedia`), viết custom hook `useCamera` để quản lý quyền truy cập camera, stream video và lifecycle <br> - Xây dựng logic chụp ảnh từ webcam, vẽ lên `<canvas>`, chuyển thành Base64 và gửi lên API <br> - Đồng bộ lại cách hiển thị trạng thái điểm danh theo các nhóm PRESENT / LATE / REJECTED để giao diện phản ánh đúng rule nghiệp vụ | 15/07/2026 | 15/07/2026 | <https://developer.mozilla.org/en-US/docs/Web/API/Canvas_API> |
| 5 | - Thiết kế trang `Attendance.jsx`: webcam nhận diện, hiển thị tên nhân viên + confidence %, hiệu ứng scanning, bảng lịch sử có filter theo ngày/ca, badge màu theo trạng thái <br> - Bổ sung các trạng thái UX: loading camera, lỗi quyền truy cập, camera không khả dụng, nhận diện thành công/thất bại <br> - Hỗ trợ rà soát thêm cách tổ chức dữ liệu hiển thị cho module Tasks và các trạng thái nghiệp vụ liên quan | 16/07/2026 | 16/07/2026 | <https://react.dev/> |
| 6 | - Kiểm thử đầu-cuối luồng đăng ký khuôn mặt và điểm danh từ giao diện đến API, ghi nhận các vấn đề về naming, kiểu dữ liệu và CORS trong quá trình tích hợp <br> - Điều chỉnh giao diện theo các thay đổi về `faceId`/`face_id`, trạng thái điểm danh và thông báo lỗi <br> - Phối hợp rà soát các edge case liên quan đến upload ảnh, dữ liệu trả về, rule điểm danh và việc hiển thị lịch sử | 17/07/2026 | 18/07/2026 | <https://fastapi.tiangolo.com/> |

## 3. Đóng góp cá nhân

* Phụ trách chính phần giao diện của tuần này:
  * Users Page hiển thị danh sách nhân sự;
  * form thêm/sửa nhân viên;
  * modal đăng ký khuôn mặt bằng Upload + Webcam;
  * custom hook `useCamera`;
  * logic chụp ảnh từ webcam qua canvas;
  * trang Attendance với webcam nhận diện, hiệu ứng scanning, filter và badge trạng thái.
* Rà soát luồng đăng ký khuôn mặt từ phía client đến API để bảo đảm tên trường, payload và cách đọc response được thống nhất.
* Làm rõ thêm cách hiển thị các trạng thái nghiệp vụ của Attendance, đặc biệt ở PRESENT, LATE, REJECTED và trường hợp điểm danh trùng lặp.
* Hỗ trợ kiểm tra các thay đổi liên quan đến dữ liệu và naming convention để tránh lệch giữa giao diện, API và phần lưu trữ.

## 4. Kết quả đạt được

* Hoàn thiện giao diện quản lý nhân sự và giao diện điểm danh bằng webcam.
* WebRTC hook và luồng capture ảnh hoạt động ổn định, phục vụ trực tiếp cho WF2 và WF3.
* Luồng đăng ký khuôn mặt và điểm danh đã kết nối được đầu-cuối từ giao diện đến API.
* Ghi nhận và xử lý được các vấn đề tích hợp quan trọng như naming không nhất quán, dữ liệu phản hồi từ nhận diện khuôn mặt và việc mất CORS khi API phát sinh lỗi.
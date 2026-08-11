---
title: "Worklog Tuần 4"
date: 2026-07-13
weight: 4
chapter: false
pre: " <b> 1.4. </b> "
---

# Tuần 4: UI Nhân sự & Điểm danh – Tích hợp Rekognition – Module Tasks – Xử lý lỗi tích hợp

## 1. Mục tiêu tuần

* Xây dựng giao diện Quản lý nhân sự và trang Điểm danh với webcam trên trình duyệt.
* Tích hợp Amazon Rekognition + S3 để hoàn thiện luồng đăng ký khuôn mặt (WF2) và điểm danh (WF3) theo hướng end-to-end.
* Mở rộng module Quản lý công việc với thiết kế DynamoDB theo access pattern và rule nghiệp vụ rõ ràng.
* Tăng cường kiểm thử tự động và xử lý các lỗi tích hợp phát sinh giữa frontend, Rekognition, DynamoDB và backend.

## 2. Nhật ký công việc chi tiết

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | - Họp nhóm chốt mục tiêu hoàn thành baseline trong tuần, ưu tiên các workflow có khả năng trình diễn rõ <br> - **Frontend:** Xây dựng trang Quản lý nhân sự (Users Page), gọi API lấy danh sách nhân viên và hiển thị bằng DataTable đã chuẩn bị từ tuần 3 <br> - **Backend:** Thiết lập S3 bucket `smart-campus-images` với Block Public Access, cấu hình CORS cho phép frontend upload trực tiếp; tạo Rekognition Collection và xây dựng wrapper `rekognition.py` cho IndexFaces, SearchFacesByImage | 13/07/2026 | 13/07/2026 | <https://docs.aws.amazon.com/rekognition/> <br> <https://docs.aws.amazon.com/AmazonS3/latest/userguide/> |
| 3 | - **Frontend:** Phát triển Modal Form thêm mới/chỉnh sửa nhân viên kèm validation đầu vào <br> - Xây dựng Modal “Đăng ký khuôn mặt” trong Users.jsx: hỗ trợ hai phương thức Upload file hoặc bật Webcam chụp trực tiếp qua `navigator.mediaDevices` <br> - **Backend:** Hoàn thành WF2 – Face Registration end-to-end: nhận ảnh base64, decode và validate JPEG/PNG (tối đa 5MB), lưu ảnh gốc lên S3, gọi IndexFaces để sinh faceId, confidence, BoundingBox | 14/07/2026 | 14/07/2026 | <https://docs.aws.amazon.com/rekognition/latest/dg/API_IndexFaces.html> |
| 4 | - **Frontend:** Nghiên cứu WebRTC API (`getUserMedia`), viết custom hook `useCamera` để quản lý quyền truy cập camera, stream video và lifecycle <br> - Xây dựng logic chụp ảnh từ webcam, vẽ lên `<canvas>`, chuyển thành Base64 và gửi lên backend <br> - **Backend:** Hoàn thành WF3 – Attendance: xây dựng Rule Engine với 3 ca (MORNING 07:00–12:00, AFTERNOON 13:00–17:30, EVENING 17:30–21:00), tự động phân loại PRESENT / LATE / REJECTED | 15/07/2026 | 15/07/2026 | <https://docs.aws.amazon.com/rekognition/latest/dg/API_SearchFacesByImage.html> |
| 5 | - **Frontend:** Thiết kế trang Attendance.jsx: webcam nhận diện, hiển thị tên nhân viên + confidence %, hiệu ứng scanning, bảng lịch sử có filter theo ngày/ca, badge màu theo trạng thái <br> - Bổ sung xử lý trạng thái UX: loading camera, lỗi quyền truy cập, camera không khả dụng, nhận diện thành công/thất bại <br> - **Backend:** Thiết kế bảng Tasks trên DynamoDB với partition key `task_id`, tạo GSI `assignee_id-status-index`; triển khai phân trang cursor (`ExclusiveStartKey` / `LastEvaluatedKey`) | 16/07/2026 | 16/07/2026 | <https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/GSI.html> |
| 6 | - **Backend:** Bổ sung rule nghiệp vụ cho Tasks: auto-assign incident theo phòng ban, kiểm soát phân quyền cập nhật theo vai trò, tự động gửi thông báo theo mốc trạng thái <br> - Nâng cao kiểm thử: viết unit test và integration test cho WF2, WF3 và Tasks; sử dụng moto giả lập Rekognition, DynamoDB, SES; cập nhật pipeline CI/CD <br> - Xử lý sự cố kỹ thuật: parse `BoundingBox` sang String trước khi ghi DynamoDB do boto3 không hỗ trợ trực tiếp kiểu Float từ Rekognition; đồng bộ `faceId` → `face_id`; bổ sung global exception handler để giữ CORS khi backend phát sinh lỗi | 17/07/2026 | 18/07/2026 | <https://github.com/getmoto/moto> |

## 3. Đóng góp cá nhân

* Phụ trách chính phần frontend của tuần này:
  * Users Page hiển thị danh sách nhân sự;
  * form thêm/sửa nhân viên;
  * modal đăng ký khuôn mặt bằng Upload + Webcam;
  * custom hook `useCamera`;
  * logic chụp ảnh từ webcam qua canvas;
  * trang Attendance với webcam nhận diện, hiệu ứng scanning, filter và badge trạng thái.
* Rà soát luồng đăng ký khuôn mặt từ phía client đến backend để bảo đảm tên trường và payload thống nhất.
* Làm rõ thêm phần rule nghiệp vụ cho Attendance, đặc biệt ở các trạng thái PRESENT, LATE, REJECTED và trường hợp điểm danh trùng lặp.
* Hỗ trợ phần Tasks ở góc độ truy vấn dữ liệu, naming convention và khả năng hiển thị về sau trên giao diện.
* Ghi nhận và xử lý các lỗi tích hợp phát sinh liên quan đến Float từ Rekognition, naming `face_id` và CORS khi exception.

## 4. Kết quả đạt được

* **Frontend:**
  * Hoàn thiện giao diện quản lý nhân sự và giao diện điểm danh bằng webcam.
  * WebRTC hook và luồng capture ảnh hoạt động ổn định, phục vụ trực tiếp cho WF2 và WF3.

* **Backend:**
  * WF2 (Face Registration) và WF3 (Attendance) hoạt động end-to-end.
  * Module Tasks đã có cấu trúc dữ liệu, GSI và cơ chế phân trang phù hợp.

* **Kiểm thử và tích hợp:**
  * Bổ sung test cho các luồng mới và cập nhật pipeline CI/CD.
  * Xử lý được các lỗi tích hợp quan trọng như kiểu dữ liệu từ Rekognition, naming không nhất quán và việc mất CORS khi backend ném exception.
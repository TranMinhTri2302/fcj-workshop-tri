---
title: "Worklog Tuần 5"
date: 2024-01-01
weight: 5
chapter: false
pre: " <b> 1.5. </b> "
---



### Mục tiêu tuần 5:

* Triển khai module Tasks theo kiến trúc tách lớp và tối ưu hóa truy vấn DynamoDB bằng GSI và cursor-based pagination.
* Bổ sung các rule nghiệp vụ quan trọng như auto-assign, phân quyền theo vai trò và gửi thông báo theo từng trạng thái.
* Tích hợp AI bằng Amazon Rekognition vào luồng điểm danh khuôn mặt để nâng cao trải nghiệm người dùng.
* Nâng cao chất lượng kiểm thử tự động và hoàn thiện pipeline CI/CD để hệ thống sẵn sàng cho các giai đoạn phát triển tiếp theo.

### Các công việc cần triển khai trong tuần này:
| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | - Triển khai module Tasks và thiết kế bảng DynamoDB với GSI `assignee_id-status-index` để truy vấn nhanh hơn | 20/07/2026 | 20/07/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 3 | - Hoàn thiện cơ chế phân trang cursor cho DynamoDB <br> - Xây dựng các rule nghiệp vụ như auto-assign, phân quyền và thông báo theo trạng thái | 21/07/2026 | 21/07/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 4 | - Tích hợp Amazon Rekognition để xây dựng API đăng ký khuôn mặt và API check-in bằng SearchFacesByImage | 22/07/2026 | 22/07/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 5 | - Áp dụng ràng buộc thời gian cho điểm danh khuôn mặt <br> - Nâng cao kiểm thử tự động với moto cho Rekognition, DynamoDB và SES | 23/07/2026 | 23/07/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 6 | - Cập nhật pipeline CI/CD trên GitHub Actions để chạy toàn bộ bộ test mới | 24/07/2026 | 24/07/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 7 | - Tổng kết tuần: tối ưu hóa thiết kế NoSQL, tích hợp AI vào luồng nghiệp vụ thực tế và củng cố phân quyền | 25/07/2026 | 25/07/2026 | <https://cloudjourney.awsstudygroup.com/> |

### Kết quả đạt được tuần 5:

* **Module Tasks và tối ưu DynamoDB:**
  * Triển khai module Tasks theo kiến trúc tách lớp, chuẩn hóa cấu hình tên bảng DynamoDB qua biến settings để dễ bảo trì.
  * Thiết kế bảng Tasks với GSI `assignee_id-status-index`, giúp truy vấn theo assignee và status hiệu quả hơn, đồng thời giảm thiểu việc sử dụng scan.
  * Hoàn thiện cơ chế phân trang theo cursor cho DynamoDB, phù hợp hơn với đặc thù NoSQL so với mô hình offset/limit truyền thống.

* **Nghiệp vụ và tích hợp AI:**
  * Bổ sung các rule nghiệp vụ như auto-assign, phân quyền theo vai trò và gửi thông báo ở từng mốc trạng thái.
  * Tích hợp Amazon Rekognition vào luồng điểm danh khuôn mặt, bao gồm đăng ký khuôn mặt, check-in và xử lý các trường hợp như không phát hiện khuôn mặt hoặc chất lượng ảnh thấp.

* **Kiểm thử và CI/CD:**
  * Nâng cao chất lượng kiểm thử tự động bằng unit test và integration test, sử dụng moto để mô phỏng Rekognition, DynamoDB và SES.
  * Cập nhật pipeline CI/CD trên GitHub Actions để chạy đầy đủ bộ test mới.
  * Hiểu sâu hơn về thiết kế GSI theo access pattern, cơ chế cursor pagination và tầm quan trọng của phân quyền kết hợp với thông báo trong hệ thống thực tế.




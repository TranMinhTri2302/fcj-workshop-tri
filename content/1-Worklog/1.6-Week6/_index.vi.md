---
title: "Worklog Tuần 6"
date: 2024-01-01
weight: 6
chapter: false
pre: " <b> 1.6. </b> "
---



### Mục tiêu tuần 6:

* Nâng cấp module Analytics/Reports và triển khai giao diện dashboard phân tích dữ liệu cho hệ thống.
* Hoàn thiện cơ chế thông báo production bằng Amazon SES/SNS và chuẩn hóa định dạng dữ liệu notification.
* Tích hợp Amazon Cognito vào luồng xác thực chính thức, hỗ trợ xử lý yêu cầu đổi mật khẩu lần đầu và khôi phục tài khoản bằng khuôn mặt.
* Tham gia sự kiện AWS FCAJ Agent Forge – Deepdive để hiểu sâu hơn về kiến trúc Agentic AI và các nguyên tắc bảo mật nhiều lớp.

### Các công việc cần triển khai trong tuần này:
| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | - Nâng cấp module Analytics/Reports bằng cách bổ sung endpoint và schema phục vụ dashboard phân tích, đồng thời hỗ trợ filter theo phòng ban | 27/07/2026 | 27/07/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 3 | - Triển khai trang Analytics trên frontend để trực quan hóa dữ liệu thống kê | 28/07/2026 | 28/07/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 4 | - Hoàn thiện cơ chế thông báo production: chuyển từ broadcast sang gửi email 1-1 bằng Amazon SES, có fallback sang SNS khi SES lỗi hoặc thiếu email | 29/07/2026 | 29/07/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 5 | - Tích hợp Amazon Cognito vào luồng xác thực chính thức: xử lý NEW_PASSWORD_REQUIRED, triển khai AuthContext và ProtectedRoute trên frontend | 30/07/2026 | 30/07/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 6 | - Phát triển tính năng khôi phục mật khẩu bằng khuôn mặt: backend nhận email và ảnh base64, xác minh qua Rekognition, rồi reset password trên Cognito | 31/07/2026 | 31/07/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 7 | - Tham gia sự kiện **AWS FCAJ Agent Forge – Deepdive (03/08/2026)** tại tầng 26 Bitexco; tìm hiểu Agentic AI, Bedrock Agent Core, Firecracker MicroVM, MCP/A2A, Guardrails và Human-in-the-loop | 03/08/2026 | 03/08/2026 | <https://cloudjourney.awsstudygroup.com/> <br> [Video workshop Agent Forge](https://www.youtube.com/watch?v=F58sam40jxk) |
| 8 | - Tổng kết hiểu biết về thiết kế hệ thống Agentic AI, Human-in-the-loop và cách lựa chọn model phù hợp theo chi phí và hiệu năng | 01/08/2026 | 01/08/2026 | <https://cloudjourney.awsstudygroup.com/> |

### Kết quả đạt được tuần 6:

* **Nâng cấp Analytics và frontend:**
  * Cải tiến module Analytics/Reports bằng cách bổ sung endpoint và schema phục vụ dashboard phân tích với khả năng filter theo phòng ban.
  * Triển khai trang Analytics trên frontend để trực quan hóa dữ liệu thống kê cho người dùng.

* **Thông báo production và bảo mật dữ liệu:**
  * Hoàn thiện cơ chế gửi thông báo theo hướng production: chuyển từ broadcast sang email cá nhân 1-1 bằng Amazon SES, đồng thời có cơ chế fallback tự động sang SNS khi SES gặp lỗi hoặc không có email hợp lệ.
  * Chuẩn hóa định dạng dữ liệu notification theo chuẩn snake_case và đảm bảo tương thích với dữ liệu cũ.

* **Xác thực và khôi phục tài khoản:**
  * Tích hợp Amazon Cognito vào luồng xác thực chính thức, xử lý trường hợp NEW_PASSWORD_REQUIRED bằng API respond-to-auth-challenge.
  * Triển khai AuthContext và ProtectedRoute trên frontend để bảo vệ các route cần đăng nhập.
  * Phát triển tính năng khôi phục mật khẩu bằng khuôn mặt, trong đó backend nhận email và ảnh base64, xác minh danh tính qua Rekognition rồi thực hiện reset password trên Cognito.

* **Sự kiện và tư duy kiến trúc nâng cao:**
  * Tham gia sự kiện AWS FCAJ Agent Forge – Deepdive, tìm hiểu sâu về Agentic AI, Bedrock Agent Core, Firecracker MicroVM, Token Vault, giao thức MCP/A2A, Gateway với Guardrails và Human-in-the-loop.
  * Nắm rõ hơn về cách thiết kế hệ thống Agentic AI một cách bài bản, tầm quan trọng của Human-in-the-loop, các lớp bảo mật đa tầng và cách lựa chọn model phù hợp như Haiku, Sonnet và Opus để tối ưu chi phí và hiệu năng.



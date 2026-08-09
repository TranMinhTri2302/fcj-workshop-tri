---
title: "Tự đánh giá"
date: 2024-01-01
weight: 6
chapter: false
pre: " <b> 6. </b> "
---

Trong thời gian thực tập tại **First Cloud AI Journey (FCAJ)** từ **tháng 6/2026** đến **tháng 8/2026**, em đã có cơ hội học hỏi, thực hành và áp dụng những kiến thức đã học vào một môi trường làm việc thực tế.

Trong kỳ thực tập, em tham gia xây dựng **Smart Campus Platform** — một hệ thống điểm danh và quản lý công việc theo kiến trúc serverless trên AWS, có tích hợp nhận diện khuôn mặt bằng AI. Thông qua dự án này, em đã cải thiện đáng kể kỹ năng ở nhiều mảng như **kiến trúc cloud, phát triển serverless, DevOps, tích hợp AI/ML, xử lý dữ liệu và hệ thống event-driven**.

Về thái độ làm việc, em luôn cố gắng hoàn thành nhiệm vụ được giao, tuân thủ nội quy của đơn vị thực tập, chủ động trao đổi với mentor và các anh/chị trong team khi gặp vấn đề. Bên cạnh đó, em cũng cố gắng tự tìm hiểu trước khi đặt câu hỏi để rèn luyện khả năng tự học và tự giải quyết vấn đề.

Để nhìn lại quá trình thực tập một cách rõ ràng hơn, em xin tự đánh giá bản thân theo các tiêu chí sau:

| STT | Tiêu chí | Mô tả | Tốt | Khá | Trung bình |
| --- | --- | --- | --- | --- | --- |
| 1 | **Kiến thức và kỹ năng chuyên môn** | Hiểu và áp dụng được kiến thức về AWS, serverless, backend, frontend và các công cụ hỗ trợ vào dự án thực tế | ✅ | ☐ | ☐ |
| 2 | **Khả năng học hỏi** | Tiếp thu nhanh kiến thức mới, đặc biệt là các dịch vụ AWS và mô hình kiến trúc cloud | ✅ | ☐ | ☐ |
| 3 | **Tính chủ động** | Chủ động tìm hiểu tài liệu, thử nghiệm giải pháp và nhận nhiệm vụ mà không phụ thuộc hoàn toàn vào hướng dẫn | ✅ | ☐ | ☐ |
| 4 | **Tinh thần trách nhiệm** | Hoàn thành công việc đúng hạn, theo dõi tiến độ và đảm bảo chất lượng đầu ra | ✅ | ☐ | ☐ |
| 5 | **Kỷ luật** | Tuân thủ giờ giấc, quy định, quy trình làm việc và bảo mật thông tin dự án | ✅ | ☐ | ☐ |
| 6 | **Tính cầu tiến** | Sẵn sàng tiếp nhận góp ý, sửa lỗi và cải thiện qua từng vòng review | ✅ | ☐ | ☐ |
| 7 | **Giao tiếp** | Trình bày ý tưởng, báo cáo tiến độ và trao đổi kỹ thuật với team | ☐ | ✅ | ☐ |
| 8 | **Hợp tác nhóm** | Phối hợp tốt với mentor, team admin và các thành viên khác trong quá trình làm việc | ✅ | ☐ | ☐ |
| 9 | **Ứng xử chuyên nghiệp** | Tôn trọng đồng nghiệp, giữ thái độ nghiêm túc và phù hợp trong môi trường làm việc | ✅ | ☐ | ☐ |
| 10 | **Tư duy giải quyết vấn đề** | Biết phân tích nguyên nhân, đề xuất hướng xử lý và kiểm tra lại kết quả | ✅ | ☐ | ☐ |
| 11 | **Đóng góp vào dự án/tổ chức** | Có đóng góp thực tế vào các workflow, dashboard, pipeline và tài liệu kỹ thuật của dự án | ✅ | ☐ | ☐ |
| 12 | **Tổng thể** | Đánh giá chung về toàn bộ quá trình thực tập | ✅ | ☐ | ☐ |

---

## Tự đánh giá chi tiết theo lĩnh vực kỹ thuật

### 1. Kiến trúc Cloud và phát triển Serverless  
**Đánh giá: ⭐⭐⭐⭐⭐**

Trong dự án, em có cơ hội tham gia thiết kế và triển khai kiến trúc **Serverless Event-Driven** trên AWS. Hệ thống sử dụng nhiều dịch vụ khác nhau như Lambda, API Gateway, DynamoDB, EventBridge, SQS, SNS, S3, Rekognition, Cognito, Athena, Glue, CloudFront, WAF, CloudWatch, X-Ray, CodePipeline và CodeBuild.

**Những điểm em đã làm được:**
- Tham gia xây dựng backend serverless với **API Gateway + Lambda**.
- Sử dụng **FastAPI kết hợp Mangum** để triển khai API Python trên môi trường Lambda.
- Thiết kế luồng xử lý bất đồng bộ bằng **EventBridge, SQS và DLQ**.
- Áp dụng mô hình kết hợp **DynamoDB cho dữ liệu giao dịch** và **Athena/S3 cho dữ liệu phân tích**.
- Tìm hiểu và áp dụng nguyên tắc **IAM Least Privilege** khi cấp quyền cho các service.

**Những kiến thức em học được:**
- Cách thiết kế hệ thống serverless có khả năng mở rộng.
- Cách tách rời các thành phần trong hệ thống bằng event-driven architecture.
- Cách xử lý retry, dead-letter queue và lỗi bất đồng bộ.
- Sự khác biệt giữa hệ thống chạy được ở mức demo và hệ thống có thể vận hành thực tế.

**Thách thức đã vượt qua:**
- Xử lý vấn đề khác biệt môi trường giữa **Windows local** và **AWS Lambda Linux**.
- Debug các lỗi liên quan đến **CORS, event loop và Mangum** trên Python 3.12.
- Điều chỉnh IAM policy để vừa đủ quyền sử dụng nhưng vẫn đảm bảo an toàn.

---

### 2. Tích hợp AI/ML  
**Đánh giá: ⭐⭐⭐⭐⭐**

Một phần quan trọng của dự án là tích hợp AI vào hệ thống điểm danh. Em đã tham gia sử dụng **Amazon Rekognition** cho chức năng nhận diện khuôn mặt và kiểm tra tính xác thực của khuôn mặt.

**Những điểm em đã làm được:**
- Tích hợp **Face Recognition** bằng Rekognition, bao gồm thao tác với collection và tìm kiếm khuôn mặt bằng `SearchFacesByImage`.
- Tìm hiểu và áp dụng **Face Liveness Detection** để hạn chế gian lận bằng ảnh in, video replay hoặc mặt nạ.
- Xử lý luồng upload ảnh an toàn thông qua **S3 pre-signed URL**.
- Lưu trữ và xử lý kết quả nhận diện phục vụ cho nghiệp vụ điểm danh.

**Những kiến thức em học được:**
- Cách lựa chọn và điều chỉnh ngưỡng confidence cho bài toán nhận diện.
- Cách xử lý dữ liệu trả về từ Rekognition trước khi lưu vào DynamoDB.
- Những rủi ro khi đưa AI vào hệ thống thực tế, ví dụ false positive, false negative và spoofing.

**Thách thức đã vượt qua:**
- Xử lý lỗi serialize dữ liệu kiểu Float khi làm việc với Boto3 và DynamoDB.
- Kiểm tra và validate ảnh đầu vào dạng base64.
- Đảm bảo ảnh được truy cập an toàn, không public trực tiếp ra ngoài.

---

### 3. Data Engineering và Analytics  
**Đánh giá: ⭐⭐⭐⭐**

Em cũng tham gia xây dựng pipeline phân tích dữ liệu cho hệ thống. Đây là phần giúp em hiểu rõ hơn sự khác nhau giữa dữ liệu phục vụ giao dịch hằng ngày và dữ liệu phục vụ báo cáo, phân tích.

**Những điểm em đã làm được:**
- Xây dựng luồng dữ liệu từ **Kinesis Firehose → S3 Data Lake → Glue Crawler → Athena**.
- Tổ chức dữ liệu trên S3 theo partition như year/month/day để hỗ trợ truy vấn hiệu quả hơn.
- Kết nối dữ liệu phân tích với dashboard frontend sử dụng **React và Recharts**.
- Hỗ trợ các góc nhìn báo cáo theo vai trò, ví dụ Director, Manager và Staff.

**Những kiến thức em học được:**
- Cách xây dựng Data Lake cơ bản trên AWS.
- Cách Glue Crawler phát hiện schema và tạo table phục vụ truy vấn Athena.
- Cách thiết kế dashboard phân tích dữ liệu dựa trên nhu cầu người dùng.
- Cách kết hợp DynamoDB cho truy vấn real-time và Athena cho dữ liệu lịch sử.

**Thách thức đã vượt qua:**
- Xử lý vấn đề schema discovery của Glue.
- Làm việc với partition và cấu hình Athena workgroup.
- Đồng bộ tên field giữa backend, data lake và frontend, ví dụ camelCase và snake_case.

---

### 4. DevOps và CI/CD  
**Đánh giá: ⭐⭐⭐⭐**

Trong quá trình thực tập, em có cơ hội tiếp cận quy trình build và deploy tự động cho cả backend và frontend. Đây là phần giúp em hiểu rõ hơn cách một hệ thống được triển khai ổn định thay vì chỉ chạy trên máy local.

**Những điểm em đã làm được:**
- Thiết lập pipeline deploy tự động bằng **CodePipeline và CodeBuild**.
- Build backend Python 3.12 với dependency tương thích môi trường AWS Lambda Linux.
- Build frontend bằng **Vite**, sau đó deploy lên S3 và invalidate cache CloudFront.
- Viết và chỉnh sửa file `buildspec.yml` cho các bước build/deploy.

**Những kiến thức em học được:**
- Cách tổ chức pipeline CI/CD cơ bản trên AWS.
- Cách build package Python cho Lambda bằng `manylinux2014_x86_64`.
- Cách cấp quyền cho CodeBuild service role.
- Cách xử lý cache khi deploy frontend qua CloudFront.

**Thách thức đã vượt qua:**
- Đóng gói Lambda zip sao cho dependency tương thích với Linux runtime.
- Xử lý lỗi 404 khi triển khai Single Page Application trên S3/CloudFront.
- Debug lỗi phát sinh sau deploy thông qua log và pipeline output.

---

### 5. Bảo mật và giám sát hệ thống  
**Đánh giá: ⭐⭐⭐⭐**

Bảo mật và giám sát là hai phần giúp em hiểu rằng một hệ thống thực tế không chỉ cần chạy đúng chức năng, mà còn phải an toàn, có thể theo dõi và dễ xử lý khi có sự cố.

**Những điểm em đã làm được:**
- Áp dụng mô hình **Defense in Depth** ở nhiều lớp.
- Sử dụng **AWS WAF** để giới hạn truy cập theo IP.
- Tích hợp **Cognito JWT** cho xác thực người dùng.
- Áp dụng phân quyền theo vai trò bằng RBAC trong hệ thống.
- Sử dụng **CloudWatch** để theo dõi log, metric và alarm.
- Tìm hiểu **X-Ray** để trace request qua nhiều service.

**Những kiến thức em học được:**
- Cách quản lý WAFv2 IPSet.
- Cách hoạt động của JWT trong xác thực API.
- Cách viết structured logging để dễ tìm lỗi.
- Cách đặt alarm và theo dõi DLQ để phát hiện lỗi bất đồng bộ.

**Thách thức đã vượt qua:**
- Xử lý yêu cầu HTTPS khi truy cập camera trên trình duyệt thông qua CloudFront.
- Cấu hình WAF với scope phù hợp cho CloudFront.
- Cân bằng giữa bảo mật và sự thuận tiện khi sử dụng hệ thống.

---

### 6. Phát triển Frontend  
**Đánh giá: ⭐⭐⭐⭐**

Ngoài backend và cloud, em cũng tham gia phát triển giao diện người dùng cho hệ thống. Phần này giúp em hiểu rõ hơn cách kết nối giữa trải nghiệm người dùng và các API phía sau.

**Những điểm em đã làm được:**
- Xây dựng frontend bằng **React + Vite**.
- Thiết kế giao diện hiện đại với phong cách Glassmorphism và hỗ trợ Dark Mode.
- Hiển thị biểu đồ bằng **Recharts**.
- Xây dựng các thành phần như dashboard, task board, thông báo real-time và upload file.
- Áp dụng render giao diện theo vai trò người dùng dựa trên RBAC.

**Những kiến thức em học được:**
- Cách tổ chức frontend project với Vite.
- Cách xử lý CORS trong môi trường local bằng Vite proxy.
- Cách kết nối frontend với API backend và S3 pre-signed URLs.
- Cách thiết kế giao diện responsive cho nhiều màn hình khác nhau.

---

## Các hạng mục chính đã hoàn thành

| Hạng mục | Mô tả | Công nghệ sử dụng | Trạng thái |
|:---|:---|:---|:---|
| **Smart Campus Platform** | Hệ thống điểm danh và quản lý công việc serverless full-stack | AWS, FastAPI, React, Rekognition, EventBridge, Athena | ✅ Hoàn thành |
| **WF1–WF8 Workflows** | Các workflow nghiệp vụ cốt lõi của hệ thống | AWS Serverless, EventBridge, SQS, DynamoDB, Bedrock | ✅ 7/8 hoàn thành |
| **Analytics Dashboard** | Dashboard báo cáo theo vai trò, có so sánh dữ liệu giữa các phòng ban | Athena, Glue, Recharts, DynamoDB | ✅ Hoàn thành |
| **CI/CD Pipeline** | Pipeline build và deploy tự động cho backend/frontend | CodePipeline, CodeBuild, S3, CloudFront | ✅ Hoàn thành |

---

## Những điểm cần cải thiện

Mặc dù đã đạt được nhiều kết quả tích cực, em nhận thấy bản thân vẫn còn một số điểm cần tiếp tục cải thiện:

1. **Kỹ năng giao tiếp kỹ thuật**  
   Em cần luyện tập thêm khả năng trình bày các quyết định kỹ thuật một cách rõ ràng, đặc biệt là khi giải thích cho người không chuyên sâu về cloud hoặc software engineering. Trong một số tình huống, em vẫn có xu hướng trình bày hơi nhiều chi tiết kỹ thuật, khiến người nghe khó nắm ý chính.

2. **Kỹ năng viết tài liệu**  
   Em cần cải thiện cách viết tài liệu kỹ thuật sao cho ngắn gọn, dễ đọc và có cấu trúc hơn, đặc biệt là các tài liệu như architecture decision record, runbook, hướng dẫn deploy hoặc tài liệu bàn giao.

3. **Thực hành testing sớm hơn**  
   Trong một số giai đoạn, em tập trung nhiều vào việc hoàn thành chức năng trước, sau đó mới bổ sung kiểm thử. Em nhận thấy mình cần áp dụng unit test và integration test sớm hơn trong quá trình phát triển, ví dụ sử dụng `pytest`, `moto` để mock AWS service hoặc viết test case cho các API quan trọng.

4. **Ước lượng thời gian công việc**  
   Với các task liên quan đến nhiều dịch vụ AWS, đôi khi thời gian debug lâu hơn dự kiến. Em cần cải thiện khả năng ước lượng effort, chia nhỏ task và dự phòng thời gian cho các lỗi phát sinh khi tích hợp hệ thống.

---

## Những trải nghiệm học tập quý giá nhất

1. **Trải nghiệm một dự án cloud end-to-end**  
   Em được tham gia vào nhiều giai đoạn của dự án, từ thiết kế kiến trúc, phát triển tính năng, triển khai, giám sát đến tối ưu hệ thống. Điều này giúp em hiểu rõ hơn vòng đời thực tế của một sản phẩm phần mềm trên cloud.

2. **Hiểu serverless ở mức thực tế hơn**  
   Trước đây, em chỉ hiểu serverless ở mức khái niệm hoặc demo đơn giản. Sau kỳ thực tập, em hiểu rõ hơn các yếu tố cần có trong hệ thống serverless production-grade như retry, DLQ, idempotency, logging, monitoring, fallback và cost control.

3. **Tích hợp AI vào hệ thống vận hành thực tế**  
   Việc sử dụng Rekognition giúp em hiểu rằng AI không chỉ là gọi API và nhận kết quả. Cần xử lý thêm nhiều vấn đề như ngưỡng confidence, lỗi nhận diện, validation dữ liệu đầu vào, bảo mật ảnh và trải nghiệm người dùng.

4. **Tư duy event-driven architecture**  
   Em học được cách tách rời các service bằng EventBridge, SQS và DLQ, đồng thời hiểu rõ hơn về eventual consistency, retry mechanism và cách thiết kế hệ thống chịu lỗi tốt hơn.

5. **Học từ mentor và môi trường làm việc thực tế**  
   Một trong những điều giá trị nhất là được học từ các anh/chị có kinh nghiệm. Mentor không “cầm tay chỉ việc” hoàn toàn, mà định hướng để em tự phân tích, tự thử nghiệm và tự rút ra bài học. Cách hướng dẫn này giúp em trưởng thành hơn trong tư duy giải quyết vấn đề.

---

## Kết luận tự đánh giá

Nhìn chung, em đánh giá quá trình thực tập của mình là **tốt**. Em đã hoàn thành phần lớn các nhiệm vụ được giao, chủ động học hỏi công nghệ mới và có đóng góp thực tế vào dự án Smart Campus Platform.

Kỳ thực tập tại FCAJ giúp em không chỉ nâng cao kỹ năng kỹ thuật, mà còn hiểu rõ hơn cách làm việc trong một dự án cloud thực tế: cần giao tiếp rõ ràng, viết code có trách nhiệm, quan tâm đến bảo mật, theo dõi hệ thống sau deploy và luôn suy nghĩ đến khả năng mở rộng, vận hành lâu dài.

Sau kỳ thực tập, em cảm thấy tự tin hơn với định hướng phát triển trong lĩnh vực **Cloud Engineering, AI Engineering và Solutions Architecture**. Em cũng nhận thức rõ hơn những điểm bản thân cần tiếp tục rèn luyện để có thể làm việc chuyên nghiệp và đóng góp hiệu quả hơn trong tương lai.
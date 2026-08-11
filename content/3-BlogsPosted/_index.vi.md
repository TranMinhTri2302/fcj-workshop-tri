---
title: "Các bài blogs đã đăng"
date: 2024-01-01
weight: 3
chapter: false
pre: " <b> 3. </b> "
---


Tại đây sẽ là phần liệt kê, giới thiệu các blogs mà các bạn đã đăng trên [AWS Study Group](https://www.facebook.com/groups/awsstudygroupfcj).

###  [Blog 1 - PAGINATION STRATEGY TRONG AMAZON DYNAMODB](3.1-Blog1/)
Blog này trình bày chiến lược pagination thiết yếu trong Amazon DynamoDB sử dụng **LastEvaluatedKey** và **ExclusiveStartKey** để chia nhỏ kết quả truy vấn thành từng trang thay vì load toàn bộ dữ liệu một lần. Bài viết chứng minh cách pagination giúp giảm chi phí Read Capacity Units (RCU) lên tới 10.000 lần, cải thiện thời gian phản hồi từ vài phút xuống còn vài trăm millisecond, và đảm bảo ứng dụng scale hiệu quả khi dữ liệu tăng trưởng.

**Link Facebook:** [https://www.facebook.com/share/p/1BxRgPHRBn/](https://www.facebook.com/share/p/1BxRgPHRBn/)

###  [Blog 2 - CHỐNG GIẢ MẠO NHẬN DIỆN KHUÔN MẶT VỚI AMAZON REKOGNITION FACE LIVENESS](3.2-Blog2/)
Blog này giải quyết một lỗ hổng bảo mật quan trọng trong hệ thống nhận diện khuôn mặt: tấn công spoofing bằng ảnh chụp, video quay sẵn, hoặc mặt nạ 3D. Bài viết giải thích cách **Amazon Rekognition Face Liveness** xác minh người dùng là người thật trước khi so khớp danh tính, sử dụng các challenge màu ngẫu nhiên và phân tích video thời gian thực. Bài viết chi tiết luồng tích hợp hoàn chỉnh: CreateFaceLivenessSession → Frontend Amplify UI Liveness Detector → GetFaceLivenessSessionResults → Quyết định dựa trên Confidence → SearchFacesByImage.

**Link Facebook:** [https://www.facebook.com/groups/awsstudygroupfcj/permalink/2240573343374292/](https://www.facebook.com/groups/awsstudygroupfcj/permalink/2240573343374292/)

###  [Blog 3 - CẢI THIỆN PHÂN TÍCH BIẾN ĐỘNG CHI PHÍ CLOUD HÀNG THÁNG BẰNG WEEKLY FINOPS CHECKPOINT](3.3-Blog3/)
Blog này trình bày quy trình **Weekly FinOps Checkpoint** chủ động để cải thiện phân tích biến động chi phí cloud hàng tháng. Thay vì chờ đến cuối tháng (quá muộn) hoặc theo dõi hàng ngày (quá nhiễu), chu kỳ rà soát hàng tuần giúp phát hiện xu hướng chi tiêu sớm, cung cấp bối cảnh rõ ràng cho báo cáo variance, và thúc đẩy sự phối hợp giữa Finance/FinOps với các đội kỹ thuật. Bài viết bao gồm 3 cách triển khai: AWS FinOps Agent (AI-powered), quy trình thủ công với Cost Explorer, và tương tác có cấu trúc với chủ tài khoản sử dụng các chỉ số WoW% và Annualized Impact.

**Link Facebook:** [https://www.facebook.com/groups/awsstudygroupfcj/permalink/2239975663434060/](https://www.facebook.com/groups/awsstudygroupfcj/permalink/2239975663434060/)
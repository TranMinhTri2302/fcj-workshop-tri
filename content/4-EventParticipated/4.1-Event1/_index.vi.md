---
title: "FCAJ Community Day - June 2026"
date: 2026-06-01
weight: 1
chapter: false
pre: " <b> 4.1. </b> "
---


# Summary Report: “FCAJ Community Day - June 2026”

## Thông tin sự kiện

- **Tên sự kiện:** FCAJ Community Day - June 2026  
- **Thời gian:** 9:00 sáng 27/06/2026  
- **Địa điểm:** Sự kiện offline, ghi chú tham gia tại tầng 26/36  
- **Vai trò:** Người tham dự, lắng nghe chia sẻ và đặt câu hỏi trong phần thảo luận  


## Mục tiêu sự kiện

FCAJ Community Day là một buổi chia sẻ cộng đồng tập trung vào các chủ đề công nghệ đang được áp dụng thực tế trong doanh nghiệp, đặc biệt là **Cloud**, **AI**, **DevOps**, **bảo mật** và **kiến trúc AWS**. Sự kiện không chỉ giới thiệu xu hướng, mà còn đi sâu vào những vấn đề thực tế khi triển khai hệ thống trong môi trường doanh nghiệp, ví dụ như chi phí hạ tầng, độ phức tạp khi hệ thống mở rộng, bảo mật dữ liệu và cách AI có thể hỗ trợ công việc kỹ thuật.

Thông qua sự kiện này, người tham dự có cơ hội hiểu rõ hơn về cách các doanh nghiệp đang ứng dụng AI vào vận hành, tuyển dụng, xử lý giọng nói, tối ưu chi phí và quản lý hạ tầng cloud.

## Diễn giả

- **Steve Trần** – Cloud Thinker  
- **Anh Nghị, anh Kiệt, anh Trung** – R AI, chia sẻ về Voice AI  
- **Chị Bảo và anh Nguyên** – Cloud Kinetic, chia sẻ về DevOps Agent  
- **Anh Trường và chị Minh Anh** – Noventis, chia sẻ về AI trong nhân sự  
- **Anh Toàn** – chia sẻ về AWS Architecture, Security và Cost Estimation  

## Nội dung chính

### Cloud Engineering trong thời đại AI

Phần đầu của sự kiện tập trung vào hành trình các doanh nghiệp chuyển dịch lên cloud và vai trò của kỹ sư cloud trong bối cảnh AI phát triển rất nhanh. Một điểm mình thấy khá thực tế là AI hiện nay có thể hỗ trợ viết code, tạo tài liệu hoặc triển khai một số tác vụ kỹ thuật nhanh hơn trước rất nhiều. Tuy nhiên, AI chưa thể thay thế hoàn toàn con người trong các quyết định quan trọng như thiết kế kiến trúc, xử lý sự cố production hoặc đưa ra lựa chọn phù hợp khi hệ thống gặp vấn đề phức tạp.

Diễn giả cũng nhấn mạnh rằng khi doanh nghiệp mở rộng quy mô, hệ thống sẽ ngày càng phức tạp. Một kỹ sư hoặc một công cụ AI rất khó có thể hiểu toàn bộ hệ thống từ tầng source code, hạ tầng, dữ liệu cho đến logic nghiệp vụ. Vì vậy, vai trò của Cloud Engineer, DevOps Engineer và Solution Architect vẫn rất quan trọng, đặc biệt là trong các hệ thống lớn cần đảm bảo tính ổn định, bảo mật và khả năng mở rộng.

### Voice AI và bài toán thực tế trong ngân hàng

Phần Voice AI là một trong những phần mình thấy ấn tượng nhất vì nội dung không dừng lại ở mức demo, mà đi vào cách một hệ thống Voice AI có thể hoạt động trong môi trường thực tế, cụ thể là trong lĩnh vực ngân hàng.

Diễn giả chia sẻ rằng một trong những khó khăn lớn khi làm Voice AI cho tiếng Việt là dữ liệu giọng nói còn hạn chế, trong khi tiếng Việt có nhiều vùng miền, cách phát âm và ngữ điệu khác nhau. Ngoài ra, hiện tại chưa có một mô hình đủ tốt để đưa trực tiếp giọng nói tiếng Việt vào LLM và để LLM hiểu hoàn toàn chính xác. Vì vậy, hệ thống cần đi qua nhiều bước xử lý trung gian.

Luồng xử lý có thể hiểu đơn giản như sau:

1. Người dùng nói bằng giọng nói tiếng Việt.  
2. Hệ thống chuyển giọng nói thành văn bản tiếng Việt.  
3. Văn bản tiếng Việt được xử lý hoặc chuyển sang tiếng Anh.  
4. Nội dung sau đó được đưa vào LLM để hiểu ý định và tạo phản hồi.  
5. Nếu cần thực hiện tác vụ, hệ thống sẽ dùng tool calling để xử lý các yêu cầu phù hợp trong ngữ cảnh ngân hàng.  

Một bạn tham dự cũng có đặt câu hỏi rằng hệ thống có hỗ trợ các giọng vùng miền của Việt Nam hay không. Diễn giả giải thích rằng thông qua quá trình sử dụng và huấn luyện từ dữ liệu giọng nói sang văn bản tiếng Việt, hệ thống có thể dần xử lý được nhiều cách phát âm khác nhau. Ngoài việc nhận diện nội dung lời nói, hệ thống còn có thể bổ sung thêm khả năng nhận diện giới tính nam hoặc nữ thông qua giọng nói.

Từ phần này, mình hiểu rõ hơn rằng xây dựng Voice AI không chỉ là kết nối speech-to-text với LLM. Để triển khai thực tế, hệ thống còn phải giải quyết các vấn đề về dữ liệu, ngôn ngữ, giọng vùng miền, độ chính xác và khả năng kiểm soát hành động của AI.

### DevOps Agent và tự động hóa xử lý sự cố

Phần DevOps Agent do Cloud Kinetic chia sẻ tập trung vào việc dùng AI để hỗ trợ đội ngũ vận hành hệ thống. Thay vì để kỹ sư phải tự đọc log, kiểm tra từng service và phân tích nguyên nhân sự cố hoàn toàn thủ công, DevOps Agent có thể hỗ trợ thu thập thông tin, phân tích dấu hiệu bất thường và gợi ý nguyên nhân có thể xảy ra.

Điểm quan trọng của phần này là AI không thay thế DevOps Engineer, mà đóng vai trò như một trợ lý giúp rút ngắn thời gian điều tra sự cố. Khi hệ thống gặp incident, việc giảm thời gian tìm nguyên nhân và khôi phục dịch vụ sẽ giúp giảm **MTTR** và tăng độ ổn định cho hệ thống.

Mình thấy đây là một hướng ứng dụng AI rất thực tế, vì trong môi trường production, tốc độ phản ứng khi có sự cố quan trọng không kém gì việc viết code nhanh.

### AI trong quy trình nhân sự

Ở phần chia sẻ của Noventis, diễn giả trình bày cách AI có thể hỗ trợ các công việc trong lĩnh vực nhân sự, đặc biệt là tuyển dụng và quản lý hồ sơ ứng viên. Một số tác vụ như sàng lọc CV, tổng hợp thông tin ứng viên hoặc quản lý dữ liệu tuyển dụng có thể được AI hỗ trợ để giảm thời gian xử lý thủ công.

Phần demo cho thấy AI, cụ thể là các công cụ như **Amazon Q**, không chỉ hữu ích cho developer mà còn có thể được áp dụng vào các bộ phận vận hành doanh nghiệp. Điều này giúp mình có góc nhìn rộng hơn rằng AI không chỉ phục vụ kỹ thuật, mà còn có thể tối ưu quy trình làm việc ở nhiều phòng ban khác nhau.

### Security, MCP Server và kiến trúc AWS private

Phần cuối của sự kiện đi sâu vào bảo mật và kiến trúc AWS, đặc biệt là cách triển khai một ứng dụng hoặc MCP Server theo hướng private để hạn chế dữ liệu đi qua internet công cộng. Đây là phần khá thực tế vì nó liên quan trực tiếp đến cách thiết kế hạ tầng an toàn cho hệ thống doanh nghiệp.

Một điểm được nhấn mạnh là không nên hard-code credential hoặc secret key trong source code. Khi tích hợp với các bên thứ ba, ví dụ như API bên ngoài, thông tin nhạy cảm nên được lưu và quản lý bằng **AWS Secrets Manager**. Cách làm này giúp hệ thống an toàn hơn và dễ quản lý hơn khi triển khai trong môi trường production.

Diễn giả cũng chia sẻ rằng thiết kế private trên AWS có thể an toàn hơn, nhưng chi phí đi kèm không hề nhỏ. Ví dụ:

- **Route 53 Resolver:** khoảng 180 USD/tháng  
- **Application Load Balancer:** khoảng 32 USD/tháng  
- Một module private cơ bản có thể rơi vào khoảng **250–350 USD/tháng**, chưa tính phí data transfer  

Điều mình rút ra là khi estimate chi phí cloud, không thể chỉ dựa vào số lượng user. Với một hệ thống có 500 người dùng truy vấn liên tục, kỹ sư cần làm việc với team Business hoặc Product để biết rõ dung lượng dữ liệu truyền tải mỗi tháng, ví dụ 5GB, 10GB hay nhiều hơn. Từ đó mới có thể tính toán chi phí hạ tầng chính xác hơn.

## Key Takeaways

### Tư duy thiết kế hệ thống

Điều quan trọng nhất mình học được là AI nên được xem như một công cụ tăng tốc, chứ không phải thứ có thể thay thế hoàn toàn vai trò của kỹ sư. Trong các hệ thống lớn, con người vẫn cần đưa ra quyết định kiến trúc, đánh giá rủi ro, xử lý sự cố và cân bằng giữa chi phí, hiệu năng, bảo mật.

Ngoài ra, khi thiết kế hệ thống cloud, mình cần nghĩ đến độ phức tạp ngay từ đầu. Hệ thống càng phát triển thì càng khó kiểm soát, vì vậy việc chia module rõ ràng, quản lý dependency và thiết kế kiến trúc dễ mở rộng là rất cần thiết.

### Kiến thức kỹ thuật

Sự kiện giúp mình hiểu thêm về nhiều hướng ứng dụng AI trong thực tế:

- Voice AI cần xử lý nhiều bước trước khi đưa dữ liệu vào LLM, đặc biệt với tiếng Việt.
- DevOps Agent có thể hỗ trợ phân tích log và giảm thời gian xử lý incident.
- AI có thể hỗ trợ các quy trình nghiệp vụ như tuyển dụng và quản lý hồ sơ ứng viên.
- AWS private architecture giúp tăng bảo mật nhưng cần đánh đổi bằng chi phí hạ tầng cao hơn.
- Credential nên được quản lý bằng dịch vụ chuyên dụng như AWS Secrets Manager thay vì hard-code.

### FinOps và Security

Một bài học lớn khác là **FinOps** và **Security** không nên được xem là phần làm sau cùng. Khi vẽ kiến trúc, kỹ sư cần nghĩ trước hệ thống sẽ tốn bao nhiêu chi phí, dữ liệu đi qua đâu, credential được lưu như thế nào và có cần private connectivity hay không.

Nếu không tính toán từ đầu, hệ thống có thể an toàn hơn nhưng lại phát sinh chi phí cao không cần thiết. Ngược lại, nếu chỉ tập trung tiết kiệm chi phí mà bỏ qua bảo mật, hệ thống có thể gặp rủi ro lớn khi đi vào vận hành thực tế.

## Áp dụng vào công việc

Sau sự kiện, mình có thể áp dụng một số bài học vào các dự án hiện tại hoặc tương lai:

- Khi thiết kế hạ tầng cloud, cần kiểm tra kỹ các thành phần networking như private endpoint, Route 53 Resolver, NAT Gateway hoặc ALB để tránh phát sinh chi phí không cần thiết.
- Khi làm việc với credential hoặc API key, nên sử dụng Secrets Manager hoặc các giải pháp quản lý secret thay vì lưu trực tiếp trong source code.
- Khi estimate chi phí AWS, cần hỏi rõ workload, data volume và tần suất sử dụng, thay vì chỉ dựa vào số lượng user.
- Với các ứng dụng AI, đặc biệt là Voice AI, cần chú ý đến dữ liệu huấn luyện, ngôn ngữ, vùng miền và khả năng kiểm soát output của mô hình.
- Có thể tận dụng AI để hỗ trợ đọc log, phân tích lỗi, kiểm tra bảo mật và tối ưu chi phí cloud.

## Trải nghiệm cá nhân

Tham gia **FCAJ Community Day - June 2026** là một trải nghiệm có giá trị với mình vì sự kiện mang lại góc nhìn rất thực tế về cách AI và cloud đang được dùng trong doanh nghiệp. Nội dung không chỉ xoay quanh xu hướng, mà còn đi vào những vấn đề kỹ thuật cụ thể như chi phí AWS, bảo mật private connection, xử lý Voice AI tiếng Việt và tự động hóa DevOps.

Mình đặc biệt ấn tượng với phần Voice AI vì nó cho thấy khi đưa AI vào thực tế, nhất là với tiếng Việt, có rất nhiều vấn đề cần giải quyết phía sau. Việc hệ thống phải chuyển từ giọng nói sang tiếng Việt, rồi xử lý tiếp sang tiếng Anh trước khi đưa vào LLM giúp mình hiểu rằng AI application không đơn giản là gọi một model và nhận kết quả. Nó là cả một pipeline cần được thiết kế cẩn thận.

Ngoài ra, phần AWS Architecture cũng giúp mình thay đổi cách nhìn về chi phí cloud. Trước đây mình thường nghĩ compute là phần chi phí chính, nhưng qua chia sẻ của diễn giả, mình nhận ra networking, private connectivity và data transfer cũng có thể chiếm chi phí đáng kể.

## Bài học rút ra

Sau sự kiện, mình rút ra một số bài học chính:

- AI có thể giúp kỹ sư làm việc nhanh hơn, nhưng tư duy hệ thống và khả năng ra quyết định vẫn rất quan trọng.
- Voice AI cho tiếng Việt cần xử lý tốt các vấn đề về dữ liệu, vùng miền, ngôn ngữ và độ chính xác.
- DevOps Agent là một hướng ứng dụng AI thực tế để giảm thời gian xử lý sự cố.
- Cloud architecture cần cân bằng giữa bảo mật, chi phí và khả năng mở rộng.
- Khi thiết kế hệ thống, cần đưa FinOps và Security vào từ đầu thay vì xử lý sau khi triển khai.
- Việc đặt câu hỏi trong sự kiện giúp mình hiểu sâu hơn về cách hệ thống Voice AI xử lý giọng vùng miền và các đặc trưng giọng nói như giới tính.

## Hình ảnh hoặc video chứng minh tham gia

- **Hình ảnh tham gia:** *Thêm hình ảnh check-in, ảnh chụp tại sự kiện hoặc screenshot video tại đây.*  

> Nhìn chung, FCAJ Community Day giúp mình có cái nhìn rõ hơn về việc ứng dụng AI trong hệ thống thực tế. Sự kiện cũng giúp mình hiểu rằng để trở thành một kỹ sư cloud hoặc DevOps tốt, mình không chỉ cần biết công cụ, mà còn cần biết cách đánh giá kiến trúc, chi phí, bảo mật và giá trị kinh doanh của hệ thống.

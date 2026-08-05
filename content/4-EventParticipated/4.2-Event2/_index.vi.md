---
title: "AWS FCAJ Agent Forge - Deepdive"
date: 2026-08-03
weight: 2
chapter: false
pre: " <b> 4.2. </b> "
---

# Báo cáo tóm tắt: "AWS FCAJ Agent Forge - Deepdive"

### Thông tin sự kiện

- **Tên sự kiện:** AWS FCAJ Agent Forge - Deepdive
- **Thời gian:** 9:00 sáng 03/08/2026
- **Địa điểm:** Sự kiện Workshop Offline kết hợp Hands-on Lab, tham gia tại tầng 26 tòa nhà Bitexco
- **Vai trò:** Người tham dự, lắng nghe lý thuyết chuyên sâu và trực tiếp thực hành trên hệ thống

### Mục tiêu sự kiện

AWS FCAJ Agent Forge là một workshop chuyên sâu cấp độ L300 (Advanced), được thiết kế dành riêng cho các kỹ sư và doanh nghiệp muốn đi sâu vào thế giới **Agentic AI**. Không giống những buổi chia sẻ xu hướng thông thường, sự kiện này đặt mục tiêu rõ ràng là hướng dẫn người tham dự cách thiết kế và triển khai một hệ thống AI Agent bài bản, từ giai đoạn Proof of Concept cho đến khi sẵn sàng đưa vào môi trường Production thực tế.

Toàn bộ nội dung xoay quanh nền tảng **Amazon Bedrock Agent Core** và các bài toán khó nhất mà doanh nghiệp phải đối mặt khi đưa AI vào vận hành, bao gồm: hiệu suất xử lý, khả năng mở rộng khi hệ thống lớn dần, bảo mật dữ liệu người dùng và cách quản trị quyền hạn của AI sao cho không mất kiểm soát.

### Diễn giả

- **Anh Nghĩa** – Phụ trách phần lý thuyết chuyên sâu: Kiến trúc hệ thống, Bedrock Agent Core, Runtime Environment, Identity & Security và Gateway
- **Anh Hải Anh** – Phụ trách dẫn dắt Hands-on Lab, hướng dẫn viết code và thiết lập hệ thống Agent thực tế

### Nội dung chính

#### Agentic AI là gì và tại sao nó khác với AI thông thường

Phần mở đầu của anh Nghĩa giúp mình hiểu rõ hơn về định nghĩa của **Agentic AI** — không chỉ đơn giản là một chatbot trả lời câu hỏi, mà là một hệ thống phần mềm có khả năng tự nhận thức (reasoning), tự lập kế hoạch (planning) và thực thi nhiệm vụ theo từng bước mà không cần con người can thiệp liên tục.

Điều mình thấy thú vị là mức độ tự chủ của một Agent có thể trải dài trên một dải rất rộng: từ luồng công việc được định sẵn hoàn toàn (deterministic workflow) — tức là AI chỉ làm đúng những gì đã được lập trình — cho đến hoàn toàn tự chủ đa tác vụ (fully autonomous multi-agent) — tức là nhiều Agent phối hợp với nhau để xử lý một bài toán phức tạp mà không cần hướng dẫn từng bước.

Tuy nhiên, diễn giả nhấn mạnh rất rõ: **trong môi trường doanh nghiệp, không bao giờ nên trao quyền tự chủ hoàn toàn cho AI**. Điều đó quá rủi ro. Thay vào đó, kỹ sư phải thiết kế cơ chế **Human-in-the-loop** — con người luôn đóng vai trò là người kiểm soát, phê duyệt hoặc can thiệp vào các quyết định quan trọng. Mình thấy điểm này rất thực tế, vì nếu AI tự quyết định hết mọi thứ mà không có ai kiểm tra, rủi ro sai sót sẽ rất cao và rất khó truy vết nguyên nhân.

Một điểm khác cũng được đề cập là về giao thức giao tiếp. Khác với REST API truyền thống mà developer đã quen, hệ thống Agent sử dụng hai chuẩn mới:
- **MCP (Model Context Protocol):** để Agent giao tiếp với các công cụ (Tools)
- **A2A (Agent to Agent):** để các Agent trong hệ thống nói chuyện với nhau

Mình chưa từng làm việc nhiều với hai chuẩn này nên đây là lần đầu mình hiểu được rõ tại sao chúng cần tồn tại song song với REST API.

#### Amazon Bedrock Agent Core — nền tảng để xây dựng Agent bài bản

Phần tiếp theo đi vào chi tiết về **Amazon Bedrock Agent Core**, đây là dịch vụ AWS giúp kỹ sư không cần phải tự xây dựng toàn bộ hạ tầng cho Agent từ đầu. Anh Nghĩa giải thích rằng để khởi tạo một Agent cơ bản, chỉ cần 3 thứ: một **Model (bộ não)**, một **System Prompt** và một bộ **Tools**. Nhưng nếu tự làm từ scratch, việc đảm bảo các tiêu chuẩn bảo mật và vận hành của ngành là cực kỳ phức tạp. Bedrock Agent Core ra đời để đóng gói và quản lý toàn bộ vòng đời của Agent một cách chuẩn mực.

Một nội dung mình thấy rất hữu ích là phần **chiến lược chọn Model**. Với họ Claude của Anthropic:
- **Claude Haiku** — phù hợp cho các truy vấn cần tốc độ phản hồi nhanh, chi phí thấp
- **Claude Sonnet** — điểm cân bằng giữa tốc độ và độ thông minh, phù hợp đa số use case
- **Claude Opus** — dành cho các tác vụ kỹ thuật phức tạp cần suy luận sâu, nhưng chi phí và thời gian phản hồi cao hơn đáng kể

Đây là bài toán đánh đổi (trade-off) mà kỹ sư phải cân nhắc kỹ: chọn model "thông minh" hơn đồng nghĩa với chi phí cao hơn và thời gian phản hồi lâu hơn. Không phải lúc nào cũng cần dùng model mạnh nhất, quan trọng là chọn đúng model cho đúng bài toán.

#### Runtime Environment — hạ tầng chạy Agent và bảo mật ở tầng phần cứng

Đây là phần mình thấy "wow" nhất trong cả buổi sáng. **Runtime Environment** là môi trường serverless mà AWS cung cấp để chạy Agent, hoạt động theo mô hình pay-as-you-go và hỗ trợ triển khai linh hoạt qua Docker/ECR hoặc file nén ZIP đẩy lên S3.

Nhưng điều đặc biệt là công nghệ bảo mật lõi phía sau: **Firecracker MicroVM**. Mình chưa từng nghe đến cái tên này trước buổi hôm nay, nhưng sau khi nghe giải thích xong, mình hiểu đây là lý do vì sao các ông lớn công nghệ có thể phục vụ hàng triệu người dùng cùng lúc mà vẫn đảm bảo dữ liệu của người này không bị lộ sang người khác.

Ý tưởng đơn giản nhưng mạnh: mỗi phiên làm việc (session) của người dùng được chạy trong một **máy ảo siêu nhỏ (MicroVM) độc lập về phần cứng và tài nguyên**. Người dùng A và người dùng B chạy trong hai MicroVM hoàn toàn tách biệt, không có cách nào để dữ liệu hay kết quả của phiên này rò rỉ sang phiên kia. Trước đây khi dùng API của OpenAI hay Gemini, mình không bao giờ nghĩ tới câu hỏi "liệu dữ liệu của mình có bị trộn với người khác không" — nhưng giờ mình mới hiểu rằng bên dưới, họ phải giải quyết bài toán này bằng công nghệ như Firecracker.

Ngoài ra, phần này cũng đề cập đến một vấn đề trải nghiệm người dùng rất thực tế: **nếu Agent xử lý tác vụ phức tạp mà buộc người dùng phải ngồi chờ màn hình trắng, trải nghiệm sẽ rất tệ**. Giải pháp là thiết kế theo hướng bất đồng bộ (asynchronous) hoặc long-running background jobs — Agent xử lý ngầm, trả về kết quả ngay khi có, thay vì bắt người dùng chờ toàn bộ pipeline hoàn thành mới hiển thị.

#### Identity & Security — bảo mật không chỉ là IAM Role

Phần bảo mật là phần mình thấy phức tạp nhất nhưng cũng quan trọng nhất. Anh Nghĩa giải thích rằng trong một hệ thống Agent, bảo mật cần được thiết kế thành một lớp riêng biệt gọi là **Identity**, chịu trách nhiệm xác thực (authentication) và phân quyền (authorization) theo cả hai chiều:

- **Inbound:** người dùng có được phép truy cập Agent không?
- **Outbound:** Agent có được phép gọi ra các Tools hay dịch vụ bên ngoài không?

Điểm đặc biệt là cơ chế **Token Vault**. Hệ thống được thiết kế để AI không bao giờ nhìn thấy trực tiếp API key hay thông tin đăng nhập nhạy cảm. Thay vào đó, quy trình chuyển đổi diễn ra như sau:

1. Người dùng đăng nhập và nhận **JWT (JSON Web Token)**
2. JWT này được đổi thành **WAT (Workload Access Token)** — là sự kết hợp giữa token người dùng và token của Agent
3. WAT sau đó mới được đổi thành **OAuth Credential** của dịch vụ bên thứ ba để thực hiện hành động

Cách làm này đảm bảo rằng AI chỉ được cấp quyền vừa đủ để làm việc cần làm, và không bao giờ có thể tự ý truy cập vào những dịch vụ mà nó không được phép. Diễn giả cũng nhắc đến **AWS Cognito** như một giải pháp đơn giản để phát hành JWT, giúp kỹ sư không cần tự thiết kế luồng xác thực từ đầu.

Mình rút ra được rằng: phân quyền trong hệ thống AI không đơn giản chỉ là gán IAM Role. Nó là cả một chuỗi chuyển đổi token phức tạp được thiết kế có chủ đích để giữ bí mật thông tin và kiểm soát hành động của AI ở từng bước.

#### Gateway — lớp trung gian không thể thiếu khi hệ thống mở rộng

Phần cuối của buổi lý thuyết là về **Gateway**, và đây là phần giúp mình hiểu vì sao cần có middleware khi hệ thống Agent bắt đầu lớn dần.

Hãy tưởng tượng một doanh nghiệp có hàng trăm Agent và hàng nghìn công cụ khác nhau. Nếu mỗi Agent phải tự kết nối trực tiếp (point-to-point) với từng Tool, việc quản lý và bảo mật sẽ trở thành ác mộng. Gateway ra đời để giải quyết bài toán này bằng cách trở thành một **điểm trung tâm** điều phối toàn bộ luồng dữ liệu.

Một tính năng mình thấy thông minh là việc Gateway tận dụng giao thức **MCP kết hợp với Semantic Search** để giúp Agent tự động tìm và chọn đúng Tool cần dùng thay vì phải hard-code từng kết nối. Về cơ bản, mỗi Tool được bọc lại bằng một đoạn mô tả (Schema), và khi Agent cần làm gì đó, nó sẽ tự tìm kiếm ngữ nghĩa để chọn Tool phù hợp — giống như tìm kiếm Google hơn là tra cứu danh bạ điện thoại.

Phần Gateway cũng đề cập đến cách triển khai **Human-in-the-loop ở quy mô lớn** thông qua Guardrails (rào cản chính sách). Ví dụ thực tế được đưa ra: Agent được phép tự động hoàn tiền cho khách hàng với giao dịch dưới 100 đô. Nhưng nếu khách yêu cầu hoàn 200 đô, Agent không thể tự quyết — Gateway sẽ đẩy yêu cầu đó lên cho nhân viên admin xem xét và phê duyệt trước khi thực hiện. Cách này giúp quy trình không bị cứng nhắc mà vẫn đảm bảo kiểm soát được rủi ro.

Ngoài ra, Gateway còn hỗ trợ:
- **Interceptor (Hook):** tự động rà quét và che giấu dữ liệu cá nhân nhạy cảm (PII) trước khi trả phản hồi cho người dùng
- **AWS PrivateLink:** kết nối an toàn với mạng nội bộ (on-premise) mà không bị lộ ra internet công cộng

### Key Takeaways

#### Tư duy thiết kế hệ thống

Bài học lớn nhất mình mang về từ sự kiện này là **làm AI trong doanh nghiệp không phải là bài toán Prompting — đó là bài toán System Architecture**. Trước đây mình thường nghĩ rằng để AI hoạt động tốt, chỉ cần viết prompt rõ ràng là xong. Nhưng sau buổi này mình mới thấy rằng phía sau một con Agent, có hàng chục lớp kỹ thuật cần được thiết kế bài bản: từ môi trường chạy, cơ chế xác thực, phân quyền, bảo mật dữ liệu, cho đến cách Agent giao tiếp với Tools và các Agent khác.

Quan trọng hơn, **AI không bao giờ được hoạt động mà không có người giám sát** trong môi trường doanh nghiệp. Human-in-the-loop và Gateway không phải là tùy chọn, mà là bắt buộc để đảm bảo hệ thống không vượt tầm kiểm soát.

#### Kiến thức kỹ thuật

Sự kiện này giúp mình hiểu thêm rất nhiều về cách xây dựng Agentic AI một cách đúng đắn:

- **Firecracker MicroVM** là công nghệ cô lập session ở tầng phần cứng, đảm bảo dữ liệu người dùng không bao giờ rò rỉ giữa các phiên làm việc.
- **MCP (Model Context Protocol)** đang dần thay thế REST API thông thường trong việc kết nối Agent với Tools, và Semantic Search giúp Agent tự tìm đúng Tool mà không cần hard-code.
- **Token Vault và chuỗi JWT → WAT → OAuth Credential** là cơ chế bảo mật quan trọng để ẩn hoàn toàn secret key khỏi AI.
- **Asynchronous processing** là cần thiết để tránh bắt người dùng chờ kết quả trong các tác vụ phức tạp.
- **Guardrails trong Gateway** là cách thiết kế Human-in-the-loop một cách có hệ thống và có thể mở rộng.

#### FinOps và Security

- Serverless Runtime là lựa chọn tối ưu chi phí cho Agent vì chỉ tính tiền khi có traffic thực sự.
- Bảo mật không dừng ở IAM Role — cần thiết kế toàn bộ luồng chuyển đổi token để AI không bao giờ tiếp xúc trực tiếp với credential nhạy cảm.
- Khi cần kết nối với hệ thống on-premise, **AWS PrivateLink** là lựa chọn bắt buộc thay vì dùng Internet Gateway, vừa bảo mật hơn vừa tránh rò rỉ dữ liệu ra ngoài.

### Áp dụng vào công việc

Sau sự kiện, mình nhận ra có một số điều có thể áp dụng ngay vào công việc thực tế:

- **Bảo vệ dữ liệu người dùng:** Khi xây dựng bất kỳ hệ thống AI nào, mình sẽ thiết kế thêm lớp Interceptor hoặc Guardrail ở middleware để tự động rà quét và che giấu thông tin nhạy cảm (PII) trước khi trả phản hồi cho người dùng.
- **Quản lý API Key đúng cách:** Thay vì nhúng API key trực tiếp vào code, mình sẽ tham khảo mô hình Token Vault và Workload Access Token để cấp phát quyền hạn một cách trung gian và an toàn hơn.
- **Cải thiện trải nghiệm người dùng:** Áp dụng luồng xử lý bất đồng bộ (async) cho các tác vụ AI phức tạp, tránh việc bắt người dùng ngồi chờ màn hình trắng.
- **Thiết kế Agent có kiểm soát:** Khi triển khai Agent, luôn xây dựng Guardrails rõ ràng để giới hạn phạm vi hành động của AI, đặc biệt với các tác vụ có ảnh hưởng thực tế như giao dịch tài chính hay thay đổi dữ liệu.
- **Chọn Model phù hợp:** Không phải lúc nào cũng cần dùng model mạnh nhất. Cần phân tích rõ yêu cầu của từng tác vụ để chọn Claude Haiku, Sonnet hay Opus cho đúng với bài toán, tránh lãng phí chi phí không cần thiết.

### Trải nghiệm cá nhân

Tham gia **AWS FCAJ Agent Forge - Deepdive** là một trải nghiệm khá "nặng đô" theo nghĩa tích cực. Gần 100 slide được trình bày chỉ trong buổi sáng, nhưng nội dung được thiết kế theo hướng đi từ lý thuyết đến ví dụ thực tế rất mượt, không cảm giác bị nhồi nhét.

Điều mình ấn tượng nhất là phần giải thích về **Firecracker MicroVM**. Đây là lần đầu tiên mình hiểu được cách các nền tảng cloud lớn bảo vệ quyền riêng tư ở tầng hạ tầng sâu nhất — không phải bằng logic phần mềm, mà bằng cách cô lập hoàn toàn ở tầng phần cứng ảo hóa. Trước đây khi dùng ChatGPT hay các API AI, mình chưa bao giờ thắc mắc liệu dữ liệu của mình có bị trộn lẫn với người khác không. Giờ mình mới biết rằng để đảm bảo điều đó, cần cả một công nghệ riêng.

Phần Hands-on Lab do anh Hải Anh dẫn dắt cũng giúp mình vỡ ra nhiều điều. Thực hành trực tiếp trên hệ thống khiến mình hiểu được sự khác biệt giữa "biết lý thuyết" và "biết cách áp dụng". Có những đoạn code nhìn qua thì đơn giản nhưng khi thực sự gõ và chạy mới thấy được những chi tiết nhỏ quan trọng mà nếu chỉ nghe thuyết trình sẽ rất dễ bỏ qua.

### Bài học rút ra

Sau sự kiện, mình rút ra một số bài học mà mình nghĩ sẽ còn nhớ lâu:

- Làm AI cho doanh nghiệp không chỉ là Prompting hay Fine-tuning — đó là một bài toán System Design bao trùm cả Networking, Identity và Serverless Infrastructure.
- **Human-in-the-loop không phải là tính năng thêm vào sau**, mà cần được thiết kế ngay từ đầu như một phần cốt lõi của kiến trúc.
- Việc chọn đúng model (Haiku / Sonnet / Opus) quan trọng không kém gì viết Prompt tốt — sai model có thể làm hệ thống chậm hoặc tốn chi phí không cần thiết.
- Bảo mật trong hệ thống Agent không phải là gán IAM Role rồi xong — cần thiết kế toàn bộ chuỗi token để AI không bao giờ thấy được secret key.
- **Firecracker MicroVM** giúp mình hiểu rằng cô lập dữ liệu thực sự có thể đạt được ở tầng phần cứng, không chỉ là lý thuyết bảo mật trên giấy.
- Khi hệ thống mở rộng, **Gateway và MCP** là lớp không thể thiếu để quản lý hàng trăm Agent và hàng nghìn Tool mà không mất kiểm soát.

### Hình ảnh hoặc video chứng minh tham gia

- **Hình ảnh tham gia sự kiện:**

![Event 2-1](/images/4-EventParticipated/EV2-1.png)
![Event 2-2](/images/4-EventParticipated/EV2-2.png)
![Event 2-3](/images/4-EventParticipated/EV2-3.png)
![Event 2-4](/images/4-EventParticipated/EV2-4.png)
![Event 2-5](/images/4-EventParticipated/EV2-5.png)

> Nhìn chung, AWS FCAJ Agent Forge giúp mình hiểu được rằng để xây dựng một hệ thống AI Agent thực sự hoạt động tốt trong doanh nghiệp, cần rất nhiều thứ hơn là một model thông minh. Phía sau là cả một hệ thống bảo mật, phân quyền, cô lập dữ liệu và quản trị rủi ro được thiết kế cẩn thận. Sự kiện này giúp mình thay đổi cách nhìn từ "làm AI" sang "thiết kế hệ thống AI" — và đó là sự khác biệt rất lớn.
---
title: "AWS FCAJ Agent Forge - Hands-on AgentCore (Deepdive day 3)"
date: 2026-08-15
weight: 4
chapter: false
pre: " <b> 4.3. </b> "
---

# Báo cáo tóm tắt: "AWS FCAJ Agent Forge - Hands-on AgentCore (Buổi 3)"

### Thông tin sự kiện

- **Tên sự kiện:** AWS FCAJ Agent Forge - Buổi 3 (Hands-on Lab & Case Study)
- **Hình thức:** Workshop kết hợp thực hành trực tiếp trên hệ thống (hands-on lab)
- **Chủ đề trọng tâm:** Triển khai thực tế các thành phần cốt lõi của **AWS Bedrock AgentCore**
- **Vai trò:** Người tham dự, vừa nghe phần case study vừa trực tiếp code theo hướng dẫn

### Mục tiêu sự kiện

Nếu như những buổi trước của chuỗi **AWS FCAJ Agent Forge** thiên về lý thuyết kiến trúc và tư duy thiết kế hệ thống Agentic AI, thì buổi thứ ba này chuyển hẳn sang hướng **thực chiến**. Mục tiêu rõ ràng của buổi học là giúp người tham dự không chỉ "hiểu" mà còn "làm được" — tức là tự tay triển khai từng thành phần trong hệ sinh thái **AgentCore**, từ bộ nhớ (Memory), cổng kết nối (Gateway), cho đến giao diện người dùng và hệ thống giám sát (Observability).

Buổi học đặt ra một thông điệp xuyên suốt: xây dựng AI Agent không chỉ là ghép các công cụ lại với nhau, mà là hiểu cách các thành phần phối hợp với nhau trong một kiến trúc hoàn chỉnh, đồng thời biết cách áp dụng đúng vào bài toán thực tế của doanh nghiệp.

### Nội dung chính

#### Tổng quan kiến trúc Agentic AI với AgentCore

Phần mở đầu giúp mình hình dung lại toàn cảnh kiến trúc của một hệ thống Agent hiện đại. Điểm mấu chốt nằm ở khái niệm **AgentCore Harness** — mình hiểu đây như một "khung xương" hay "cơ thể" bao bọc lấy bộ não AI. Nếu mô hình ngôn ngữ (Model) là bộ não chịu trách nhiệm suy luận, thì Harness chính là phần cơ thể giúp bộ não đó thực sự làm được việc: kết nối với môi trường thực thi mã (Code Interpreter), trình duyệt web (Browser), và các công cụ mở rộng khác.

Điều mình thấy tâm đắc nhất trong phần này là triết lý thiết kế **"Everything is a plugin"** (mọi thứ đều là plugin). Thay vì xây dựng một hệ thống cứng nhắc, đóng khối, kiến trúc AgentCore được module hóa hoàn toàn. Nghĩa là mỗi năng lực của Agent — dù là khả năng chạy code, duyệt web hay gọi một API bên ngoài — đều được thiết kế như một mảnh ghép có thể tháo lắp linh hoạt. Cách tiếp cận này giúp việc mở rộng, tùy biến và bảo trì hệ thống trở nên dễ dàng hơn rất nhiều, đặc biệt khi bài toán doanh nghiệp thay đổi liên tục theo thời gian.

Mình nhận ra rằng tư duy plugin này không chỉ là một lựa chọn kỹ thuật, mà còn là một triết lý thiết kế bền vững: hệ thống nào càng dễ mở rộng và thay thế từng phần thì càng sống lâu và càng dễ thích nghi với thay đổi.

#### Case Study 1 — Hệ thống Multi-Agent cho QA và sửa lỗi tự động

Đây là một trong hai case study khiến mình hào hứng nhất. Diễn giả giới thiệu một kiến trúc **Multi-Agent** (nhiều Agent phối hợp) được áp dụng vào bài toán rất quen thuộc với dân kỹ thuật: kiểm thử phần mềm và sửa lỗi.

Ý tưởng là kết hợp hai Agent với vai trò tách biệt:
- **QA Agent** — chịu trách nhiệm kiểm thử phần mềm, phát hiện lỗi và các vấn đề tiềm ẩn trong code
- **Bug-fixing Agent** — tự động đề xuất và thực hiện việc sửa các lỗi được phát hiện

Điều mình thấy hợp lý là hệ thống này **không loại bỏ hoàn toàn con người** ra khỏi quy trình. Mặc dù hai Agent có thể xử lý phần lớn công việc lặp đi lặp lại và tốn thời gian, nhưng khâu kiểm soát chất lượng cuối cùng vẫn thuộc về con người — cụ thể là bước **Senior Review (Human-in-the-loop)**. Kỹ sư senior sẽ là người xem xét và phê duyệt trước khi các thay đổi được áp dụng chính thức.

Case study này cho mình thấy rất rõ giá trị thực tế của Multi-Agent: nó không thay thế con người, mà giải phóng con người khỏi những tác vụ nhàm chán để tập trung vào những quyết định quan trọng cần tư duy và kinh nghiệm. Đây chính xác là mô hình "con người + AI" mà mình nghĩ sẽ trở thành chuẩn mực trong tương lai gần.

#### Case Study 2 — Ứng dụng IoT & AI theo dõi chỉ số sinh học qua tín hiệu Wi-Fi

Case study thứ hai thì thực sự bất ngờ và mở mang tầm mắt của mình. Diễn giả giới thiệu một giải pháp ứng dụng **tín hiệu Wi-Fi** để theo dõi các chỉ số sinh học của con người như **nhịp tim và nhịp thở** — mà hoàn toàn **không cần dùng đến camera**.

Ý tưởng này thoạt nghe có vẻ khó tin, nhưng khi được giải thích, mình hiểu rằng chuyển động cực nhỏ của cơ thể (như lồng ngực phồng lên xẹp xuống khi thở, hay nhịp đập của tim) tạo ra những nhiễu loạn rất nhỏ trong sóng Wi-Fi lan truyền trong không gian. Bằng cách phân tích những nhiễu loạn đó với sự hỗ trợ của AI, hệ thống có thể suy ra được các chỉ số sinh học.

Điều mình thấy giá trị nhất ở giải pháp này là khía cạnh **quyền riêng tư**. Việc theo dõi sức khỏe mà không cần camera mở ra một hướng đi rất tiềm năng cho các ứng dụng nhà thông minh (smart home) và chăm sóc sức khỏe người cao tuổi — nơi mà việc lắp camera giám sát thường gây cảm giác khó chịu và xâm phạm sự riêng tư. Đây là một ví dụ tuyệt vời cho thấy AI khi kết hợp với IoT có thể giải quyết những bài toán thực tế theo cách mà trước đây mình chưa từng nghĩ tới.

#### Hands-on Lab — Triển khai từng thành phần của AgentCore

Đây là phần "nặng đô" và cũng bổ ích nhất của buổi học, khi cả nhóm bắt tay vào thực hành trực tiếp. Nội dung lab đi qua từng thành phần cốt lõi trong hệ sinh thái AgentCore:

**Cấu hình AgentCore Memory**
Mình học được cách thiết lập hệ thống bộ nhớ để lưu trữ và quản lý ngữ cảnh (context) cho Agent. Đây là một điểm cực kỳ quan trọng vì nếu không có memory, Agent sẽ "quên" mọi thứ sau mỗi lần tương tác, không thể duy trì được cuộc hội thoại liền mạch hay ghi nhớ những thông tin đã trao đổi trước đó. Việc quản lý context tốt chính là yếu tố phân biệt một Agent "thông minh thực sự" với một chatbot đơn giản.

**Sử dụng AgentCore Gateway**
Phần này hướng dẫn cách dùng Gateway để quản lý và kết nối các API, các công cụ bên ngoài, và đặc biệt là chuẩn giao thức **MCP (Model Context Protocol)**. Mình hiểu rằng Gateway đóng vai trò như một cổng trung tâm giúp Agent giao tiếp với thế giới bên ngoài một cách có tổ chức và an toàn, thay vì phải kết nối trực tiếp và rời rạc với từng công cụ.

**Phát triển giao diện với Streamlit**
Để biến một Agent thành sản phẩm mà người dùng có thể tương tác, cần có giao diện. Buổi lab sử dụng **Streamlit** — một công cụ giúp dựng nhanh giao diện web tương tác chỉ bằng Python, mà không cần kiến thức sâu về frontend. Mình thấy đây là lựa chọn rất hợp lý cho việc làm demo và prototype nhanh, giúp kiểm chứng ý tưởng mà không tốn quá nhiều thời gian vào phần giao diện.

**Tích hợp Observability**
Thành phần cuối cùng nhưng không kém phần quan trọng là **Observability** — khả năng giám sát. Mình học cách cấu hình các công cụ theo dõi hiệu suất của Agent, quan sát luồng xử lý bên trong (Agent đang gọi tool nào, suy luận ra sao), và theo dõi chi phí vận hành. Đây là phần mà mình nghĩ nhiều người mới thường bỏ qua, nhưng lại cực kỳ quan trọng khi đưa hệ thống vào production — bởi nếu không "nhìn thấy" được hệ thống đang làm gì, ta sẽ không thể debug hay tối ưu khi có sự cố.

### Key Takeaways

#### Tư duy thiết kế hệ thống

Bài học lớn nhất mình rút ra từ buổi này là **kiến trúc module hóa (plugin-based) là chìa khóa để xây dựng hệ thống AI bền vững**. Triết lý "Everything is a plugin" không chỉ giúp code dễ bảo trì hơn, mà còn giúp hệ thống linh hoạt thích nghi với những thay đổi liên tục của bài toán doanh nghiệp.

Bên cạnh đó, mô hình **Multi-Agent kết hợp Human-in-the-loop** cho thấy tương lai của AI trong doanh nghiệp không phải là thay thế con người, mà là hợp tác cùng con người — AI lo phần việc lặp lại, con người giữ quyền quyết định cuối cùng.

#### Kiến thức kỹ thuật

- **AgentCore Harness** là khung vận hành đóng vai trò "cơ thể" cho bộ não AI, kết nối Model với Code Interpreter, Browser và các tool khác.
- **AgentCore Memory** là thành phần quyết định khả năng duy trì ngữ cảnh của Agent qua các lượt tương tác.
- **AgentCore Gateway + MCP** giúp quản lý kết nối tới API và tool bên ngoài một cách tập trung và an toàn.
- **Streamlit** là công cụ tuyệt vời để dựng nhanh giao diện demo cho ứng dụng AI.
- **Observability** là yếu tố bắt buộc để vận hành Agent trong production, giúp giám sát hiệu suất, luồng xử lý và chi phí.

#### Ứng dụng thực tế

- Hệ thống Multi-Agent QA + Bug-fixing cho thấy AI có thể tự động hóa phần lớn quy trình kiểm thử và sửa lỗi, giải phóng kỹ sư khỏi các tác vụ nhàm chán.
- Ứng dụng theo dõi sinh học qua tín hiệu Wi-Fi mở ra hướng đi mới cho smart home và healthcare mà vẫn bảo vệ quyền riêng tư (không cần camera).

### Áp dụng vào công việc

Sau buổi học, mình nhận ra một số điều có thể áp dụng ngay:

- **Thiết kế theo hướng plugin:** Khi xây dựng bất kỳ hệ thống AI nào, mình sẽ ưu tiên kiến trúc module hóa để dễ mở rộng và bảo trì về sau.
- **Không bỏ qua Memory:** Với các ứng dụng cần hội thoại liền mạch, mình sẽ chú trọng thiết kế cơ chế quản lý context ngay từ đầu.
- **Đầu tư vào Observability sớm:** Thay vì chờ đến khi có sự cố mới lo giám sát, mình sẽ tích hợp Observability ngay từ giai đoạn phát triển.
- **Dùng Streamlit cho prototype:** Với các ý tưởng cần demo nhanh, mình sẽ dùng Streamlit để tiết kiệm thời gian dựng giao diện.
- **Luôn giữ Human-in-the-loop:** Với các tác vụ có tác động thực tế (như sửa code, giao dịch), mình sẽ luôn thiết kế bước phê duyệt của con người.

### Thông điệp cốt lõi

Điều mình mang về lớn nhất từ buổi học không chỉ là các kỹ thuật hay công cụ, mà là một nhận thức quan trọng: **bên cạnh việc làm chủ công cụ xây dựng AI Agent, người phát triển cần chú trọng vào tri thức chuyên ngành (Domain Knowledge)**. Công nghệ chỉ là phương tiện — muốn áp dụng AI giải quyết đúng và hiệu quả các bài toán thực tế của doanh nghiệp, ta phải thực sự hiểu sâu về chính bài toán đó. Một kỹ sư giỏi công cụ nhưng không hiểu bài toán sẽ khó tạo ra giá trị thực sự.

### Trải nghiệm cá nhân

Buổi 3 của **AWS FCAJ Agent Forge** để lại cho mình cảm giác rất "thực" so với những buổi thiên lý thuyết trước đó. Việc được trực tiếp gõ code và chạy thử trên hệ thống giúp mình hiểu ra sự khác biệt lớn giữa "biết" và "làm được". Có những chi tiết khi nghe giảng thì thấy đơn giản, nhưng lúc tự tay triển khai mới phát hiện ra vô số điểm nhỏ cần lưu ý.

Hai case study — đặc biệt là ứng dụng theo dõi sinh học qua Wi-Fi — thực sự truyền cảm hứng cho mình về việc AI có thể được ứng dụng sáng tạo đến mức nào khi kết hợp với các lĩnh vực khác như IoT. Nó nhắc mình rằng giới hạn của AI không nằm ở công nghệ, mà nằm ở trí tưởng tượng và sự hiểu biết về bài toán của người áp dụng.

### Bài học rút ra

- Xây dựng AI Agent là bài toán tích hợp nhiều thành phần (Memory, Gateway, UI, Observability), không chỉ là gọi một model.
- Kiến trúc plugin-based giúp hệ thống linh hoạt và sống lâu hơn.
- Human-in-the-loop vẫn là yếu tố cốt lõi ngay cả trong các hệ thống Multi-Agent tự động.
- Observability phải được coi trọng ngay từ đầu, không phải là thứ thêm vào sau.
- Domain Knowledge quan trọng không kém kỹ năng công nghệ — hiểu bài toán mới là chìa khóa để AI tạo ra giá trị thực.

### Hình ảnh hoặc video chứng minh tham gia

- **Hình ảnh tham gia sự kiện:**

![Event 4-1](/images/4-EventParticipated/EV4-1.png)
![Event 4-2](/images/4-EventParticipated/EV4-2.png)
![Event 4-3](/images/4-EventParticipated/EV4-3.png)
![Event 4-4](/images/4-EventParticipated/EV4-4.png)
![Event 4-5](/images/4-EventParticipated/EV4-5.png)

> Nhìn chung, buổi 3 của AWS FCAJ Agent Forge giúp mình chuyển từ tư duy "hiểu lý thuyết" sang "triển khai thực tế". Qua các bài lab về AgentCore và hai case study đầy cảm hứng, mình nhận ra rằng để xây dựng một hệ thống AI Agent thành công, cần cả sự vững vàng về kỹ thuật lẫn hiểu biết sâu sắc về bài toán thực tế — và quan trọng nhất, con người vẫn luôn giữ vai trò trung tâm trong việc kiểm soát và định hướng AI.

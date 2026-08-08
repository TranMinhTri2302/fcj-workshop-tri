---
title: "AWS FCAJ Agent Forge - Agent Core in Production (Deepdive day 2)"
date: 2026-08-08
weight: 3
chapter: false
pre: " <b> 4.3. </b> "
---

# Báo cáo tóm tắt: "AWS FCAJ Agent Forge – Agent Core in Production"

### Thông tin sự kiện

- **Tên sự kiện:** AWS FCAJ Agent Forge – Agent Core in Production (Session tiếp nối chuỗi Deepdive)
- **Thời gian:** 08/08/2026
- **Hình thức:** Sự kiện Workshop Offline kết hợp Hands-on Lab, tham gia tại tầng 26 tòa nhà Bitexco
- **Vai trò:** Người theo dõi và thực hành lại toàn bộ Hands-on Lab trên tài khoản AWS cá nhân

### Mục tiêu sự kiện

Nếu như session Deepdive trước đó tập trung trả lời câu hỏi **"Agentic AI là gì và kiến trúc nền tảng ra sao"**, thì session lần này đi thẳng vào câu hỏi khó hơn nhiều: **"Làm sao để đưa một AI Agent từ bản demo chạy trên máy cá nhân lên môi trường Production mà không mất kiểm soát?"**

Toàn bộ nội dung xoay quanh bốn trụ cột vận hành của **Amazon Bedrock Agent Core**:

1. **Memory** – Agent nhớ gì, nhớ như thế nào, và nhớ bao lâu
2. **Observability** – Làm sao nhìn thấy được bên trong "hộp đen" khi Agent chạy thật
3. **Evaluation** – Đo lường chất lượng câu trả lời bằng số liệu thay vì cảm tính
4. **Policy & Security** – Khóa quyền hạn của Agent lại trước khi nó làm điều dại dột

Ngoài ra, phần mở đầu còn có một nội dung mình không ngờ tới nhưng lại rất giá trị: **định hướng phát triển nghề nghiệp cho kỹ sư trẻ trong lĩnh vực Cloud/AI**. Và phần cuối là một bài **Hands-on Lab kéo dài gần 1 tiếng** để triển khai một Refund Assistant hoàn chỉnh.

---

### Nội dung chính

#### 1. Định hướng phát triển nghề nghiệp Cloud/AI

Phần này diễn ra ngay đầu session và mình nghĩ nó xứng đáng được ghi lại cẩn thận, vì đây là góc nhìn từ người đã đi làm lâu năm chứ không phải lời khuyên chung chung trên mạng.

**Mô hình kỹ năng chữ T (T-Shaped Skills)**

Diễn giả nhấn mạnh rằng lộ trình của một kỹ sư Cloud/AI không nên phát triển theo một đường thẳng, mà phải là hình chữ **T** — có cả chiều sâu và chiều rộng:

- **Năm đầu tiên (chiều dọc – Depth):** Đây là giai đoạn phải đào thật sâu vào một chuyên môn cốt lõi. Đừng học lan man. Chọn một mảng (ví dụ: hạ tầng serverless, hoặc data pipeline, hoặc model deployment) và làm cho tới nơi tới chốn.
- **Năm 2–3 (chiều ngang – Breadth):** Sau khi đã có nền, bắt đầu mở rộng sang các môi trường triển khai thực tế (**production**) — nơi mọi thứ khác hoàn toàn so với môi trường local. Song song đó là bồi đắp **domain knowledge** (kiến thức nghiệp vụ) trong những lĩnh vực cụ thể như **EdTech, HealthTech, FinTech**.

Một câu mình thấy khá "thấm": **bằng cấp và chứng chỉ AWS chỉ là điều kiện cần, chưa phải điều kiện đủ**. Chứng chỉ giúp mình lọt qua vòng hồ sơ, nhưng thứ giữ mình lại trong nghề là khả năng giải quyết bài toán thật của doanh nghiệp thật.

**Kỹ năng mềm và Đạo đức AI**

Phần này khiến mình phải suy nghĩ lại về cách mình đang đầu tư thời gian. Diễn giả nói rất rõ rằng **kỹ năng giao tiếp là kỹ năng sống còn** — cụ thể là khả năng diễn giải một khái niệm kỹ thuật phức tạp cho những người **non-technical** (khách hàng, quản lý, bộ phận kinh doanh) hiểu được.

Mình liên hệ ngay tới thực tế: nhiều khi mình hiểu rất rõ vấn đề trong đầu, nhưng khi phải giải thích cho người không cùng chuyên môn thì lại bí. Mà trong doanh nghiệp, nếu không thuyết phục được người ra quyết định thì giải pháp hay đến mấy cũng không được duyệt.

Hai yếu tố khác được nhấn mạnh:

- **Trách nhiệm khi sử dụng AI (Responsible AI):** Cần có tư duy về việc AI mình xây ra có an toàn không, có thiên lệch không, có gây hại cho người dùng không.
- **Tinh thần làm chủ (Ownership):** Không chỉ làm xong task rồi thôi, mà phải coi hệ thống mình xây như của mình — chịu trách nhiệm từ lúc viết code đến lúc nó chạy trên production và cả khi nó lỗi lúc 2 giờ sáng.

**Cơ hội cọ xát thực tế**

Lời khuyên cụ thể nhất: tham gia các cuộc thi **Hackathon** do doanh nghiệp và startup tổ chức. Lý do rất thực tế — Hackathon là môi trường buộc mình phải biến kiến thức "chay" (chỉ đọc trên tài liệu) thành một sản phẩm chạy được trong thời gian giới hạn. Đó là quá trình mà việc học qua khóa online không thể thay thế.

> **Cảm nhận cá nhân:** Mình từng nghĩ cứ học nhiều chứng chỉ là đủ. Sau phần này mình nhận ra tấm bằng chỉ chứng minh mình *biết*, còn sản phẩm và dự án mới chứng minh mình *làm được*. Kế hoạch của mình sắp tới là chọn một domain cụ thể để đào sâu thay vì học dàn trải như hiện tại.

---

#### 2. Kiến trúc cốt lõi của Agent Core

**Ranh giới giữa Chatbot và AI Agent**

Đây là phần định nghĩa lại một thứ mà mình tưởng mình đã hiểu. Diễn giả phân biệt rất rõ:

| Tiêu chí | Chatbot thông thường | AI Agent |
|---|---|---|
| Đầu ra | Chỉ trả về văn bản | Có thể thực thi hành động thật |
| Khả năng | Hỏi – Đáp | Tự chủ lập kế hoạch và hành động |
| Công cụ | Không có | Gọi được Tools bên ngoài |
| Trạng thái | Thường không nhớ | Có Memory ngắn hạn và dài hạn |

Nói ngắn gọn: **Chatbot nói cho bạn biết phải làm gì. Agent thì tự đi làm việc đó.**

**Vòng lặp nhận thức: Thinking – Reasoning – Tooling**

Diễn giả giải thích vòng lặp mà một Agent thực hiện mỗi khi nhận yêu cầu:

1. **Reasoning (Suy luận):** Agent phân tích ngữ cảnh — người dùng thực sự đang muốn gì? Thông tin nào đã có, thông tin nào còn thiếu?
2. **Thinking (Lập kế hoạch):** Agent vạch ra các bước cần làm để giải quyết yêu cầu. Bước nào làm trước, bước nào làm sau, cần dữ liệu gì ở mỗi bước.
3. **Tool Use (Sử dụng công cụ):** Agent tự động gọi các công cụ bên ngoài — có thể là truy vấn database, gọi API, tra cứu internet — để thu thập dữ liệu hoặc thực thi hành động.

Vòng lặp này lặp đi lặp lại cho tới khi nhiệm vụ hoàn thành. Điểm quan trọng là **Agent tự quyết định gọi Tool nào và gọi khi nào**, chứ không phải lập trình viên hard-code từng bước.

> **Cảm nhận cá nhân:** Điều làm mình thay đổi cách nghĩ là: khi build Agent, việc của mình không còn là viết logic "nếu A thì làm B" nữa, mà là **mô tả thật rõ các Tool có sẵn và để Agent tự chọn**. Đây là sự dịch chuyển tư duy khá lớn từ lập trình mệnh lệnh (imperative) sang lập trình khai báo (declarative).

---

#### 3. Memory – Quản trị Bộ nhớ của Agent

Đây là phần mình thấy giá trị nhất về mặt kiến trúc trong cả session, vì nó giải thích cụ thể cách một Agent "nhớ" — thứ mà trước đây mình chỉ hiểu mơ hồ là "nhét lịch sử chat vào prompt".

**Trí nhớ ngắn hạn (Short-term Memory)**

Short-term Memory quản lý **trạng thái tức thời ngay trong một phiên làm việc (session)** để giữ được mạch hội thoại liền lạc.

Ví dụ được đưa ra rất dễ hiểu — một kịch bản mua giày chạy bộ:

- Người dùng: "Tôi muốn mua giày chạy bộ"
- Agent hỏi thêm: Hãng nào? → **N**
- Agent hỏi tiếp: Màu gì? → **Đen**
- Agent hỏi tiếp: Size bao nhiêu? → **42**

Toàn bộ các tin nhắn này được lưu dưới dạng **raw text** (văn bản thô, chưa qua xử lý) và được xử lý **đồng bộ (synchronous)** — nghĩa là Agent phải đọc xong toàn bộ ngữ cảnh này rồi mới phản hồi. Chính vì đồng bộ nên nó cho tốc độ phản hồi ngay lập tức, nhưng đổi lại là **không thể nhét quá nhiều** vào (giới hạn context window và chi phí token).

**Trí nhớ dài hạn (Long-term Memory)**

Điểm mình thích nhất ở đây: **lập trình viên không cần tự build từ đầu**. Bedrock Agent Core cung cấp sẵn tính năng này.

Cơ chế hoạt động:

- Có một module tên là **Memory Extraction** chạy **ngầm song song (asynchronously)** với luồng hội thoại chính.
- Module này tự động đọc luồng chat ngắn hạn, **trích xuất ra các thông tin quan trọng** (key insights / knowledge) — ví dụ: "khách hàng này thích hãng N", "size chân là 42", "ưu tiên màu tối".
- Những thông tin cô đọng đó được đẩy vào kho lưu trữ dài hạn.

Điểm hay của việc chạy **bất đồng bộ** là quá trình trích xuất này **không làm chậm phản hồi** cho người dùng. Người dùng vẫn nhận câu trả lời ngay, còn việc "ghi nhớ" diễn ra ở background.

**Memory Strategy – Bài toán đánh đổi**

Hệ thống cung cấp **nhiều chiến lược (strategy) khác nhau** để chuyển đổi dữ liệu từ ngắn hạn sang dài hạn. Mỗi chiến lược có ưu và nhược điểm riêng:

- Chiến lược lưu chi tiết nhiều → Agent nhớ tốt hơn nhưng **tốn chi phí lưu trữ và token** hơn
- Chiến lược tóm tắt gọn → Rẻ hơn, nhanh hơn nhưng **có thể mất thông tin quan trọng**

Diễn giả nhấn mạnh: **không có chiến lược nào là đúng tuyệt đối**. Kỹ sư phải dựa vào bài toán cụ thể để **trade-off** giữa chất lượng ngữ cảnh và chi phí vận hành.

> **Cảm nhận cá nhân:** Trước đây mình cứ nghĩ Memory là cứ nhét hết lịch sử chat vào prompt là xong. Giờ mình mới hiểu đó là cách làm vừa đắt vừa không hiệu quả. Kiến trúc hai tầng (ngắn hạn đồng bộ – dài hạn bất đồng bộ) là một pattern rất đáng học và mình sẽ áp dụng ngay vào project cá nhân.

---

#### 4. Observability – Giám sát Hệ thống

**Giải quyết bài toán "Hộp đen"**

Diễn giả mở đầu bằng một câu rất đúng: **khi chạy trên môi trường Production, không thể nhắm mắt để AI tự chạy**. Khác với một API thông thường mà input–output rõ ràng, AI Agent có thể tự quyết định gọi Tool này, bỏ qua Tool kia, suy luận theo hướng không ai lường trước. Nếu không nhìn thấy được bên trong, khi có sự cố sẽ không thể biết Agent đã sai ở bước nào.

Giải pháp gồm hai cơ chế:

- **Logging:** Ghi lại toàn bộ nội dung tương tác — người dùng hỏi gì, Agent trả lời gì, gọi Tool nào với tham số ra sao.
- **Tracing:** Theo dõi **toàn bộ vòng đời của một request** — từ lúc người dùng gửi yêu cầu, qua từng bước reasoning, từng lần gọi Tool, cho tới khi trả về kết quả cuối cùng. Đây là thứ cho phép mình trả lời câu hỏi "tại sao Agent lại trả lời như vậy?".

**Metrics & Alerting**

Ngoài việc ghi log, hệ thống còn theo dõi các chỉ số vận hành:

- **Mức tiêu thụ tài nguyên:** GPU / CPU / memory
- **Độ trễ (Latency):** Agent mất bao lâu để phản hồi
- **Traffic:** Số lượng request theo thời gian

Từ các metrics này, kỹ sư thiết lập **Alerting** — cảnh báo tự động khi có bất thường. Và quan trọng hơn, kết nối với cơ chế **auto-scaling** để hệ thống tự động mở rộng khi traffic tăng đột biến, thay vì để người dùng gặp lỗi timeout.

> **Cảm nhận cá nhân:** Phần này khiến mình nhận ra một điều: **build được Agent chỉ là 30% công việc, 70% còn lại là vận hành nó**. Một Agent chạy tốt trên máy mình nhưng không có log, không có trace thì khi lên production sẽ trở thành cơn ác mộng khi debug.

---

#### 5. Evaluation – Đánh giá hiệu suất Agent

Đây là phần trả lời câu hỏi: **làm sao biết Agent của mình trả lời tốt hay dở?**

**Đo lường bằng Metrics thay vì cảm tính**

Cách tiếp cận là so sánh trực diện giữa hai thứ:

- **Predicted Response:** Câu trả lời do AI sinh ra
- **Ground Truth:** Câu trả lời tham chiếu chuẩn mực (do con người soạn sẵn, được coi là đúng)

Việc so sánh này cho ra các chỉ số **định lượng**, thay vì kiểu đánh giá cảm tính "thấy nó trả lời cũng ổn". Diễn giả nhấn mạnh rằng nếu không có bộ Ground Truth, mình không có cách nào biết được thay đổi prompt hôm nay có làm hệ thống tốt lên hay tệ đi.

**Human-in-the-loop và vai trò của SME**

Điểm quan trọng nhất trong phần này: **quá trình đánh giá không thể tự động hóa 100%**.

Bắt buộc phải có sự tham gia của các **SME (Subject Matter Experts)** — chuyên gia nghiệp vụ trong lĩnh vực đó — để thẩm định:

- Câu trả lời có **chính xác về mặt nghiệp vụ** không?
- Câu trả lời có **hợp lý** trong bối cảnh thực tế không?
- Có vi phạm quy định ngành nào không?

Ví dụ: một Agent tư vấn y tế có thể trả lời rất "trôi chảy" và khớp về mặt ngôn ngữ với Ground Truth, nhưng chỉ bác sĩ mới biết câu trả lời đó có an toàn cho bệnh nhân hay không.

> **Cảm nhận cá nhân:** Trước đây mình đánh giá chất lượng AI hoàn toàn bằng cảm tính — chạy thử vài câu thấy ổn là coi như xong. Giờ mình hiểu rằng cần xây dựng một **bộ test case với Ground Truth** ngay từ đầu dự án, giống như viết unit test cho code vậy. Không có nó thì mỗi lần sửa prompt là một lần đánh cược.

---

#### 6. Policy & Security – Bảo mật và Phân quyền

**Cedar Language – Ngôn ngữ định nghĩa quyền hạn**

Hệ thống sử dụng một ngôn ngữ chuyên biệt để thiết lập **Authorization** — tức là định nghĩa Agent được phép truy cập những gì trong hệ thống. Thay vì viết logic phân quyền rải rác trong code, toàn bộ chính sách được khai báo tập trung dưới dạng policy có thể đọc và audit được.

**Strict vs. Permissive Mode**

Đây là phần mình thấy cực kỳ thực dụng:

- **Permissive Mode:** Dùng trong giai đoạn **development**. Ở chế độ này Agent được "rảnh tay" hoạt động, ít bị chặn, giúp lập trình viên thử nghiệm nhanh mà không phải cấu hình quyền cho từng thao tác.
- **Strict Mode:** **Bắt buộc phải bật khi lên Production.** Mọi quyền hạn bị khóa chặt, Agent chỉ được làm đúng những gì đã được cấp phép tường minh.

Lý do đây là yếu tố sống còn: nó đảm bảo nguyên tắc **Least Privilege (Quyền hạn tối thiểu)**. Nếu để Permissive trên production, hậu quả có thể là:

- Agent gọi nhầm một API có tính phá hủy (xóa dữ liệu, hủy đơn hàng hàng loạt)
- Agent truy cập vào dữ liệu nhạy cảm mà nó không có nghiệp vụ gì để đọc → **rò rỉ dữ liệu**
- Bị khai thác qua prompt injection để thực hiện hành động ngoài ý muốn

> **Cảm nhận cá nhân:** Đây là loại lỗi mà mình nghĩ rất dễ mắc phải — dev xong ở chế độ thoải mái, đến lúc deploy thì quên không siết lại. Mình sẽ đưa việc "kiểm tra Strict Mode" vào checklist bắt buộc trước mỗi lần release.

---

#### 7. Mở rộng năng lực Agent với các Tools chuyên biệt

Phần này giới thiệu ba công cụ dựng sẵn giúp Agent vượt ra khỏi giới hạn của một mô hình ngôn ngữ thuần túy.

**Browser Tool**

Trao quyền cho Agent **truy cập Internet** để lấy dữ liệu theo **thời gian thực**. Điều này giải quyết một hạn chế cố hữu của LLM: kiến thức bị đóng băng tại thời điểm training. Với Browser Tool, Agent có thể tra cứu giá cả hiện tại, tin tức mới, tình trạng hàng tồn kho — những thứ thay đổi liên tục.

**Code Interpreter**

Một môi trường **sandbox (hộp cát)** — tức là môi trường cô lập an toàn — để Agent có thể:

- Tự viết mã code
- Chạy mã đó để thực hiện tính toán phức tạp
- Vẽ biểu đồ, xử lý dữ liệu
- Trả kết quả (dạng số hoặc hình ảnh) về cho người dùng

Việc chạy trong sandbox rất quan trọng: nếu Agent viết ra một đoạn code lỗi hoặc nguy hiểm, nó cũng chỉ ảnh hưởng trong môi trường cô lập, không đụng tới hệ thống chính.

**Payment Integration**

Khả năng gọi trực tiếp các **API cổng thanh toán**, biến Agent từ một trợ lý tư vấn thành một **trợ lý bán hàng tự động** — có thể chốt đơn và xử lý giao dịch trọn vẹn trong cùng một cuộc hội thoại.

> **Cảm nhận cá nhân:** Nhìn ba tool này đặt cạnh nhau mình mới thấy rõ ranh giới giữa Chatbot và Agent. Với Payment Integration, AI không còn chỉ "nói" nữa mà đã **chạm được vào tiền thật** — và đó chính xác là lý do vì sao phần Policy & Security ở trên lại quan trọng đến vậy.

---

#### 8. Hands-on Lab: Triển khai Refund Assistant

Đây là phần thực hành dài nhất, kéo dài gần một tiếng, xây dựng một Agent xử lý yêu cầu hoàn tiền hoàn chỉnh.

**Bước 1 – Thiết lập dự án**

Bộ công cụ được sử dụng:

- **Agent CLI:** Công cụ dòng lệnh để khởi tạo cấu trúc dự án Agent
- **Node.js:** Môi trường chạy
- **AWS CDK (Cloud Development Kit):** Định nghĩa hạ tầng bằng code

Sau khi khởi tạo mã nguồn, CDK sẽ sinh ra template và triển khai hạ tầng **Serverless** thông qua **CloudFormation**. Điểm quan trọng về mặt chi phí: **hạ tầng chỉ tính tiền khi có lệnh invoke** (khi thực sự có người gọi Agent chạy). Không có traffic thì không mất tiền — đây là lợi thế lớn cho giai đoạn thử nghiệm và cho các workload có lưu lượng không đều.

**Bước 2 – Tích hợp Tool hoàn tiền**

Xây dựng một Agent xử lý yêu cầu **Refund**. Điểm khác biệt so với chatbot thông thường:

- Chatbot: "Bạn vui lòng liên hệ bộ phận CSKH để được hỗ trợ hoàn tiền."
- Agent: **Tự động gọi hàm (tool)** để tra cứu trạng thái đơn hàng thật, kiểm tra điều kiện hoàn tiền, rồi phản hồi dựa trên dữ liệu thực tế.

**Bước 3 – CLI Invoke & Mock Data**

Lập trình viên chạy lệnh gọi trực tiếp từ **Terminal** để mô phỏng việc Client gọi xuống API. Điểm mình thấy thú vị nhất: hệ thống hiển thị rõ ràng **quá trình Thinking và Reasoning của Agent ngay trên màn hình terminal**. Mình thấy được từng bước Agent suy nghĩ — "người dùng muốn hoàn tiền → cần tra mã đơn hàng → gọi tool lookup_order → nhận kết quả → kiểm tra điều kiện → phản hồi".

Về dữ liệu, tại lab này tác giả dùng **Mock Data đính kèm trực tiếp vào System Prompt** thay vì kết nối cơ sở dữ liệu thật như **DynamoDB**. Lý do rất thực tế: mục tiêu của lab là **test nhanh logic của Agent**, không phải xây dựng hệ thống hoàn chỉnh. Việc tách biệt logic Agent khỏi tầng dữ liệu giúp lặp lại nhanh hơn nhiều trong giai đoạn đầu.

**Bước 4 – Phân luồng Log và Trace**

Một chi tiết kiến trúc nhỏ nhưng đáng học:

| Loại | Nơi lưu | Mục đích |
|---|---|---|
| **Log** | Ổ cứng nội bộ (local) | Phục vụ debug trong quá trình phát triển |
| **Trace** | Dịch vụ Cloud Observability | Theo dõi hành trình request để giám sát vận hành |

Việc tách hai luồng này giúp không làm phình dung lượng lưu trữ trên cloud bằng những log chi tiết chỉ cần thiết lúc dev, trong khi vẫn giữ được khả năng giám sát end-to-end trên production.

> **Cảm nhận cá nhân:** Lab này giúp mình hiểu được sự khác biệt rõ rệt giữa "xem người khác demo" và "tự gõ lệnh chạy". Đặc biệt là khoảnh khắc nhìn thấy quá trình reasoning hiện ra trên terminal — trước đó mình chỉ coi Agent như một hộp đen nhận input trả output, giờ thì mình thấy được từng bước nó suy nghĩ. Việc dùng Mock Data cũng là một mẹo hay mà mình sẽ áp dụng: **đừng vội kết nối database thật khi logic Agent còn chưa ổn định**.

---

### Key Takeaways

#### Tư duy vận hành hệ thống AI

Bài học lớn nhất mình mang về từ session này là: **khoảng cách giữa một Agent demo và một Agent production lớn hơn rất nhiều so với mình tưởng**.

Một Agent chạy được trên máy cá nhân chỉ cần Model + Prompt + vài Tool. Nhưng một Agent chạy được trên production cần thêm: chiến lược Memory hợp lý, hệ thống Observability đầy đủ, quy trình Evaluation có Ground Truth, chính sách Security ở chế độ Strict, và cơ chế auto-scaling. Đó là bốn đến năm lớp kỹ thuật hoàn toàn không liên quan gì tới việc "viết prompt cho hay".

#### Kiến thức kỹ thuật

- **Memory hai tầng:** Short-term xử lý **đồng bộ** để phản hồi tức thì, Long-term dùng **Memory Extraction chạy bất đồng bộ** để không làm chậm trải nghiệm. Đây là pattern kiến trúc đáng áp dụng rộng rãi.
- **Memory Strategy là bài toán trade-off**, không có đáp án chung — phải cân giữa độ giàu ngữ cảnh và chi phí token/lưu trữ.
- **Logging và Tracing là hai thứ khác nhau:** Log cho biết *cái gì đã xảy ra*, Trace cho biết *nó xảy ra theo trình tự nào và mất bao lâu ở mỗi bước*.
- **Predicted Response vs Ground Truth** là nền tảng của việc đánh giá định lượng — nhưng vẫn **cần SME thẩm định**, không thể tự động hóa 100%.
- **Cedar + Strict Mode** là cơ chế bảo vệ cuối cùng để đảm bảo Least Privilege khi Agent lên production.
- **Sandbox cho Code Interpreter** là bắt buộc — không bao giờ để AI chạy code sinh ra trong môi trường có quyền truy cập hệ thống thật.

#### FinOps

- Hạ tầng **Serverless chỉ tính tiền khi invoke** — cực kỳ phù hợp cho giai đoạn PoC và cho workload có traffic không đều.
- **Memory Strategy ảnh hưởng trực tiếp tới hóa đơn:** nhét càng nhiều ngữ cảnh vào prompt thì càng tốn token cho mỗi lần gọi. Tối ưu Memory chính là tối ưu chi phí.
- **Dùng Mock Data trong giai đoạn dev** giúp tiết kiệm cả chi phí lẫn thời gian so với việc dựng database thật ngay từ đầu.

#### Nghề nghiệp

- Phát triển theo mô hình **chữ T**: một năm đầu đào sâu chuyên môn, năm 2–3 mở rộng sang production và domain knowledge.
- **Chứng chỉ là điều kiện cần, sản phẩm là điều kiện đủ.**
- **Kỹ năng giao tiếp với người non-technical** quan trọng ngang với kỹ năng kỹ thuật.
- **Hackathon** là con đường ngắn nhất để chuyển kiến thức lý thuyết thành kinh nghiệm thực chiến.

---

### Áp dụng vào công việc

Sau session này, mình liệt kê ra những thứ có thể áp dụng ngay:

- **Thiết kế Memory hai tầng cho project cá nhân:** Thay vì nhét toàn bộ lịch sử chat vào prompt như hiện tại, mình sẽ tách thành short-term (giữ vài lượt gần nhất) và long-term (trích xuất các thông tin cốt lõi về người dùng, chạy ngầm).
- **Xây bộ Ground Truth trước khi viết prompt:** Chuẩn bị sẵn 20–30 cặp câu hỏi – câu trả lời chuẩn để mỗi lần chỉnh prompt đều có thể đo được là tốt lên hay tệ đi, thay vì đoán.
- **Bật Tracing ngay từ ngày đầu, không đợi đến lúc lỗi:** Cấu hình trace end-to-end ngay từ khi khởi tạo dự án. Log để local, trace đẩy lên cloud.
- **Đưa "kiểm tra Strict Mode" vào checklist release:** Trước mỗi lần deploy lên production, bắt buộc rà soát lại toàn bộ policy để đảm bảo không sót chế độ Permissive nào từ giai đoạn dev.
- **Dùng Mock Data để iterate nhanh:** Khi bắt đầu một Agent mới, dùng dữ liệu giả nhúng vào System Prompt để ổn định logic trước, sau đó mới thay bằng kết nối DynamoDB thật.
- **Áp dụng Serverless cho các workload AI có traffic thấp:** Với các tool nội bộ chỉ dùng vài chục lần một ngày, serverless sẽ rẻ hơn nhiều so với giữ một instance chạy 24/7.
- **Chọn một domain để đào sâu:** Thay vì học lan man, mình sẽ chọn một lĩnh vực nghiệp vụ cụ thể để tích lũy domain knowledge song song với kỹ thuật.

---

### Trải nghiệm cá nhân


Điều mình ấn tượng nhất là **cách session được sắp xếp**: bắt đầu bằng định hướng nghề nghiệp (phần "mềm"), rồi đi dần vào kiến trúc, sau đó là bốn trụ cột vận hành, và kết thúc bằng một bài lab thực chiến. Cấu trúc này khiến mình không bị choáng — mỗi phần lý thuyết đều có một mảnh ghép tương ứng xuất hiện lại trong lab.

Phần khiến mình bất ngờ nhất là **Evaluation**. Trước đây mình gần như bỏ qua bước này hoàn toàn — cứ chạy thử vài câu, thấy trả lời hợp lý là coi như xong. Sau khi nghe về Predicted Response vs Ground Truth và vai trò bắt buộc của SME, mình nhận ra cách làm cũ của mình về bản chất là **không có kiểm soát chất lượng gì cả**. Nếu đưa lên production thì mỗi lần chỉnh prompt là một lần đánh cược mù.

Phần **Hands-on Lab** thì đúng là "làm mới hiểu". Có những chi tiết như việc tách Log để local còn Trace đẩy lên cloud — nghe thì rất hiển nhiên, nhưng nếu không thực sự chạy và nhìn thấy output ở hai nơi khác nhau thì mình sẽ không bao giờ để tâm. Khoảnh khắc thấy được quá trình **Thinking và Reasoning hiện ra trên terminal đen** cũng là một trải nghiệm khá đặc biệt — nó biến thứ trừu tượng thành thứ nhìn được.

Cuối cùng, phần **định hướng nghề nghiệp** ở đầu session tuy ngắn nhưng lại là phần mình suy nghĩ nhiều nhất sau khi xem xong. Nó khiến mình phải nhìn lại cách mình đang đầu tư thời gian: mình đang học rộng hay học sâu? Mình có đang tích lũy domain knowledge nào không, hay chỉ đang gom chứng chỉ?

---

### Bài học rút ra

- **Xây được Agent chỉ là 30% công việc — 70% còn lại là Memory, Observability, Evaluation và Security.** Đó mới là thứ quyết định Agent có sống nổi trên production hay không.
- **Bất đồng bộ là chìa khóa của trải nghiệm tốt.** Mọi thứ không cần trả về ngay cho người dùng (như Memory Extraction) đều nên đẩy xuống background.
- **Không đo được thì không cải thiện được.** Không có Ground Truth thì mọi việc tinh chỉnh prompt đều chỉ là cảm tính.
- **Con người vẫn không thể thay thế trong khâu đánh giá.** SME là mắt xích bắt buộc, đặc biệt trong các lĩnh vực có tính nghiệp vụ cao.
- **Permissive là để dev, Strict là để sống.** Quên siết quyền trước khi lên production là một trong những lỗi tốn kém nhất có thể mắc phải.
- **Memory Strategy là bài toán tiền bạc, không chỉ là bài toán kỹ thuật.** Mỗi token ghi nhớ đều có giá.
- **Serverless là bạn của giai đoạn thử nghiệm** — không traffic thì không tốn tiền, cho phép thử sai thoải mái.
- **Sự nghiệp cũng cần kiến trúc như hệ thống:** đào sâu trước, mở rộng sau, và đừng quên chiều nghiệp vụ lẫn kỹ năng giao tiếp.

---

### Hình ảnh hoặc video chứng minh tham gia

![Event 3-1](/images/4-EventParticipated/EV3-1.png)
![Event 3-2](/images/4-EventParticipated/EV3-2.png)
![Event 3-3](/images/4-EventParticipated/EV3-3.png)
![Event 3-4](/images/4-EventParticipated/EV3-4.png)
![Event 3-5](/images/4-EventParticipated/EV3-5.png)

> Nhìn chung, session này bổ sung đúng mảnh ghép còn thiếu sau buổi Deepdive trước: nếu buổi trước cho mình biết **Agent được xây bằng gì**, thì buổi này cho mình biết **làm sao để nó sống được ngoài thực tế**. Memory quyết định Agent thông minh tới đâu, Observability quyết định mình có kiểm soát được nó không, Evaluation quyết định mình có biết nó tốt hay dở không, và Security quyết định nó có gây họa hay không. Thiếu bất kỳ mảnh nào trong bốn mảnh đó, Agent vẫn chỉ là một bản demo đẹp mắt chứ chưa phải một sản phẩm.
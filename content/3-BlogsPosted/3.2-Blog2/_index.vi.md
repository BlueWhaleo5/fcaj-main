---
title: "Blog 2 - Amazon Bedrock AgentCore"
date: 2026-07-30
weight: 2
chapter: false
pre: " <b> 3.2. </b> "
includeInReport: true
---

# Amazon Bedrock AgentCore: Lời Giải Cho Bài Toán Đưa AI Agent Từ Demo ra Production

Hi mọi người, đợt vừa rồi mình có tham gia buổi workshop [AgentForge - Ho Chi Minh City](../../4-EventParticipated/4.2-Event2/) thuộc AWS Connected Community. Cá nhân mình thấy đây là buổi workshop làm mình ấn tượng nhất từ trước giờ mình từng đi, hôm nay mình muốn chia sẻ với mọi người đôi nét về AWS Bedrock AgentCore.

---

Nếu từng tự tay dựng một AI Agent (tác tử AI) bằng các framework phổ biến như LangGraph, CrewAI hay LlamaIndex, hẳn bạn đã trải qua cảm giác "lên thiên đường rồi xuống địa ngục":

Chạy thử nghiệm trên máy local (PoC) thì cực kỳ mượt mà, nhưng khi đưa lên môi trường thực tế (Production) cho hàng nghìn người dùng, bạn ngay lập tức đụng trúng bức tường lửa: Bảo mật sandbox ở đâu? Quản lý bộ nhớ (Memory) thế nào? Làm sao cấp quyền OAuth cho Agent gọi API bên ngoài? Và tại sao phải trả tiền máy chủ 24/7 chỉ để AI... ngồi chờ LLM phản hồi?

Đó chính là lý do AWS tung ra **Amazon Bedrock AgentCore**, nền tảng chuyên biệt giúp dựng, triển khai và vận hành các AI Agent cấp doanh nghiệp trên quy mô lớn mà không cần bận tâm về quản lý hạ tầng.

## 1. Điểm Tách Biệt: AgentCore Không Thay Thế Framework Của Bạn

Một hiểu lầm phổ biến là AgentCore sẽ "giành sân" với LangGraph hay CrewAI. Thực tế hoàn toàn ngược lại.

AgentCore đóng vai trò như lớp hạ tầng (Infrastructure Layer). Bạn cứ thoải mái dùng framework yêu thích hoặc viết code Python/TypeScript thuần, sau đó thả ứng dụng lên AgentCore để nó lo toàn bộ phần "việc nặng vô hình" như runtime serverless, quản lý memory, gateway và security.

## 2. Các "Mảnh Ghép" Cốt Lõi Trong AgentCore

AgentCore được thiết kế theo dạng mô-đun hóa. Bạn có thể dùng trọn bộ hoặc nhặt riêng từng dịch vụ để tích hợp vào ứng dụng của mình:

* **AgentCore Runtime (Serverless Chuyên Cho Agent):** Máy chủ truyền thống tính phí theo từng giây chạy. Nhưng công việc của AI Agent lại dành tới 30% - 70% thời gian chỉ để "đợi" (chờ LLM trả lời, chờ API phản hồi). Runtime của AgentCore áp dụng cơ chế tính phí theo tài nguyên thực tế tiêu thụ. Trong khoảng thời gian chờ đợi I/O, bạn không phải trả tiền CPU. Nó hỗ trợ cả các phiên tương tác độ trễ thấp lẫn các tiến trình chạy ngầm kéo dài tới 8 tiếng.
* **AgentCore Gateway & Support MCP:** Chuyển đổi các API hiện có hoặc hàm AWS Lambda thành các công cụ (Tools) mà Agent hiểu được. Dịch vụ này tích hợp sẵn với Model Context Protocol (MCP), giúp Agent dễ dàng kết nối với các nguồn dữ liệu và công cụ bên ngoài mà không cần viết lại adapter.
* **AgentCore Memory (Bộ Nhớ Ngắn Hạn & Dài Hạn):** Thay vì tự quản lý Vector Database hay Redis phức tạp để lưu hội thoại, AgentCore Memory tự động duy trì bối cảnh (Context) qua nhiều phiên làm việc, giúp Agent ngày càng "hiểu" người dùng hơn mà bạn không cần đụng tay vào quản lý database.
* **Identity, Policy & Sandboxing:** Giúp Agent đại diện cho người dùng (hoặc tự nó) đăng nhập an toàn vào các hệ thống như GitHub, Salesforce, Slack qua OAuth. Đồng thời, nó cung cấp môi trường isolated sandbox để Agent tự do viết/chạy code Python hoặc duyệt web thao tác UI mà không sợ nguy cơ lộ đòn tấn công Prompt Injection vào hệ thống nội bộ.
* **Observability (Giám Sát Hệ Thống):** Cho phép theo dõi vết (tracing) chi tiết từng bước suy luận, gọi tool của Agent thông qua chuẩn OpenTelemetry, giúp lập trình viên nhanh chóng tìm ra lỗi logic hoặc nghẽn cổ chai trong quy trình xử lý.

## Lời Kết

Thời kỳ các AI Agent chỉ dừng lại ở các bài demo trên Terminal đã qua. Với sự xuất hiện của Amazon Bedrock AgentCore, khoảng cách từ một bài code mẫu thử nghiệm đến một hệ thống AI Agent sẵn sàng phục vụ hàng triệu người dùng doanh nghiệp đã thu hẹp lại chỉ còn vài dòng lệnh.

### Bài Đăng Gốc

* **Link:** [Amazon Bedrock AgentCore: Lời Giải Cho Bài Toán Đưa AI Agent Từ Demo ra Production](https://www.facebook.com/groups/awsstudygroupfcj/?multi_permalinks=2231620424968674&notif_id=1785401468277838&notif_t=feedback_reaction_generic&ref=notif)
    
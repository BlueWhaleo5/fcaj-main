---
title: "Event 2: AgentForge - Ho Chi Minh City"
date: 2026-07-30
weight: 2
chapter: false
pre: " <b> 4.2. </b> "
---

### Thông Tin Sự Kiện

* **Tên sự kiện:** AgentForge - Ho Chi Minh City
* **Thời gian:** ngày 21/7/2026
* **Địa điểm:** Tầng 26, tòa nhà Bitexco, số 02 đường Hải Triều, phường Sài Gòn, Thành phố Hồ Chí Minh
* **Vai trò:** Người tham dự

### Mục Đích Của Sự Kiện

Một khóa training chuyên sâu, kéo dài cả ngày, mức độ L300-400 về **Amazon Bedrock AgentCore**, dành cho những người muốn đi sâu vào hệ thống AI agentic. Trình bày toàn bộ AgentCore stack (Runtime, Gateway, Identity, Memory, Evaluations, Observability, Registry, Policy) từ kiến trúc đến production, thông qua các phần trình bày chuyên sâu, một use case DevOps thực tế, và các bài lab thực hành.

### Danh Sách Diễn Giả

- **Vasileios Vonikakis** - Sr. GTM Specialist Solution Architect
- **Isaac Ibrahim** - GTM Specialist SA – AI/ML
- **Tim Wu** - Sr. GTM Spec. SA AIML
- **Brian Bae** - Head of ASEAN AI GTM

### Nội Dung Nổi Bật

- Các phần trình bày chuyên sâu về toàn bộ nền tảng Amazon Bedrock AgentCore - Runtime, Gateway, Identity, Memory, Evaluations, Observability, Registry, và Policy
- Một use case DevOps thực tế minh họa AgentCore chạy trong production
- Lab thực hành: deploy một base agent, sau đó nâng cấp thêm Evaluations và Policy enforcement, cộng thêm một phần mở rộng tự chọn
- Best practice để thiết kế, triển khai, và vận hành hệ thống agentic sẵn sàng cho production (mức L300-400 - hướng đến làm chủ thực sự, không chỉ dừng ở biết sơ)

### Những Gì Học Được

- AgentCore tách cái mà trước đây cảm giác như một khối "agent" duy nhất thành các thành phần riêng biệt - Runtime (thực thi), Gateway (truy cập tool/API), Identity (xác thực cho chính agent), Memory (trạng thái xuyên suốt các lượt/phiên), Policy (guardrails), Evaluations (kiểm thử chất lượng/regression), Observability (tracing) - mỗi thành phần một mối quan tâm riêng
- Policy enforcement và Evaluations được coi là một phần bắt buộc để đưa một base agent đến trạng thái "sẵn sàng production", không phải phần thêm tùy chọn
- Use case DevOps cho thấy các vấn đề vận hành thực tế ngoài happy path: ranh giới phân quyền (Identity/Gateway), khả năng audit, và cách memory của agent tồn tại/hết hạn qua các phiên

### Ứng Dụng Vào Công Việc

Thấy Gateway đóng vai trò lớp expose tool/API cho agent, kết hợp với Identity cấp quyền riêng cho từng agent, khiến tôi nghĩ ngay đến một khoảng trống trong REST API của ServerlessFinance (Phase 4) - hiện tại một API key duy nhất có toàn quyền truy cập mọi endpoint. Nếu đặt một AgentCore Gateway phía trước, có thể giới hạn chính xác agent được phép gọi endpoint nào (vd đọc kết quả backtest nhưng không được đặt lệnh paper trading), thay vì một key có tất cả hoặc không có gì.

### Trải Nghiệm Trong Event

Một ngày trọn vẹn với lab thực hành, deploy một base agent rồi lần lượt thêm Evaluations, Policy enforcement, và một phần mở rộng tự chọn, khiến các mảnh ghép của AgentCore trở nên rõ ràng theo cách mà chỉ xem demo không làm được. Nó cũng khiến tôi nhìn lại API của ServerlessFinance theo hướng khác: coi "ai được phép gọi cái gì" là một lớp riêng biệt, tường minh (Gateway + Identity) thay vì dùng chung một API key, là một khoảng trống nhỏ nhưng có thật mà tôi chưa từng nghĩ tới trước sự kiện này.

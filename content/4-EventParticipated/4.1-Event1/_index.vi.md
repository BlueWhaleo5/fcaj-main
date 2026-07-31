---
title: "Event 1: FCAJ Community Day"
date: 2026-07-30
weight: 1
chapter: false
pre: " <b> 4.1. </b> "
---

### Thông Tin Sự Kiện

* **Tên sự kiện:** FCAJ Community Day
* **Thời gian:** ngày 27/6/2026
* **Địa điểm:** Tầng 26, tòa nhà Bitexco, số 02 đường Hải Triều, phường Sài Gòn, Thành phố Hồ Chí Minh
* **Vai trò:** Người tham dự

### Mục Đích Của Sự Kiện

Một buổi chia sẻ đa chủ đề, nơi các diễn giả đến từ nhiều doanh nghiệp khác nhau trình bày về các chủ đề AWS/AI thực tế, tổng cộng 5 bài nói, xoay quanh hạ tầng agent bảo mật, công cụ AI năng suất cho doanh nghiệp, DevOps agent, voice agent, và vận hành hệ thống agentic trên cloud.

### Chủ Đề

- Building secure private MCP for Quick
- AI-powered productivity worker planning for Enterprise
- AWS DevOps Agent
- Building Voice Agent At Scale
- AgenticOps for your Cloud

### Danh Sách Diễn Giả

- **Mr.Steve Trần** - CEO of Cloud Thinker
- **Mr.Nghị Danh** - ReNova Cloud
- **Mr.Trung** - AWS Builder
- **Mr.Trung** - CEO Revve AI
- **Ms.Bảo** - Cloud Engineer
- **Mr.Nguyên** - Could Engineer
- **Mr.Trường** - Noventiq Viet Nam
- **Ms.Minh Anh** - Noventiq Viet Nam
- **Mr.Toàn Nguyễn** - AWS Security Builder

### Nội Dung Nổi Bật

- **Building secure private MCP for Quick**: kiến trúc vận hành một MCP (Model Context Protocol) server nội bộ, để lộ tool/data ra cho AI agent mà không vượt ra khỏi ranh giới mạng của doanh nghiệp
- **AI-powered productivity worker planning for Enterprise**: cách các doanh nghiệp đang thử nghiệm dùng AI agent để lên kế hoạch và ưu tiên công việc giữa các team
- **AWS DevOps Agent**: một agent hỗ trợ CI/CD pipeline, phân loại sự cố (incident triage), và thay đổi hạ tầng trên AWS
- **Building Voice Agent At Scale**: kiến trúc và các thách thức khi scale voice agent cho lượng người dùng đồng thời lớn
- **AgenticOps for your Cloud**: các thực hành vận hành (monitoring, guardrails, rollback) cần thiết khi agent bắt đầu tự thay đổi hạ tầng cloud

### Những Gì Học Được

- Nhiều doanh nghiệp không liên quan nhau nhưng cùng gặp một vấn đề - cấp quyền truy cập an toàn, có giới hạn cho AI agent vào tool/data nội bộ - cho thấy đây là mối quan tâm thực tế trong production, không phải lý thuyết
- Voice agent và DevOps agent đều cần cùng một kỷ luật về độ tin cậy như các buổi AgentForge trước đã nói: guardrails, tracing, và có con người can thiệp khi cần
- "AgenticOps" đang dần trở thành một mảng vận hành riêng, tách khỏi DevOps truyền thống, khi agent có thể trực tiếp hành động lên hạ tầng chứ không chỉ trả lời câu hỏi

### Ứng Dụng Vào Công Việc

Bài nói về private MCP chỉ thẳng vào một khoảng trống trong ServerlessFinance - REST API (Phase 4) hiện chưa có lớp truy cập nào giới hạn phạm vi và an toàn cho agent. Nếu expose nó thành một MCP server thay vì REST API thuần, một AI agent có thể truy vấn kết quả backtest hoặc đặt lệnh paper trading trong phạm vi quyền hạn giống như bài nói đề cập, thay vì cần API key có quyền truy cập toàn bộ.

### Trải Nghiệm Trong Event

Năm doanh nghiệp khác nhau cùng trình bày trong một buổi chiều cho tôi góc nhìn rộng và thực tế hơn nhiều so với một buổi nói chuyện từ một hãng duy nhất. Bài về bảo mật MCP và AgenticOps để lại ấn tượng nhất với tôi vì chỉ thẳng ra những khoảng trống tôi có thể liên hệ ngay với project của mình, còn phần voice agent và DevOps agent tuy ít liên quan trực tiếp đến những gì tôi đang xây nhưng vẫn là bối cảnh hữu ích.

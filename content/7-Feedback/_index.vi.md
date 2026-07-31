---
title: "Chia sẻ, đóng góp ý kiến"
date: 2026-07-30
weight: 7
chapter: false
pre: " <b> 7. </b> "
includeInReport: true
---

### Đánh giá chung

**1. Cấu trúc chương trình**
Chương trình First Cloud AI Journey cân bằng khá tốt giữa một project tự thân (ServerlessFinance, được xây dựng và chỉnh sửa qua 8 phase) và các điểm chạm cộng đồng (AWS Study Group, các sự kiện trực tiếp). Việc được tự thiết kế phạm vi và nhịp độ của project, nhưng vẫn phải chịu trách nhiệm với deadline thật và hạ tầng AWS thật, khiến những gì học được cảm giác thực chất chứ không phải được cho sẵn.

**2. Các sự kiện cộng đồng**
Các sự kiện đã tham gia trong chương trình (FCAJ Community Day, AgentForge - Ho Chi Minh City, FCAJ x Agentic AI Build Week, và AgentForge - Deep Dive) đều có chất lượng tốt và mang tính thực hành cao, chứ không chỉ mang tính quảng bá. Đặc biệt AgentForge - Ho Chi Minh City (một buổi training cả ngày, mức L300-400 về Amazon Bedrock AgentCore) là buổi có chiều sâu kỹ thuật nhất mà em từng tham gia; nó ảnh hưởng trực tiếp đến cách em nhìn nhận lỗ hổng về phân quyền truy cập trong chính API của ServerlessFinance.

**3. Sự phù hợp giữa công việc và chuyên ngành học**
Việc xây dựng ServerlessFinance chạm đến hệ thống phân tán, infrastructure-as-code, và tối ưu hiệu năng (lõi Rust/PyO3) đều liên quan trực tiếp đến nền tảng Artificial Intelligence, đồng thời vượt xa những gì thường học trên lớp để chạm vào các mối quan tâm thực tế của production: quản lý chi phí, observability, và ranh giới bảo mật (API key, IAM least privilege).

**4. Cơ hội học hỏi & phát triển kỹ năng**
Định dạng project buộc phải học qua thực hành thay vì học qua tutorial: nhiều vấn đề thật xảy ra mà không có đáp án sẵn trong sách vở (một dịch vụ AWS trong kế hoạch không khả dụng ở region mục tiêu, giới hạn concurrency của Lambda chặn một workload phân tán, lỗi Rust toolchain trong Docker multi-stage build) và phải tự truy vết nguyên nhân, sửa trực tiếp trên tài khoản AWS. Kinh nghiệm debug kiểu đó rất khó có được theo cách nào khác.

**5. Văn hóa cộng đồng**
Việc đăng bài lên AWS Study Group và thấy blog được đọc, được phản hồi khi có thắc mắc khiến văn hóa "chia sẻ những gì mình học được" trở nên thực chất. Workshop AgentForge cũng vậy, có văn hóa chia sẻ từ người thực hành khá mạnh (diễn giả cộng đồng từ nhiều công ty, không chỉ nhân viên AWS), khiến nội dung cảm giác bám sát kinh nghiệm production thật thay vì tài liệu marketing chung chung.

---

### Một số câu hỏi khác
- **Điều hài lòng nhất trong chương trình:** các mentor rất nhiệt huyết, các anh chị AWS Builder support nhiệt tình, luôn chủ động hỏi thăm và quan tâm tình hình của các bạn tham gia chương trình. Chương trình cũng là nơi giúp em đem những kiến thức học trên sách vở áp dụng thực tế vào một dự án kiểu doanh nghiệp.
- **Điều công ty/chương trình có thể cải thiện cho các thực tập sinh sau:** lịch lên văn phòng nên được chia đều hơn giữa các trường, tránh tình trạng các bạn phải tranh nhau suất lên văn phòng, ảnh hưởng đến việc đảm bảo chuyên cần.
- **Có giới thiệu chương trình này cho bạn bè không:** Tất nhiên. Sự kết hợp giữa một project AWS thật, tự thiết kế phạm vi, và một cộng đồng thực chất (mentor nhiệt tình, sự kiện, study group) là điều khá hiếm gặp ở một chương trình thực tập.

---

### Đề xuất & mong muốn
- Tổ chức thêm buổi review project giữa các nhóm/trường, hoặc một ngày giao lưu riêng dành cho từng trường, để các bạn học hỏi lẫn nhau nhiều hơn.
- Mời thêm diễn giả từ nhiều doanh nghiệp về chia sẻ kinh nghiệm, kết hợp job fair, phỏng vấn thử,... để chương trình gắn kết hơn với cơ hội việc làm thực tế.
- Em mong muốn được tiếp tục gắn bó với chương trình đến khi hoàn thành trọn vẹn cả 6 track, chứ không chỉ dừng lại ở phần thực tập.

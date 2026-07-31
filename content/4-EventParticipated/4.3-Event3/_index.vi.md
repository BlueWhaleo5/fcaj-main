---
title: "Event 3: FCAJ x Agentic AI Build Week"
date: 2026-07-30
weight: 3
chapter: false
pre: " <b> 4.3. </b> "
---

### Thông Tin Sự Kiện

* **Tên sự kiện:** FCAJ x Agentic AI Build Week
* **Thời gian:** ngày 25/7/2026
* **Địa điểm:** Tầng 36, tòa nhà Bitexco, số 02 đường Hải Triều, phường Sài Gòn, Thành phố Hồ Chí Minh
* **Vai trò:** Người tham dự

### Mục Đích Của Sự Kiện

Một sự kiện dạng hackathon/demo day, nơi các team xây dựng và trình bày giải pháp AI cho 5 bài toán thực tế khác nhau.

### Chủ Đề

- **AI-powered Conversation Ordering** - Lấy cảm hứng từ bài toán AI Drive-Thru của McDonald's, một agent nhận order của khách bằng hội thoại, xử lý cả việc thay thế món và upsell
- **Signal Scout** - Thu thập các thông tin công khai rời rạc về một doanh nghiệp, gộp thành một hồ sơ chi tiết, có cấu trúc
- **Solution Architect Professional (Native App)** - Một ứng dụng native giúp thí sinh ôn luyện chứng chỉ AWS Solutions Architect Professional
- **Smart Human Flow** - Đánh giá, dự đoán và phát hiện nguy hiểm trong luồng di chuyển của con người/đám đông, tự động phản hồi và điều phối
- **Adaptive AML Workflow Engine** - Tự động hóa việc làm giàu dữ liệu điều tra cho các vụ chống rửa tiền (AML), biến hàng giờ tra cứu thủ công thành một báo cáo tuân thủ pháp lý

### Danh Sách Diễn Giả
*Teams:*
- **One Team**  
- **Dream AI Team**  
- **Plan V**  
- **Team 3KA**  
- **Six Pillars Team**  

### Nội Dung Nổi Bật

- Năm team lần lượt pitch và demo một prototype hoạt động thật cho một trong 5 bài toán ở trên
- Một điểm chung nổi bật giữa các team: dùng LLM/agent để nén hàng giờ công việc thủ công, rời rạc - thu thập dữ liệu, viết báo cáo, nhận order - thành một output có cấu trúc duy nhất
- Phần Q&A chủ yếu xoay quanh mức độ sẵn sàng cho production: độ chính xác dữ liệu, tuân thủ, và độ trễ khi chịu tải thực tế

### Những Gì Học Được

- Những demo thuyết phục nhất là những demo có sự đối chiếu "trước và sau" rõ ràng cho một quy trình thủ công thực sự tốn thời gian (như viết báo cáo AML, hay tra cứu doanh nghiệp trong Signal Scout), chứ không chỉ là một giao diện chatbot mới lạ
- Tạo ra output có cấu trúc, kiểm chứng được (như một báo cáo tuân thủ) là một tiêu chuẩn khó hơn nhiều so với chatbot Q&A thông thường - độ chính xác và khả năng truy vết nguồn quan trọng hơn độ trôi chảy
- Giao diện thời gian thực/voice (agent nhận order) đặt ra một nhóm vấn đề kỹ thuật khác - độ trễ, xử lý ngắt lời - so với các agent kiểu nghiên cứu/báo cáo

### Ứng Dụng Vào Công Việc

Pattern của Signal Scout / AML - biến dữ liệu rời rạc thành một báo cáo có cấu trúc - giống hệt hình dạng pipeline của chính ServerlessFinance: kéo dữ liệu OHLCV thô qua data lake để tạo ra một báo cáo backtest có cấu trúc duy nhất. Thấy cùng pattern "gom dữ liệu → cấu trúc hóa → báo cáo" được áp dụng cho compliance và business intelligence càng khẳng định rằng pattern này tổng quát hóa tốt, không chỉ riêng cho tài chính.

### Trải Nghiệm Trong Event

Xem năm team rất khác nhau giải quyết bài toán của mình trong một buổi chiều là một lời nhắc hay về việc "hình dạng" của một sản phẩm AI tốt thường lặp lại giữa các lĩnh vực, dù đề bài (order drive-thru vs điều tra AML) chẳng liên quan gì nhau. Demo của AML và Signal Scout đọng lại với tôi nhiều nhất, vì pattern bên dưới, gom dữ liệu rời rạc, cấu trúc hóa, tạo ra một báo cáo đáng tin cậy, khá gần với những gì ServerlessFinance đang làm cho kết quả backtest.

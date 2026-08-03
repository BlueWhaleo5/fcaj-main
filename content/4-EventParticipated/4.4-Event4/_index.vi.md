---
title: "Event 4: AgentForge - Deep Dive"
date: 2026-07-30
weight: 4
chapter: false
pre: " <b> 4.4. </b> "
---

### Thông Tin Sự Kiện

* **Tên sự kiện:** AgentForge - Deep Dive
* **Thời gian:** ngày 1/8/2026
* **Địa điểm:** Tầng 26, tòa nhà Bitexco, số 02 đường Hải Triều, phường Sài Gòn, Thành phố Hồ Chí Minh
* **Vai trò:** Người tham dự

### Mục Đích Của Sự Kiện

Một buổi hands-on nửa ngày, tiếp nối sự kiện AgentForge L300-400 (Event 2), tập trung vào việc đưa một agent Amazon Bedrock AgentCore từ con số 0 thành một web app hoàn chỉnh, có xác thực: deploy một agent cơ bản, kết nối với external tools và Knowledge Base, rồi đưa lên một web UI được bảo vệ bởi Amazon Cognito.

### Danh Sách Diễn Giả

- **Nghĩa Trần** - Agentic SA
- **Anh Phạm** - Cloud Consuitant C-Asiapacific Vietnam

### Nội Dung Nổi Bật

- Buổi sáng chia làm 2 phần: 1 tiếng thuyết trình nền tảng về Amazon Bedrock AgentCore Overview (Runtime, Gateway, Identity) tiếp theo là 1 tiếng hands-on lab
- Lab hướng dẫn deploy một agent cơ bản trên AgentCore Runtime, sau đó mở rộng bằng cách kết nối external tools và một Knowledge Base thông qua Gateway
- Bước cuối của lab bọc agent trong một web UI được bảo vệ bởi Amazon Cognito, biến một agent endpoint trần trụi thành thứ mà một người dùng thật có thể đăng nhập và sử dụng
- Nhịp độ nhanh và cô đọng hơn nhiều so với cả ngày AgentForge L300-400. Cùng các thành phần Runtime/Gateway/Identity, nhưng toàn bộ hành trình từ "agent đã deploy" đến "app khả dụng, có xác thực" gói gọn trong một tiếng

### Những Gì Học Được

- Runtime, Gateway và Identity tạo thành một bộ tối thiểu cần có cho một agent hướng đến production: Runtime thực thi agent, Gateway là cách agent tiếp cận tools và Knowledge Base, Identity là cách cả agent lẫn người dùng cuối được xác thực
- Cognito đặt trước web UI thực chất là cùng một bài toán "ai được phép gọi cái này" như phần thảo luận Gateway/Identity ở Event 2, chỉ là chuyển sang phía người dùng thay vì phía agent-gọi-tool
- Việc kết nối một Knowledge Base được xem như một tích hợp Gateway chính danh, chứ không phải một pipeline RAG chắp vá thêm, retrieval chỉ là một tool khác mà agent gọi qua đúng con đường truy cập như mọi thứ khác

### Ứng Dụng Vào Công Việc

Cognito đặt trước một web UI chính là mảnh còn thiếu ở phía người dùng của ServerlessFinance: REST API (Phase 4) hiện chỉ được bảo vệ bởi một API key dùng chung, không có khái niệm người dùng riêng lẻ, và hiện tại cũng chưa có UI nào cả, chỉ có gọi `curl` và `test-invoke-method`. Áp dụng đúng pattern được demo ở đây (Cognito ở cổng vào, API/agent phía sau) sẽ cho ServerlessFinance cơ chế xác thực theo từng user thay vì một key dùng chung, cộng thêm một front end thật cho các endpoint backtest/results thay vì gọi API thô.

### Trải Nghiệm Trong Event

Đi từ "agent đã được deploy" đến "một người dùng đã đăng nhập có thể mở một trang web và sử dụng nó" chỉ trong một tiếng lab khiến bộ ba Runtime -> Gateway -> Identity bớt trừu tượng hơn hẳn so với các bài thuyết trình deep-dive ở Event 2. Nhìn Cognito được gắn vào phía trước một agent như một ranh giới xác thực khác (cùng hình dạng với phần thảo luận Gateway/Identity của cả ngày AgentForge) giúp em nhận ra rằng lớp "ai là user" còn thiếu của ServerlessFinance không phải một vấn đề tách biệt với lớp "agent nào" còn thiếu; đó là cùng một pattern được áp dụng hai lần.

---
title: "Blog 1 - Hành Trình Của Amazon SQS"
date: 2026-07-30
weight: 1
chapter: false
pre: " <b> 3.1. </b> "
includeInReport: true
---

# Hành Trình Của Amazon SQS: Dịch Vụ Đầu Tiên Định Hình Nên Đế Chế Đám Mây AWS

Khi nhắc đến AWS, đa phần chúng ta sẽ nghĩ ngay đến những cái tên "quốc dân" như máy chủ ảo EC2, kho lưu trữ S3, hay cỗ máy cơ sở dữ liệu RDS. Thế nhưng, bạn có biết dịch vụ AWS đầu tiên chính thức ra mắt toàn cầu là gì không?

Không phải máy chủ, cũng chẳng phải kho lưu trữ dữ liệu. Đó chính là **Amazon SQS (Simple Queue Service)** - một dịch vụ nghe qua có vẻ khiêm tốn: Hàng đợi tin nhắn (Message Queue).

Hôm nay, hãy cùng tìm hiểu xem điều gì khiến SQS trở thành "viên gạch đầu tiên" cho đế chế đám mây hàng nghìn tỷ đô, và tại sao nó lại quan trọng đến thế trong kiến trúc phần mềm hiện đại.

## 1. Bài toán năm 2004: Khi các hệ thống "ngủ gật" cùng nhau

Vào những năm đầu thập niên 2000, các ứng dụng web thường được thiết kế theo dạng Monolith (khối đồng nhất) hoặc kết nối trực tiếp với nhau (Synchronous).

Hãy hình dung một kịch bản đơn giản trên trang thương mại điện tử Amazon:

1. Khách hàng nhấn "Đặt hàng".
2. Hệ thống xử lý thanh toán thẻ tín dụng.
3. Hệ thống gửi email xác nhận.
4. Hệ thống báo kho cập nhật tồn kho.

Nếu cả 4 bước này nối đuôi nhau xử lý trực tiếp, chỉ cần bước gửi email bị nghẽn mạng hay cổng thanh toán phản hồi chậm 5 giây, toàn bộ trang web sẽ quay mòng mòng. Người dùng chán nản hủy đơn, còn máy chủ thì đổ gục vì quá tải.

Đặt ra yêu cầu: Cần một "trạm trung chuyển" giúp các thành phần trong hệ thống có thể nói chuyện với nhau một cách bất đồng bộ (Asynchronous). Nhóm A gửi tin nhắn xong là xong việc của mình, Nhóm B khi nào rảnh thì vào lấy tin nhắn ra xử lý, không ai phải ngồi chờ ai.

Chính nhu cầu cấp thiết đó đã khai sinh ra Amazon SQS vào năm 2004 (thậm chí trước cả khi thương hiệu AWS chính thức ra đời vào năm 2006).

## 2. SQS hoạt động như thế nào?

Ví dụ dễ hình dung nhất về SQS chính là Quầy Order ở quán Cà phê:

```
[ Khách hàng / Producer ]
        |
        v
[ Tờ hóa đơn trên Dây treo / SQS Queue ]
        |
        v
[ Barista / Consumer ]
```

* **Producer (Bên gửi):** Nhóm thu ngân nhận đơn và in ra một tờ giấy order (Gửi Message vào Queue).
* **SQS (Queue):** Dây treo hóa đơn. Nó giữ các order theo đúng thứ tự, an toàn, không lo bị gió thổi bay.
* **Consumer (Bên nhận):** Barista pha chế. Họ lấy từng tờ hóa đơn xuống để làm. Bàn thu ngân không cần đứng chờ Barista pha xong ly cà phê mới được nhận khách tiếp theo.

## 3. Hai "vũ khí" chính của Amazon SQS

SQS cung cấp 2 loại hàng đợi để giải quyết các bài toán khác nhau:

### Standard Queues (Hàng đợi tiêu chuẩn)

* **Tốc độ:** Gần như không giới hạn số lượng tin nhắn mỗi giây (Unlimited throughput).
* **Đặc điểm:** Cam kết tin nhắn được giao ít nhất một lần (At-least-once delivery). Tuy nhiên, đôi khi thứ tự tin nhắn đến có thể bị xáo trộn nhẹ.
* **Phù hợp cho:** Các công việc cần xử lý nhanh, khối lượng cực lớn và không quá khắt khe về thứ tự (như xử lý ảnh, gửi email thông báo).

### FIFO Queues (First-In-First-Out)

* **Tốc độ:** Giới hạn tốc độ xử lý hơn so với Standard.
* **Đặc điểm:** Đúng như tên gọi "Vào trước - Ra trước", đảm bảo tin nhắn được xử lý chính xác theo thứ tự và chỉ đúng 1 lần (Exactly-once processing).
* **Phù hợp cho:** Các giao dịch tài chính, ngân hàng, đặt vé máy bay, nơi mà thứ tự các bước quyết định tính đúng đắn của dữ liệu.

## 4. Tại sao SQS lại là "Cứu tinh" cho các Developer?

* **Khả năng tự co giãn (Auto-Scaling):** Dù ứng dụng của bạn nhận 10 tin nhắn/ngày hay 10 tỷ tin nhắn/ngày, SQS đều cân trọn mà bạn không cần phải bấm nút "nâng cấp RAM" hay lo lắng hạ tầng bị sập.
* **Decoupling (Tách rời hệ thống):** Giúp các microservices hoạt động độc lập. Nếu dịch vụ xử lý email bị hỏng, tin nhắn vẫn nằm an toàn trong SQS. Khi dịch vụ email hoạt động trở lại, nó chỉ cần vào SQS lấy dữ liệu ra xử lý tiếp mà không bị mất dữ liệu của khách hàng.
* **Dead Letter Queue (DLQ):** Khi một tin nhắn bị lỗi và không thể xử lý sau nhiều lần thử, SQS sẽ tự động chuyển nó sang một "hàng đợi chứa thư rác" (DLQ) để lập trình viên vào kiểm tra nguyên nhân mà không làm nghẽn các tin nhắn khác.

## Lời kết

Là dịch vụ mở màn cho AWS, Amazon SQS chính là minh chứng rõ nhất cho triết lý thiết kế của Amazon: Bắt đầu từ những bài toán cốt lõi nhất, đơn giản nhất nhưng phải hoạt động cực kỳ bền bỉ trên quy mô lớn.

Dù không hào nhoáng như các công cụ AI hay Big Data hiện đại, SQS vẫn âm thầm chảy trong mạch máu của hàng triệu ứng dụng toàn cầu mỗi giây.

### Bài Đăng Gốc

* **Link:** [Hành Trình Của Amazon SQS](https://www.facebook.com/groups/awsstudygroupfcj/?multi_permalinks=2225888418176118&notif_id=1785400015854661&notif_t=feedback_reaction_generic&ref=notif)

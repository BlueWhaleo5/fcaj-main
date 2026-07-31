---
title: "Bản đề xuất"
date: 2026-07-30
weight: 2
chapter: false
pre: " <b> 2. </b> "
includeInReport: true
---

# ServerlessFinance - Nền tảng Backtest Chiến lược Tài chính & Paper Trading Serverless

## Giải pháp AWS Serverless hoàn chỉnh cho backtest song song và paper trading thời gian thực

### 1. Tóm tắt điều hành

ServerlessFinance là nền tảng chạy hoàn toàn trên AWS serverless, cho phép một trader (crypto/quant) backtest một chiến lược giao dịch trên hàng loạt tổ hợp tham số song song, sau đó paper-trade chiến lược đó theo giá thị trường thực mà không rủi ro vốn thật. Dữ liệu nến OHLCV (Open/High/Low/Close/Volume) lịch sử được lấy từ API công khai của Binance và lưu vào data lake trên S3; một lưới tham số (ví dụ vài chục tổ hợp chu kỳ SMA nhanh/chậm) được phân tán song song trên nhiều Lambda worker thông qua Step Functions Distributed Map; kết quả được xếp hạng theo Sharpe ratio và phục vụ qua REST API. Một module thứ hai, độc lập, mô phỏng đặt lệnh paper trading theo giá thực và đẩy kết quả khớp lệnh về client theo thời gian thực qua WebSocket. Toàn bộ hệ thống được thiết kế, xây dựng, triển khai và kiểm chứng đầu-cuối qua 8 phase, mỗi phase đều được deploy, test trên hạ tầng AWS, và tháo dỡ/dựng lại khi cần để kiểm soát chi phí.

### 2. Tuyên bố vấn đề

**Vấn đề hiện tại**

Kiểm thử một chiến lược giao dịch đúng cách đòi hỏi chạy nó trên rất nhiều tổ hợp tham số (ví dụ mọi cặp chu kỳ moving-average nhanh/chậm) trên nhiều năm dữ liệu lịch sử, đây là bài toán "song song hoá tự nhiên" (embarrassingly parallel), nhưng nếu chạy tuần tự trên một máy sẽ ngày càng chậm khi lưới tham số hoặc lượng dữ liệu tăng lên. Đồng thời, trader muốn xác thực một chiến lược tiềm năng bằng giá *thực* trước khi dùng tiền thật, điều này đòi hỏi một tài khoản paper trading luôn sẵn sàng và phản hồi thời gian thực, hạ tầng này lại quá dư thừa nếu phải chạy 24/7 trên máy cá nhân hoặc VM thuê chỉ để thỉnh thoảng backtest.

**Giải pháp**

ServerlessFinance thay thế mô hình "một máy chạy backtest tuần tự" bằng "hàng trăm lượt gọi Lambda, mỗi lượt chạy một tổ hợp tham số, chạy song song, chỉ trong thời gian job thực thi". Step Functions Distributed Map phân tán lưới tham số của một job ra nhiều Worker Lambda; Aggregator xếp hạng kết quả; REST API cung cấp trạng thái job và top kết quả. Một module riêng, cũng hoàn toàn serverless, cho phép cùng trader đó đặt lệnh market/limit giả lập khớp theo giá Binance thực, theo dõi tiền mặt/vị thế trong DynamoDB, và đẩy thông báo khớp lệnh qua WebSocket, ở quy mô nhỏ và rẻ hơn nhiều, kiến trúc mà một nền tảng giao dịch thật sẽ dùng.

### 3. Kiến trúc giải pháp

Hệ thống được tổ chức thành 5 CDK stack độc lập, mỗi stack có thể deploy và test riêng, chỉ liên kết với nhau qua đúng output mà stack kia cần (tham chiếu S3 bucket, Lambda function, ARN của Step Functions state machine):

![Kiến trúc ServerlessFinance](/images/2-Proposal/serverless_architecture.png)

- *Ba đường kẻ ngang màu xám mờ, đánh dấu "25", chạy từ Worker Lambda, Step Functions (và các Lambda khác) đến CloudWatch. Đây là luồng metrics/logs. Tất cả Lambda và Step Functions tự động gửi logs và metrics đến CloudWatch chạy ngầm, không phải là một lời gọi API rõ ràng trong luồng chính. Vì nó chạy song song và bất đồng bộ với luồng chính (không phải "bước tiếp theo" trong chuỗi 1 → 27), nên dùng đường kẻ mờ để phân biệt.*

- *Đường kẻ dọc mờ "On Failure" (từ Ingestion Lambda xuống SQS DLQ) là luồng lỗi, chỉ dùng khi Ingestion Lambda thất bại (Lambda retry nhưng vẫn lỗi), không phải luồng bình thường.*

**Dịch vụ AWS sử dụng**
- **Amazon S3** - lưu dữ liệu OHLCV dạng Parquet có partition (`symbol/interval/year/month`), đúng định dạng mà engine backtest đọc trực tiếp.
- **AWS Lambda** - toàn bộ bước tính toán: ingestion, orchestration, worker backtest (có phần lõi tăng tốc bằng Rust/PyO3, xem Phase 6), aggregation, hai REST API, và các handler connect/disconnect WebSocket.
- **AWS Step Functions (Distributed Map)** - phân tán lưới tham số của một job ra nhiều Worker Lambda chạy song song và tổng hợp kết quả, có sẵn retry/catch để job lỗi được đánh dấu `FAILED` thay vì treo mãi.
- **Amazon DynamoDB** (on-demand) - trạng thái job/kết quả, tài khoản/vị thế/lệnh/P&L paper trading, và theo dõi kết nối WebSocket. Được chọn thay cho ElastiCache/Timestream trong kế hoạch ban đầu chính vì gần như $0 khi rảnh (xem Mục 4).
- **Amazon API Gateway** - một REST API (bảo vệ bằng API key) cho engine backtest, một REST API thứ hai cho paper trading, và một WebSocket API để đẩy thông báo khớp lệnh thời gian thực.
- **AWS Glue + Amazon Athena** - catalog dùng partition projection trên data lake S3, cho phép kiểm tra dữ liệu đã ingest bằng SQL thuần mà không cần crawler.
- **Amazon EventBridge** - lịch chạy hàng ngày giữ watchlist symbol luôn cập nhật.
- **Amazon CloudWatch + Amazon SNS** - dashboard và 8 alarm (lỗi Lambda, throttle Worker, lỗi Step Functions) trên toàn bộ Lambda và state machine, kết nối tới một SNS topic.
- **Amazon SQS** - dead-letter queue cho Lambda duy nhất được gọi bất đồng bộ (hàm ingestion do EventBridge kích hoạt); mọi Lambda khác đều được gọi đồng bộ nên không cần DLQ.

{{% notice note %}}
**Toàn bộ stack không dùng VPC, subnet, hay Availability Zone.** Mọi service dùng ở đây đều là managed service của AWS, truy cập qua AWS API plane. VPC chỉ thực sự cần khi có resource bắt buộc phải tách khỏi Internet công cộng, ví dụ như các dịch vụ RDS, ElastiCache/Redis, hay container nằm sau ALB nội bộ, mà hệ thống này không dùng cái nào cả. DynamoDB on-demand đảm nhận vai trò mà ElastiCache và Timestream lẽ ra phải làm (xem phần thay đổi thiết kế bên dưới).
{{% /notice %}}

**Thiết kế thành phần**
- **Data Lake**: Ingestion Lambda lấy dữ liệu klines từ REST API của Binance và ghi thành Parquet, partition theo symbol/interval/year/month, có lịch EventBridge hàng ngày giữ watchlist luôn mới.
- **Lõi engine backtest**: module Python thuần (`services/common/engine.py`) hiện thực interface chiến lược, simulator equity curve theo từng nến, và các metric tổng hợp (Sharpe ratio, max drawdown, win rate), unit test cục bộ, không cần AWS.
- **Compute phân tán**: Orchestrator Lambda biến một request thành một lượt thực thi Step Functions; Distributed Map chạy một Worker Lambda cho mỗi tổ hợp tham số; Aggregator xếp hạng kết quả theo Sharpe ratio vào DynamoDB.
- **Lớp API**: REST API đứng trước Orchestrator (tạo job) và một Lambda chỉ đọc (trạng thái/kết quả), bảo vệ bằng API key.
- **Paper trading thời gian thực**: Trading Lambda khớp lệnh giả lập theo giá ticker Binance thực, cập nhật DynamoDB, và đẩy kết quả khớp lệnh tới mọi kết nối WebSocket đã đăng ký cho tài khoản đó.
- **Observability**: dashboard CloudWatch và bộ alarm được xây thành một stack riêng, chỉ đọc metric theo tên/ARN từ 4 stack còn lại, không có ràng buộc IAM hai chiều.

### 4. Triển khai kỹ thuật

**Các giai đoạn triển khai**

Dự án được xây dựng và deploy lên hạ tầng AWS ở cuối mỗi phase:

| Phase | Nội dung |
|---|---|
| 0 | Project scaffolding: CDK app (Python), cấu trúc repo |
| 1 | Data Lake: Ingestion Lambda từ Binance, S3 Parquet, Glue/Athena |
| 2 | Lõi engine backtest: interface chiến lược, simulator, metrics (có unit test) |
| 3 | Compute phân tán: Step Functions Distributed Map, Orchestrator/Worker/Aggregator Lambda |
| 4 | Lớp API: REST API + xác thực bằng API key |
| 5 | Paper trading thời gian thực: khớp lệnh, trạng thái DynamoDB, đẩy WebSocket |
| 6 | Tối ưu chi phí & hiệu năng: S3 byte-range fetch, cache `/tmp`, lõi Rust/PyO3 |
| 7 | Observability & hardening: dashboard/alarm CloudWatch, SQS DLQ |

**Hai điểm lệch có chủ đích so với thiết kế ban đầu**, cả hai đều phát hiện và xử lý trong lúc thực sự xây dựng trên AWS chứ không phải giả định trước:
- **ElastiCache → DynamoDB on-demand** cho trạng thái paper trading: ElastiCache Serverless tính phí storage + ECPU ngay cả khi không có traffic, trong khi DynamoDB on-demand gần như miễn phí lúc rảnh, đánh đổi hợp lý cho một dự án không chạy liên tục.
- **Amazon Timestream → DynamoDB** cho lịch sử P&L: Timestream for LiveAnalytics hóa ra **không có mặt ở region `ap-southeast-1`**, được xác nhận qua một lần deploy lỗi (`Unrecognized resource types: [AWS::Timestream::Table, AWS::Timestream::Database]`), không phải suy đoán nên các điểm dữ liệu P&L được ghi vào bảng DynamoDB thay thế.

**Yêu cầu kỹ thuật**
- AWS CDK (Python) cho toàn bộ hạ tầng - một CDK app, năm stack, mỗi stack deploy độc lập.
- Lambda runtime: Python 3.11 thuần cho các handler nhẹ (API, orchestrator, trading); container-image Lambda cho những phần cần pandas/pyarrow (ingestion, worker) vì các thư viện này vượt giới hạn 250MB của zip package; extension Rust/PyO3 (build qua `maturin` trong Dockerfile multi-stage) cho phần tính toán metrics tốn CPU nhất của worker.
- REST API công khai của Binance (`/api/v3/klines` cho dữ liệu lịch sử, `/api/v3/ticker/price` cho giá khớp lệnh paper trading thời gian thực) - miễn phí, không cần license/API key, phù hợp cho dự án crypto-quant cá nhân.

### 5. Lộ trình & Mốc triển khai

- **Tháng 1**: Kiến thức nền AWS, thiết lập account/CLI, Phase 0–2 (scaffolding, data lake, lõi engine backtest).
- **Tháng 2**: Phase 3–5 (compute phân tán, lớp API, paper trading thời gian thực) - phần lõi sản phẩm đầu-cuối.
- **Tháng 3**: Phase 6–7 (tối ưu hiệu năng Rust/PyO3, observability/hardening), kiểm chứng cuối cùng, tháo dỡ hạ tầng, và viết báo cáo (tài liệu này).

### 6. Ước tính ngân sách

Mọi dịch vụ AWS dùng trong dự án đều tính phí theo request kiểu serverless (Lambda, API Gateway, Step Functions) hoặc lưu trữ on-demand (S3, DynamoDB). Hệ thống được destroy (`cdk destroy --all`) mỗi khi không đang được xây dựng/demo, nên chi phí thực tế trong suốt kỳ thực tập chỉ là một phần nhỏ so với ước tính "dùng nhẹ" bên dưới.

**Ước tính chi phí hàng tháng ở mức sử dụng cá nhân nhẹ** (vài chục job backtest/tháng, hoạt động paper trading không thường xuyên, dashboard + 8 alarm luôn bật):

| Dịch vụ | Chi phí ước tính/tháng |
|---|---|
| Lambda (toàn bộ function, worker tăng tốc Rust) | < 0,50 USD |
| API Gateway (2× REST + 1× WebSocket) | < 0,10 USD |
| Step Functions (Distributed Map) | < 0,10 USD |
| DynamoDB (on-demand, cả 8 bảng) | < 0,20 USD |
| S3 (lưu trữ Parquet OHLCV) | < 0,10 USD |
| CloudWatch (dashboard + 8 alarm) | ~0,80 USD |
| SNS / SQS | không đáng kể |
| **Tổng** | **≈ 1–2 USD/tháng khi dùng tích cực** |

Con số này khớp với kết quả benchmark thật ở Phase 6: chuyển Worker Lambda từ Python thuần sang kết hợp byte-range fetch + cache `/tmp` + lõi Rust giúp giảm billed duration trung bình từ 419,4 ms xuống 44,4 ms mỗi lần gọi, **nhanh gấp 9,4 lần và giảm ~89% chi phí compute Lambda** cho function được gọi nhiều nhất hệ thống, đo trên AWS thật chứ không phải ước tính.

### 7. Đánh giá rủi ro

**Ma trận rủi ro**
- Tính sẵn sàng của API bên thứ ba (Binance): ảnh hưởng trung bình, xác suất thấp, lịch chạy hàng ngày và luồng backfill thủ công của ingestion Lambda đều "graceful degrade" (một lần chạy lỗi không làm hỏng các partition đã ingest trước đó).
- Quota dịch vụ ở cấp tài khoản: ảnh hưởng trung bình, xác suất trung bình, phát hiện thực tế khi benchmark Phase 6, quota concurrent-execution mặc định của Lambda trên tài khoản này (10) thấp hơn `max_concurrency` cấu hình cho Distributed Map (50), nên một job backtest đủ lớn có thể bị throttle.
- Tính sẵn sàng của dịch vụ theo region: ảnh hưởng thấp, xác suất thấp một khi đã biết, Amazon Timestream không khả dụng ở `ap-southeast-1`, phát hiện qua một lần deploy lỗi ở Phase 5.

**Chiến lược giảm thiểu**
- Phụ thuộc Binance: gắn SQS dead-letter queue cho Lambda duy nhất được gọi bất đồng bộ (ingestion) để một lần chạy lịch bị lỗi không âm thầm mất dữ liệu.
- Quota concurrency: đã ghi rõ trong README của dự án như một giới hạn đã biết; cách khắc phục (xin tăng quota, hoặc giảm `max_concurrency`) chỉ là thay đổi một dòng/một ticket, cố tình chưa áp dụng trước khi thực sự cần.
- Tính sẵn sàng theo region: kiểm chứng thực tế cho từng dịch vụ AWS thay vì suy đoán từ tài liệu, và thiết kế lại (dùng DynamoDB) thay vì đi vòng (mở thêm region).

**Kế hoạch dự phòng**
- Mỗi stack có thể destroy và deploy lại độc lập (`cdk destroy`/`cdk deploy` theo từng stack), nên một lần deploy lỗi luôn khôi phục được trong vài phút.
- Các bảng DynamoDB chứa kết quả backtest hoặc trạng thái paper trading thật dùng `RemovalPolicy.RETAIN`, nên việc tháo dỡ stack không bao giờ âm thầm làm mất dữ liệu quan trọng.

### 8. Kết quả kỳ vọng

**Cải tiến kỹ thuật**
Một hệ thống serverless đầu-cuối hoạt động thật, được kiểm chứng trên hạ tầng AWS thật ở mọi phase, không chỉ thiết kế trên giấy: job backtest thật chạy qua Step Functions, lệnh paper trading thật khớp theo giá Binance thực, alarm CloudWatch thật được quan sát ở trạng thái `OK`, và cải thiện hiệu năng 9,4 lần thật, đo được, từ phase tối ưu Rust.

**Giá trị dài hạn**
Codebase và các CDK stack có thể tái sử dụng làm nền tảng để thêm chiến lược mới, thêm sàn giao dịch, hoặc xây frontend thật; các điểm lệch đã ghi lại (dùng DynamoDB thay ElastiCache/Timestream, dùng WebSocket API thay AppSync) và các giới hạn account/region đã phát hiện được lưu lại để không phải khám phá lại ở dự án serverless tiếp theo.

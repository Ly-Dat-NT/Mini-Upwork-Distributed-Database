# Sơ đồ thiết kế khái niệm

[Trang chủ](../../README.md) · [Báo cáo gốc](../report/Project_Report_DB.docx)

Ba góc nhìn dưới đây được dựng lại từ danh mục thực thể, chuyên biệt hóa và bảng quan hệ ở mục 2.2–2.4 của báo cáo. Đây là bản diễn giải bằng Mermaid cho GitHub, không phải ảnh gốc của các Figure 2.1–2.3. File Word hiện có chú thích hình nhưng không chứa ảnh sơ đồ nhúng.

Các sơ đồ mô tả thiết kế dự kiến, chưa thể hiện lược đồ SQL hay kiến trúc phân tán đã triển khai.

## 1. Tổng quan các nhóm thực thể

```mermaid
flowchart TB
    subgraph Identity["Tài khoản và hồ sơ"]
        U["USER"]
        C["CLIENT"]
        F["FREELANCER"]
        U --> C
        U --> F
    end
    subgraph Marketplace["Công việc và đề xuất"]
        CAT["CATEGORY"]
        S["SKILL"]
        FS["FREELANCER_SKILL"]
        J["JOB"]
        FJ["FIXED_PRICE_JOB"]
        HJ["HOURLY_JOB"]
        JS["JOB_SKILL"]
        P["PROPOSAL"]
        CAT --> J
        F --> FS
        S --> FS
        S --> JS
        J --> JS
        J --> FJ
        J --> HJ
        J --> P
        F --> P
    end
    subgraph Execution["Hợp đồng và thực hiện"]
        CT["CONTRACT"]
        FC["FIXED_PRICE_CONTRACT"]
        HC["HOURLY_CONTRACT"]
        M["MILESTONE"]
        T["TIME_ENTRY"]
        I["WEEKLY_INVOICE"]
        CT --> FC
        CT --> HC
        FC --> M
        HC --> T
        HC --> I
        I --> T
    end
    subgraph Settlement["Thanh toán và đánh giá"]
        E["ESCROW_FUNDING"]
        PAY["PAYMENT_TRANSACTION"]
        R["REVIEW"]
        RS["RATING_SUMMARY"]
        M --> E
        E --> PAY
        I --> PAY
        CT --> PAY
        CT --> R
        R -.-> RS
    end
    C --> J
    P --> CT
```

Mũi tên tổng quan biểu thị liên hệ giữa các nhóm, không biểu thị lực lượng quan hệ hoặc thứ tự xử lý. Góc nhìn này không vẽ mọi quan hệ payer/payee và reviewer/reviewee để giữ dễ đọc.

## 2. Marketplace, kỹ năng và proposal

```mermaid
classDiagram
    USER <|-- CLIENT
    USER <|-- FREELANCER
    JOB <|-- FIXED_PRICE_JOB
    JOB <|-- HOURLY_JOB
    CLIENT "1" --> "0..*" JOB : posts
    CATEGORY "0..1" --> "0..*" CATEGORY : parent_of
    CATEGORY "1" --> "0..*" JOB : classifies
    FREELANCER "1" --> "0..*" FREELANCER_SKILL : has
    SKILL "1" --> "0..*" FREELANCER_SKILL : identifies
    JOB "1" --> "0..*" JOB_SKILL : requires
    SKILL "1" --> "0..*" JOB_SKILL : identifies
    FREELANCER "1" --> "0..*" PROPOSAL : submits
    JOB "1" --> "0..*" PROPOSAL : receives
    PROPOSAL "1" --> "0..1" CONTRACT : creates
```

- USER → CLIENT/FREELANCER: **total, overlapping**; mỗi user có ít nhất một vai trò, có thể có cả hai.
- JOB → FIXED_PRICE_JOB/HOURLY_JOB: **total, disjoint**; mỗi job thuộc đúng một loại.
- JOB_SKILL được phép chưa có khi nháp; trước khi OPEN, job phải có ít nhất một kỹ năng yêu cầu.
- Mỗi cặp freelancer–job có tối đa một proposal; lần gửi lại dùng cùng vòng đời.
- Chỉ proposal ACCEPTED mới tạo hợp đồng. Mỗi job có tối đa một proposal được chấp nhận và một hợp đồng trong phạm vi đồ án.
- Không được ứng tuyển job của chính tài khoản mình.

## 3. Hợp đồng, công việc, thanh toán và đánh giá

```mermaid
classDiagram
    CONTRACT <|-- FIXED_PRICE_CONTRACT
    CONTRACT <|-- HOURLY_CONTRACT
    FIXED_PRICE_CONTRACT "1" --> "0..*" MILESTONE : contains
    HOURLY_CONTRACT "1" --> "0..*" TIME_ENTRY : records
    HOURLY_CONTRACT "1" --> "0..*" WEEKLY_INVOICE : bills
    WEEKLY_INVOICE "0..1" --> "1..*" TIME_ENTRY : summarizes
    MILESTONE "1" --> "0..*" ESCROW_FUNDING : funding_history
    CONTRACT "1" --> "0..*" PAYMENT_TRANSACTION : owns
    ESCROW_FUNDING "0..1" --> "0..*" PAYMENT_TRANSACTION : source_A
    WEEKLY_INVOICE "0..1" --> "0..*" PAYMENT_TRANSACTION : source_B
    USER "1" --> "0..*" PAYMENT_TRANSACTION : payer
    USER "1" --> "0..*" PAYMENT_TRANSACTION : payee
    PAYMENT_TRANSACTION "0..1" --> "0..*" PAYMENT_TRANSACTION : original_for_correction
    CONTRACT "1" --> "0..2" REVIEW : permits
    USER "1" --> "0..*" REVIEW : reviewer
    USER "1" --> "0..*" REVIEW : reviewee
    USER "1" --> "0..1" RATING_SUMMARY : has
```

- CONTRACT → FIXED_PRICE_CONTRACT/HOURLY_CONTRACT: **total, disjoint** và phải khớp loại job.
- Hợp đồng trọn gói phải có ít nhất một milestone trước ACTIVE; tổng số tiền milestone phải bằng agreed_amount.
- Hợp đồng theo giờ có time entry và invoice; mỗi time entry thuộc tối đa một invoice, mỗi invoice tổng hợp ít nhất một entry đủ điều kiện.
- **Diễn giải lịch sử escrow:** sơ đồ cho phép nhiều funding trong lịch sử; BR-30 giới hạn **tối đa một funding đang hoạt động** cho mỗi milestone. Chính sách tái cấp vốn cần được chốt ở bước thiết kế logic.
- **XOR nguồn thanh toán:** source_A và source_B là tùy chọn khi xét riêng; mỗi payment bắt buộc có đúng một nguồn. Nguồn phải thuộc cùng hợp đồng.
- Payer/payee phải là hai bên khác nhau của hợp đồng, phù hợp loại giao dịch. Giao dịch thành công chỉ được điều chỉnh bằng giao dịch bù liên kết bản gốc.
- Chỉ hợp đồng COMPLETED cho phép review; mỗi bên tối đa một review, điểm nguyên từ 1 đến 5.
- RATING_SUMMARY được suy ra từ các review hợp lệ nhận được.

## Cách đọc và cập nhật

Tam giác rỗng biểu thị kế thừa/chuyên biệt hóa. Các nhãn 1, 0..1, 0..* và 1..* biểu thị lực lượng quan hệ. Các ràng buộc total/disjoint/overlapping, XOR, trạng thái và tính duy nhất có điều kiện phải đọc cùng ghi chú; đường nối không tự thể hiện đầy đủ chúng.

Sửa trực tiếp các khối Mermaid trong file này khi mô hình được chốt lại. Chưa có số site, quy tắc phân mảnh hoặc cấu hình sao chép trong báo cáo nên repo chưa công bố sơ đồ triển khai phân tán.

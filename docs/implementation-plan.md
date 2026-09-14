## Kế hoạch triển khai

[Trang chủ](../README.md) · [Sơ đồ](diagrams/README.md) · [Báo cáo gốc](report/Project_Report_DB.docx)

Dự án đang ở giai đoạn lập kế hoạch. Các bước dưới đây là lộ trình đề xuất từ yêu cầu trong báo cáo, chưa phải cam kết tiến độ hoặc kết quả triển khai.

## 1. Chốt mô hình logic

- Ánh xạ 21 thực thể sang các quan hệ; xác định PK, FK, UNIQUE và các miền giá trị.
- Phân tích phụ thuộc hàm, khóa ứng viên và dạng chuẩn cho từng quan hệ.
- Làm rõ các trạng thái còn thiếu chuyển tiếp, như tiếp tục hợp đồng PAUSED và xử lý hóa đơn DISPUTED.
- Chốt cách lưu lịch sử escrow: báo cáo giới hạn một bản ghi funding đang hoạt động trên mỗi milestone, không đồng nghĩa chỉ có một bản ghi trong toàn bộ lịch sử.
- Chốt ý nghĩa từng loại giao dịch và hướng payer/payee trước khi hiện thực hóa escrow.

## 2. Xây dựng cơ sở dữ liệu

Bổ sung SQL tạo lược đồ, dữ liệu mẫu, các giao dịch nghiệp vụ, phân quyền và truy vấn minh họa. Mỗi quy tắc BR cần được liên kết với cơ chế thực thi và kịch bản kiểm tra phù hợp.

Chỉ bổ sung hướng dẫn cài đặt khi có lệnh chạy thực tế và thứ tự thực thi đã được kiểm chứng.

## 3. Kiểm chứng theo báo cáo

| Kịch bản | Kết quả mong đợi khi triển khai |

| Hợp đồng trọn gói | Milestone và escrow hợp lệ; thanh toán tham chiếu funding; không có time entry hoặc invoice theo giờ |
| Hợp đồng theo giờ | Thời gian không chồng lấn; hóa đơn tính đúng; thanh toán tham chiếu invoice |
| Người dùng hai vai trò | Được đăng việc và ứng tuyển việc khác; bị từ chối khi tự ứng tuyển |
| Proposal không hợp lệ | Từ chối bản ghi trùng cặp freelancer–job và ứng tuyển job chưa OPEN |
| Nguồn thanh toán sai | Từ chối khi có cả hai nguồn, không có nguồn hoặc nguồn khác hợp đồng |
| Bên giao dịch và điều chỉnh | Từ chối bên không thuộc hợp đồng; bảo vệ giao dịch thành công khỏi sửa trực tiếp |
| Review không hợp lệ | Từ chối người ngoài hợp đồng, đánh giá sớm và đánh giá trùng |

Các kết quả trong bảng là kỳ vọng từ mục 2.6 của báo cáo; chưa có bài kiểm thử được chạy.

## 4. Đánh giá và công bố minh chứng

- Đo truy vấn tìm kiếm với 10.000 jobs và 50.000 proposals; ghi rõ cấu hình, dữ liệu, chỉ mục, truy vấn và cách đo. Đối chiếu mục tiêu dưới 2 giây.
- Kiểm tra phân quyền, bảo vệ thông tin xác thực và dữ liệu nhạy cảm trong truy vấn hồ sơ công khai.
- Thực hành backup/restore và công bố điểm khôi phục đã kiểm chứng.
- Bổ sung kết quả thực nghiệm, hướng dẫn tái hiện và các giới hạn còn tồn tại.


## BankStatementAnalysing

## Phần 1: Yêu cầu phần mềm
# Chức năng chính của hệ thống:

1. **Tải dữ liệu**: Người dùng có thể tải lên các file sao kê giao dịch ngân hàng (CSV, Excel, hoặc các định dạng khác).

2. **Phân tích dữ liệu**: Hệ thống phân tích các giao dịch trong file, tính toán các chỉ số như tổng số tiền, số giao dịch, số tiền thu/chi, và số dư cuối kỳ,...

3. **Xuất kết quả**: Lưu kết quả phân tích vào thiết bị lưu trữ dưới dạng file (PDF, CSV, hoặc Excel).

---

# Tác nhân liên quan đến hệ thống:

- **Khách hàng**: Người sử dụng hệ thống để tải lên và phân tích dữ liệu giao dịch.
- **Hệ thống Ngân hàng**: Cung cấp các file sao kê giao dịch cho khách hàng.


---

UseCase Diagram
![markdown](https://www.planttext.com/plantuml/png/T991JWCn303lVeMrztu15LRm04hFoCgQ4f5DgjYHA5LVne4dyGMKj5asPJdQux5i9z-VNsjHYff61uvV53LWjBjNnS56Dcg31o2Z8MBN9z4mSkoG16jGuHtvDzmSH7aiFCS0kGNvdMFidY9veT8HRpsvWrYPX2CW8YPXGyBORVSkq81pvS4wBest_VCPvqohbROVFFUaEXSsHwtSapF9aJ9kbDIAMZF94_j5hNp_lWzHyP4bEPLynFcyPtg1S0AenyFLwkPgeMlKkj0KrvgbtPRJrLXQQ3dzpnpbkVzElsO_vA_q0m00__y30000)

---

# Đặc tả ca sử dụng: Thống kê giao dịch ngân hàng

## Tên ca sử dụng: Phân tích và thống kê giao dịch ngân hàng

### Tác nhân tham gia:
- **Khách hàng**

### Mô tả tóm tắt:
Người dùng tải lên file sao kê giao dịch, hệ thống phân tích dữ liệu và cung cấp báo cáo thống kê bao gồm tổng số giao dịch, số tiền thu/chi, số dư cuối kỳ, và các chỉ số liên quan.

### Tiền điều kiện:
- Người dùng đã đăng nhập vào hệ thống.
- File sao kê giao dịch phải có định dạng hợp lệ (CSV, Excel, v.v.).

### Luồng sự kiện chính:
1. Người dùng chọn chức năng "Tải lên file giao dịch".
2. Hệ thống hiển thị giao diện để người dùng chọn file từ thiết bị.
3. Người dùng tải file lên.
4. Hệ thống kiểm tra định dạng file:
   - Nếu hợp lệ, chuyển sang bước 5.
   - Nếu không, hiển thị thông báo lỗi và yêu cầu tải lại.
5. Hệ thống phân tích dữ liệu và hiển thị kết quả thống kê (tổng số giao dịch, tổng số tiền thu/chi, số dư cuối kỳ).
6. Người dùng chọn chức năng "Lưu kết quả".
7. Hệ thống lưu file kết quả vào thiết bị lưu trữ của người dùng.

### Luồng sự kiện phụ:
- **Khi file bị lỗi**: Hệ thống thông báo lỗi, yêu cầu người dùng tải lại file đúng định dạng.
- **Khi phân tích thất bại**: Hệ thống thông báo lỗi kỹ thuật và khuyến nghị thử lại sau.

### Hậu điều kiện:
- Kết quả thống kê được hiển thị và/hoặc lưu trữ thành công.

### Các ngoại lệ:
- Lỗi kết nối mạng khi tải file.
- File quá lớn không thể xử lý.
- Lỗi phân tích do dữ liệu trong file không hợp lệ.

---
# Yêu cầu phi chức năng

- Hệ thống phải có khả năng xử lý phân tích một tệp sao kê ngân hàng có kích thước 100MB trong thời gian không quá 5 phút.

---
## Phần 2: Thiết kế và cài đặt

## Vấn đề 1: Sự phụ thuộc mạnh giữa `BankStatement` và `TransactionParser`

### Mô tả:
`BankStatement` đang sử dụng trực tiếp `TransactionParser` thông qua phương thức `parseFile()`. Điều này vi phạm nguyên tắc thiết kế **Dependency Inversion Principle (DIP)** vì `BankStatement` phụ thuộc vào cách thức cụ thể mà `TransactionParser` thực hiện việc phân tích tệp. Điều này làm giảm tính linh hoạt của hệ thống, vì mỗi lần có sự thay đổi trong cách thức phân tích tệp, `BankStatement` cũng cần phải thay đổi.

## Vấn đề 2: `Transaction` chịu trách nhiệm quá nhiều (God Class)

### Mô tả:
Lớp `Transaction` hiện đang gánh quá nhiều trách nhiệm, bao gồm việc xử lý loại giao dịch (phân loại giao dịch - `categorize`) và kiểm tra tính hợp lệ của giao dịch (kiểm tra tính hợp lệ - `isValid`). Điều này vi phạm nguyên tắc **Single Responsibility Principle (SRP)**, vì mỗi lớp chỉ nên có một lý do để thay đổi. Việc để một lớp đảm nhiệm nhiều trách nhiệm làm cho lớp đó trở nên khó bảo trì và mở rộng.


## Vấn đề 3: `StatementAnalyzer` có thể thiếu tính mở rộng

### Mô tả:
`StatementAnalyzer` hiện đang sử dụng danh sách `TransactionParser`, nhưng không chỉ rõ cách thức nó chọn parser phù hợp để phân tích tệp (ví dụ: CSV, PDF). Điều này có thể gây ra vấn đề khi hệ thống cần mở rộng để hỗ trợ thêm các định dạng tệp mới. Nếu không có cơ chế để chọn parser phù hợp, việc thêm hỗ trợ cho các định dạng mới sẽ trở nên phức tạp và làm giảm tính linh hoạt của hệ thống.


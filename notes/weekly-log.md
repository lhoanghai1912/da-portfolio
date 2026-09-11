# Nhật ký tuần

Mỗi Chủ nhật viết 150 từ **bằng lời của mình**: tuần này học được gì, chỗ nào còn mơ hồ.
Đây là phần dễ bỏ nhất nhưng tạo khác biệt lớn nhất — nó biến "đã xem" thành "giải thích được", mà phỏng vấn chỉ đo cái thứ hai.

## Week 0 — Dựng môi trường
- Đã cài: duckdb 1.5.5 · uv 0.12.10 · docker 29.6.1 · git 2.50.1 · DBeaver · VS Code
- Python: pandas 3.0.5, jupyter, seaborn, scipy, matplotlib — import OK
- Repo: https://github.com/lhoanghai1912/da-portfolio
- 3 thứ chưa rõ:
  1.
  2.
  3.

## Week 1 — Tư duy dữ liệu

### Buổi 1 (W1.1) — Grain, kiểu dữ liệu, NULL, giới hạn dữ liệu

Học được gì:

**Grain** là khái niệm nền. Một dòng Superstore = 1 sản phẩm trong 1 đơn hàng, nên 9.994 dòng chỉ ứng với 5.009 đơn. Đếm nhầm grain thì AOV ra 229,9 thay vì 458,6 — sai gần một nửa mà query vẫn chạy bình thường. Mẹo nhận biết nhanh: nhìn hai dòng cùng một đơn, cột nào bị chép lại giống hệt thì thuộc grain cao hơn, không cộng/đếm thẳng được.

**Kiểu dữ liệu do công cụ đoán, phải soát lại.** Cùng một file, DuckDB đọc `Postal Code` thành text giữ được `06824`, còn pandas đọc thành số làm mất số 0 đầu. Lỗi này không báo gì, chỉ âm thầm làm JOIN rơi mất dòng.

**NULL khác 0.** `AVG` bỏ qua NULL nên mẫu số nhỏ đi và con số bị thổi lên — ví dụ 10% thay vì 6,7%. Khi điều tra ra sự thật thì phải điền vào; khi không điều tra được thì báo kèm độ phủ và khoảng dao động thay vì giấu.

**Phần đáng giá nhất là biết dữ liệu KHÔNG nói được gì.** `Profit` hóa ra là biên gộp tính sẵn từ giá bán và chiết khấu, không phải lợi nhuận kế toán — nên không kết luận được về hiệu quả kinh doanh. Cột `Segment` nhìn như công cụ phân khúc nhưng doanh thu mỗi khách ba nhóm gần bằng nhau (2.840 / 2.992 / 2.903) nên thực chất vô dụng để target. Và toàn bộ dữ liệu bắt đầu từ lúc đơn hàng đã hình thành, nên không dựng được funnel.

Ba chỗ sai và đã sửa: viết grain thiếu ngữ cảnh đơn hàng · xếp nhầm `Profit` vào nhóm cộng vô nghĩa · biết sự thật rồi mà vẫn báo con số cũ thay vì điền lại dữ liệu.

### Buổi 2 (W1.2) — chưa làm


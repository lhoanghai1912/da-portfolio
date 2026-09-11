# W1.1 — Tư duy dữ liệu (Superstore) — ĐÃ HOÀN THÀNH

Dataset: `data/superstore_utf8.csv` · Buổi 1 · Lý thuyết: https://lhoanghai1912.github.io/da-roadmap/ly-thuyet/l1-foundation

---

## Phần 1 — Grain

**1.1 Một dòng đại diện cho cái gì?**

> **1 dòng = 1 sản phẩm trong 1 đơn hàng.**

Ghi chú cách diễn đạt: câu "1 dòng = 1 sản phẩm" là **chưa đủ** — thiếu ngữ cảnh đơn hàng. Một sản phẩm được bán trong nhiều đơn nên xuất hiện ở nhiều dòng. Và gọi theo **khóa** (`Product ID`) chứ không theo nhãn (`Product Name`).

**1.2 Khách đặt 1 đơn gồm 3 sản phẩm → bảng có mấy dòng?** → 3 dòng.

**1.3 Khóa nghiệp vụ thật?** → cặp `(Order ID, Product ID)`.

Kiểm chứng:
```
so_dong            = 9.994
Order ID duy nhat  = 5.009     -> khong phai khoa
Product ID duy nhat= 1.862     -> khong phai khoa
cap (Order, Product)= 9.986    -> gan bang so dong -> DAY la khoa
```
Lệch 8 dòng = 8 dòng trùng cặp → **dữ liệu bẩn**, cần hỏi người vận hành: khách mua cùng sản phẩm 2 lần trong 1 đơn, hay nhập liệu lặp?

---

## Phần 2 — Các con số nền

| # | Chỉ số | Giá trị |
|---|---|---|
| 2.1 | Số dòng | 9.994 |
| 2.2 | Số đơn hàng duy nhất | 5.009 |
| 2.3 | Số khách duy nhất | 793 |
| 2.4 | Số sản phẩm duy nhất | 1.862 |
| 2.5 | Dòng trung bình mỗi đơn | 2,0 |
| 2.6 | Số cột có ô trống | 0 |
| 2.7 | Dòng trùng theo khóa nghiệp vụ | 8 |

---

## Phần 3 — Kiểu dữ liệu

**3.1 Cột nào công cụ đoán sai kiểu?**

DuckDB đoán **đúng hết** — kể cả `Postal Code` để `VARCHAR`.
Nhưng pandas đọc cùng file thì `Postal Code` thành `int64` → `06824` biến thành `6824`, **mất số 0 đầu**.
Sửa: `pd.read_csv(..., dtype={"Postal Code": str})`.

Pandas cũng đọc `Order Date` / `Ship Date` thành `str` nếu không truyền `parse_dates`.

**3.2 Cột kiểu số nhưng cộng lại vô nghĩa?**

| Cột | Vì sao |
|---|---|
| `Row ID` | số thứ tự nhân tạo, cộng ra 49.950.015 — vô nghĩa |
| `Postal Code` | mã định danh |
| `Discount` | là **tỷ lệ**, 20% + 30% ≠ 50% |

`Profit` **cộng được** — `SUM(Profit)` = 286.397, có ý nghĩa rõ. (Chỗ này ban đầu nhầm.)

Quy luật: **số để đo** thì cộng được · **số định danh** và **tỷ lệ** thì không.

---

## Phần 4 — NULL

**4.1 `Discount = 0` vs `NULL` khác nhau thế nào?**

- `0` = có đo, kết quả bằng không (đơn này thực sự không giảm giá)
- `NULL` = **không biết** đơn này có giảm giá hay không

**4.2 `AVG` xử lý NULL ra sao?** → **Bỏ qua NULL**, mẫu số nhỏ đi.

Ví dụ 3 đơn, giảm giá `0 / NULL / 0,2`:
```
AVG(giam_gia)              = 0,100 = 10,0%   (chia cho 2)
AVG(COALESCE(giam_gia,0))  = 0,067 =  6,7%   (chia cho 3)
```
Chênh 50%.

**Nguyên tắc rút ra:**
- Điều tra ra sự thật → **điền vào**, NULL không còn là NULL nữa
- Không điều tra được → báo số **kèm độ phủ và khoảng dao động**: *"10%, tính trên 2/3 đơn có dữ liệu; con số thật nằm trong khoảng 6,7%–13,3%"* (phân tích độ nhạy)
- Không bao giờ điền bừa `mean` hay `0` mà không kiểm

**4.3 Giá trị `0` có đáng tin không?** 4.798/9.994 dòng có `Discount = 0`. Bốn cách kiểm:

| Cách | Kết quả Superstore | Đọc |
|---|---|---|
| Ổn định theo thời gian | 47,0% · 48,7% · 48,2% · 48,0% | phẳng → không có dấu hiệu default fill |
| Phân bố theo nhóm | Central 35,6% → West 53,6% | chênh vừa phải, giải thích được bằng chính sách |
| Thang giá trị | 0 · 0,1 · 0,15 · 0,2 · 0,3 · 0,32 · 0,4... | bậc rời rạc → `0` là lựa chọn hợp lệ |
| Logic nghiệp vụ | nhóm 0% lãi TB +66,9 · nhóm 40% lỗ −111,9 | hành xử đúng kỳ vọng |

→ Tạm tin là giá trị thật. **Vẫn cần xác nhận với bộ phận vận hành.**

---

## Phần 5 — Giới hạn của dữ liệu

### 5.1 `Profit` không phải lợi nhuận thật

Kiểm chứng: cùng sản phẩm `FUR-CH-10002965`, cùng mức chiết khấu → tỷ lệ `Profit/Sales` **giống hệt** (0,1 → 0,189 · 0,2 → 0,088 · 0,3 → −0,043).
→ `Profit` là cột **tính ra** từ giá bán và chiết khấu, là **biên gộp ước tính**, không phải lợi nhuận kế toán.

Thiếu: chi phí vận hành · lương · mặt bằng · cước vận chuyển thật · marketing · hàng trả lại.

| Kết luận vẫn dùng được | Kết luận KHÔNG được rút ra |
|---|---|
| So sánh tương đối giữa nhóm (Tech 17,4% vs Furniture 2,5%) | "Công ty lãi 286K" — đó là lãi gộp |
| Ngưỡng chiết khấu 30% làm biên âm | "Nên dừng bán Tables" — chưa trừ chi phí chung, chưa biết Tables có kéo khách mua món khác |

### 5.2 Dữ liệu khách hàng — có tín hiệu, nhưng `Segment` thì không

Làm được: RFM, phân khúc theo giá trị, đo độ trung thành.
```
793 khach · 6,3 don/khach · chi 12 khach mua 1 lan
Nhom 20% chi nhieu nhat -> 48,1% doanh thu
```

**Phát hiện quan trọng:** `Segment` (Consumer / Corporate / Home Office) có doanh thu mỗi khách gần như bằng nhau — 2.840 / 2.992 / 2.903, chênh 5%.
→ **`Segment` không phân biệt được giá trị khách hàng**, không dùng để target được.

> Bài học: có một cột phân loại ≠ cột đó mang thông tin hữu ích. Phải kiểm bằng cách so chỉ số giữa các nhóm.

Thiếu để target được: nhân khẩu học · kênh tiếp cận · hành vi trước khi mua · phản hồi · lịch sử tương tác.

### 5.3 Không có dữ liệu hành vi trước khi mua

Dữ liệu bắt đầu **tại thời điểm đơn hàng đã hình thành**.
Không dựng được funnel · không tính được tỷ lệ bỏ giỏ hàng · không biết thời gian cân nhắc.

Chỉ tính được phần sau khi mua:
```
Thoi gian giao TB 3,96 ngay (0-7)
  Same Day 0,0 · First Class 2,2 · Second Class 3,2 · Standard Class 5,0
```
Max chỉ 7 ngày, không có đơn trễ bất thường → dữ liệu đã làm sạch sẵn, **đáng nghi so với dữ liệu vận hành thật**.

> **Câu tóm cho cả Phần 5:** đây là dữ liệu **giao dịch**, không phải dữ liệu **hành vi**. Trả lời tốt "bán được gì, cho ai, lãi gộp bao nhiêu". Không trả lời được "vì sao mua" và "vì sao không mua".

---

## Phần 7 — Kết luận buổi

### Ba chỗ đã hiểu sai và đã sửa

| Chỗ sai | Sửa thành |
|---|---|
| "1 dòng = 1 productName" | thiếu ngữ cảnh đơn hàng → **"1 sản phẩm trong 1 đơn hàng"**; gọi theo khóa, không theo nhãn |
| Xếp `Profit` vào nhóm "cộng vô nghĩa" | `Profit` **cộng được**; cột thứ ba là `Postal Code` |
| Khi đã điều tra ra sự thật vẫn báo 10% | biết sự thật rồi thì **điền vào**, con số đúng là 6,7% |

### Một câu về grain

> Grain là câu trả lời cho "một dòng trong bảng này là cái gì". Trả lời sai thì mọi con số sau đó sai, và không có lỗi nào báo ra.

### Còn lấn cấn — đưa vào đầu buổi sau

1. Khi nào thì được phép sửa dữ liệu, khi nào phải giữ nguyên và chỉ ghi chú
2. Cách trình bày khoảng dao động (phân tích độ nhạy) trong báo cáo cho người không chuyên
3. Làm sao biết một cột phân loại có mang thông tin hay không, trước khi dùng nó

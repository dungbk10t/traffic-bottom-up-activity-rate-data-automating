# Quy trình xử lý dữ liệu lưu lượng phương tiện từ camera giao thông

## Bước 1: Crawl & cắt video theo từng giờ
Từ video camera real-time, ứng với mỗi tuyến đường:
- Crawl và cắt dữ liệu theo từng giờ ứng với từng ngày.
- Lưu trữ thành thư mục theo ngày cụ thể (như data Viettel Camera trước đây).

---

## Bước 2: Lập lịch chạy YOLO để đếm lưu lượng phương tiện
Thiết lập scheduler chạy model YOLO vào khung giờ quota thấp (ví dụ 01:00 sáng).

Với mỗi video theo từng tuyến đường → đếm và xử lý lưu lượng phương tiện.

**Output:** Dữ liệu lưu lượng phương tiện của từng đường, theo giờ cụ thể của ngày cụ thể  
(hoặc có thể quy đổi sang “Ngày trong tuần” / “Ngày cuối tuần”).

### Ghi chú
- Chạy vào khung giờ cố định, đã lập lịch từ trước.
- Phải ghi LOG (ví dụ: chạy lúc 01:00 ngày 10/10/2025 → đếm lưu lượng ngày 09/10/2025).

---

## Bước 3: Chuyển hóa lưu lượng phương tiện từ đường camera (a) → đoạn đường trong CSDL (b)

Áp dụng cho các đoạn đường (b) có ánh xạ với đường camera giao thông (a).

### Trường hợp 1 — Có đủ dữ liệu 24h
→ Lưu lượng của đường (a) được chuyển trực tiếp thành lưu lượng của đoạn đường (b).

### Trường hợp 2 — Không đủ dữ liệu 24h
- Sử dụng bảng **hệ số lưu lượng theo giờ**.
- Từ lưu lượng “tiêu chuẩn 1 giờ”:
  - Nhân ngược với bảng hệ số để sinh ra lưu lượng của các khung giờ thiếu.

---

## Bước 4: Sinh dữ liệu cho các đoạn đường không được ánh xạ (Unmapped Segments)

Đối với các đoạn đường **không có camera ánh xạ**, lưu lượng phương tiện được tính theo công thức:

Q = Q1 × k × l


Trong đó:

| Ký hiệu | Ý nghĩa |
|--------|-----------------------------------------------------------|
| **Q**  | Lưu lượng phương tiện cần tính cho đoạn đường không ánh xạ |
| **Q1** | Lưu lượng tham chiếu lấy từ đoạn đường gần nhất hoặc có đặc tính tương tự |
| **k**  | Hệ số loại đường (theo cấp đường / loại hình giao thông) |
| **l**  | Hệ số chiều dài của đoạn đường |

Công thức này cho phép ước lượng lưu lượng cho các đoạn đường không có dữ liệu trực tiếp, dựa trên những đoạn đường tương tự có dữ liệu thực đo từ camera.

---

## Lưu ý về cơ chế cập nhật (drop & insert)
Quy trình từ Bước 1 → 4 là cơ chế:

Tức là mỗi lần có dữ liệu mới, toàn bộ lưu lượng của đoạn đường trong khung 24h được sinh lại.

---

## Thắc mắc 1
**Lần cập nhật thứ nhất đã đầy đủ dữ liệu Bước 4, lần 2 có cần cập nhật nữa không?**

→ **Có.**  
Vì cơ chế luôn là “drop and insert”, bất kể dữ liệu đã đầy đủ hay chưa.

---

## Thắc mắc 2
**Nên để năm hoạt động là 2024 hay 2025?**

- Nếu hệ thống đang chạy dữ liệu mới từ camera → dùng **2025**.
- Nếu backfill hoặc mô phỏng dữ liệu lịch sử → dùng **2024**.

---

## Bước 5: Refresh cache
Sau khi hoàn thành import dữ liệu, chạy refresh cho:
- Mapbox Emission  
- Emission By Province  
- Emission By Year  

Để cập nhật giao diện và dữ liệu phân tích.

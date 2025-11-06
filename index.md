# ĐẶC TẢ TÍNH NĂNG “MÃ ƯU ĐÃI” CHO TRANG NẠP (ONE-PAGER)

> **Phạm vi**: Chuẩn hoá logic & yêu cầu nghiệp vụ để AI dev triển khai vào `index.php`, tách khởi tạo DB sang `data.php`, bổ sung `admin.php`. **Không kèm mã nguồn**.

---

## 1) Mục tiêu
- Thêm tính năng **Mã Ưu Đãi** với 3 loại: **+% Xu**, **+Xu cố định**, **Giảm tiền VND**.
- Giao diện có **4 tab**: **Nạp Xu** (mặc định), **Mã Ưu Đãi**, **Sự kiện**, **Lịch sử**.
- **data.php**: tự tạo DB/bảng & cung cấp cấu hình (bỏ tự tạo DB trong `index.php`).
- **admin.php**: trang quản trị; chỉ tài khoản có `type = 2` trong `ny_auth` truy cập (khác → **404**).
- Dọn “config trong PHP” → **đưa hết vào DB**; trải nghiệm **mượt**, có **xem trước (preview)**.

---

## 2) Thuật ngữ & Tỉ lệ
- **Xu**: đơn vị ảo. Tỉ lệ cơ bản: **1 VND = 1 Xu** (đọc từ DB).
- **Đơn nạp**: yêu cầu nạp tiền → thanh toán → cộng Xu.
- **Điều kiện nạp tối thiểu**: số VND tối thiểu để mã hợp lệ.

---

## 3) Loại ưu đãi (chọn **1 mã** duy nhất khi thanh toán)
### [1] +% Xu (Bonus phần trăm Xu)
- Xu thưởng = `Xu gốc * %`.
- Ví dụ: 100k VND → 100k Xu; **+100%** → +100k Xu, **tổng 200k Xu**.

### [2] +Xu cố định
- Xu thưởng = **một số Xu cố định**.
- Ví dụ: 100k VND → 100k Xu; **+100k Xu** → **tổng 200k Xu**.

### [3] Giảm tiền nạp VND
- 2 kiểu: **% VND** _hoặc_ **VND cố định**.
- **Giảm số tiền phải trả**, **Xu vẫn theo số tiền gốc**.
- Ví dụ: 100k với **-10%** → **trả 90k** nhưng **nhận 100k Xu**.
- Ví dụ: giảm **100k**, điều kiện đơn ≥ **200k** → **trả 100k**, **nhận 200k Xu**.

> Cả 3 loại có thể **có điều kiện** (min VND) hoặc **không điều kiện**.

---

## 4) Quy tắc áp dụng mã
- Người dùng chỉ **chọn 1 mã** trong **tối đa 3 mã gợi ý** (hoặc **nhập mã riêng** do admin cấp).
- **Mã công khai**: hiển thị danh sách, bấm chọn.
- **Mã riêng**: nhập chuỗi mã, hệ thống kiểm tra hợp lệ tức thì.
- **Bắt buộc hợp lệ** đồng thời:
  1. `is_active = 1`
  2. **Chưa hết hạn** (HSD)
  3. **Đạt điều kiện nạp tối thiểu** (nếu có)
  4. **Còn lượt tổng** (`max_uses` > `used_count` nếu có)
  5. **Chưa vượt lượt/người** (`per_user_limit` nếu có)

---

## 5) Quy trình người dùng (User Flow)
1. Ở tab **Nạp Xu** (mặc định).
2. Chuyển sang **Mã Ưu Đãi** (tuỳ chọn):
   - Thấy **mã công khai** còn hạn/còn lượt (hiển thị **HSD**, **số lượt còn lại**, **ảnh**).
   - Hoặc **nhập mã** do admin cấp → kiểm tra hợp lệ.
   - **Chọn mã** → quay về **Nạp Xu** (mã đã “được chọn”).
3. **Nạp Xu**: nhập **VND** → hệ thống **preview** ngay:
   - **Xu gốc**, **Xu thưởng**, **Tổng Xu**, **Số tiền phải trả**.
4. **Tạo đơn** → (Bước sau mới tạo QR/thanh toán).
5. Thanh toán **thành công**:
   - Cộng Xu: `Xu gốc + Xu thưởng`.
   - Cập nhật **tích luỹ** người dùng (phục vụ **Sự kiện**).
   - Nếu dùng mã: ghi **lượt dùng**.
   - Ghi **Lịch sử**: “Tạo đơn”, “Dùng mã”, “Nhận quà sự kiện”.

---

## 6) Tab giao diện (4 tab)
- **Nạp Xu**: nhập VND, xem preview, tạo đơn.
- **Mã Ưu Đãi**:
  - Danh sách mã công khai còn hạn/còn lượt (hiển thị **HSD**, **lượt còn lại**, **ảnh 1024×512** được scale; thiếu ảnh dùng **ảnh mặc định**).
  - Ô **nhập mã** (mã do admin cấp).
- **Sự kiện**:
  - Hiển thị các sự kiện **đang bật** trong khung thời gian.
  - **Nhận quà**: thưởng **Xu** hoặc **Mã ưu đãi** (tự phát khi đạt điều kiện).
  - **Điều kiện tạm thời**: **tích luỹ Xu nạp** hoặc **tích luỹ VND nạp**.
- **Lịch sử**:
  - Liệt kê 3 loại bản ghi: **ORDER**, **PROMO_USE**, **EVENT_REWARD**.
  - Cho phép **xoá lịch sử** hiển thị của người dùng.

---

## 7) Công thức & Tính toán
- **Xu gốc** = `amount_vnd * rate_vnd_to_xu`.
- **Xu thưởng**:
  - [1] `floor(Xu gốc * % / 100)`
  - [2] `Xu cố định`
  - [3] `0`
- **Số tiền phải trả**:
  - [1]/[2]: `payable = amount_vnd`
  - [3]:
    - `% VND`: `payable = amount_vnd - floor(amount_vnd * % / 100)`
    - `VND cố định`: `payable = max(0, amount_vnd - flat_vnd)`
- **Tổng Xu nhận** = `Xu gốc + Xu thưởng`.

---

## 8) Dữ liệu cần lưu (các nhóm & thuộc tính)
### Cấu hình
- `rate_vnd_to_xu` (int)
- `default_promo_image_url` (string)
- … (các cấu hình khác nếu cần)

### Mã ưu đãi (Promotion)
- `code` (string, có thể rỗng = công khai), `title` (string)
- `type` ∈ {`PERCENT_XU`, `FLAT_XU`, `DISCOUNT_VND`}
- Nếu `PERCENT_XU`: `percent_xu` (decimal %)
- Nếu `FLAT_XU`: `flat_xu` (int)
- Nếu `DISCOUNT_VND`:  
  - `discount_subtype` ∈ {`PERCENT_VND`, `FLAT_VND`}
  - `percent_vnd` (decimal %) / `flat_vnd` (int)
- `min_amount_vnd` (int) — điều kiện tối thiểu
- `is_public` (bool), `is_active` (bool)
- `max_uses` (int, 0 = ∞), `used_count` (int)
- `per_user_limit` (int, 0 = ∞)
- `image_url` (string, có thể null)
- `expires_at` (datetime)

### Lượt dùng mã theo người
- `promotion_id`, `user_id`, `order_id`, `used_at`

### Đơn nạp (Order)
- `user_id`
- `original_amount_vnd`, `payable_amount_vnd`
- `xu_base`, `xu_bonus`
- `promotion_id` (nullable)
- `status` ∈ {`PENDING`, `PAID`, `CANCELLED`}
- `created_at`, `paid_at`

### Lịch sử người dùng
- `user_id`
- `type` ∈ {`ORDER`, `PROMO_USE`, `EVENT_REWARD`}
- `ref_id` (id liên quan), `note`, `created_at`

### Sự kiện
- `name`, `is_active`
- `starts_at`, `ends_at`
- `cond_type` ∈ {`ACCUM_XU`, `ACCUM_VND`}
- `cond_amount` (ngưỡng tích luỹ)
- `reward_type` ∈ {`XU`, `PROMO`}
- `reward_value` (số Xu hoặc `promotion_id`)

### Tích luỹ người dùng
- `user_id` (PK)
- `total_xu`, `total_vnd`, `updated_at`

---

## 9) Quy định giao diện “Mã Ưu Đãi”
- Mỗi mã hiển thị: **tiêu đề**, **mã/code** (nếu có), **HSD**, **lượt còn lại** (= `max_uses - used_count` hoặc `∞`), **ảnh** (1024×512, scale theo UI; thiếu ảnh dùng mặc định).
- Danh sách mã công khai **lọc sẵn**: chỉ hiển thị **còn hạn** & **chưa hết lượt**.
- Người dùng **chỉ được chọn 1 mã** (tối đa đề xuất 3 mã phù hợp).

---

## 10) Quản trị (admin.php)
- Chỉ cho phép **`ny_auth.type = 2`**; không đủ quyền → **404 Not Found**.
- **Quản lý cấu hình**: tỉ lệ VND→Xu, ảnh mặc định, …
- **Quản lý mã ưu đãi**: tạo/sửa/xoá; loại [1]/[2]/[3]; điều kiện tối thiểu; HSD; công khai/riêng tư; giới hạn lượt tổng & mỗi user; trạng thái bật/tắt; ảnh.
- **Quản lý sự kiện**: bật/tắt; thời gian; điều kiện (tích luỹ Xu/VND); thưởng (Xu/mã); phát thưởng tự động khi đạt.
- **Đơn hàng**: xem danh sách/trạng thái; (khi tích hợp thanh toán thật: nhận webhook cập nhật trạng thái).
- **Lịch sử**: tra cứu (read-only).

---

## 11) Tiêu chí nghiệm thu (Acceptance)
1. Chọn **1 mã** (trong 3 mã gợi ý hoặc nhập mã) → **Preview** hiển thị **Xu gốc / Xu thưởng / Tổng Xu / Phải trả** chính xác.
2. [1]/[2] **không đổi** số tiền phải trả; [3] **chỉ** đổi số tiền phải trả; **Xu luôn tính theo số tiền gốc**.
3. Kiểm tra đủ điều kiện: **hoạt động**, **HSD**, **tối thiểu VND**, **lượt tổng**, **lượt/người**.
4. Mã công khai hiển thị đúng **HSD**, **lượt còn lại**, **ảnh** (fallback ảnh mặc định nếu thiếu).
5. Sự kiện: **tự phát thưởng** đúng loại (Xu/mã) khi tích luỹ đạt ngưỡng; chỉ hoạt động trong **khung thời gian**.
6. Lịch sử có đủ 3 loại; cho phép người dùng **xoá** bản ghi hiển thị.
7. Người không có quyền admin → **404** khi truy cập `admin.php`.

---

## 12) Trải nghiệm & Hiệu năng (Guidelines)
- **Preview realtime** khi nhập VND/đổi mã.
- **Phân trang/lấy theo lô** danh sách mã; lọc server-side **hết hạn/hết lượt**.
- **Đưa config vào DB** (không hardcode trong PHP).
- **Tối ưu cache nhẹ** cho danh sách mã công khai (nếu traffic lớn).
- **Bảo mật**: kiểm soát quyền, ràng buộc input, chống XSS (khi render), CSRF cho thao tác quan trọng.
- **Xoá lịch sử** nên là “xoá hiển thị” (soft delete) nếu cần audit (tuỳ chính sách).

---

## 13) Ghi chú triển khai (không code, chỉ dàn ý)
- `data.php`: kết nối DB, khởi tạo bảng nếu thiếu, seed cấu hình cơ bản, expose hàm đọc config.
- `index.php`:  
  - 4 tab; luồng chọn mã → preview → tạo đơn → thanh toán thành công → cộng Xu & ghi lịch sử.  
  - API nhẹ (AJAX) cho: danh sách mã công khai; kiểm tra mã nhập; preview; tạo đơn; đánh dấu thanh toán (hook webhook sau này); lịch sử (list/xoá).
- `admin.php`:  
  - CRUD **mã ưu đãi**, **sự kiện**; xem **đơn**; bảo vệ quyền; trả **404** nếu không phải admin.

---

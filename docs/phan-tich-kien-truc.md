# Phan Tich Kien Truc Landing Page BeanStalk
# Phân Tích Kiến Trúc Landing Page BeanStalk

> Ngày phân tích: 14/04/2026
> Dự án: Landing Page BeanStalk — Nobinobi.vn
> Stack hiện tại: HTML tĩnh + Tailwind CSS + Firebase Hosting + GitHub Actions CI/CD

---

## 1. Tổng Quan 4 Hướng Triển Khai

### Hướng 1: Site tĩnh + Auto Rebuild (SSG)

```
[nobinobi API] ──giá đổi──→ [GitHub Actions Cron] ──build──→ [HTML mới]
                                                                 │
                                                                 ▼
                                   [Khách] ◄──────── [Firebase Hosting]
                                                      (HTML/CSS tĩnh)

❌ Không có Backend
❌ Không tạo đơn hàng tự động
❌ Turnstile chỉ verify client-side
```

### Hướng 2: Site động (Next.js + Vercel)

```
[Khách] ──request──→ [Vercel Server] ──gọi API──→ [nobinobi API]
                           │                            │
                           ◄────── render HTML ◄────────┘
                           │        (giá mới nhất)
                           ▼
                     [Trả HTML cho khách]

✅ Backend xử lý tất cả
✅ Tạo đơn + verify Turnstile server-side
⚠️  Viết lại toàn bộ code
```

### Hướng 3: Tĩnh + Firebase Cloud Functions (KHUYẾN NGHỊ)

```
[Khách] ──truy cập──→ [Firebase Hosting] ──trả HTML tĩnh──→ [Khách]
                                                                │
                            JS gọi API (fetch)                  │
                                   │                            ▼
            [Firebase Functions] ◄──┘                   [Hiển thị trang]
                   │
       ┌───────────┼───────────┐
       ▼                       ▼
[Cloudflare API]        [nobinobi API]
(verify Turnstile)      (lấy giá / tạo đơn)

✅ Giữ nguyên code hiện tại
✅ Chỉ thêm functions/
✅ Tạo đơn + verify Turnstile server-side
```

### Hướng 4: Cloudflare Pages + Workers (Edge Computing)

```
[Khách] ──truy cập──→ [Cloudflare Edge VN] ──trả HTML──→ [Khách]
                             │                               │
                  Workers chạy tại Edge                      ▼
                        │                           [Hiển thị trang]
            ┌───────────┼───────────┐
            ▼                       ▼
   [Turnstile verify]        [nobinobi API]
   (nội bộ, cực nhanh)      (lấy giá / tạo đơn)
            │                       │
            └───────┬───────────────┘
                    ▼
            [KV Store Cache]

✅ Nhanh nhất (Edge VN)
✅ Bảo mật cao nhất (DDoS miễn phí)
⚠️  Cần học Cloudflare ecosystem
```

---

## 2. Phân Công Frontend / Backend Mỗi Hướng

| Thành phần | HƯỚNG 1: Rebuild | HƯỚNG 2: Next.js | HƯỚNG 3: Firebase | HƯỚNG 4: Cloudflare |
|------------|-----------------|------------------|-------------------|---------------------|
| **FRONTEND** | | | | |
| Hiển thị HTML | ✅ | ✅ | ✅ | ✅ |
| Giỏ hàng (localStorage) | ✅ | ✅ | ✅ | ✅ |
| Turnstile widget | ✅ Client only | ✅ | ✅ | ✅ |
| Form đặt hàng | ❌ | ✅ | ✅ | ✅ |
| Gọi API Backend | ❌ Không có BE | ✅ | ✅ | ✅ |
| SEO (HTML tĩnh) | ✅ | ⚠️ Cần config | ✅ | ✅ |
| **BACKEND** | | | | |
| Lấy giá từ nobinobi | ❌ (GitHub Actions) | ✅ Server render | ✅ /api/products | ✅ Workers |
| Cache dữ liệu | ❌ HTML tĩnh | ✅ Server cache | ✅ Firestore | ✅ KV Store |
| Verify Turnstile server | ❌ | ✅ | ✅ | ✅ (nội bộ CF) |
| Validate dữ liệu | ❌ | ✅ | ✅ | ✅ |
| Tạo đơn trên nobinobi | ❌ | ✅ | ✅ | ✅ |
| Chống spam server-side | ❌ | ✅ | ✅ | ✅ |
| Giữ bí mật API Key | ❌ | ✅ | ✅ | ✅ |
| DDoS Protection | ❌ | ⚠️ Tùy plan | ⚠️ Tùy plan | ✅ Miễn phí |

---

## 3. Bảng So Sánh Chi Tiết

| Tiêu chí | H1: Rebuild | H2: Next.js | H3: Firebase | H4: Cloudflare |
|-----------|------------|------------|-------------|---------------|
| **Hạ tầng** | | | | |
| Hosting | Firebase | Vercel | Firebase | Cloudflare Pages |
| Backend | Không có | Next.js API Routes | Cloud Functions | CF Workers |
| Cache | HTML tĩnh | Server cache | Firestore | KV Store |
| Turnstile verify | Client only | Server | Server | Server (nội bộ) |
| **Đánh giá** | | | | |
| Tốc độ tải trang | ★★★★★ | ★★★★ | ★★★★★ | ★★★★★ |
| Cập nhật giá | ★★★ (trễ) | ★★★★★ (real-time) | ★★★★ (gần RT) | ★★★★ (gần RT) |
| Bảo mật | ★★★ | ★★★★ | ★★★★ | ★★★★★ |
| SEO | ★★★★★ | ★★★★ | ★★★★★ | ★★★★★ |
| Dễ triển khai | ★★★★ | ★★ | ★★★★ | ★★★ |
| Dễ bảo trì | ★★★★★ | ★★★ | ★★★★ | ★★★ |
| **Chi phí & thời gian** | | | | |
| Chi phí/tháng | ~0đ | 0-500K | ~0đ | ~0đ |
| Thay đổi code | Ít | Viết lại | Thêm ít | Trung bình |
| Thời gian triển khai | 2-3 ngày | 1-2 tuần | 2-3 ngày | 3-5 ngày |
| Độ khó (người mới) | Dễ | Khó | Dễ - TB | TB - Khó |
| **Tính năng** | | | | |
| Tạo đơn tự động | ❌ | ✅ | ✅ | ✅ |
| Verify Turnstile server | ❌ | ✅ | ✅ | ✅ |
| Cập nhật giá tự động | ✅ Có trễ | ✅ Real-time | ✅ Gần real-time | ✅ Gần real-time |
| Chống spam server-side | ❌ | ✅ | ✅ | ✅ |
| Chống DDoS | ❌ | ⚠️ Tùy plan | ⚠️ Tùy plan | ✅ Miễn phí |
| Giữ nguyên code cũ | ✅ | ❌ | ✅ | ⚠️ Cần migrate |

---

## 4. Luồng Đặt Hàng Hoàn Chỉnh (Hướng 3 - Khuyến nghị)

```
Khách truy cập Landing Page
    │
    ▼
[FRONTEND] Gọi Backend /api/products → Lấy giá mới nhất → Hiển thị
    │
    ▼
Khách thêm sản phẩm vào giỏ hàng
    │
    ▼
[FRONTEND] Lưu giỏ hàng vào localStorage
    │
    ▼
Khách bấm "Tiến hành đặt hàng"
    │
    ▼
[FRONTEND] Hiển thị Cloudflare Turnstile → Khách xác thực là người thật
    │
    ▼
[FRONTEND] Hiển thị form: Tên, SĐT, Địa chỉ, Ghi chú
    │
    ▼
Khách bấm "Xác nhận đặt hàng"
    │
    ▼
[FRONTEND] Gửi dữ liệu lên Backend /api/order:
    - Turnstile token
    - Thông tin khách (tên, SĐT, địa chỉ, ghi chú)
    - Giỏ hàng (danh sách sản phẩm + số lượng)
    │
    ▼
[BACKEND /api/order] Xử lý:
    1. Verify Turnstile token với Cloudflare → Nếu bot → Từ chối
    2. Validate dữ liệu: SĐT đúng format? Tên không trống? Có SP? → Nếu sai → Trả lỗi
    3. Gọi API nobinobi.vn tạo đơn hàng (trạng thái: "Chờ xác nhận")
    4. Trả kết quả về Frontend
    │
    ▼
[FRONTEND] Hiển thị kết quả:
    "Đặt hàng thành công! Mã đơn: #NB12345"
    "Nhân viên CSKH sẽ gọi lại trong 15 phút để xác nhận"
    │
    ▼
[NOBINOBI] CS thấy đơn mới → Gọi điện xác nhận → Đóng gói → Giao hàng
```

---

## 5. Phân Tích Chi Tiết Frontend & Backend (Hướng 3)

### Frontend — Nhiệm vụ

| STT | Nhiệm vụ | Mô tả |
|-----|----------|-------|
| 1 | Hiển thị sản phẩm | Render tên, giá, hình ảnh, mô tả lên giao diện |
| 2 | Cập nhật giá mới | Gọi Backend /api/products → cập nhật giá trên giao diện |
| 3 | Quản lý giỏ hàng | Thêm / xóa sản phẩm, tính tổng tiền (localStorage) |
| 4 | Thu thập thông tin khách | Form nhập: Họ tên, Số điện thoại, Địa chỉ, Ghi chú |
| 5 | Xác thực Turnstile | Hiển thị widget Cloudflare, lấy token gửi Backend |
| 6 | Gửi đơn hàng | Gom tất cả thông tin → gửi Backend /api/order |
| 7 | Hiển thị kết quả | Thông báo thành công (mã đơn, CS sẽ gọi lại) hoặc lỗi |
| 8 | SEO | HTML tĩnh có sẵn nội dung mặc định để Google quét được |
| 9 | Noscript fallback | Hiển thị sản phẩm + liên hệ khi trình duyệt tắt JavaScript |

**Frontend KHÔNG được làm:**
- Không giữ API Key nobinobi (sẽ bị lộ ra người dùng)
- Không tự verify Turnstile token (bot có thể bypass)
- Không gọi thẳng API nobinobi (lộ endpoint + API key)

### Backend — Nhiệm vụ

| STT | API Endpoint | Nhiệm vụ | Chi tiết |
|-----|-------------|----------|----------|
| 1 | GET /api/products | Lấy sản phẩm + giá | Gọi API nobinobi, cache 5-10 phút, trả JSON cho Frontend |
| 2 | POST /api/order | Tạo đơn hàng | Verify Turnstile → Validate data → Gọi API nobinobi tạo đơn |
| 3 | (Bảo mật chung) | Giữ bí mật | API Key nobinobi + Turnstile Secret Key lưu trong env variables |
| 4 | (Bảo mật chung) | Chống spam | Rate limit, kiểm tra honeypot, thời gian submit |

**Backend KHÔNG làm:**
- Không render giao diện (Frontend đảm nhận)
- Không lưu trữ dữ liệu khách hàng (nobinobi đảm nhận)
- Không xử lý thanh toán (CS xác nhận trước khi giao hàng)

---

## 6. API Cần Xin Từ Leader nobinobi.vn

### API 1: Lấy danh sách sản phẩm + giá

- Mục đích: Landing page tự động cập nhật tên, giá, hình ảnh, mô tả sản phẩm
- Endpoint mẫu: GET https://nobinobi.vn/api/products
- Response cần:
  - ID sản phẩm
  - Tên sản phẩm
  - Giá gốc + giá khuyến mãi
  - Hình ảnh (URL)
  - Mô tả ngắn
  - Trạng thái còn hàng / hết hàng

### API 2: Tạo đơn hàng

- Mục đích: Khách đặt trên landing page → tự động tạo đơn "Chờ xác nhận" trên nobinobi
- Endpoint mẫu: POST https://nobinobi.vn/api/orders
- Request body:
  - Tên khách hàng
  - Số điện thoại
  - Địa chỉ giao hàng
  - Ghi chú
  - Danh sách sản phẩm (ID + số lượng)
- Response cần:
  - Mã đơn hàng
  - Trạng thái đơn

### API 3: Webhook thông báo thay đổi giá (tùy chọn)

- Mục đích: Khi nobinobi cập nhật giá → trigger landing page cập nhật, không cần chờ cron
- Nếu không có webhook: landing page tự kiểm tra định kỳ (cache 5-10 phút)

### Thông tin bổ sung cần hỏi

| Câu hỏi | Lý do |
|----------|-------|
| nobinobi chạy trên nền tảng gì? (Haravan, Sapo, WooCommerce, tự build?) | Nếu có sẵn API chuẩn thì khỏi build thêm |
| API có rate limit không? (bao nhiêu request/phút) | Để cấu hình cache phù hợp |
| Có môi trường test/sandbox không? | Để dev test mà không ảnh hưởng đơn thật |
| Đơn hàng tạo qua API, CS xem ở đâu? | Đảm bảo CS nhận được đơn để gọi xác nhận |
| Cách xác thực API? (API Key, Bearer Token?) | Để Backend gọi đúng cách |

---

## 7. Khuyến Nghị Cuối Cùng

| Hướng | Tóm tắt | Khi nào chọn |
|-------|---------|-------------|
| Hướng 1 | Rebuild HTML khi giá đổi | Chỉ cần cập nhật giá, KHÔNG cần tạo đơn tự động |
| Hướng 2 | Viết lại bằng Next.js | Xây hệ thống lớn, nhiều trang, nhiều tính năng, có dev kinh nghiệm |
| **Hướng 3** | **Giữ nguyên + thêm Firebase Functions** | **Cần tạo đơn + cập nhật giá + triển khai NHANH + chi phí 0** |
| Hướng 4 | Chuyển sang Cloudflare toàn bộ | Ưu tiên hiệu năng + bảo mật tối đa, sẵn sàng học thêm |

**→ Khuyến nghị chọn Hướng 3: Phù hợp nhất với stack hiện tại, đội ngũ hiện tại, và yêu cầu kinh doanh.**

---

*Tài liệu được tạo bởi đội kỹ thuật Landing Page BeanStalk*

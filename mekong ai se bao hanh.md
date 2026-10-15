# Prototype mẫu — Track SE: Tiếp nhận phiếu bảo hành

Repo **mẫu** của học phần Chuyên đề tốt nghiệp 1 (Khoa CNTT, Trường Đại học Văn Lang).
Đây là ví dụ tham chiếu cho mức độ và cấu trúc mà Bài tập 2 và Bài tập 3 yêu cầu —
**không phải bài mẫu để sinh viên sao chép**. Luồng nghiệp vụ: L2 — tiếp nhận thiết bị
vào bảo hành (case study Smart CRM – Mekong Mobile).

## Hệ thống làm được gì

- Tiếp nhận một thiết bị vào bảo hành: kiểm hạn bảo hành, tính hạn xử lý theo SLA, tạo phiếu.
- Phân công kỹ thuật viên, có kiểm tải tối đa.
- Tra cứu phiếu theo số serial và theo mã phiếu.
- Cập nhật trạng thái phiếu, có kiểm soát truy cập theo trung tâm.

Bốn quy tắc nghiệp vụ được hiện thực: **QT-05** (còn hạn bảo hành), **QT-08** (SLA theo
mức ưu tiên), **QT-09** (một serial không có hai phiếu đang mở), **QT-11** (kỹ thuật viên
không quá 8 phiếu đang mở).

## Yêu cầu môi trường

- Docker Engine 24 trở lên và Docker Compose v2 (`docker compose version`).
- Cổng 8000 và 5432 còn trống (đổi được qua biến môi trường).
- Không cần cài Node.js trên máy để chạy hệ thống.
- Để chạy bộ kiểm thử **không cần Docker**: Node.js 20 trở lên (dùng `node --test`, không cần cài gói nào).

## Chạy hệ thống — hai lệnh

```bash
cp .env.example .env
docker compose up -d --build
```

## Kiểm tra hệ thống đã chạy đúng

```bash
docker compose ps                      # cột STATUS của db và api phải là "healthy"
curl -s http://localhost:8000/health   # {"status":"ok","db":"ok"}

# Dữ liệu mẫu đã được nạp tự động: 6 trung tâm, 8 kỹ thuật viên, 12 khách hàng,
# 10 sản phẩm, 3 phiếu bảo hành.
curl -s http://localhost:8000/api/tickets | head -c 400

# Tiếp nhận một thiết bị mới
curl -s -X POST http://localhost:8000/api/tickets \
  -H 'Content-Type: application/json' -H 'X-User-Id: 1' \
  -d '{"customerId":1,"serialNo":"SN-90001","productId":1,"centerId":1,
       "categoryId":1,"priority":"CAO","soldAt":"2026-05-01"}'
```

## Chạy bộ kiểm thử

```bash
npm test          # node --test — KHÔNG cần npm install
```

Bộ kiểm thử gồm 14 test case ở ba loại: luồng đúng, giá trị biên, dữ liệu sai.
Chúng chỉ dùng thư viện chuẩn của Node và các đối tượng giả, nên chạy được ngay
sau khi clone, không cần cơ sở dữ liệu và không cần cài phụ thuộc.

## Cấu trúc thư mục

```
src/routes/          Tầng 1 — nhận HTTP, dịch mã nghiệp vụ sang mã HTTP
src/services/        Tầng 2 — quy tắc nghiệp vụ, KHÔNG biết gì về HTTP
src/repositories/    Tầng 3 — truy cập CSDL, KHÔNG chứa quy tắc nghiệp vụ
src/validators/      Kiểm tra định dạng dữ liệu vào (tầng 1 của ba tầng kiểm tra)
migrations/          Tạo bảng, chạy tự động khi container db khởi tạo lần đầu
seeds/               Dữ liệu mẫu, chạy tự động sau migrations
tests/               Bộ kiểm thử
docs/                Toàn bộ tài liệu mà Bài tập 2 và Bài tập 3 yêu cầu
```

**Phép thử ranh giới tầng** (chạy được ngay, phải trả về rỗng):

```bash
grep -n "req\.\|res\.\|status(\|SELECT" src/services/*.js
grep -n "if (.*qua_tai\|SLA_HOURS" src/repositories/*.js
```

## Lưu ý về `package-lock.json`

Repo mẫu này không cần phụ thuộc nào để chạy test, nên chưa có `package-lock.json`.
Khi em chạy `npm install` (để dùng `express` và `pg` cho phần chạy thật ngoài Docker),
npm sẽ sinh ra tệp đó — **bắt buộc commit tệp đó vào repo của em**, đó là mục kiểm
chứng C6 của Bài tập 2.

## Tài liệu

| Tệp | Nội dung |
|---|---|
| `docs/test-plan.md` | Test Plan rút gọn 4 mục |
| `docs/test-report.md` | 14 test case đã chạy, đủ 8 cột |
| `docs/bug-log.md` | 3 bug đã sửa, mỗi bug 7 dòng |
| `docs/deployment-guide.md` | Deployment Guide 5 mục |
| `docs/checklist-bao-mat.md` | Checklist bảo mật 7 mục đã tự đánh giá |
| `docs/api-docs.md` | API docs — thành phần 6 của Bài tập 3 cho track SE |
| `docs/thay-doi-thiet-ke.md` | Nhật ký thay đổi so với thiết kế ban đầu |
| `docs/ai-disclosure.md` | Bảng khai báo sử dụng công cụ AI |

# Prototype mẫu — Track DA: Pipeline làm sạch và hợp nhất dữ liệu khách hàng

Repo **mẫu** của học phần Chuyên đề tốt nghiệp 1 (Khoa CNTT, Trường Đại học Văn Lang).
Ví dụ tham chiếu cho mức độ và cấu trúc mà Bài tập 2 và Bài tập 3 yêu cầu —
**không phải bài mẫu để sinh viên sao chép**. Luồng nghiệp vụ: L6 — hợp nhất khách
hàng trùng lặp từ nhiều cửa hàng (case study Smart CRM – Mekong Mobile).

## Pipeline làm được gì

```
nguồn CSV  →  extract  →  transform  →  quality  →  load  →  kho dữ liệu
                                          ↓
                                   bảng reject + báo cáo chất lượng
```

- **extract:** đọc `customers_raw.csv` và `orders_2024_2026.csv`, KHÔNG biến đổi gì.
- **transform:** chuẩn hoá số điện thoại (4 định dạng), đọc ngày (3 định dạng có thật:
  `dd/mm/yyyy` 62,3% · `d-m-yy` 23,5% · `yyyy.mm.dd` 13,8%),
  sinh khoá hợp nhất, hợp nhất khách hàng trùng lặp.
- **quality:** đo **ba chiều** chất lượng (đầy đủ · chính xác · nhất quán) —
  bảy phép đo, mỗi phép trả về số dòng lỗi và tỉ lệ lỗi.
- **load:** nạp vào bảng staging, dimension và fact; mọi dòng không xử lý được
  đều vào `reject_rows` kèm lý do viết cho người đọc.

**Ba mục kiểm chứng cốt lõi:** chạy bằng một lệnh · chạy lần hai không nhân đôi
dữ liệu · kiểm chất lượng chạy TRƯỚC khi nạp, không đạt ngưỡng thì dừng.

## Yêu cầu môi trường

- Docker Engine 24 trở lên và Docker Compose v2.
- Cổng 5433 và 8501 còn trống (đổi được qua biến môi trường).
- Để chạy **bộ kiểm thử** không cần Docker: Python 3.11 trở lên, `pip install -r requirements.txt`.

## Chạy hệ thống — hai lệnh

```bash
cp .env.example .env
docker compose up -d --build
```

Lệnh thứ hai dựng kho dữ liệu, chạy pipeline một lần trên dữ liệu mẫu, rồi khởi
động dashboard. Dịch vụ `pipeline` chạy một lần rồi thoát — đó là hành vi đúng.

## Kiểm tra hệ thống đã chạy đúng

```bash
# 1. Kho dữ liệu phải "healthy"
docker compose ps

# 2. Báo cáo chất lượng của lần chạy vừa rồi
ls -1 reports/ && cat reports/chat-luong-*.md | head -30

# 3. Bảng đích đã có dữ liệu (dữ liệu mẫu: 400 bản ghi khách thô → ~330 khách hợp nhất)
docker compose exec -T warehouse psql -U "$DWH_USER" -d "$DWH_NAME" \
  -c "SELECT COUNT(*) FROM dim_customer;" \
  -c "SELECT ma_quy_tac, COUNT(*) FROM reject_rows GROUP BY 1 ORDER BY 2 DESC;"

# 4. Dashboard mở được: http://localhost:8501
```

## Phép thử NẠP LẠI ĐƯỢC — mục kiểm chứng quan trọng nhất

```bash
docker compose exec -T warehouse psql -U "$DWH_USER" -d "$DWH_NAME" \
  -tAc "SELECT COUNT(*) FROM dim_customer;"          # ghi lại số này

docker compose run --rm pipeline python -m src.pipeline   # chạy lại lần hai

docker compose exec -T warehouse psql -U "$DWH_USER" -d "$DWH_NAME" \
  -tAc "SELECT COUNT(*) FROM dim_customer;"          # PHẢI ra đúng số trên
```

## Chạy bộ kiểm thử

```bash
pip install -r requirements.txt
pytest -q
```

15 test cho `src/transform/rules.py` và `src/quality/checks.py` — đều là hàm thuần,
chạy trong vài trăm mili giây, không cần cơ sở dữ liệu.

## Cấu trúc thư mục

```
src/extract/     Đọc dữ liệu thô. KHÔNG biến đổi.
src/transform/   Làm sạch và biến đổi. KHÔNG đọc ghi CSDL.
src/quality/     Đo ba chiều chất lượng. Mỗi phép đo trả về MỘT SỐ.
src/load/        Ghi vào đích. KHÔNG biến đổi.
src/pipeline.py  Điều phối bốn pha, sinh báo cáo, trả mã thoát theo kết quả.
sql/             Tạo bảng staging, reject, dimension, fact
dashboard/       Streamlit — hai biểu đồ, xử lý đủ ba trạng thái
data/mau/        Dữ liệu mẫu nhỏ để người khác chạy thử được
reports/          Báo cáo chất lượng sinh tự động mỗi lần chạy
```

**Phép thử ranh giới pha** (chạy được ngay, cả hai phải trả về rỗng):

```bash
grep -n "connect\|cursor\|to_sql\|read_sql" src/transform/*.py src/quality/*.py
grep -n "\.replace(\|\.strip()\|astype" src/load/*.py
```

## Mã thoát của pipeline

| Mã | Ý nghĩa | Công cụ hẹn lịch nên làm gì |
|---|---|---|
| 0 | Thành công | Không làm gì |
| 1 | Lỗi kỹ thuật (không đọc được nguồn, mất kết nối) | Cảnh báo cho người vận hành, thử lại sau |
| 2 | Dừng vì chất lượng dữ liệu không đạt ngưỡng | Cảnh báo cho người phụ trách dữ liệu, **không** tự thử lại |

## Tài liệu

| Tệp | Nội dung |
|---|---|
| `docs/test-plan.md` | Test Plan rút gọn 4 mục |
| `docs/test-report.md` | 18 test case đã chạy, đủ 8 cột |
| `docs/bug-log.md` | 3 bug đã sửa, mỗi bug 7 dòng |
| `docs/deployment-guide.md` | Deployment Guide 5 mục |
| `docs/checklist-bao-mat.md` | Checklist bảo mật 7 mục đã tự đánh giá |
| `docs/pipeline-doc.md` | Pipeline doc 6 mục — thành phần 6 của BT3 cho track DA |
| `docs/data-dictionary.md` | Data dictionary — mọi cột, mỗi cột 6 trường |
| `docs/thay-doi-thiet-ke.md` | Nhật ký thay đổi so với thiết kế ban đầu |
| `docs/ai-disclosure.md` | Bảng khai báo sử dụng công cụ AI |

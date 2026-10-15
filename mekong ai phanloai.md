# Prototype mẫu — Track AI: Phân loại tự động mô tả sự cố

Repo **mẫu** của học phần Chuyên đề tốt nghiệp 1 (Khoa CNTT, Trường Đại học Văn Lang).
Ví dụ tham chiếu cho mức độ và cấu trúc mà Bài tập 2 và Bài tập 3 yêu cầu —
**không phải bài mẫu để sinh viên sao chép**. Luồng nghiệp vụ: L10 — gợi ý nhóm
lỗi khi nhân viên tiếp nhận nhập mô tả sự cố (case study Smart CRM – Mekong Mobile).

## Hệ thống làm được gì

- Huấn luyện mô hình phân loại mô tả sự cố thành sáu nhóm lỗi: `MAN_HINH`, `PIN`,
  `SAC`, `PHAN_MEM`, `NUOC_VAO`, `KHAC`.
- **Chia dữ liệu TRƯỚC** khi học tham số — mọi bộ biến đổi chỉ `fit` trên tập train.
- So sánh **hai mô hình cơ sở** và **hai mô hình thật**; chọn mô hình trên tập
  validation, đo tập test **đúng một lần**.
- Báo cáo chỉ số **tách theo từng nhãn**, confusion matrix, và **chênh lệch giữa
  các nhóm** (đo thiên lệch).
- Đóng gói mô hình thành dịch vụ dự đoán FastAPI, trả về top 3 kèm độ tin cậy và
  cờ **cần người xem lại**.

## Ba nguyên tắc được kiểm chứng bằng test

| Nguyên tắc | Test kiểm nó |
|---|---|
| Không rò rỉ dữ liệu | `tests/test_no_leakage.py` — thí nghiệm có đối chứng hai chiều (RR-03) **và** kiểm trên chính tệp mô hình sẽ triển khai (RR-06). Xem BUG-05, BUG-06 trong `docs/bug-log.md` để biết vì sao phải làm cả hai |
| Tái lập được | `tests/test_tai_lap.py` — hai lần huấn luyện cùng hạt giống cho `metrics.json` giống hệt |
| Tập test dùng một lần | `src/models/train.py` — hàm `do_lan_cuoi` có bộ đếm, gọi lần thứ hai sẽ ném lỗi |

## Yêu cầu môi trường

- Docker Engine 24 trở lên và Docker Compose v2, **hoặc** Python 3.11 trở lên.
- Cổng 8080 còn trống (đổi bằng `MODEL_PORT`).

## Chạy — hai lệnh

```bash
cp .env.example .env
docker compose up -d --build
```

Ảnh Docker **tự huấn luyện mô hình khi dựng** (`RUN python -m src.models.train`),
nên gói đóng gói tự đủ: người khác clone về là chạy được, không cần tệp mô hình.

## Chạy không dùng Docker

```bash
pip install -r requirements.txt
python -m src.models.train --config configs/train.yaml   # sinh artifacts + reports
uvicorn src.serve.app:app --port 8080
```

## Kiểm tra hệ thống đã chạy đúng

```bash
# 1. Dịch vụ mô hình phải "healthy"
docker compose ps
curl -s http://localhost:8080/health
#    Kết quả mong đợi: {"status":"ok","phien_ban_mo_hinh":"...","so_nhan":6}

# 2. Dự đoán với một mô tả THẬT
curl -s -X POST http://localhost:8080/du-doan \
  -H 'Content-Type: application/json' \
  -d '{"mo_ta":"May sac khong vao dien, cam sac khong len bao gi"}'
#    Kết quả mong đợi: JSON có top_3, do_tin_cay, can_nguoi_xem_lai, phien_ban_mo_hinh

# 3. Dữ liệu vào không hợp lệ phải bị TỪ CHỐI, không làm sập dịch vụ
curl -s -o /dev/null -w "%{http_code}\n" -X POST http://localhost:8080/du-doan \
  -H 'Content-Type: application/json' -d '{"mo_ta":"abc"}'
#    Kết quả mong đợi: 422

# 4. Đọc báo cáo chỉ số
cat reports/metrics.json | python -m json.tool | head -40
```

## Phép thử TÁI LẬP ĐƯỢC — mục kiểm chứng quan trọng nhất

```bash
python -m src.models.train --config configs/train.yaml
cp reports/metrics.json /tmp/lan1.json

python -m src.models.train --config configs/train.yaml
diff /tmp/lan1.json reports/metrics.json && echo "TAI LAP DUOC: hai lan giong het"
```

## Chạy bộ kiểm thử

```bash
pip install -r requirements.txt
pytest -q
```

## Cấu trúc thư mục

```
notebooks/01_eda.md   KHAM PHA du lieu — KHONG phai san pham nop
src/data/load.py      Nap du lieu tho
src/data/split.py     CHIA du lieu — lam TRUOC moi bien doi hoc tham so
src/features/text.py  Lam sach van ban (GIU dau tieng Viet), vector hoa
src/models/baseline.py  Hai mo hinh co so BAT BUOC de so sanh
src/models/train.py     Huan luyen, chon mo hinh, do lan cuoi
src/evaluate.py         Confusion matrix, chi so theo nhan, do thien lech
src/serve/app.py        Dich vu du doan FastAPI
configs/train.yaml      Sieu tham so va HAT GIONG ngau nhien
artifacts/              Mo hinh da luu — KHONG commit (xem .gitignore)
reports/metrics.json    Ket qua do, sinh tu dong
```

**Notebook không phải sản phẩm nộp.** `notebooks/01_eda.md` là ghi chép khám phá
dữ liệu. Mục kiểm chứng của Bài tập 2 là `python -m src.models.train` chạy được
bằng một lệnh và sinh ra `reports/metrics.json`.

## Tài liệu

| Tệp | Nội dung |
|---|---|
| `docs/test-plan.md` | Test Plan rút gọn 4 mục |
| `docs/test-report.md` | 16 test case đã chạy, đủ 8 cột |
| `docs/bug-log.md` | 3 bug đã sửa, mỗi bug 7 dòng |
| `docs/deployment-guide.md` | Deployment Guide 5 mục |
| `docs/checklist-bao-mat.md` | Checklist bảo mật 7 mục đã tự đánh giá |
| `docs/model-card.md` | Model card 10 mục — thành phần 6 của BT3 cho track AI |
| `docs/thay-doi-thiet-ke.md` | Nhật ký thay đổi so với thiết kế ban đầu |
| `docs/ai-disclosure.md` | Bảng khai báo sử dụng công cụ AI |

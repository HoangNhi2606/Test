# Skill Matrix — Plant Floor

Web app theo dõi ma trận kỹ năng công nhân, cân bằng chuyền (line balance), kế hoạch sản xuất và phân tích thiếu hụt kỹ năng.

- **Frontend**: `static/index.html` (React chạy trực tiếp trên trình duyệt, giữ nguyên bản gốc)
- **Backend**: `backend/main.py` (FastAPI + SQLite), cung cấp toàn bộ `/api/...` mà giao diện gọi
- Backend phục vụ luôn trang web tại `/`, nên chỉ cần chạy **một** server.

> ⚠️ **GitHub Pages không chạy được app này** vì cần backend. Hãy chạy trên máy của bạn hoặc trên Render/Docker (bên dưới).

## 1. Chạy trên máy (Windows/Mac/Linux)

```bash
python -m venv .venv
# Windows: .venv\Scripts\activate      Mac/Linux: source .venv/bin/activate
pip install -r requirements.txt

# Đặt mật khẩu admin (bỏ qua nếu chỉ dùng trên máy mình thì không cần mật khẩu)
# Windows (PowerShell):  $env:ADMIN_TOKEN="matkhau-cua-ban"
# Mac/Linux:             export ADMIN_TOKEN=matkhau-cua-ban

uvicorn backend.main:app --port 8000
```

Mở http://localhost:8000. Dữ liệu lưu trong `data/skillmatrix.db`.

## 2. Đưa lên GitHub

```bash
git init
git add .
git commit -m "Skill Matrix"
git branch -M main
git remote add origin https://github.com/<ten-ban>/skillmatrix.git
git push -u origin main
```

Không có Git? Vào github.com → New repository → "uploading an existing file" → kéo thả toàn bộ nội dung thư mục (đã giải nén) vào.

## 3. Chạy online trên Render

1. Render → **New → Blueprint** → chọn repo này (`render.yaml` đã cấu hình sẵn).
2. Nhập `ADMIN_TOKEN` = mật khẩu admin khi được hỏi.
3. Xong, mở link `https://<ten>.onrender.com`.

**Lưu ý dữ liệu:** gói Free của Render dùng ổ đĩa tạm, nên dữ liệu **mất khi server khởi động lại hoặc deploy lại**. Hãy bấm **Manage → Download backup** thường xuyên, hoặc dùng gói trả phí kèm Disk (đặt `DB_PATH=/var/data/skillmatrix.db`).

Docker:
```bash
docker build -t skillmatrix .
docker run -p 8000:8000 -e ADMIN_TOKEN=matkhau -v $(pwd)/data:/app/data skillmatrix
```

## Quyền admin

Ai cũng **xem** được. Mọi thao tác **sửa/thêm/xóa/import** và **tải backup** cần bấm **Admin sign-in** (góc trên phải) rồi nhập đúng `ADMIN_TOKEN`. Nếu không đặt `ADMIN_TOKEN`, server chạy chế độ mở (ai cũng sửa được), chỉ nên dùng trên máy cá nhân.

## Nhập toàn bộ dữ liệu bằng file Excel (khuyên dùng)

File `sample_data/Skill_Matrix_Input_Template.xlsx` gom mọi dữ liệu đầu vào: **Employees, Skills, Levels (mức 0–4 từng công nhân × kỹ năng), Cycle Times, Stations, Parts, Monthly Plan**. Sheet "Hướng dẫn" giải thích chi tiết.

1. Mở file Template, điền vào các ô **màu vàng** (ô xám là tự động), lưu bằng Excel.
2. Trong app: **Manage → Backup & restore → "Restore from backup file…"** → chọn file `.xlsx`.
3. App báo số dòng đã thêm/cập nhật. Nếu có lỗi, app ghi rõ **sheet + số dòng** và **không lưu gì cả**.

`Skill_Matrix_Input_Example.xlsx` là bản đã điền dữ liệu mẫu để thử. Nạp lại nhiều lần được (ID trùng thì cập nhật, không nhân đôi). Chạy `python tools/make_input_workbook.py` để tạo lại hai file.

## Nhập từng bảng bằng CSV (cách cũ)

Thư mục `sample_data/` có CSV mẫu. Thứ tự nhập:

1. `employees.csv`: tab Skill Matrix → Import CSV
2. `skills.csv`: tab Manage → Import CSV
3. `parts.csv`: tab Planning → Import parts CSV
4. `plan_2026-10.csv`: chọn tháng 2026-10 → Import month's plan CSV

CSV nhận dấu `,` hoặc `;`, ngày dạng `YYYY-MM-DD` hoặc `DD/MM/YYYY`, có hoặc không có BOM. Nếu có dòng lỗi thì **không lưu gì cả** và báo rõ dòng nào sai.

## Quy tắc tính

- Cấp độ 0–4; **"đạt" = L3 trở lên**.
- **Coverage** của một line = số ô đạt / (số công nhân × số kỹ năng áp dụng). Kỹ năng có *Group* trùng tên một department chỉ tính cho department đó; kỹ năng không có Group tính cho mọi line.
- **Bottleneck alert**: kỹ năng có ≤ 1 người đạt L3+ trong line; *critical* nếu kỹ năng đánh dấu Critical, ngược lại *warning*.
- **Line balance**: trạm có CT thực tế (nhập ở ô ma trận của người được gán) lớn hơn Target CT là "over target"; trạm có CT lớn nhất là bottleneck của line.
- **Planning**: Target hours = Quantity × CT Target / 3600; Expected hours = Quantity × CT Actual / 3600.

## Kiểm thử

```bash
pip install -r requirements-dev.txt
pytest
```


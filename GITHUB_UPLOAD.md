# Đưa phần thực nghiệm lên GitHub để leader xem

## 1. Chuẩn bị trong thư mục dự án

Giải nén bộ bàn giao cạnh bốn notebook, giữ tên file `.gitignore` có dấu chấm ở đầu. Nếu dự án đã có README, requirements hoặc `.gitignore`, ghép nội dung phù hợp vào tệp hiện có thay vì ghi đè phần của người khác.

Các tệp cần đưa lên gồm README, requirements, bốn notebook, bảng chỉ số tổng hợp và hình trong `reports/`. Dữ liệu Excel, `data/`, `models/` và `.venv/` được chia sẻ hoặc tạo lại riêng. File Excel đã dùng khoảng 233 MiB; GitHub chặn tệp lớn hơn 100 MiB trong Git thông thường ([tài liệu](https://docs.github.com/en/repositories/working-with-files/managing-large-files/about-large-files-on-github)).

Trong README, kiểm tra hai bảng số liệu với `reports/final_cf/test_comparison.csv` trên máy bạn. Các số có ký hiệu % trong README đã nhân 100; CSV gốc lưu thang 0–1.

## 2. Chọn đúng repository trước khi push

- Link `recommenders-team/recommenders` trong trao đổi là tài liệu tham khảo.
- Dùng URL repository của nhóm hoặc repository do bạn tạo để leader xem.
- Nếu repository nhóm đã có code, ưu tiên nhánh riêng và Pull Request theo cách làm của nhóm.
- Với repository riêng, chọn quyền truy cập theo quy định nhóm và bảo đảm leader có quyền xem.

Để viết đúng lệnh remote/push, cần URL repository đích. Đừng chạy nguyên các URL mẫu bên dưới.

## 3. Nếu nhóm đã có repository

Thực hiện ở một thư mục checkout riêng để giữ thư mục dự án đang chạy:

```powershell
git clone "URL_REPOSITORY_CUA_NHOM" RecommenderSystem-review
cd RecommenderSystem-review
git switch -c cuong-quoc/cf-baseline
```

Copy mã nguồn và kết quả tổng hợp vào vị trí mà nhóm quy định. Giữ đường dẫn các tệp tương thích với README. Nếu nhóm đặt phần này trong thư mục con, mở chính thư mục chứa bốn notebook làm workspace VS Code để `Path.cwd()` đúng vị trí.

Ghép `.gitignore` vào quy tắc của repository; sau đó kiểm tra thay đổi và stage các tệp bàn giao. Lệnh dưới đây giả định bốn notebook ở gốc repository:

```powershell
git status
git add README.md requirements.txt .gitignore GITHUB_UPLOAD.md
git add 01_data_preparation.ipynb 02_item_based_cf.ipynb 03_tuning_and_test.ipynb 04_results_and_demo.ipynb
git add reports
git diff --cached --stat
git diff --cached --name-only
```

Danh sách staged nên gồm mã, bảng tổng hợp và ảnh. Nếu một tệp dữ liệu/môi trường đã được theo dõi từ trước, `.gitignore` không tự bỏ theo dõi tệp đó ([tài liệu](https://docs.github.com/en/get-started/git-basics/ignoring-files)); cần xử lý đúng tệp đã theo dõi trước khi commit.

Khi staged đúng phần bàn giao:

```powershell
git commit -m "Add CF baseline, temporal split evaluation and result reports"
git push -u origin cuong-quoc/cf-baseline
```

Mở GitHub, tạo Pull Request từ nhánh này theo quy trình nhóm và gửi link để leader review. Mô tả nên nêu: đầu vào, cách chạy 01→04, bảng kết quả, tăng độ phủ, khoảng tin cậy NDCG@10 chứa 0, phạm vi KG/TKG/LLM còn ở giai đoạn tích hợp.

## 4. Nếu tạo repository mới, trống

Dùng nhánh này của hướng dẫn khi thư mục dự án chưa là Git repository và repository đích vừa tạo chưa có commit. Tạo repository trống trên tài khoản/nhóm phù hợp; sau đó trong thư mục dự án:

```powershell
git init -b main
git add README.md requirements.txt .gitignore GITHUB_UPLOAD.md
git add 01_data_preparation.ipynb 02_item_based_cf.ipynb 03_tuning_and_test.ipynb 04_results_and_demo.ipynb
git add reports
git diff --cached --stat
git diff --cached --name-only
git commit -m "Add CF baseline, evaluation and result reports"
git remote add origin "URL_REPOSITORY_MOI_CUA_BAN"
git push -u origin main
```

Nếu dự án đã là Git repository, xem `git status` và `git remote -v` để dùng cấu hình hiện có. Nếu remote đã có commit, dùng quy trình clone/nhánh riêng ở mục 3; tránh tạo hai lịch sử riêng rồi ghi đè lên nhau.

## 5. Kiểm tra phần leader sẽ xem

Mở trang repository/nhánh đã push:

1. README hiển thị đúng tiếng Việt và các bảng kết quả.
2. Link tới notebook, `test_comparison.csv` và `presentation_notes.md` mở được.
3. Hai ảnh minh họa trong README hiện đầy đủ nhờ thư mục `reports/cf_presentation/figures/` đã commit.
4. Leader biết cách lấy file Excel qua kênh chia sẻ dữ liệu của nhóm để chạy lại.

Tài liệu chính thức: [Đưa code cục bộ lên GitHub](https://docs.github.com/en/migrations/importing-source-code/using-the-command-line-to-import-source-code/adding-locally-hosted-code-to-github).

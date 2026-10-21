# Kết quả hướng 1 — Lọc cộng tác

Người thực hiện: Cường Quốc.

## Thiết kế thí nghiệm

- Chia theo thời gian toàn cục; giữ nguyên mốc đã lưu ở notebook 01.
- Tương tác tích cực: rating >= 4; mỗi cặp user–item tích cực đóng góp một lần vào mô hình.
- Thử N = [10, 20, 50, 100] trên validation; chọn theo NDCG@10 toàn bộ người dùng, rồi Recall@10, rồi N nhỏ hơn.
- Cấu hình chọn: N = 10. Chốt trước khi đọc test và huấn luyện lại trên train + validation.
- Lịch sử cuối gồm 624,827 review và 102,963 sản phẩm.
- Loại sản phẩm từng được người dùng đánh giá trong lịch sử, ở bất kỳ mức sao nào.
- Giữ sản phẩm chưa xuất hiện trong lịch sử ở ground truth; mô hình không gợi ý được thì tính miss.
- Precision@K chia cho K; Recall@K chia cho số sản phẩm liên quan; NDCG@K dùng relevance nhị phân. Trung bình theo người dùng.
- Nhóm toàn bộ gồm người có ít nhất một sản phẩm tích cực mới trong test; nhóm có lịch sử tích cực là tập con đã có ít nhất một cặp tích cực trong train + validation.
- Lịch sử và mô hình được cố định trong suốt giai đoạn test.

## Kết quả test

| model | k | group | users | Precision@K | Recall@K | NDCG@K | CatalogCoverage@K |
| --- | --- | --- | --- | --- | --- | --- | --- |
| Most Popular | 10 | Toàn bộ | 43656 | 0.00163551 | 0.01621770 | 0.00732834 | 0.0107% |
| Most Popular | 10 | Có lịch sử tích cực | 2533 | 0.00078958 | 0.00769838 | 0.00311049 | 0.0107% |
| Most Popular | 20 | Toàn bộ | 43656 | 0.00140072 | 0.02777014 | 0.01028960 | 0.0214% |
| Most Popular | 20 | Có lịch sử tích cực | 2533 | 0.00076984 | 0.01500197 | 0.00499570 | 0.0214% |
| ItemCF + Popular fallback | 10 | Toàn bộ | 43656 | 0.00164468 | 0.01629787 | 0.00740466 | 5.8720% |
| ItemCF + Popular fallback | 10 | Có lịch sử tích cực | 2533 | 0.00094749 | 0.00908014 | 0.00442593 | 5.8720% |
| ItemCF + Popular fallback | 20 | Toàn bộ | 43656 | 0.00140531 | 0.02785031 | 0.01036184 | 6.3285% |
| ItemCF + Popular fallback | 20 | Có lịch sử tích cực | 2533 | 0.00084880 | 0.01638373 | 0.00624065 | 6.3285% |

## Chênh lệch và khả năng áp dụng

- Chênh lệch NDCG@10 (CF trừ Most Popular), toàn bộ người dùng: 0.00007632.
- Bootstrap theo người dùng 1,000 lần, seed 42; khoảng percentile 95%: [-0.00001416, 0.00017774]. Khoảng tin cậy chứa 0: chưa có bằng chứng rõ về cải thiện NDCG@10 trong phép đánh giá này.
- Độ phủ danh mục ở K=10: Most Popular 0.0107%, CF 5.8720%. Đây là tỷ lệ sản phẩm xuất hiện trong ít nhất một danh sách trên danh mục lịch sử của giai đoạn test.
- 94.20% người dùng được đánh giá không có lịch sử tích cực; 95.94% danh sách CF ở K=10 hoàn toàn dùng Most Popular.
- 35.71% cặp tích cực mới trong test có sản phẩm chưa xuất hiện trong lịch sử; CF hiện tại không có cách đưa các sản phẩm đó vào danh sách.
- Khoảng tin cậy chỉ mô tả lấy mẫu người dùng test hiện tại, không bao gồm bất định khi huấn luyện lại hoặc thay đổi thời gian.
- Độ phủ cao hơn là dấu hiệu danh sách đa dạng hơn theo sản phẩm; chưa chứng minh công bằng theo cửa hàng hoặc cải thiện doanh thu. Dữ liệu đang dùng là review và thời điểm review.

## Bàn giao và bước tiếp theo

- `selected_config.json`: tham số và quy tắc chọn trên validation.
- `validation_grid.csv`: toàn bộ cấu hình đã thử.
- `test_comparison.csv`: bảng kết quả cuối của hai mô hình, Top-10/Top-20 và hai nhóm người dùng.
- `test_per_user.csv.gz`: chỉ số từng người dùng để đối chiếu.
- `test_ndcg10_paired_bootstrap.csv`: chênh lệch NDCG@10 và khoảng tin cậy.
- `test_diagnostics.json`: quy mô, mốc thời gian, sản phẩm chưa thấy và mã kiểm tra dữ liệu.
- `demo_top_k.csv`: ví dụ khuyến nghị có ghi nguồn CF hoặc Most Popular.
- `models/item_cf_final/`: ma trận, ánh xạ, độ phổ biến, lịch sử đã đánh giá và cấu hình. Trong notebook, chạy ô khai báo lớp trước khi gọi `ItemCFModel.load(MODEL_DIR)`.

Phần CF tạo chuẩn so sánh cho các hướng đồ thị tri thức và LLM. Các hướng sau cần giữ cùng định nghĩa relevance, mốc chia thời gian, nhóm đánh giá và chính sách tập ứng viên, hoặc báo cáo rõ phần thay đổi. Sản phẩm mới và người dùng ít lịch sử là những trường hợp cần nghiên cứu thêm bằng thông tin sản phẩm.

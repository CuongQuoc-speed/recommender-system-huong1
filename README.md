# Hướng 1 — Temporal Recommendation Baseline

**Người thực hiện: Cường Quốc** · **Trạng thái: Đã hoàn thành thực nghiệm**  
Kết quả chạy ngày 24/10/2026 · Tổng hợp ngày 25/10/2026.

> **Kết luận chính:** Thời gian đã làm đổi thứ hạng ở nhóm có đủ lịch sử, nhưng chưa có bằng chứng rõ về cải thiện NDCG@10. Mô hình và ứng viên có điểm được chuẩn bị để Hướng 2 tiếp tục bổ sung tri thức sản phẩm.

## 1. Bài toán và mục tiêu

Mục tiêu chung là giúp cửa hàng tiếp cận khách hàng và tăng cơ hội bán hàng. Hướng 1 xây dựng baseline và kiểm chứng tác động của thời gian lên gợi ý sản phẩm.

**Dữ liệu:** All Beauty, **694.252 review**; khoảng **93,19% người dùng chỉ có một review**. Lịch sử quá ít làm hệ thống khó cá nhân hóa.

**Câu hỏi thực nghiệm:** Ưu tiên review gần đây có làm đổi và cải thiện gợi ý không? Tác động xuất hiện ở nhóm người dùng nào?

## 2. Phương pháp đã thực hiện

| Mô hình | Cách tạo gợi ý |
| --- | --- |
| **Most Popular** | Xếp hạng theo số người dùng đánh giá tích cực của sản phẩm |
| **ItemCF + Popular fallback** | Gợi ý từ các sản phẩm có lịch sử tương tác tương tự; thiếu ứng viên thì bổ sung Most Popular |
| **Time-decay ItemCF + Popular fallback** | Giữ cosine và cơ chế fallback của ItemCF; giảm đóng góp của review cũ, ưu tiên review gần đây |

Cấu hình chọn: **N = 10 lân cận, h = 180 ngày**. Trọng số review giảm một nửa sau mỗi 180 ngày; review tích cực có **rating ≥ 4**.

Chia dữ liệu theo thời gian **70% / 20% / 10%**; chọn tham số bằng validation rồi so sánh trên cùng test với lịch sử cố định. Timestamp là thời điểm review.

## 3. Kết quả chính

**Chất lượng gợi ý trên 43.656 người dùng test đủ điều kiện:**

| Mô hình | NDCG@10 |
| --- | ---: |
| Most Popular | 0,00732834 |
| ItemCF + fallback | 0,00740466 |
| Time-decay ItemCF + fallback | 0,00740848 |

NDCG@10 càng cao thì chất lượng thứ hạng càng tốt. Mức tăng của temporal rất nhỏ; **khoảng tin cậy 95% chứa 0 nên chưa có bằng chứng rõ về cải thiện**.

**Tác động của thời gian theo số sản phẩm tích cực khác nhau trong lịch sử huấn luyện:**

| Lịch sử tích cực | Số người dùng | Danh sách Top-10 thay đổi so với ItemCF |
| --- | ---: | ---: |
| 0 sản phẩm | 41.123 | 0 |
| 1 sản phẩm | 1.850 | 0 |
| Từ 2 sản phẩm | 683 | **379 — tương đương 55,49%** |

- **Thời gian đã tác động trực tiếp đến thứ hạng:** 55,49% người có từ 2 sản phẩm tích cực đổi Top-10.
- Nhóm 0 sản phẩm tích cực dùng Most Popular; nhóm 1 sản phẩm có cùng hệ số nhân cho mọi điểm CF nên thứ hạng giữ nguyên.
- Chỉ **1,56%** người dùng test có từ 2 sản phẩm tích cực, nên tác động trên toàn bộ còn nhỏ.

Kết quả này kiểm chứng tác động của thời gian trên dữ liệu thưa; chưa chứng minh tăng doanh thu hoặc lượt mua.

## 4. Khoảng trống còn lại và định hướng phát triển — Hướng 2

- **Khoảng trống của Hướng 1:** ItemCF có trọng số thời gian vẫn phụ thuộc lịch sử tương tác, chưa khai thác thuộc tính và quan hệ sản phẩm. **95,94% danh sách Top-10 vẫn hoàn toàn dùng Most Popular**, cho thấy giới hạn cá nhân hóa trên dữ liệu thưa.
- **Định hướng sang Hướng 2:** Xây dựng **đồ thị tri thức từ metadata**, khai thác thương hiệu, danh mục, thuộc tính và quan hệ sản phẩm để bổ sung thông tin và nguồn ứng viên. So sánh với baseline Hướng 1 để kiểm chứng hiệu quả.

Hướng 1 dùng thời gian ở mức trọng số lịch sử; phần biểu diễn quan hệ bằng đồ thị tri thức được phát triển tiếp ở Hướng 2.

## 5. Đầu ra và bàn giao cho Member 2

| Đầu ra | Kết quả |
| --- | --- |
| Quy trình thực nghiệm | Đủ 4 notebook; giữ baseline và bổ sung thí nghiệm thời gian trong notebook 03 |
| Mô hình | ItemCF baseline và Time-decay ItemCF; đã kiểm tra lưu và nạp lại |
| Ứng viên và điểm | Tối đa 100 sản phẩm/người dùng cho mỗi biến thể; **8.731.200 dòng**, có điểm, thứ hạng và nguồn CF/fallback |
| Báo cáo | So sánh mô hình, phân tích dữ liệu thưa, thay đổi thứ hạng, khoảng tin cậy và biểu đồ |

**Gói đã tạo:** `models/member2_handoff.zip`; đã kiểm tra nạp lại độc lập thành công. Gói cần gửi riêng cho Member 2. Ứng viên được xuất cho nhóm người dùng test đủ điều kiện; Hướng 2 sử dụng thêm dữ liệu và metadata chung.

**Phạm vi:** baseline, thí nghiệm thời gian, phân tích dữ liệu thưa và đầu ra bàn giao; không phụ trách chính thuật toán giảm popularity bias.

**Đối chiếu:** [Báo cáo kết quả](reports/temporal_cf/temporal_summary.md) · [Bảng so sánh](reports/temporal_cf/test_comparison.csv) · [Biểu đồ](reports/temporal_cf/temporal_comparison.png) · [Gói bàn giao](reports/temporal_cf/handoff_summary.json).

**Notebook:** [01 — Dữ liệu](01_data_preparation.ipynb) · [02 — ItemCF](02_item_based_cf.ipynb) · [03 — Đánh giá và temporal](03_tuning_and_test.ipynb) · [04 — Biểu đồ và demo](04_results_and_demo.ipynb).

**Tham khảo:** [Recommenders](https://github.com/recommenders-team/recommenders) · [Koren (2009) — Collaborative Filtering with Temporal Dynamics](https://doi.org/10.1145/1557019.1557072).

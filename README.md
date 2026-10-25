# Hướng 1 — Temporal Recommendation Baseline

**Người thực hiện: Cường Quốc** · **Đã hoàn thành thực nghiệm**  
Lần chạy: 24/10/2026 · Tổng hợp: 25/10/2026.

> **Kết luận chính:** Đã xây dựng baseline và kiểm chứng tác động trực tiếp của thời gian lên ItemCF. Temporal làm đổi Top-10 ở **55,49% nhóm có từ 2 sản phẩm tích cực**, nhưng mức tăng NDCG@10 còn nhỏ và chưa có bằng chứng rõ về cải thiện. Mô hình, ứng viên và điểm được chuẩn bị cho Member 2 phát triển Hướng 2.

## 1. Bài toán, dữ liệu và mục tiêu

Mục tiêu chung là giúp cửa hàng tiếp cận khách hàng và tăng cơ hội bán hàng. Phần việc của Hướng 1 là tạo **baseline gợi ý có thể đối chiếu** và kiểm tra việc ưu tiên tương tác gần đây có giúp cải thiện gợi ý khi dữ liệu thưa hay không.

| Dữ liệu All Beauty đã làm sạch | Số lượng |
| --- | ---: |
| Review | 694.252 |
| Người dùng | 631.986 |
| Sản phẩm có tương tác | 112.565 |
| Người dùng chỉ có một review | 93,19% |

Lịch sử quá ít làm hệ thống khó cá nhân hóa. Timestamp là thời điểm **review**; rating **≥ 4** được quy ước là tương tác tích cực.

## 2. Phương pháp và cách đánh giá

| Mô hình | Nội dung thực hiện |
| --- | --- |
| **Most Popular** | Xếp hạng theo số người dùng đánh giá tích cực của sản phẩm |
| **ItemCF + Popular fallback** | Tính cosine từ các cặp user–item tích cực; gợi ý bằng tổng độ tương tự. Thiếu ứng viên thì bổ sung Most Popular |
| **Time-decay ItemCF + Popular fallback** | Giữ cosine và fallback của ItemCF; nhân đóng góp của mỗi sản phẩm lịch sử với trọng số giảm theo tuổi review |

Trọng số temporal là `w = 2 ** (-age_days / h)`: review gần đây đóng góp nhiều hơn. Với cặp user–item lặp, lấy timestamp review tích cực mới nhất trong lịch sử.

- **Chia dữ liệu:** theo thời gian, khoảng 70% train / 20% validation / 10% test.
- **Chọn tham số bằng validation:** thử N = 10, 20, 50, 100; h = 90, 180, 365 ngày. Kết quả chọn **N = 10, h = 180 ngày**, theo NDCG@10 toàn bộ người dùng đủ điều kiện.
- **Đánh giá test:** học lại trên train + validation rồi cố định lịch sử; cùng người dùng, ground truth và catalog cho ba mô hình. Loại sản phẩm đã đánh giá; sản phẩm ngoài catalog lịch sử vẫn được giữ ở ground truth và tính miss nếu không gợi ý được.
- **Độ dài danh sách:** K = 10 và 20; N là số lân cận, K là số sản phẩm trả về. Test không dùng để chọn tham số.

## 3. Kết quả và ý nghĩa

**So sánh ba mô hình trên 43.656 người dùng test đủ điều kiện.** Đây là kết quả test sau khi chốt tham số; lưới validation dùng để chọn N và h được lưu riêng.

| Mô hình | K | P@K (%) | R@K (%) | NDCG@K | Cov@K (%) | CF (%) | Fallback (%) |
| --- | --- | --- | --- | --- | --- | --- | --- |
| Most Popular | 10 | 0,1636 | 1,6218 | 0,00732834 | 0,0107 | 0,0000 | 100,0000 |
| ItemCF | 10 | 0,1645 | 1,6298 | 0,00740466 | 5,8720 | 2,6986 | 95,9387 |
| Time-decay ItemCF | 10 | 0,1645 | 1,6298 | 0,00740848 | 5,8992 | 2,6986 | 95,9387 |
| Most Popular | 20 | 0,1401 | 2,7770 | 0,01028960 | 0,0214 | 0,0000 | 100,0000 |
| ItemCF | 20 | 0,1405 | 2,7850 | 0,01036184 | 6,3285 | 1,7439 | 95,9387 |
| Time-decay ItemCF | 20 | 0,1406 | 2,7862 | 0,01036878 | 6,3537 | 1,7439 | 95,9387 |

**Ý nghĩa các chỉ số:**

- **P@K — Precision:** mức chính xác của danh sách Top-K; **R@K — Recall:** mức thu hồi sản phẩm liên quan.
- **NDCG@K:** chất lượng thứ hạng; **Cov@K — Catalog coverage:** tỷ lệ sản phẩm trong catalog xuất hiện trong các danh sách.
- **CF:** tỷ lệ vị trí do CF tạo; **Fallback:** tỷ lệ danh sách hoàn toàn lấy từ Most Popular. P, R, Cov, CF và Fallback hiển thị theo **%**; NDCG giữ thang **0–1**.

**Nhận xét từ kết quả:**

- Ở K = 10, NDCG tăng từ **0,00740466 → 0,00740848**; Precision và Recall không đổi. Temporal có tác động đến thứ hạng, nhưng mức tăng chất lượng còn rất nhỏ.
- Độ phủ Top-10 tăng từ **5,8720% → 5,8992%**. Tỷ lệ vị trí CF không đổi; khoảng **95,94% danh sách vẫn hoàn toàn fallback** ở cả hai biến thể ItemCF.
- Paired bootstrap theo người dùng, 1.000 lần, seed 42: chênh lệch NDCG@10 temporal trừ ItemCF có **CI95% [0,00000000; 0,00001527]**. Khoảng này chứa 0, nên **chưa có bằng chứng rõ temporal cải thiện NDCG@10**.

**Phân tích theo số sản phẩm tích cực khác nhau trong lịch sử train + validation:**

| Lịch sử tích cực | Người dùng | NDCG@10 ItemCF | NDCG@10 temporal | Top-10 thay đổi | Tỷ lệ (%) |
| --- | --- | --- | --- | --- | --- |
| 0 sản phẩm | 41.123 | 0,00758814 | 0,00758814 | 0 | 0,00 |
| 1 sản phẩm | 1.850 | 0,00483546 | 0,00483546 | 0 | 0,00 |
| Từ 2 sản phẩm | 683 | 0,00331668 | 0,00356070 | 379 | 55,49 |

- **0 sản phẩm tích cực:** dùng Most Popular; **1 sản phẩm:** mọi điểm CF cùng nhân một hệ số dương nên thứ hạng giữ nguyên.
- **Từ 2 sản phẩm:** trọng số thời gian có thể đổi đóng góp tương đối. Thực tế **379/683 người đổi Top-10 (55,49%)**, xác nhận yếu tố thời gian đã tác động trực tiếp đến thuật toán.
- Nhóm từ 2 sản phẩm chỉ chiếm **1,56%** người dùng test đủ điều kiện; đây là giới hạn lớn của cá nhân hóa trên bộ dữ liệu này.

Kết quả offline từ review chưa chứng minh tăng doanh thu hoặc lượt mua. Bảng số liệu đầy đủ và biểu đồ là đầu ra của lần chạy thực nghiệm.

## 4. Khoảng trống còn lại và định hướng sang Hướng 2

- **Khoảng trống của Hướng 1:** ItemCF có trọng số thời gian vẫn phụ thuộc lịch sử tương tác, chưa khai thác thuộc tính và quan hệ sản phẩm. Dữ liệu thưa khiến mô hình còn phụ thuộc nhiều vào Most Popular; thay trọng số thời gian chưa bổ sung được tri thức sản phẩm để bù cho thiếu tương tác.
- **Định hướng phát triển — Hướng 2:** Xây dựng **đồ thị tri thức từ metadata**, khai thác thương hiệu, danh mục, thuộc tính và quan hệ sản phẩm để bổ sung thông tin và nguồn ứng viên. Kết hợp với baseline, ứng viên và điểm của Hướng 1 để kiểm chứng hiệu quả trên cùng quy trình đánh giá.

Hướng 1 dùng thời gian ở mức trọng số lịch sử, chưa biểu diễn quan hệ sản phẩm bằng đồ thị tri thức. Đây là giới hạn còn lại của baseline trong dự án và cơ sở phát triển tiếp sang Hướng 2.

## 5. Đầu ra và bàn giao cho Member 2

| Đầu ra | Kết quả |
| --- | --- |
| Quy trình | Đủ 4 notebook; giữ baseline và bổ sung temporal trong notebook 03 |
| Mô hình | ItemCF baseline và Time-decay ItemCF; đã kiểm tra lưu/nạp lại |
| Ứng viên và điểm | Tối đa 100 sản phẩm/người dùng/biến thể; **8.731.200 dòng**, có điểm, thứ hạng và nguồn CF/fallback |
| Báo cáo | Đủ Precision, Recall, NDCG, độ phủ, tỷ lệ CF/fallback; phân tích nhóm, thay đổi thứ hạng, bootstrap và biểu đồ |

**Gói đã tạo:** `models/member2_handoff.zip`, gồm hai mô hình, mã Python nạp độc lập, ứng viên/điểm/thứ hạng và cấu hình; kiểm tra nạp lại thành công. Gói cần gửi riêng cho Member 2. Phạm vi ứng viên là nhóm người dùng test đủ điều kiện; Hướng 2 sử dụng thêm dữ liệu và metadata chung.

**Phạm vi của Cường Quốc:** baseline, thí nghiệm thời gian, phân tích dữ liệu thưa và đầu ra bàn giao; không phụ trách chính thuật toán giảm popularity bias.

**Bằng chứng thực nghiệm:** [Bảng test](reports/temporal_cf/test_comparison.csv) · [Chọn tham số trên validation](reports/temporal_cf/validation_grid.csv) · [Phân tích nhóm](reports/temporal_cf/personalization_groups.csv) · [Thay đổi thứ hạng](reports/temporal_cf/rank_change_by_group.csv) · [Khoảng tin cậy](reports/temporal_cf/test_ndcg10_paired_bootstrap.csv) · [Báo cáo tổng hợp](reports/temporal_cf/temporal_summary.md) · [Biểu đồ](reports/temporal_cf/temporal_comparison.png) · [Thông tin bàn giao](reports/temporal_cf/handoff_summary.json).

**Notebook:** [01 — Dữ liệu](01_data_preparation.ipynb) · [02 — ItemCF](02_item_based_cf.ipynb) · [03 — Đánh giá và temporal](03_tuning_and_test.ipynb) · [04 — Biểu đồ và demo](04_results_and_demo.ipynb).

**Tham khảo:** [Recommenders](https://github.com/recommenders-team/recommenders) · [Koren (2009) — Collaborative Filtering with Temporal Dynamics](https://doi.org/10.1145/1557019.1557072).

# Hướng 1 — Temporal Recommendation Baseline

**Người thực hiện: Cường Quốc**  
**Cập nhật kết quả sau khi chạy Notebook 03: 24/10/2026**  
Đề tài chung: Hệ thống khuyến nghị thương mại điện tử dựa trên đồ thị tri thức thời gian và mô hình ngôn ngữ.

Phần việc này xây dựng mốc so sánh bằng Most Popular và Item-based Collaborative Filtering (ItemCF), sau đó bổ sung một thí nghiệm để thời gian tác động trực tiếp đến điểm gợi ý. Đầu ra gồm mô hình, kết quả đánh giá, phân tích giới hạn cá nhân hóa và danh sách ứng viên có điểm xếp hạng cho Member 2.

| Hạng mục | Trạng thái của phiên bản này |
| --- | --- |
| Chuẩn bị dữ liệu, Most Popular, ItemCF, đánh giá và biểu đồ baseline | Đã có kết quả từ lần chạy trước; giữ lại làm mốc so sánh |
| Time-decay ItemCF trong notebook 03 | Đã chạy trên dữ liệu All Beauty; chọn N = 10, h = 180 ngày bằng validation |
| Phân tích nhóm có 0, 1 và từ 2 sản phẩm tích cực trong lịch sử | Đã có số liệu, kiểm tra thay đổi thứ hạng và khoảng tin cậy |
| Chuẩn bị gói bàn giao Member 2 | Đã tạo module Python, hai mô hình, 8.731.200 dòng ứng viên/điểm và ZIP; kiểm tra nạp lại thành công |
| Kết quả temporal trên dữ liệu All Beauty đầy đủ | **Đã hoàn tất lần chạy và xuất báo cáo vào `reports/temporal_cf/`** |

Kết quả trong README này được đối chiếu với Notebook 03 đã chạy đủ 19 ô mã, các CSV và JSON của lần chạy All Beauty ngày 24/10/2026. Phép kiểm tra bằng dữ liệu mô phỏng trước đó phục vụ kiểm tra logic; các số liệu dưới đây lấy từ lần chạy dữ liệu thật của dự án. Gói mô hình và CSV ứng viên đầy đủ được tạo trên máy Cường Quốc để chia sẻ riêng cho Member 2.

## 1. Bài toán và phạm vi được phân công

Mục tiêu kinh doanh chung là giúp cửa hàng tiếp cận khách hàng và tăng cơ hội bán hàng. Với dữ liệu review hiện có, nhiệm vụ được cụ thể hóa thành **xếp hạng Top-10/Top-20 sản phẩm phù hợp với lịch sử đánh giá**, đồng thời xuất Top-100 ứng viên để bước tiếp theo có thể xếp hạng lại.

Theo yêu cầu leader, phần việc của Cường Quốc gồm:

- Giữ bốn notebook và các kết quả baseline đã có.
- Bổ sung tối thiểu một thí nghiệm mà yếu tố thời gian tác động trực tiếp đến thuật toán.
- Phân tích khả năng và giới hạn cá nhân hóa khi dữ liệu rất thưa.
- Bàn giao mô hình baseline, danh sách ứng viên và điểm xếp hạng cho Member 2.

Thuật toán giảm popularity bias không thuộc trách nhiệm chính của phần này. Most Popular, tỷ lệ fallback và độ phủ được dùng để so sánh, chẩn đoán mô hình. Việc xây dựng Temporal KG và LLM được thực hiện trong các phần việc tiếp theo của nhóm.

Các chỉ số offline từ review chưa chứng minh mức tăng doanh thu, lượt mua hoặc hiệu quả tiếp cận của cửa hàng. Những kết luận đó cần log kinh doanh và thí nghiệm phù hợp.

## 2. Quá trình và nội dung thực hiện

| Bước | Nội dung | Kết quả |
| --- | --- | --- |
| 1 | Kiểm tra dữ liệu đã làm sạch, xác định trường lõi và ý nghĩa timestamp | Bộ dữ liệu dùng thống nhất cho thực nghiệm |
| 2 | Chuẩn hóa kiểu dữ liệu và chia theo thời gian toàn cục | Train/validation/test dạng Parquet và cấu hình chia tập |
| 3 | Xây Most Popular và ItemCF bằng ma trận thưa | Baseline chạy CPU, hướng tới máy RAM 16 GB |
| 4 | Thử số lân cận N trên validation, chốt cấu hình rồi đánh giá test | Bảng chỉ số, bootstrap, mô hình cuối và ví dụ gợi ý |
| 5 | Tạo biểu đồ và minh họa có tên sản phẩm | Các tệp trình bày baseline trong notebook 04 |
| 6 | Chạy Time-decay ItemCF trong notebook 03 | Chọn h = 180 ngày; đánh giá test, phân tích nhóm lịch sử, tạo biểu đồ và báo cáo temporal |
| 7 | Tạo gói dùng cho Member 2 | Đã xuất mã nạp mô hình, hai biến thể, 8.731.200 dòng ứng viên/điểm/thứ hạng, cấu hình và manifest; kiểm tra nạp lại thành công |

### 2.1. Dữ liệu đã kiểm tra

Nguồn thực nghiệm là `All_Beauty_Cleaned_Merged.xlsx` đã được nhóm làm sạch và ghép metadata. Các số dưới đây là kết quả kiểm tra của lần chạy dữ liệu trước đó.

| Nội dung | Kết quả |
| --- | ---: |
| Bản ghi review | 694.252 |
| Người dùng khác nhau | 631.986 |
| Sản phẩm có tương tác | 112.565 |
| Sản phẩm trong sheet metadata | 112.590 |
| Người dùng chỉ có một review | 588.942, khoảng 93,19% |
| Giá trị thiếu ở bốn trường lõi | 0 |
| Sự kiện trùng user, item và timestamp | 0 |

Mô hình dùng `user_id`, `parent_asin`, `rating`, `timestamp`. Giữ ID dạng chuỗi; rating dùng float32 và timestamp dùng int64, đơn vị mili giây. **Timestamp là thời điểm viết review**, không tự diễn giải thành thời điểm mua hàng.

Trong dữ liệu ghép, `Merged_Data.title` là tiêu đề review; `Merged_Data.product_title` và `Product_Metadata.title` dùng cho tên sản phẩm. Metadata chỉ được ghép để hiển thị ở notebook 04, không tham gia tính điểm CF trong phiên bản này.

### 2.2. Chia tập và quy tắc tương tác

Chia theo thứ tự thời gian toàn cục, tỷ lệ xấp xỉ 70%/20%/10%. Các sự kiện cùng timestamp nằm cùng một phía của mốc chia.

| Tập | Số review | Điều kiện timestamp, mili giây |
| --- | ---: | --- |
| Train | 485.977 | `timestamp <= 1607221537757` |
| Validation | 138.850 | `1607221537757 < timestamp <= 1643201980125` |
| Test | 69.425 | `timestamp > 1643201980125` |

Mốc cuối train: `2020-12-06T02:25:37.757Z`. Mốc cuối validation: `2022-01-26T12:59:40.125Z`. Notebook đọc các mốc từ `data/processed/split_config.json` và kiểm tra tính nhất quán.

- Review có `rating >= 4` được xem là tương tác tích cực.
- Mỗi cặp user–item tích cực đóng góp một lần vào ma trận nhị phân và số người dùng tích cực của sản phẩm.
- Giữ sự kiện lặp khác timestamp trước khi chia tập; không loại toàn bộ cặp user–item trên cả dữ liệu rồi mới chia.
- Khi gợi ý, loại mọi sản phẩm người dùng đã đánh giá trong lịch sử, ở bất kỳ mức sao nào.
- Tại test, dùng lịch sử train + validation cố định; không cập nhật bằng review xuất hiện trong test.

## 3. Baseline giữ nguyên

**Most Popular:** xếp sản phẩm theo số người dùng đánh giá tích cực khác nhau trong lịch sử; nếu bằng nhau, theo `parent_asin` tăng dần.

**ItemCF + Popular fallback:** tạo ma trận user–item nhị phân, tính cosine giữa sản phẩm, bỏ liên kết với chính sản phẩm và giữ tối đa N lân cận có cosine dương. Điểm ứng viên là tổng độ tương tự từ các sản phẩm tích cực trong lịch sử người dùng. Khi không đủ ứng viên CF, bổ sung Most Popular.

Cosine được tính theo khối ma trận thưa, không tạo ma trận đặc có kích thước số sản phẩm nhân số sản phẩm. Đây là cách triển khai phù hợp hơn với giới hạn RAM của máy đang dùng; thời gian và RAM thực tế vẫn phụ thuộc dữ liệu, môi trường.

Thử **N = 10, 20, 50, 100** trên validation; chọn theo NDCG@10 toàn bộ người dùng đủ điều kiện, rồi Recall@10 cao hơn, rồi N nhỏ hơn. Lần chạy baseline trước chọn **N = 10**. N là số lân cận; K = 10/20 là số sản phẩm gợi ý, hai tham số có ý nghĩa khác nhau.

Notebook 03 mới giữ mã tính baseline và quy tắc đánh giá. Khi Run All, các tệp baseline đang có được sao lưu trước vào `models/baseline_archive/<thời điểm chạy>/`, rồi baseline được chạy lại theo mã cũ. Chỉ dời bước giải phóng train/validation đến sau khi tạo trọng số temporal. Notebook 01, 02, 04 và các hình baseline của notebook 04 được giữ nguyên.

## 4. Khoảng trống và thí nghiệm thời gian trong notebook 03

### 4.1. Vấn đề cần kiểm chứng

Baseline đã chia dữ liệu theo thời gian nhưng coi các tương tác tích cực trong lịch sử có cùng trọng số. **Chia tập theo thời gian chưa đồng nghĩa thuật toán đã dùng thời gian để chấm điểm.**

Thí nghiệm bổ sung kiểm tra việc ưu tiên các review gần đây có làm thay đổi thứ hạng và cải thiện khả năng khôi phục sản phẩm tích cực trong tương lai hay không. Đây là một phép thử cụ thể trên dữ liệu của dự án, chưa phải tuyên bố một khoảng trống nghiên cứu mới đã được xác nhận bằng tổng quan tài liệu.

### 4.2. Time-decay ItemCF

Giữ cosine và số lân cận của baseline. Với mỗi cặp người dùng–sản phẩm tích cực, lấy timestamp của review tích cực mới nhất trong lịch sử, rồi tính:

```text
age_days(u,j) = (t_ref - latest_positive_timestamp(u,j)) / 86400000
w(u,j)       = 2 ** (-age_days(u,j) / h)
score_time(u,i) = sum_j w(u,j) * S[j,i]
```

Trong đó `S[j,i]` là độ tương tự đã giữ từ sản phẩm lịch sử j đến ứng viên i; h là chu kỳ bán rã tính bằng ngày. Khi tuổi review bằng h, trọng số bằng 0,5.

| Thành phần | ItemCF baseline | Time-decay ItemCF |
| --- | --- | --- |
| Cosine, số lân cận N | Học từ lịch sử nhị phân | Dùng chung với baseline tương ứng |
| Trọng số mỗi cặp tích cực | 1 | `2 ** (-age_days / h)` |
| Catalog, loại sản phẩm đã đánh giá | Theo lịch sử huấn luyện | Giữ giống baseline |
| Fallback | Most Popular | Giữ giống baseline |
| Tác động trực tiếp của thời gian | Không | Thay đóng góp vào điểm CF |

Thử **h = 90, 180, 365 ngày**. Đối chứng h = ∞ dùng trọng số 1 để kiểm tra tái tạo baseline. Chỉ trọng số lịch sử được đổi; đây chưa phải mô hình Temporal KG hoặc mô phỏng cập nhật sở thích trực tuyến.

### 4.3. Chọn tham số và đánh giá

1. Giữ quy tắc chọn N của baseline trên validation.
2. Với N đã chọn, học cosine từ **train**; thử ba h hữu hạn trên validation. Mốc tính tuổi review là cuối train.
3. Chọn h theo NDCG@10 của toàn bộ người dùng validation đủ điều kiện; nếu bằng nhau, chọn Recall@10 cao hơn, rồi h nhỏ hơn. Lưu `reports/temporal_cf/selected_config.json` **trước ô đọc test**.
4. Học baseline cuối bằng **train + validation**; tạo ma trận trọng số temporal trên cùng lịch sử, với mốc cuối lịch sử đó.
5. So sánh Most Popular, ItemCF và Time-decay ItemCF trên cùng test; không chọn lại N hoặc h theo test.

Lần chạy này chọn **N = 10 và h = 180 ngày** từ validation. NDCG@10 validation của ItemCF là **0,00665796**, của temporal được chọn là **0,00665916**. Giữ h đã chọn khi đánh giá test.

Giữ h hữu hạn để thực hiện thí nghiệm kể cả khi không vượt baseline; kết quả không cải thiện vẫn cần báo cáo. Baseline test đã được xem trong giai đoạn trước; đây là thí nghiệm mở rộng với tham số chọn trên validation. Một giai đoạn holdout mới sẽ giúp xác nhận độc lập hơn.

### 4.4. Độ thưa và giới hạn cá nhân hóa

| Nhóm theo lịch sử huấn luyện | Khả năng của temporal weighting |
| --- | --- |
| 0 sản phẩm tích cực | Dùng Most Popular; không có bằng chứng tích cực để cá nhân hóa bằng CF |
| 1 sản phẩm tích cực | Mọi điểm CF cùng nhân một hệ số dương nên thứ hạng giữ nguyên |
| Từ 2 sản phẩm tích cực khác nhau | Khác biệt thời điểm có thể đổi đóng góp tương đối; cần đo thứ hạng và chất lượng thực tế |

Notebook báo cáo riêng ba nhóm này, toàn bộ người dùng đủ điều kiện và nhóm có ít nhất một sản phẩm tích cực. Số sản phẩm trong lịch sử được tính từ **train + validation**, không dùng review test để phân nhóm.

Đo tỷ lệ danh sách thay đổi so với ItemCF, số ứng viên CF trong Top-K, tỷ lệ vị trí do CF tạo và tỷ lệ danh sách hoàn toàn fallback. Nếu nhóm từ 2 sản phẩm có ít người, cần đọc kết quả cùng số lượng người dùng và khoảng tin cậy, tránh suy rộng.

Temporal weighting không tạo lịch sử cho người mới hoặc đưa sản phẩm ngoài catalog lịch sử vào ứng viên. Nếu không có review mới, việc dịch chung mốc dự đoán chỉ nhân tất cả đóng góp của một người dùng theo cùng hệ số, nên chưa tạo ra thay đổi thứ hạng theo thời gian một cách độc lập.

## 5. Quy trình đánh giá và kết quả baseline đã có

Người dùng đủ điều kiện đánh giá phải có ít nhất một sản phẩm tích cực mới trong tập đánh giá sau khi loại sản phẩm đã đánh giá trong lịch sử. Giữ sản phẩm chưa xuất hiện trong catalog lịch sử ở ground truth; nếu mô hình không gợi ý được thì tính miss. Người dùng không có hit vẫn nằm trong mẫu số.

| Chỉ số | Ý nghĩa |
| --- | --- |
| Precision@K | Số sản phẩm liên quan trong Top-K chia cho K |
| Recall@K | Số sản phẩm liên quan trong Top-K chia cho số sản phẩm liên quan của người dùng |
| NDCG@K | Chất lượng thứ hạng với relevance nhị phân, chiết khấu theo vị trí |
| CatalogCoverage@K | Số sản phẩm khác nhau được gợi ý chia cho catalog lịch sử |
| `CF_slot_fraction` | Tỷ lệ vị trí trong Top-K do CF cung cấp |
| `popular_only_list_fraction` | Tỷ lệ danh sách hoàn toàn dùng Most Popular |

Các chỉ số lấy trung bình theo người dùng. Phân tích độ bất định bằng paired bootstrap theo cùng người dùng, 1.000 lần, seed 42, khoảng percentile 95%. Temporal được so với ItemCF tĩnh; baseline ItemCF so với Most Popular như trước. Khoảng tin cậy không bao gồm bất định khi học lại hoặc thay đổi giai đoạn thời gian.

### 5.1. Số liệu baseline tham chiếu

Các số dưới đây giữ từ lần chạy baseline trước; **không phải kết quả temporal**. Đối chiếu với CSV gốc trong `reports/final_cf/`. CSV lưu thang 0–1; các cột có ký hiệu % trong bảng đã nhân 100.

**Toàn bộ: 43.656 người dùng test đủ điều kiện.** Lịch sử cuối gồm 624.827 review và 102.963 sản phẩm.

| Mô hình | K | Precision (%) | Recall (%) | NDCG | Catalog coverage (%) |
| --- | ---: | ---: | ---: | ---: | ---: |
| Most Popular | 10 | 0,1636 | 1,6218 | 0,00732834 | 0,0107 |
| ItemCF + fallback | 10 | 0,1645 | 1,6298 | 0,00740466 | 5,8720 |
| Most Popular | 20 | 0,1401 | 2,7770 | 0,01028960 | 0,0214 |
| ItemCF + fallback | 20 | 0,1405 | 2,7850 | 0,01036184 | 6,3285 |

**Nhóm có lịch sử tích cực: 2.533 người dùng**, là tập con của nhóm toàn bộ.

| Mô hình | K | Precision (%) | Recall (%) | NDCG |
| --- | ---: | ---: | ---: | ---: |
| Most Popular | 10 | 0,0790 | 0,7698 | 0,00311049 |
| ItemCF + fallback | 10 | 0,0947 | 0,9080 | 0,00442593 |
| Most Popular | 20 | 0,0770 | 1,5002 | 0,00499570 |
| ItemCF + fallback | 20 | 0,0849 | 1,6384 | 0,00624065 |

Chênh lệch NDCG@10 của ItemCF trừ Most Popular trên toàn bộ là **0,00007632**, CI95% **[-0,00001416; 0,00017774]**. Khoảng này chứa 0 nên chưa có bằng chứng rõ về cải thiện NDCG@10 trong phép đánh giá đó.

ItemCF mở rộng số sản phẩm xuất hiện ở Top-10 từ 11 lên 6.046 sản phẩm. Tuy nhiên, khoảng **95,94%** danh sách hoàn toàn lấy từ Most Popular; CF trực tiếp chỉ chiếm khoảng **2,70%** vị trí trên toàn bộ. Độ phủ cao hơn chưa chứng minh giảm popularity bias, công bằng theo cửa hàng hoặc tăng doanh thu.

### 5.2. Kết quả temporal đã chạy trên All Beauty

Cấu hình cuối là **N = 10, h = 180 ngày**, được chọn trên validation rồi giữ nguyên khi đánh giá test. Ba mô hình được so sánh trên cùng **43.656 người dùng đủ điều kiện**, cùng lịch sử train + validation và cùng ground truth.

| Mô hình | K | Precision (%) | Recall (%) | NDCG | Catalog coverage (%) |
| --- | --- | --- | --- | --- | --- |
| Most Popular | 10 | 0.1636 | 1.6218 | 0.00732834 | 0.0107 |
| ItemCF + fallback | 10 | 0.1645 | 1.6298 | 0.00740466 | 5.8720 |
| Time-decay ItemCF + fallback | 10 | 0.1645 | 1.6298 | 0.00740848 | 5.8992 |
| Most Popular | 20 | 0.1401 | 2.7770 | 0.01028960 | 0.0214 |
| ItemCF + fallback | 20 | 0.1405 | 2.7850 | 0.01036184 | 6.3285 |
| Time-decay ItemCF + fallback | 20 | 0.1406 | 2.7862 | 0.01036878 | 6.3537 |

Nguồn: [test_comparison.csv](reports/temporal_cf/test_comparison.csv). Các cột Precision, Recall và coverage có ký hiệu % đã nhân 100; NDCG giữ thang 0–1.

Ở Top-10, temporal có NDCG cao hơn ItemCF một lượng nhỏ, trong khi Precision và Recall không đổi trong lần chạy này. Kết quả đó phù hợp với việc thời gian tác động vào vị trí của sản phẩm liên quan trong danh sách; không đồng nghĩa thu hồi thêm sản phẩm liên quan ở K = 10.

### 5.3. Tác động của thời gian và giới hạn cá nhân hóa

Các nhóm được xác định bằng số **sản phẩm tích cực khác nhau trong train + validation**:

| Lịch sử tích cực | Người dùng | NDCG@10 ItemCF | NDCG@10 temporal | Top-10 thay đổi | Tỷ lệ thay đổi (%) |
| --- | --- | --- | --- | --- | --- |
| 0 sản phẩm tích cực | 41,123 | 0.00758814 | 0.00758814 | 0 | 0.00 |
| 1 sản phẩm tích cực | 1,850 | 0.00483546 | 0.00483546 | 0 | 0.00 |
| Từ 2 sản phẩm tích cực | 683 | 0.00331668 | 0.00356070 | 379 | 55.49 |

Nguồn: [personalization_groups.csv](reports/temporal_cf/personalization_groups.csv) và [rank_change_by_group.csv](reports/temporal_cf/rank_change_by_group.csv). Số người dùng trong bảng dùng dấu phẩy để phân cách hàng nghìn.

- Trong 683 người dùng có từ 2 sản phẩm tích cực, **379 người đổi Top-10, tương đương 55,49%**; Top-20 thay đổi ở **400 người, tương đương 58,57%**. Điều này xác nhận yếu tố thời gian đã tác động trực tiếp đến kết quả thuật toán.
- Nhóm không có lịch sử tích cực và nhóm chỉ có một sản phẩm tích cực đều không đổi thứ hạng, đúng với cơ chế fallback và phép nhân chung một hệ số dương.
- Nhóm từ 2 sản phẩm tích cực chỉ chiếm khoảng **1,56%** toàn bộ người dùng test đủ điều kiện. Vì vậy tỷ lệ danh sách đổi Top-10 trên toàn bộ chỉ khoảng **0,87%**, dù trong nhóm này đạt 55,49%.
- Trên toàn bộ, khoảng **95,94%** danh sách Top-10 vẫn hoàn toàn dùng Most Popular. Temporal weighting chưa khắc phục thiếu lịch sử hoặc thiếu sản phẩm trong catalog ứng viên.

![So sánh temporal và khả năng cá nhân hóa](reports/temporal_cf/temporal_comparison.png)

### 5.4. Khoảng tin cậy và kết luận thực nghiệm

Paired bootstrap theo cùng người dùng, 1.000 lần, seed 42:

| Nhóm | Chênh lệch NDCG@10 temporal − ItemCF | CI95% | Chứa 0 |
| --- | --- | --- | --- |
| Toàn bộ | 0.00000382 | [0.00000000; 0.00001527] | Có |
| Từ 2 sản phẩm tích cực | 0.00024402 | [0.00000000; 0.00073206] | Có |

Nguồn: [test_ndcg10_paired_bootstrap.csv](reports/temporal_cf/test_ndcg10_paired_bootstrap.csv).

**Thời gian đã làm thay đổi thứ hạng ở nhóm có đủ lịch sử, nhưng chưa có bằng chứng rõ về cải thiện NDCG@10**, vì khoảng tin cậy chứa 0. Không diễn giải mức tăng điểm nhỏ thành cải thiện đáng tin cậy, tăng doanh thu hoặc giải quyết popularity bias.

### 5.5. Tệp đối chiếu cho leader

| Nội dung | Tệp |
| --- | --- |
| Các h đã thử và cấu hình chọn | [validation_grid.csv](reports/temporal_cf/validation_grid.csv), [selected_config.json](reports/temporal_cf/selected_config.json) |
| So sánh ba mô hình trên test | [test_comparison.csv](reports/temporal_cf/test_comparison.csv) |
| Chất lượng theo lịch sử tích cực | [personalization_groups.csv](reports/temporal_cf/personalization_groups.csv) |
| Tỷ lệ danh sách đổi thứ hạng | [rank_change_by_group.csv](reports/temporal_cf/rank_change_by_group.csv) |
| Temporal trừ ItemCF và khoảng tin cậy | [test_ndcg10_paired_bootstrap.csv](reports/temporal_cf/test_ndcg10_paired_bootstrap.csv) |
| Biểu đồ và diễn giải tự xuất | [temporal_comparison.png](reports/temporal_cf/temporal_comparison.png), [temporal_summary.md](reports/temporal_cf/temporal_summary.md) |
| Mốc thời gian và fingerprint | [test_protocol.json](reports/temporal_cf/test_protocol.json) |
| Quy mô và kiểm tra gói dùng cho Member 2 | [handoff_summary.json](reports/temporal_cf/handoff_summary.json) |

Các báo cáo trên đã được tạo từ lần chạy thật và đưa lên repository. README là bản tổng hợp viết riêng, không tự thay đổi khi nhấn Run All; khi chạy thí nghiệm khác cần đối chiếu các CSV mới rồi cập nhật nội dung tương ứng.

## 6. Bốn notebook và cách chạy bản cập nhật

| Notebook | Vai trò | Đầu ra chính |
| --- | --- | --- |
| [01_data_preparation.ipynb](01_data_preparation.ipynb) | Kiểm tra dữ liệu, chuẩn hóa và chia tập thời gian | `data/processed/` |
| [02_item_based_cf.ipynb](02_item_based_cf.ipynb) | Most Popular và ItemCF ban đầu trên validation | `reports/item_cf/`, `models/item_cf/` |
| [03_tuning_and_test.ipynb](03_tuning_and_test.ipynb) | Giữ baseline; thêm time-decay, phân tích dữ liệu thưa và bàn giao Member 2 | `reports/final_cf/`, `reports/temporal_cf/`, mô hình và gói handoff |
| [04_results_and_demo.ipynb](04_results_and_demo.ipynb) | Giữ biểu đồ và demo baseline, ghép tên sản phẩm | `reports/cf_presentation/` |

### 6.1. Chạy lại trên VS Code

Lần chạy ngày 24/10/2026 đã hoàn tất. Các bước dưới đây dùng để tái lập kết quả; không cần chạy lại chỉ để cập nhật README.

1. Mở thư mục dự án chứa bốn notebook; giữ tên `03_tuning_and_test.ipynb` tại thư mục gốc.
2. Kiểm tra đã có `data/processed/train.parquet`, `validation.parquet`, `test.parquet`, `split_config.json` từ notebook 01. Nếu đủ, **không cần chạy lại notebook 01 và 02** chỉ để dùng bản cập nhật.
3. Mở notebook 03, chọn kernel `.venv`, lưu tệp rồi chọn **Restart Kernel → Run All**. Kiểm tra `Path.cwd()` in đúng thư mục gốc dự án.
4. Giữ mặc định `HALF_LIFE_GRID = (90, 180, 365)` và `CANDIDATE_K = 100`. Notebook tự sao lưu đầu ra baseline cũ trước khi tái tạo baseline.
5. Chờ chạy hết bước bàn giao, sau đó xem `reports/temporal_cf/temporal_summary.md` và `handoff_summary.json`.

Notebook 03 mới chạy độc lập với biến của notebook 02. Không cần đọc lại Excel cho bước này. Không cần chạy notebook 04 để có biểu đồ temporal vì notebook 03 đã tự tạo hình riêng. Notebook 04 tiếp tục dùng mô hình và kết quả baseline.

Lần chạy này đã xuất **8.731.200 dòng ứng viên**. Notebook ghi CSV nén theo luồng và in tiến độ, tránh nạp toàn bộ bảng vào RAM. Thời gian chạy lại phụ thuộc CPU, số ứng viên và ổ đĩa.

### 6.2. Môi trường

Môi trường dự án: Windows, VS Code, Python/Jupyter, CPU và RAM 16 GB. Phiên bản được ghi nhận trong lần chạy: Python 3.12.10, pandas 3.0.6, NumPy 2.5.4, SciPy 1.18.1 và scikit-learn 1.9.1. Các phiên bản này mô tả môi trường đã chạy, không phải yêu cầu bắt buộc nâng cấp mọi máy lên cùng phiên bản. Không cần API key hoặc GPU cho thí nghiệm này. Dùng `requirements.txt` hiện có; phần mới dùng các thư viện thông dụng đã phục vụ baseline và biểu đồ: numpy, pandas, scipy, scikit-learn, matplotlib; đọc Parquet cần pyarrow.

Nếu thiết lập từ đầu, chạy PowerShell tại thư mục dự án:

```powershell
py -3.12 -m venv .venv
.\.venv\Scripts\python.exe -m pip install --upgrade pip
.\.venv\Scripts\python.exe -m pip install -r requirements.txt
```

Nếu còn thiếu thư viện khi chọn đúng kernel:

```powershell
.\.venv\Scripts\python.exe -m pip install numpy pandas scipy scikit-learn matplotlib pyarrow ipykernel
```

Ghi lại phiên bản từ môi trường đã chạy thành công:

```powershell
.\.venv\Scripts\python.exe -m pip freeze > requirements-lock.txt
```

## 7. Gói bàn giao cho Member 2

Mục 13 của notebook 03 đã chạy thành công và tạo các tệp sau trên máy người thực hiện:

```text
models/member2_handoff/
    README_MEMBER2.md
    baseline_models.py
    baseline_model/
    temporal_model/
    candidates_top100.csv.gz
    static_selected_config.json
    temporal_selected_config.json
    test_comparison.csv
    personalization_groups.csv
    rank_change_by_group.csv
    test_ndcg10_paired_bootstrap.csv
    manifest.json
models/member2_handoff.zip
```

Gói gồm mô hình ItemCF baseline và biến thể temporal được chọn, module nạp/dự đoán độc lập với trạng thái notebook, các cấu hình và báo cáo tổng hợp. Kết quả `handoff_summary.json` ghi nhận **43.656 người dùng**, **8.731.200 dòng**, gồm **4.365.600 dòng cho mỗi biến thể**, `zip_created = true` và `standalone_load_check = passed`. Notebook đã kiểm tra nạp lại hai mô hình và đối chiếu dự đoán cùng điểm số.

Gói ZIP đầy đủ nằm trong `models/`, được loại khỏi Git và cần chia sẻ riêng cho Member 2. Repository chứa báo cáo tổng hợp và bản xem nhanh ứng viên, không chứa toàn bộ mô hình hoặc CSV ứng viên lớn. Việc tạo gói và việc Member 2 nhận gói là hai bước bàn giao riêng.

### 7.1. Phạm vi ứng viên và schema

Xuất tối đa **100 ứng viên cho mỗi biến thể trên mỗi người dùng test đủ điều kiện đánh giá**. Hai biến thể có danh sách riêng; định dạng CSV dùng một dòng cho một ứng viên của một biến thể. Phạm vi này không gồm mọi tài khoản hoặc mọi người viết review trong test. Không xuất nhãn relevance test vào bảng ứng viên.

| Trường | Ý nghĩa |
| --- | --- |
| `model` | `itemcf_static` hoặc `itemcf_time_decay` |
| `user_id`, `parent_asin` | ID dạng chuỗi thống nhất với dữ liệu dự án |
| `rank` | Thứ hạng cuối bắt đầu từ 1; dùng làm thứ tự chuẩn |
| `score` | Điểm vô hướng giữ nguyên chính sách CF trước, fallback sau |
| `cf_score` | Tổng cosine hoặc tổng cosine có trọng số thời gian; fallback bằng 0 |
| `positive_user_count` | Số người dùng tích cực khác nhau của sản phẩm trong lịch sử |
| `ranking_tier` | 1 cho CF, 0 cho fallback |
| `score_source` | `CF` hoặc `Most Popular fallback` |
| `history_positive_count` | Số sản phẩm tích cực khác nhau của người dùng trong lịch sử |
| `prediction_timestamp_ms` | Mốc dự đoán, đơn vị mili giây |

Khóa duy nhất của vị trí là (`model`, `user_id`, `rank`); sản phẩm không lặp trong từng danh sách. `score` dùng `1 + cf_score/(1+cf_score)` cho CF và `positive_user_count/(1+max_positive_user_count)` cho fallback. Phép biến đổi này giữ CF ở trước fallback, không thêm thuật toán điều chỉnh công bằng. Nếu bằng điểm, theo `parent_asin` tăng dần. Điểm chưa được hiệu chỉnh để so sánh giữa người dùng hoặc diễn giải thành xác suất mua.

Member 2 có thể rerank các ứng viên đã xuất. Các sản phẩm ngoài Top-100 hoặc ngoài catalog lịch sử cần nguồn ứng viên bổ sung nếu muốn xét đến; khi đó phải ghi rõ thay đổi quy trình so sánh.

### 7.2. Ví dụ nạp mô hình

Giải nén gói, đứng tại thư mục có `baseline_models.py`:

```python
from pathlib import Path
import pandas as pd
from baseline_models import ItemCFModel, TimeDecayItemCF, scored_recommendations

root = Path('.')
baseline = ItemCFModel.load(root / 'baseline_model')
temporal = TimeDecayItemCF.load(root / 'temporal_model', baseline)

user_id = 'THAY_BANG_USER_ID_CAN_GOI_Y'
top_k = pd.DataFrame(scored_recommendations(temporal, user_id, k=100))
```

Giữ hai thư mục mô hình cùng nhau: temporal dùng cosine và các ánh xạ của baseline. Đọc CSV ứng viên theo `chunksize` nếu cần để tránh nạp toàn bộ tệp vào RAM. Chi tiết nằm trong `README_MEMBER2.md`; `manifest.json` lưu số dòng, cấu hình, phiên bản thư viện và SHA256 tệp.

## 8. Đưa lên GitHub để leader xem

| Nội dung | Cách bàn giao |
| --- | --- |
| Bốn notebook, README, requirements, `.gitignore` | Commit vào repository |
| `reports/final_cf/`, `reports/cf_presentation/` | Giữ các báo cáo tổng hợp và hình baseline |
| `reports/temporal_cf/` | Commit CSV tổng hợp, JSON cấu hình, Markdown và PNG sau khi chạy thành công |
| `reports/temporal_cf/test_per_user.csv.gz` | Chia sẻ riêng; `.gitignore` hiện có đã loại mẫu tệp này |
| Excel, Parquet, `data/` | Chia sẻ riêng |
| `models/`, gói handoff, bản sao lưu baseline | Chia sẻ riêng; nằm dưới thư mục đã được `.gitignore` loại |
| `.venv/` | Tạo lại bằng requirements |

Leader có thể xem ngay [báo cáo temporal](reports/temporal_cf/temporal_summary.md), [so sánh test](reports/temporal_cf/test_comparison.csv), [phân tích nhóm](reports/temporal_cf/personalization_groups.csv) và [thay đổi thứ hạng](reports/temporal_cf/rank_change_by_group.csv). Các tệp kết quả này đã có trong repository. Khi cập nhật README sau lần chạy này, chỉ cần commit và push README mới.

## 9. Tổng kết phần việc

Đã hoàn thành phần thực nghiệm **Temporal Recommendation Baseline** gồm chuẩn bị dữ liệu, Most Popular, ItemCF, một phép thử time-decay trực tiếp vào điểm ứng viên, đánh giá test và phân tích cá nhân hóa theo độ dày lịch sử. Bốn notebook được giữ trong quy trình; baseline và các báo cáo trước vẫn là mốc đối chiếu.

Cấu hình chọn là **N = 10, h = 180 ngày**. Trên nhóm có từ 2 sản phẩm tích cực, thời gian làm thay đổi Top-10 ở **379/683 người dùng (55,49%)**. NDCG@10 toàn bộ tăng từ **0,00740466 lên 0,00740848**, nhưng khoảng tin cậy chứa 0 nên **chưa có bằng chứng rõ về cải thiện chất lượng gợi ý**. Đóng góp của thí nghiệm là kiểm chứng tác động thực tế của thời gian và chỉ ra giới hạn do dữ liệu rất thưa, không mặc định mô hình temporal sẽ tốt hơn.

Đã tạo gói gồm hai mô hình, module nạp độc lập và **8.731.200 dòng ứng viên/điểm/thứ hạng** cho bước tiếp theo của Member 2; kiểm tra nạp lại thành công. Cần gửi riêng tệp `models/member2_handoff.zip` cho Member 2. Thuật toán giảm popularity bias không thuộc trách nhiệm chính của phần này; kết quả hiện tại chưa chứng minh hiệu quả doanh thu hay công bằng theo cửa hàng.

## Tài liệu tham khảo

- [Recommenders — repository tham khảo](https://github.com/recommenders-team/recommenders).
- Koren, Y. (2009). [Collaborative Filtering with Temporal Dynamics](https://doi.org/10.1145/1557019.1557072) — tài liệu nền về mô hình hóa thời gian; phép thử decay đơn giản trong notebook không phải bản tái hiện đầy đủ phương pháp của bài báo.
- [scikit-learn — Cross-validation](https://scikit-learn.org/stable/modules/cross_validation.html).
- [SciPy — lưu ma trận thưa](https://docs.scipy.org/doc/scipy/reference/generated/scipy.sparse.save_npz.html).

Repository Recommenders được dùng để tham khảo. Mã nguồn triển khai và kết quả dự án được theo dõi tại [recommender-system-huong1](https://github.com/CuongQuoc-speed/recommender-system-huong1).

# Temporal Recommendation Baseline — Cường Quốc

## Thiết kế

- Giữ baseline Most Popular và ItemCF; cosine nhị phân, catalog, seen-item filter và fallback không đổi.
- N=10 lấy từ quy tắc validation của baseline. Thử h=[90, 180, 365] ngày trên validation; chọn h=180 ngày bằng NDCG@10 toàn bộ, rồi Recall@10, rồi h nhỏ hơn.
- Mỗi cặp tích cực dùng review tích cực mới nhất trong lịch sử. Trọng số là `2 ** (-age_days / h)` và đi trực tiếp vào tổng điểm ứng viên.
- Validation dùng lịch sử train; test dùng lịch sử train + validation, cố định tại 1643201980125 ms. Không cập nhật bằng review test.
- Đối chứng h=∞ tái tạo baseline. Đây là phần mở rộng sau khi baseline đã được báo cáo; không chọn h theo test. Muốn xác nhận độc lập hơn cần một giai đoạn holdout mới.

## Kết quả Top-10 theo số sản phẩm tích cực khác nhau trong lịch sử

| model | group | users | NDCG@K | Recall@K | CF_slot_fraction |
| --- | --- | --- | --- | --- | --- |
| Most Popular | Toàn bộ | 43656 | 0.00732834 | 0.01621770 | 0.00000000 |
| Most Popular | Có lịch sử tích cực | 2533 | 0.00311049 | 0.00769838 | 0.00000000 |
| Most Popular | 0 sản phẩm tích cực | 41123 | 0.00758814 | 0.01674246 | 0.00000000 |
| Most Popular | 1 sản phẩm tích cực | 1850 | 0.00398858 | 0.01000000 | 0.00000000 |
| Most Popular | Từ 2 sản phẩm tích cực | 683 | 0.00073206 | 0.00146413 | 0.00000000 |
| ItemCF + Popular fallback | Toàn bộ | 43656 | 0.00740466 | 0.01629787 | 0.02698598 |
| ItemCF + Popular fallback | Có lịch sử tích cực | 2533 | 0.00442593 | 0.00908014 | 0.46510067 |
| ItemCF + Popular fallback | 0 sản phẩm tích cực | 41123 | 0.00758814 | 0.01674246 | 0.00000000 |
| ItemCF + Popular fallback | 1 sản phẩm tích cực | 1850 | 0.00483546 | 0.01027027 | 0.34616216 |
| ItemCF + Popular fallback | Từ 2 sản phẩm tích cực | 683 | 0.00331668 | 0.00585652 | 0.78726208 |
| Time-decay ItemCF + Popular fallback | Toàn bộ | 43656 | 0.00740848 | 0.01629787 | 0.02698598 |
| Time-decay ItemCF + Popular fallback | Có lịch sử tích cực | 2533 | 0.00449173 | 0.00908014 | 0.46510067 |
| Time-decay ItemCF + Popular fallback | 0 sản phẩm tích cực | 41123 | 0.00758814 | 0.01674246 | 0.00000000 |
| Time-decay ItemCF + Popular fallback | 1 sản phẩm tích cực | 1850 | 0.00483546 | 0.01027027 | 0.34616216 |
| Time-decay ItemCF + Popular fallback | Từ 2 sản phẩm tích cực | 683 | 0.00356070 | 0.00585652 | 0.78726208 |

## Tác động đến thứ hạng

| k | group | users | changed_lists | changed_list_fraction |
| --- | --- | --- | --- | --- |
| 10 | Toàn bộ | 43656 | 379 | 0.00868150998717244 |
| 10 | Có lịch sử tích cực | 2533 | 379 | 0.1496249506514015 |
| 10 | 0 sản phẩm tích cực | 41123 | 0 | 0.0 |
| 10 | 1 sản phẩm tích cực | 1850 | 0 | 0.0 |
| 10 | Từ 2 sản phẩm tích cực | 683 | 379 | 0.554904831625183 |

## Độ bất định và giới hạn

Chênh lệch NDCG@10 temporal trừ static trên toàn bộ: 0.00000382; CI95% [0.00000000; 0.00001527]. Khoảng tin cậy chứa 0; chưa có bằng chứng rõ temporal cải thiện NDCG@10.
Bootstrap theo người dùng 1000 lần, seed 42; không bao gồm bất định khi huấn luyện lại hoặc thay đổi giai đoạn.

- Nhóm 0 sản phẩm tích cực dùng Popular. Nhóm 1 sản phẩm tích cực nhận cùng thứ hạng CF vì mọi ứng viên cùng nhân một hệ số dương.
- Với từ 2 sản phẩm tích cực, chênh lệch thời điểm có thể đổi đóng góp tương đối và thứ hạng; cải thiện phải dựa vào số liệu, không được mặc định.
- Nếu không có review mới, dịch chung mốc dự đoán chỉ nhân chung một hệ số; mô hình này chưa mô phỏng cập nhật sở thích trực tuyến.
- Temporal weighting không tạo dữ liệu cho người mới, không đưa sản phẩm ngoài catalog lịch sử vào tập ứng viên và không giải quyết chính thuật toán popularity bias.
- Dữ liệu là review và thời điểm review. Không kết luận hiệu quả doanh thu hay xác suất mua từ điểm CF.
- Handoff Top-100 chỉ bao gồm người dùng test đủ điều kiện đánh giá. Member 2 cần giữ quy tắc đánh giá và ghi rõ thay đổi candidate set nếu mở rộng.

## Tệp kết quả

- `validation_grid.csv`, `selected_config.json`: phép thử và lựa chọn từ validation.
- `test_comparison.csv`, `personalization_groups.csv`, `rank_change_by_group.csv`.
- `test_ndcg10_paired_bootstrap.csv`, `test_per_user.csv.gz`, `test_protocol.json`.
- `temporal_comparison.png`; mô hình temporal ở `models/item_cf_temporal/`.
- Gói dùng cho Member 2 được tạo ở bước bàn giao kế tiếp, trong `models/member2_handoff/` và `models/member2_handoff.zip`.

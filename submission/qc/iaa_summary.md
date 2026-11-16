# Tóm tắt IAA (taxonomy v1.0)

- Ảnh được so sánh (có >=2 người gán): **40**; số người gán: 2
- Lượt gán đủ **80/80**.
- IoU trung bình và % cùng lớp **chỉ tính trên các cặp box đã ghép** (IoU >= 0.5, ghép 1-1 không phân biệt lớp). Box chưa ghép được thống kê riêng ở cột bỏ sót/thừa.
- Mẫu số: 240 cặp box ghép (tổng mọi cặp người gán, ảnh chung).

| Cặp | Ảnh | Cặp box ghép | IoU TB | Cùng lớp | Cùng occluded_level | Cùng truncated | Chưa ghép (A/B) | Bỏ qua do ignore | Đạt IAA |
|---|---|---|---|---|---|---|---|---|---|
| **TỔNG** | 40 | 240 | 0.861 | 99.2% | 68.3% | 99.2% | 22/32 | 8 | ĐẠT |
| 101|102 | 40 | 240 | 0.861 | 99.2% | 68.3% | 99.2% | 22/32 | 8 | ĐẠT |

Ngưỡng IAA: IoU TB >= 0.85, đồng thuận lớp >= 95%.

Số ca bất thường: 277 ({'co_can_review': 38, 'iou_thap': 73, 'sai_thuoc_tinh': 76, 'bat_dong_ignore': 60, 'bo_sot_thua': 28, 'sai_lop': 2}), trên 32 ảnh cần calibration.

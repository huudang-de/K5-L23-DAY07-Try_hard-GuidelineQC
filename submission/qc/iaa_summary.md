# Tóm tắt IAA (taxonomy v1.0)

- Ảnh được so sánh (có >=2 người gán): **40**; số người gán: 5
- Lượt gán đủ **120/120**.
- IoU trung bình và % cùng lớp **chỉ tính trên các cặp box đã ghép** (IoU >= 0.5, ghép 1-1 không phân biệt lớp). Box chưa ghép được thống kê riêng ở cột bỏ sót/thừa.
- Mẫu số: 706 cặp box ghép (tổng mọi cặp người gán, ảnh chung).

| Cặp | Ảnh | Cặp box ghép | IoU TB | Cùng lớp | Cùng occluded_level | Cùng truncated | Chưa ghép (A/B) | Bỏ qua do ignore | Đạt IAA |
|---|---|---|---|---|---|---|---|---|---|
| **TỔNG** | 120 | 706 | 0.848 | 96.3% | 47.3% | 87.5% | 405/235 | 0 | CHƯA ĐẠT |
| 101|102 | 13 | 93 | 0.883 | 96.8% | 72.0% | 87.1% | 1/83 | 0 | ĐẠT |
| 101|103 | 12 | 89 | 0.814 | 100.0% | 28.1% | 80.9% | 34/38 | 0 | CHƯA ĐẠT |
| 101|104 | 12 | 80 | 0.820 | 100.0% | 57.5% | 87.5% | 37/7 | 0 | CHƯA ĐẠT |
| 101|105 | 11 | 55 | 0.862 | 94.5% | 41.8% | 81.8% | 33/19 | 0 | CHƯA ĐẠT |
| 102|103 | 13 | 103 | 0.806 | 100.0% | 25.2% | 95.1% | 84/2 | 0 | CHƯA ĐẠT |
| 102|104 | 11 | 54 | 0.895 | 100.0% | 35.2% | 77.8% | 78/1 | 0 | ĐẠT |
| 102|105 | 11 | 60 | 0.896 | 93.3% | 65.0% | 100.0% | 51/3 | 0 | CHƯA ĐẠT |
| 103|104 | 11 | 61 | 0.833 | 98.4% | 47.5% | 83.6% | 39/16 | 0 | CHƯA ĐẠT |
| 103|105 | 12 | 53 | 0.829 | 86.8% | 45.3% | 94.3% | 25/32 | 0 | CHƯA ĐẠT |
| 104|105 | 14 | 58 | 0.884 | 86.2% | 62.1% | 84.5% | 23/34 | 0 | CHƯA ĐẠT |

Ngưỡng IAA: IoU TB >= 0.85, đồng thuận lớp >= 95%.

Số ca bất thường: 1449 ({'bo_sot_thua': 498, 'iou_thap': 262, 'sai_thuoc_tinh': 414, 'co_can_review': 29, 'sai_lop': 26, 'bat_dong_ignore': 220}), trên 37 ảnh cần calibration.

# Tóm tắt IAA (taxonomy v1.0)

- Ảnh được so sánh (có >=2 người gán): **40**; số người gán: 5
- Lượt gán đủ **120/120**.
- IoU trung bình và % cùng lớp **chỉ tính trên các cặp box đã ghép** (IoU >= 0.5, ghép 1-1 không phân biệt lớp). Box chưa ghép được thống kê riêng ở cột bỏ sót/thừa.
- Mẫu số: 659 cặp box ghép (tổng mọi cặp người gán, ảnh chung).

| Cặp | Ảnh | Cặp box ghép | IoU TB | Cùng lớp | Cùng occluded_level | Cùng truncated | Chưa ghép (A/B) | Bỏ qua do ignore | Đạt IAA |
|---|---|---|---|---|---|---|---|---|---|
| **TỔNG** | 120 | 659 | 0.858 | 96.5% | 53.4% | 87.9% | 330/282 | 0 | ĐẠT |
| 101|102 | 13 | 67 | 0.819 | 98.5% | 64.2% | 80.6% | 0/109 | 0 | CHƯA ĐẠT |
| 101|103 | 12 | 67 | 0.778 | 100.0% | 32.8% | 73.1% | 15/60 | 0 | CHƯA ĐẠT |
| 101|104 | 12 | 87 | 1.000 | 100.0% | 100.0% | 100.0% | 0/0 | 0 | ĐẠT |
| 101|105 | 11 | 49 | 0.822 | 95.9% | 55.1% | 79.6% | 15/25 | 0 | CHƯA ĐẠT |
| 102|103 | 13 | 103 | 0.806 | 100.0% | 25.2% | 95.1% | 84/2 | 0 | CHƯA ĐẠT |
| 102|104 | 11 | 54 | 0.895 | 100.0% | 35.2% | 77.8% | 78/1 | 0 | ĐẠT |
| 102|105 | 11 | 60 | 0.896 | 93.3% | 65.0% | 100.0% | 51/3 | 0 | CHƯA ĐẠT |
| 103|104 | 11 | 61 | 0.833 | 98.4% | 47.5% | 83.6% | 39/16 | 0 | CHƯA ĐẠT |
| 103|105 | 12 | 53 | 0.829 | 86.8% | 45.3% | 94.3% | 25/32 | 0 | CHƯA ĐẠT |
| 104|105 | 14 | 58 | 0.884 | 86.2% | 62.1% | 84.5% | 23/34 | 0 | CHƯA ĐẠT |

Ngưỡng IAA: IoU TB >= 0.85, đồng thuận lớp >= 95%.

Số ca bất thường: 1323 ({'bo_sot_thua': 477, 'iou_thap': 243, 'sai_thuoc_tinh': 341, 'co_can_review': 30, 'sai_lop': 23, 'bat_dong_ignore': 209}), trên 37 ảnh cần calibration.

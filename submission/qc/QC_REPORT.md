# Báo cáo QC — Nhóm Try_hard

## 1. Mục đích, dữ liệu và phiên bản

- Mục đích và giới hạn sử dụng: Thực hành đặc tả guideline và kiểm định chất lượng (QC) nhãn bounding box 2D (9 lớp phương tiện/người + `ignore_region`) trên 40 ảnh đường phố tuyển chọn luyện ca biên; độ đại diện cho dữ liệu vận hành chưa được đánh giá.
- Nguồn và phép sử dụng dữ liệu: Theo `data/ATTRIBUTION.csv` gồm 40 ảnh từ Wikimedia Commons (15 `CC BY-SA 4.0`, 7 `CC BY 2.0`, 6 `CC BY-SA 2.0`, 5 `CC BY 4.0`, 3 `CC0`, 3 `CC BY-SA 3.0`, 1 `Public domain`), giữ nguyên ghi công tác giả, giấy phép và `source_url` trong `batch_manifest.json`.
- Manifest và checksum nhãn cuối: `submission/qc/batch_manifest.json` — `labels_calibrated.json` SHA-256 `088b7423be8250348af41148c8c07897654ec6014aa0d018b5f3f2eb539da0c2` (40 ảnh, 521 annotation gồm 514 đối tượng và 7 `ignore_region`); `labels_calibrated.csv` SHA-256 `3cced2f45bd6869d177e80e8d6888d49169a4db447756060adfcc2baf5e6c22c`.
- Guideline và taxonomy khi gán / khi bàn giao: Gán độc lập theo `guideline_v1.0_agreed.md` và `taxonomy.json` (`v1.0`); bàn giao theo `guideline_v1.1_calibrated.md` (`kind: editorial` trong `submission/guideline_change.json`), giữ nguyên `taxonomy.json` (`v1.0`) và toàn bộ ngưỡng `rules`.

## 2. Phân công và đồng thuận trước calibration

- Lượt hoàn thành / được giao: 120/120 lượt (5 thành viên `101`–`105`, K = 3 lượt/ảnh, 24 ảnh/người, đủ 40/40 ảnh).
- Số cặp box ghép, IoU TB, đồng thuận lớp và mẫu số: 706 cặp box ghép (trên 120 lượt ảnh giao nhau), IoU trung bình `0.848` (ngưỡng `>= 0.85`, CHƯA ĐẠT), đồng thuận lớp `96.3%` (ngưỡng `>= 95%`, ĐẠT); đồng thuận `occluded_level` `47.3%`, `truncated` `87.5%` (nguồn: `submission/qc/iaa_summary.md` và `iaa_summary.json`).
- Box chưa ghép, loại do ignore, bất đồng vùng ignore: Chưa ghép `405/235` (498 ca `bo_sot_thua`), loại do ignore `0`, bất đồng vùng ignore `220` ca; tổng `1,449` ca bất thường trên `37/40` ảnh bị cờ trong `submission/qc/flagged_images.txt`.
- Ca hai người có thể cùng sai sau khi đối chiếu tham chiếu: `img_001.jpg` / `img_012.jpg` (dễ gán pickup thành `car` nếu không bám tiêu chí thùng rời cabin `R-07`), `img_017.jpg` / `img_027.jpg` / `img_033.jpg` (tượng/ma-nơ-canh dễ bị gán nhầm `person`/`truck` thay vì để trống theo `R-01`, `R-18`), và `img_003.jpg` / `img_035.jpg` (có xu hướng kéo box bao luôn hàng hóa trên người/tuk-tuk nếu không tách theo `R-09`, `R-10`).

## 3. Calibration và kiểm lại

- Ca đã xử lý, rule ID, bản sửa và người xác nhận: Đã chốt đủ `1,449/1,449` dòng (`C001`–`C1449`) trong `submission/qc/calibration_decisions.csv` (dẫn rule `R-01`..`R-18`, người chốt `101`, `pham_vi=anh`), áp dụng sửa trên 37 ảnh bị cờ tại `submission/qc/labels_calibrated_1bda6829c92de0f4.json` và chuyển sang kiểm tra chéo Pha 6.
- Tác động v1.1 lên nhãn và ảnh: `guideline_v1.1_calibrated.md` bổ sung làm rõ biên tập (`editorial`) và ví dụ cho các quy tắc hiện có (`R-01`, `R-05`, `R-07`, `R-11`, `R-13`, `R-17`, `R-18`), không đổi cấu trúc lớp/thuộc tính của `taxonomy.json` v1.0; cập nhật 37 ảnh flagged khi gộp `labels_calibrated.json`.
- Tham chiếu bằng chứng khi Rework: Vòng 1 phát hiện lỗi trên 10/40 ảnh (`25.0% > 2%`); nếu mở vòng 2, giữ nguyên toàn bộ thư mục `submission/qc/reviews/round-01/`, sửa các object đã chỉ rõ trong `qc_review.csv` (`img_001.jpg`, `img_003.jpg`, `img_005.jpg`, `img_012.jpg`, `img_019.jpg`, `img_032.jpg`, `img_034.jpg`, `img_035.jpg`, `img_036.jpg`, `img_040.jpg`) qua Pha 5 rồi chạy `qc_sample.py --round 2`.

## 4. Kết quả duyệt

- Vòng QC: 1
- Người review và ngày: 105, 2026-11-17
- Đã duyệt / cả lô: 40/40 ảnh (nhóm `ca_kho`: 37/37, nhóm `con_lai`: 3/3)
- Ảnh lỗi / ảnh đã duyệt: 10/40 = 25.0% (`ca_kho`: 10/37 = 27.0%, `con_lai`: 0/3 = 0.0%)
- Annotation đối tượng hiện có bị lỗi / annotation đối tượng đã duyệt: 20/514 = 3.9%
- Vật bỏ sót, ảnh và vùng: 1 vật bỏ sót ghi riêng tại `img_003.jpg` (vùng xe đạp giữa ảnh cạnh người cúi, khoảng `x=235–420 y=230–430`, bị gộp chung vào box `48` thay vì tách box riêng theo `R-04`).
- Ca khó / phần còn lại, lỗi lặp và ví dụ: Trong `submission/qc/reviews/round-01/qc_review.csv` có 21 dòng lỗi thuộc 10 ảnh đều nằm ở nhóm `ca_kho` (`sai_lop`: 9 lỗi tại `img_019.jpg` [#241–#244] và `img_032.jpg` [#415, #422–#425] do gán `car` lên người/đám đông; `sai_vi_tri_kich_thuoc`: 6 lỗi tại `img_001.jpg` [#7], `img_003.jpg` [#33, #48], `img_005.jpg` [#72], `img_036.jpg` [#479], `img_040.jpg` [#520]; `thua`: 4 lỗi tại `img_012.jpg` [#189, #190], `img_034.jpg` [#445], `img_035.jpg` [#468]; `bo_sot`: 1 lỗi tại `img_003.jpg`; `sai_thuoc_tinh`: 1 lỗi tại `img_040.jpg` [#514]).
- Một lỗi máy chặn được và một lỗi cần người đọc ảnh: Máy (`validate_labels.py` / `qc_compare.py`) chặn được box `< 20 px`, tọa độ vượt khung ảnh, thiếu thuộc tính và chênh lệch IoU/lớp giữa các người gán; người đọc ảnh (`105` ở Pha 6) phát hiện lỗi ngữ nghĩa trên bản đã gộp như gán `car` lên đám đông khán giả/người chờ xe ở `img_019.jpg` và `img_032.jpg` hoặc gộp 2 xe đạp vào một box ở `img_003.jpg`.

## 5. Quyết định bàn giao

- Quyết định: Rework
- Lý do, người nhận và phạm vi sử dụng: Khớp `submission/qc/reviews/round-01/qc_result.json` (`REWORK`): tỉ lệ ảnh lỗi `10/40 = 25.0%` vượt ngưỡng `error_rate_img_max = 2%` trong `taxonomy.json`. Người nhận (nhóm sửa calibration & coach) chỉ dùng bản `labels_calibrated.json` hiện tại làm cơ sở rà sửa vòng 2 trên 10 ảnh lỗi, chưa dùng để huấn luyện mô hình chính thức.
- Nếu còn việc: Sửa lại 21 lỗi trên 10 ảnh (`img_001`, `img_003`, `img_005`, `img_012`, `img_019`, `img_032`, `img_034`, `img_035`, `img_036`, `img_040`) — người xử lý: `103` (sửa CVAT theo `guideline_v1.1_calibrated.md`), `101` (chạy lại `build_final.py`), `105` (mở `--round 2` duyệt lại và chạy `qc_report.py --round 2`).
- Kiểm ngược từ báo cáo tới ảnh, rule và nhãn: Truy ngược 3 ca đại diện từ `submission/qc/reviews/round-01/qc_review.csv`: (1) `img_001.jpg` object `#7` (`truck`, bbox `[230.37, 100.36, 729.63, 539.64]` trong `submission/qc/labels_calibrated.json`) đối chiếu `R-05` trong `submission/guidelines/guideline_v1.1_calibrated.md` → lỗi `sai_vi_tri_kich_thuoc` (box dư sang trái tới `x=230` dù thân pickup bắt đầu khoảng `x=386`, bao qua người đi bộ); (2) `img_003.jpg` object `#48` (`bicycle`, bbox `[0.0, 227.06, 422.65, 371.13]`) đối chiếu `R-04` & `R-05` → gộp 2 xe đạp thành 1 box kèm 1 `bo_sot` cho xe đạp ở giữa; (3) `img_019.jpg` object `#241` (`car`, bbox `[298.76, 147.55, 63.63, 173.79]`) đối chiếu mục 2.1 taxonomy `car` → `sai_lop` (box đặt lên người trong đám đông sau hàng rào).

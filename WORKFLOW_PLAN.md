# Kế hoạch phân công, Ma trận Gantt & Checklist — Nhóm `Try_hard` (5 thành viên)

> **Quy tắc chung:**
> - Nhóm **5 thành viên** (`101`–`105`): hệ thống tự động chia **K = 3 người/ảnh** (40 ảnh $\times$ 3 = 120 lượt $\to$ **24 ảnh/người**).
> - Mọi file cá nhân, nhiệm vụ trên CVAT và phiếu kiểm chỉ dùng **mã thành viên 3 chữ số** (`101`, `102`, `103`, `104`, `105`). Tuyệt đối **không** ghi họ tên, MSHV, email hay tài khoản vào tên file hoặc tiêu đề tài liệu. Không commit thư mục `work/`.

---

## 1. Phân công vai trò (5 thành viên)

| Mã | Vai trò | Nhiệm vụ trọng tâm |
|---|---|---|
| **`101`** *(Lead)* | **Team Lead & Pipeline Owner** | Điều phối tiến độ & GitHub, chạy chia ảnh (`split_assign.py`), gán 24 ảnh, điều phối chốt Calibration, chạy `build_final.py`, hoàn thiện `QC_REPORT.md`, chạy `check_submission.py` & nộp bài. |
| **`102`** | **Taxonomy & CVAT Setup Lead** | Phụ trách `submission/taxonomy.json` + `make_cvat_labels.py` (kiểm schema, sinh `cvat/labels_generated.json`, kiểm cấu hình Project CVAT), gán 24 ảnh, đối chiếu thống kê sau sửa. |
| **`103`** | **Guideline Architect & Calibration Fixer** | Biên soạn `guideline_v1.0_agreed.md` (mã quy tắc `R-01..Rn`) & `guideline_v1.1_calibrated.md` + `guideline_change.json`, gán 24 ảnh, chạy `prepare_calibration.py`, sửa box trong CVAT & điền `calibration_decisions.csv`. |
| **`104`** | **IAA & Disagreement Analyst** | Thu thập `image_issues.csv`, chạy `validate_labels.py --all`, `qc_compare.py`, `viz_disagreement.py --images flagged` (Pha 4), phân tích `iaa_summary.md` & ảnh chồng nhãn, đối chiếu ca tham chiếu Coach, gán 24 ảnh. |
| **`105`** | **Independent QC Reviewer** | Tham gia phản biện ca biên ở Pha 2, kiểm lại chéo (`kiem_lai`) các box `103` sửa ở Pha 5, **độc lập thực hiện toàn bộ Pha 6** (`qc_sample.py`, điền `qc_review.csv`, chạy `qc_report.py`, chốt Hold/Rework/Release), gán 24 ảnh. |

---

## 2. Ma trận Gantt tiến độ 4 giờ (0:00 – 4:00)

| Mốc / Pha | Thời gian | `101` (Lead) | `102` (Taxonomy) | `103` (Guideline/Fixer) | `104` (IAA Analyst) | `105` (QC Reviewer) |
|---|---|---|---|---|---|---|
| **CP0 — Chuẩn bị** | `0:00 – 0:15` (15') | Cài env, `TEAM.md`, cấp quyền repo | Cài env, test CVAT, mã `102` | Cài env, test CVAT, mã `103` | Cài env, test CVAT, mã `104` | Cài env, test CVAT, mã `105` |
| **CP1 — Guideline cá nhân** | `0:15 – 0:50` (35') | Đọc 40 ảnh $\to$ `guideline_101.md` | Đọc 40 ảnh $\to$ `guideline_102.md` | Đọc 40 ảnh $\to$ `guideline_103.md` | Đọc 40 ảnh $\to$ `guideline_104.md` | Đọc 40 ảnh $\to$ `guideline_105.md` |
| **CP2 — Chốt đặc tả v1.0** | `0:50 – 1:20` (30') | Chủ trì họp chốt `v1.0`, khóa nhánh | Sửa `taxonomy.json`, chạy `make_cvat_labels.py` | Ghi `guideline_v1.0_agreed.md` (`R-xx`) | Đề xuất xử lý ca mơ hồ & `ignore_region` | Phản biện phép thử đồng nhất số box |
| *Nghỉ giải lao* | `1:20 – 1:30` (10') | Nghỉ + chạy sẵn `split_assign.py` | Nghỉ | Nghỉ | Nghỉ | Nghỉ |
| **CP3 — Gán độc lập** *(24 ảnh/người, K=3)* | `1:30 – 2:30` (60') | Gán 24 ảnh (`101`) + `validate_labels` | Gán 24 ảnh (`102`) + `validate_labels` | Gán 24 ảnh (`103`) + `validate_labels` | Gán 24 ảnh (`104`) + `validate_labels` | Gán 24 ảnh (`105`) + `validate_labels` |
| **CP4 — Đo IAA** | `2:30 – 2:50` (20') | Gom 5 file JSON, khóa bản độc lập | Đối chiếu cấu trúc thuộc tính export | Lập danh sách ca từ `calibration_candidates.csv` | Chạy `validate --all`, `qc_compare`, `viz_disagreement` | Soi ca tham chiếu Coach, tìm lỗi cả nhóm lệch |
| **CP5 — Calibration** | `2:50 – 3:30` (40') | Họp chốt ca & chạy `build_final.py` | Chạy `make_cvat_labels --final` (nếu đổi schema) | Chạy `prepare_calibration`, sửa box, viết `v1.1` | Hỗ trợ xem `work/viz/` khi họp | Kiểm lại chéo (`kiem_lai`) ca đã sửa |
| **CP6 — QC & Nộp** | `3:30 – 4:00` (30') | Viết `QC_REPORT.md`, `note_101`, `check_submission` | Viết `individual_notes/102.md` | Viết `individual_notes/103.md` | Viết `individual_notes/104.md` | Chạy `qc_sample`, điền `qc_review.csv`, `qc_report` + `note_105` |

---

## 3. Checklist công việc chi tiết theo từng người

### Checklist của `101` (Team Lead)
- [ ] **Pha 0:** Cài `python -m pip install -r requirements.txt`; cấp quyền repo Private cho 4 thành viên (`102`–`105`) và Coach; kiểm tra `TEAM.md`.
- [ ] **Pha 1:** Xem đủ 40 ảnh trong `data/images/`, viết `submission/guidelines/individual/guideline_101.md` và gửi riêng trước khi họp.
- [ ] **Pha 2:** Điều phối họp 30' chốt `v1.0`; chạy lệnh chia ảnh cho đúng 5 người (hệ thống tự động đặt `K=3`, mỗi người **24 ảnh**):
  ```bash
  python scripts/split_assign.py --members 101,102,103,104,105 --copy-images
  ```
- [ ] **Pha 3:** Tạo Task CVAT từ `cvat/labels_generated.json`, upload đúng 24 ảnh trong `work/101/images/`, gán độc lập $\to$ Export COCO 1.0 lưu vào `submission/labels/101.json` $\to$ chạy `python scripts/validate_labels.py submission/labels/101.json`.
- [ ] **Pha 4:** Gom đủ 5 file `101.json`–`105.json` sau khi mọi người đã khóa nhãn riêng.
- [ ] **Pha 5:** Điều phối chốt quyết định calibration; sau khi `103` sửa nhãn xong và `105` kiểm lại, chạy `python scripts/build_final.py` để xuất `labels_calibrated.json`, `labels_calibrated.csv`, `batch_manifest.json`.
- [ ] **Pha 6:** Viết `submission/individual_notes/101.md`; điền `submission/qc/QC_REPORT.md` từ mẫu; chạy `python scripts/check_submission.py --round 1` (đảm bảo `0 lỗi`); đảm bảo không nộp `work/` và gửi link/commit chốt bài cho Coach.

---

### Checklist của `102` (Taxonomy & CVAT Setup Lead)
- [ ] **Pha 0:** Cài `python -m pip install -r requirements.txt`, đăng nhập được CVAT, kiểm tra mở được ảnh.
- [ ] **Pha 1:** Đọc 40 ảnh (chú ý các lớp lạ: tuk-tuk, xe chở xe, tượng, ma-nơ-canh) $\to$ viết `submission/guidelines/individual/guideline_102.md`.
- [ ] **Pha 2:** Sửa `submission/taxonomy.json` khớp bản họp nhóm (đặt `version: "v1.0"`, **giữ nguyên mục `rules`**) $\to$ chạy `python scripts/make_cvat_labels.py` để kiểm schema và sinh `cvat/labels_generated.json` hướng dẫn cả nhóm import vào CVAT.
- [ ] **Pha 3:** Gán độc lập 24 ảnh trong `work/102/images/` $\to$ export COCO 1.0 vào `submission/labels/102.json` $\to$ chạy `python scripts/validate_labels.py submission/labels/102.json` $\to$ push.
- [ ] **Pha 4:** Kiểm tra sau export xem cả 5 file JSON có bị mất thuộc tính `occluded_level`, `truncated`, `can_review` không.
- [ ] **Pha 5:** Nếu nhóm có đổi taxonomy ở `v1.1`, cập nhật `submission/taxonomy_v1.1.json` và chạy `python scripts/make_cvat_labels.py --final`.
- [ ] **Pha 6:** Viết `submission/individual_notes/102.md` và push lên repo.

---

### Checklist của `103` (Guideline Architect & Calibration Fixer)
- [ ] **Pha 0:** Cài `python -m pip install -r requirements.txt`, đăng nhập CVAT.
- [ ] **Pha 1:** Đọc 40 ảnh, ghi lại mã ảnh `img_0xx` kèm ca khó $\to$ viết `submission/guidelines/individual/guideline_103.md`.
- [ ] **Pha 2:** Làm thư ký họp nhóm, biên soạn `submission/guidelines/guideline_v1.0_agreed.md` (gắn mã `R-01`, `R-02`… cho các quy tắc trọng tâm).
- [ ] **Pha 3:** Gán độc lập 24 ảnh trong `work/103/images/` $\to$ export COCO 1.0 vào `submission/labels/103.json` $\to$ chạy `python scripts/validate_labels.py submission/labels/103.json` $\to$ push.
- [ ] **Pha 4:** Đọc `calibration_candidates.csv` để chuẩn bị danh sách ca cần thảo luận ở Pha 5.
- [ ] **Pha 5 (Trọng tâm):**
  - Chạy `python scripts/build_final.py` lần đầu để tạo file mẫu `submission/qc/calibration_decisions.csv`.
  - Chạy `python scripts/prepare_calibration.py`, sửa ảnh trên CVAT theo quyết định nhóm $\to$ export COCO đúng đường dẫn `labels_calibrated_<phiên-bản>.json`.
  - Điền đầy đủ các cột trong `calibration_decisions.csv` (ghi rõ `nguoi_chot`, `phieu_sua`, `pham_vi=anh/lo`).
  - Lưu `submission/guidelines/guideline_v1.1_calibrated.md` và `submission/guideline_change.json` (`kind: "editorial"` hoặc `"semantic"`).
- [ ] **Pha 6:** Viết `submission/individual_notes/103.md` và sửa bổ sung ngay nếu `105` trả về `Rework`.

---

### Checklist của `104` (IAA & Disagreement Analyst)
- [ ] **Pha 0:** Cài `python -m pip install -r requirements.txt`, đăng nhập CVAT.
- [ ] **Pha 1:** Đọc 40 ảnh, tập trung vào trường hợp che khuất, xe chồng lấn và vật nhỏ $\to$ viết `submission/guidelines/individual/guideline_104.md`.
- [ ] **Pha 2:** Đóng góp ý kiến vào bản `guideline_v1.0_agreed.md`, thống nhất ngưỡng tối thiểu kích thước box và xử lý `ignore_region`.
- [ ] **Pha 3:** Gán độc lập 24 ảnh trong `work/104/images/` $\to$ export COCO 1.0 vào `submission/labels/104.json` $\to$ chạy `python scripts/validate_labels.py submission/labels/104.json` $\to$ push.
- [ ] **Pha 4 (Trọng tâm):**
  - Thu thập các ghi chú chưa vẽ được box của nhóm vào `submission/qc/image_issues.csv` (cột `image,annotator,region,note`).
  - Chạy bộ 3 lệnh đo IAA và tạo ảnh chồng nhãn:
    ```bash
    python scripts/validate_labels.py --all
    python scripts/qc_compare.py
    python scripts/viz_disagreement.py --images flagged
    ```
  - Đọc `submission/qc/iaa_summary.md` và mở `work/viz/` để phân loại rõ: bất đồng do **guideline mơ hồ** (cả 3 cùng lệch) hay do **1 người gán sai**.
- [ ] **Pha 5:** Chiếu các ảnh `work/viz/` trong cuộc họp để cả nhóm chốt quyết định nhanh.
- [ ] **Pha 6:** Viết `submission/individual_notes/104.md` và push lên repo.

---

### Checklist của `105` (Independent QC Reviewer)
- [ ] **Pha 0:** Cài `python -m pip install -r requirements.txt`, đăng nhập CVAT.
- [ ] **Pha 1:** Đọc 40 ảnh, tập trung vào các ca ngoài scope (tượng, ma-nơ-canh, phản chiếu gương, biển quảng cáo) $\to$ viết `submission/guidelines/individual/guideline_105.md`.
- [ ] **Pha 2:** Kiểm tra chéo bản `guideline_v1.0_agreed.md` bằng phép thử: *"cùng 1 ảnh người đi xe máy, 5 người có dự đoán cùng số box không?"*.
- [ ] **Pha 3:** Gán độc lập 24 ảnh trong `work/105/images/` $\to$ export COCO 1.0 vào `submission/labels/105.json` $\to$ chạy `python scripts/validate_labels.py submission/labels/105.json` $\to$ push.
- [ ] **Pha 4:** Sau khi khóa nhãn độc lập, đối chiếu ca tham chiếu do Coach cung cấp để tìm tình huống cả nhóm làm giống nhau nhưng vẫn sai quy tắc.
- [ ] **Pha 5:** Kiểm tra lại chéo (`kiem_lai`) các box mà `103` đã sửa trong bước calibration và xác nhận kết quả vào `calibration_decisions.csv`.
- [ ] **Pha 6 (Trọng tâm):** Đóng vai **Independent QC Reviewer**:
  - Chạy `python scripts/qc_sample.py --round 1`.
  - Mở toàn bộ ảnh overlay trong `work/review/round-01/`, đối chiếu với ảnh gốc & guideline `v1.1`, điền đủ phiếu `submission/qc/reviews/round-01/qc_review.csv` (`reviewer=105`, dùng đúng mã lỗi `khong`, `thua`, `sai_lop`, `sai_vi_tri_kich_thuoc`, `sai_thuoc_tinh`, `bo_sot`).
  - Chạy `python scripts/qc_report.py --round 1` và xác nhận quyết định cuối cùng (**Hold $\to$ Rework $\to$ Release**).
  - Viết `submission/individual_notes/105.md` và kiểm thử khả năng truy vết ngược 1 ca từ `QC_REPORT.md` tới ảnh, rule và nhãn.

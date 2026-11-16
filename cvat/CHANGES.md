# Ghi chép thay đổi bộ nhãn CVAT

## 2026-11-16 — taxonomy v1.0 (người sửa: 102)

### 1. Sinh bộ nhãn từ taxonomy v1.0

- `submission/taxonomy.json` lên `v1.0`: thêm 3 lớp `bus`, `tuk_tuk`, `bicycle` (tổng 9 lớp, gồm `ignore_region`);
  định nghĩa lớp khớp `submission/guidelines/guideline_v1.0_agreed.md`. Mục `attributes` và `rules` **giữ nguyên**.
- Chạy `python scripts/make_cvat_labels.py` → schema hợp lệ, sinh `cvat/labels_generated.json` (taxonomy v1.0).

### 2. Sửa tay `cvat/labels_generated.json` cho thuộc tính checkbox

**Lỗi:** dán file vào CVAT (Labels → Raw) bị từ chối:

```
Attribute: "can_review": attribute values must be a non-empty array
```

**Nguyên nhân:** `scripts/make_cvat_labels.py` (dòng 34) luôn ghi `"values": []` cho thuộc tính kiểu `checkbox`,
trong khi CVAT yêu cầu mảng `values` khác rỗng; quy ước của CVAT cho checkbox là `values = [giá trị mặc định]`.

**Cách sửa:** sửa trực tiếp file sinh ra, **không** sửa script và **không** sửa `taxonomy.json`
(schema `schemas/taxonomy.schema.json` bắt buộc checkbox có `values` rỗng):

```diff
         "input_type": "checkbox",
         "default_value": "false",
-        "values": []
+        "values": ["false"]
```

Áp dụng cho `can_review` ở cả 8 lớp có thuộc tính (`ignore_region` không có thuộc tính nên không đổi).
Tên lớp, màu, thuộc tính `occluded_level` / `truncated` không đổi.

**Kiểm lại:**

- `cvat/labels_generated.json` không còn mảng `"values": []` nào.
- Dán lại vào CVAT (Labels → Raw) không còn báo lỗi.
- `python -m unittest discover -s tests` (với `PYTHONUTF8=1`): 19/20 OK; test còn lại lỗi do Windows không có quyền tạo symlink
  (`WinError 1314`), không liên quan thay đổi này.

**Lưu ý:** nếu chạy lại `make_cvat_labels.py` (hoặc `--final` ở Pha 5), file sinh ra sẽ quay về `"values": []`;
phải sửa tay lại như trên trước khi dán vào CVAT.

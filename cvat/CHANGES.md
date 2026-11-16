# Ghi chép thay đổi bộ nhãn CVAT

## 2026-11-16 — taxonomy v1.0 (người sửa: 102)

### 1. Sinh bộ nhãn từ taxonomy v1.0

- `submission/taxonomy.json` lên `v1.0`: thêm 3 lớp `bus`, `tuk_tuk`, `bicycle` (tổng 9 lớp, gồm `ignore_region`);
  định nghĩa lớp khớp `submission/guidelines/guideline_v1.0_agreed.md`. Mục `attributes` và `rules` **giữ nguyên**.
- Chạy `python scripts/make_cvat_labels.py` → schema hợp lệ, sinh `cvat/labels_generated.json` (taxonomy v1.0).

### 2. Sửa `scripts/make_cvat_labels.py` cho thuộc tính checkbox

**Lỗi:** dán file vào CVAT (Labels → Raw) bị từ chối:

```
Attribute: "can_review": attribute values must be a non-empty array
```

**Nguyên nhân:** `scripts/make_cvat_labels.py` (dòng 34) luôn ghi `"values": []` cho thuộc tính kiểu `checkbox`,
trong khi CVAT yêu cầu mảng `values` khác rỗng; quy ước của CVAT cho checkbox là `values = [giá trị mặc định]`.

**Cách sửa:** sửa script để checkbox nhận `values = [default]`. **Không** sửa `taxonomy.json`
(schema `schemas/taxonomy.schema.json` bắt buộc checkbox trong taxonomy có `values` rỗng; script chỉ đổi định dạng khi xuất cho CVAT).

```diff
-                    "values": [str(v) for v in a.get("values", [])] if a["type"] == "select" else [],
+                    # CVAT yêu cầu checkbox có values khác rỗng: quy ước là [default] (vd. ["false"]).
+                    "values": ([str(v) for v in a.get("values", [])] if a["type"] == "select"
+                               else [str(a.get("default", "false"))]),
```

Kết quả trong `cvat/labels_generated.json`: `can_review` ở cả 8 lớp có thuộc tính đổi `"values": []` → `"values": ["false"]`
(`ignore_region` không có thuộc tính nên không đổi). Tên lớp, màu, `occluded_level`, `truncated` không đổi.
Lần đầu (commit `1b5bb40`) file này được sửa tay; sau khi sửa script, chạy lại `make_cvat_labels.py` cho ra file **giống hệt** bản sửa tay.
Lệnh `--final` ở Pha 5 dùng cùng đoạn code nên `cvat/labels_final.json` cũng đúng định dạng.

### 3. Thêm `"type": "rectangle"` cho mỗi nhãn

**Lỗi:** sau bước 2, CVAT báo tiếp:

```
Label "car": unknown label type "undefined"
```

**Nguyên nhân:** CVAT bản đang dùng bắt buộc mỗi nhãn có trường `type`; script không ghi trường này.

**Cách sửa:** script ghi thêm `"type": "rectangle"` cho cả 9 nhãn (lab chỉ vẽ box chữ nhật):

```diff
-                       "attributes": attrs})
+                       # CVAT bản mới bắt buộc có "type"; lab chỉ dùng box chữ nhật.
+                       "attributes": attrs, "type": "rectangle"})
```

Sau khi sửa, `cvat/labels_generated.json` **giống hệt** `cvat/labels_local.json` (commit `1537b95`, một thành viên đã sửa tay
cho CVAT Local), so sánh bằng `json.load(a) == json.load(b)` → `True`.

**Kiểm lại (sau cả bước 2 và 3):**

- Chạy lại `python scripts/make_cvat_labels.py`: schema hợp lệ, `cvat/labels_generated.json` không còn mảng `"values": []` nào.
- Dán lại vào CVAT (Labels → Raw) không còn báo lỗi.
- `python -m unittest discover -s tests` (với `PYTHONUTF8=1`): 19/20 OK, kể cả 2 test của `make_cvat_labels`; test còn lại lỗi do
  Windows không có quyền tạo symlink (`WinError 1314`), không liên quan thay đổi này.

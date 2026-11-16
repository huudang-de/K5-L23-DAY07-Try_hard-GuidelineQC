# Guideline gán nhãn bounding box 2D — v1.0 — ảnh giao thông

- Phiên bản: v1.0-draft
- Ngày: 2026-11-16
- Người viết / duyệt: Bản nháp; chờ thành viên ghi mã và reviewer xác nhận
- Phạm vi: 40 ảnh `data/images/img_001.jpg`–`img_040.jpg`; box 2D, gán bằng CVAT, xuất COCO 1.0

## 1. Mục tiêu

Gán nhãn người và phương tiện để huấn luyện, đánh giá mô hình phát hiện đối tượng trên ảnh giao thông. Bỏ sót người hoặc phân loại sai phương tiện có thể làm sai cảnh báo va chạm và thống kê giao thông; box lệch hoặc nhãn không nhất quán làm giảm độ tin cậy của mô hình. Vì vậy, chỉ gán đối tượng thuộc taxonomy, vẽ box theo cùng một quy ước, và đánh dấu ca chưa chắc chắn để reviewer xử lý.

## 2. Lớp và thuộc tính

### Lớp

| Lớp | Định nghĩa (một câu) | Ví dụ trong bộ ảnh | Phản ví dụ / dễ nhầm với |
|---|---|---|---|
| `car` | Ô tô con chở người, gồm sedan, hatchback, SUV, coupe và minivan gia đình. | Ô tô chở trên xe chuyên dụng (`img_002`); ô tô trong phố (`img_015`, `img_031`). | Xe bán tải có khoang chở hàng (`truck`); xe van chở hàng; tuk-tuk (`tuk_tuk`). |
| `truck` | Xe cơ giới chở hàng có khoang hàng tách biệt, gồm xe tải, xe đầu kéo, xe chở ô tô và xe bán tải. | Xe chở ô tô (`img_002`); xe bán tải (`img_001`, `img_037`); xe tải nhỏ (`img_036`). | Xe van thân liền (`van`); xe tải đồ chơi (`img_038`). |
| `van` | Xe chở người hoặc hàng có thân dạng khoang kín liền khối, không tách cabin khỏi khoang sau. | Không có ví dụ đủ rõ để làm ca tham chiếu trong bộ ảnh. | Xe bán tải gắn nắp thùng (`truck`); xe tải có thùng riêng (`truck`); xe khách lớn (`bus`). |
| `bus` | Xe chuyên chở khách cỡ lớn, sức chứa trên 16 người, có thân dài và nhiều cửa lên xuống. | Xe buýt trên phố (`img_009`, `img_031`). | Xe van hoặc xe khách nhỏ không đạt ngưỡng trên. |
| `tuk_tuk` | Phương tiện cơ giới ba bánh chuyên chở khách hoặc hàng, gồm auto-rickshaw và xe lam. | Tuk-tuk (`img_004`, `img_016`, `img_035`). | Xe đạp/xích lô đạp; xe máy hai bánh (`motorcycle`). |
| `motorcycle` | Phương tiện gắn động cơ có hai bánh, gồm xe máy, xe tay ga, mô tô, xe máy điện và mô tô có sidecar. | Người đi xe máy trong phố (`img_005`, `img_021`, `img_026`). | Xe đạp và xích lô đạp; tuk-tuk. |
| `person` | Người thật còn sống ở mọi tư thế, gồm người đi bộ, người lái xe, hành khách và người mặc trang phục hóa trang. | Người bán hàng (`img_003`); người đi xe máy (`img_005`); người chờ xe (`img_032`). | Tượng (`img_017`), hình in trên biển (`img_018`), ma-nơ-canh (`img_033`), búp bê (`img_038`). |
| `ignore_region` | Vùng đánh dấu nơi không cần tính lỗi bỏ sót từng đối tượng vì vật quá nhỏ hoặc không thể tách riêng đáng tin cậy. | Cụm giao thông/đám đông quá nhỏ, dày đặc ở xa (`img_008`, `img_021`). | Không dùng thay cho box đối tượng nhìn rõ và cao ít nhất 20 px. |

Vật không thuộc lớp nào thì không gán. Không thêm lớp mới. Xe đạp, xích lô đạp, động vật, hàng hóa rời và cơ sở hạ tầng không thuộc taxonomy; người đi hoặc ngồi trên phương tiện phi cơ giới vẫn được gán `person`. Tượng, hình ảnh in, ma-nơ-canh, đồ chơi và phản chiếu không được gán. Chỉ dùng `ignore_region` theo quy tắc R-12, không dùng làm nhãn chung cho vật ngoài taxonomy. Minivan gia đình xếp `car`; xe van dạng thân liền dùng để chở hàng hoặc chuyên chở khách xếp `van`; khi không phân biệt được mục đích/kiểu thân thì bật `can_review`.

### Thuộc tính

| Thuộc tính | Mức cố định | Ngưỡng / ý nghĩa |
|---|---|---|
| `occluded_level` | `0`, `1`, `2`, `3` | Ước lượng phần diện tích đối tượng bị vật khác che: `0` = 0%; `1` = trên 0% đến 30%; `2` = trên 30% đến 70%; `3` = trên 70% đến 90%. Nếu bị che trên 90% và không thể nhận diện chắc chắn thì không gán box. |
| `truncated` | `no`, `yes` | `yes` nếu đối tượng tiếp tục ra ngoài ảnh và bị một cạnh ảnh cắt mất một phần; ngược lại `no`. Chỉ chạm cạnh nhưng không bị cắt thì `no`. |
| `can_review` | bật / tắt | Bật khi chưa chắc về danh tính, lớp hoặc biên box; nếu không có băn khoăn thì tắt. Ghi thêm vấn đề cấp ảnh khi không thể tạo box hợp lệ. |

## 3. Quy tắc vẽ box

- **R-01 — Tách người và xe:** Vẽ box riêng cho từng người và từng phương tiện; box người ôm phần thân người nhìn thấy, box xe ôm phần xe nhìn thấy. Hai box có thể chồng lấn. Người lái và hành khách đều là `person`.
- **R-02 — Vật mô phỏng và phản chiếu:** Không gán tượng, ảnh in, ma-nơ-canh, đồ chơi hoặc hình phản chiếu (`img_017`, `img_018`, `img_020`, `img_033`, `img_038`).
- **R-03 — Xe chở xe:** Gán một box `truck` cho xe chở và box `car` riêng cho từng ô tô trên sàn nếu chiều cao từng xe đạt ít nhất 20 px (`img_002`).
- **R-04 — Phương tiện phi cơ giới:** Không gán xe đạp hoặc xích lô; vẫn gán riêng người đạp và hành khách (`img_006`, `img_023`, `img_034`, `img_039`).
- **R-05 — Tuk-tuk:** Gán `tuk_tuk` cho xe ba bánh có động cơ; box bao phần xe nhìn thấy, không bao người hoặc hàng hóa rời (`img_004`, `img_035`).
- **R-06 — Xe buýt:** Gán `bus` khi sức chứa trên 16 người; không suy lớp chỉ từ màu sơn hoặc tên xe.
- **R-07 — Hàng hóa:** Box xe chỉ bao phương tiện, không kéo lên theo hàng hóa chất cao trên xe (`img_029`, `img_035`).
- **R-08 — Xe bán tải:** Gán pickup là `truck`, kể cả khi có nắp thùng/canopy (`img_001`, `img_037`).
- **R-09 — Nhòe và thiếu sáng:** Nếu còn nhận diện chắc chắn được lớp và ranh giới, gán theo phần nhìn thấy và bật `can_review` khi biên khó xác định. Nếu chỉ còn vệt nhòe không thể nhận diện, không đoán lớp; ghi vấn đề ảnh hoặc dùng `ignore_region` chỉ khi đúng điều kiện R-12.
- **R-10 — Hóa trang:** Người thật mặc mascot/trang phục hóa trang vẫn là `person` (`img_011`).
- **R-11 — Xe chủ và gương:** Không gán phần xe gắn camera/ego-vehicle hoặc hình trong gương chiếu hậu (`img_020`).
- **R-12 — Vật nhỏ và vùng ignore:** Chỉ gán box riêng khi chiều cao box ít nhất 20 px. Đối tượng nhỏ hơn ngưỡng hoặc đám đông/xe quá dày không thể tách đáng tin cậy được gom trong vùng `ignore_region`; không dùng vùng này để bỏ qua đối tượng rõ ở tiền cảnh.
- **R-13 — Độ khít và mép ảnh:** Box chữ nhật ôm sát biên ngoài của phần nhìn thấy, không bao khoảng trống không thuộc đối tượng. Box nằm trong ảnh: `0 ≤ x`, `0 ≤ y`, `x + width ≤ W`, `y + height ≤ H`. Gắn `truncated=yes` khi cạnh ảnh thực sự cắt mất đối tượng.

## 4. Ca khó

| Ảnh | Tình huống | Rule ID | Quyết định | Lý do |
|---|---|---|---|---|
| `img_002` | Xe chuyên dụng chở nhiều ô tô | R-03, R-13 | Một `truck` cho xe chở; từng `car` riêng nếu cao ≥20 px; vẽ trên phần nhìn thấy. | Xe chở và mỗi xe con là đối tượng riêng. |
| `img_003` | Người bán hàng cạnh xe đạp và hàng hóa | R-01, R-04 | Gán `person`; không gán xe đạp, rổ hàng hoặc cân. | Chỉ người thuộc taxonomy. |
| `img_005` | Nhiều người đi xe máy, người và xe chồng lấn | R-01, R-13 | Tách từng người và từng xe máy; gán occlusion theo diện tích bị che. | Không gộp người vào box xe. |
| `img_006` | Người lái và khách trên xích lô giữa giao thông | R-01, R-04 | Gán từng người; không gán xích lô đạp. | Xích lô phi cơ giới nằm ngoài taxonomy. |
| `img_008` | Chợ đêm đông xe máy và người | R-09, R-12 | Gán vật tách rõ, đủ 20 px; vùng xa quá nhỏ/dày dùng `ignore_region`. | Không bỏ qua đối tượng tiền cảnh có thể nhận diện. |
| `img_014` | Người qua đường và phương tiện bị nhòe chuyển động | R-09 | Chỉ gán nếu còn nhận diện được lớp; bật `can_review` khi biên không chắc. | Không suy đoán vật thể từ vệt nhòe. |
| `img_017` | Tượng người cưỡi ngựa | R-02 | Không gán `person` hay phương tiện. | Đây là tượng, không phải người/vật đang tham gia giao thông. |
| `img_020` | Ảnh đường phía sau xuất hiện trong gương xe | R-02, R-11 | Không gán xe chủ và các vật chỉ xuất hiện trong gương. | Loại ego-vehicle và phản chiếu. |
| `img_026` | Nhiều người cùng ngồi trên xe máy | R-01 | Gán từng người riêng và một box xe máy; box có thể chồng nhau. | Mỗi người là một đối tượng độc lập. |
| `img_028` | Pickup chở tượng ngựa | R-02, R-08 | Gán pickup là `truck`; không gán tượng ngựa. | Phân biệt phương tiện thật với vật mô phỏng. |
| `img_033` | Ma-nơ-canh trong cửa hàng | R-02 | Không gán `person`. | Không phải người thật. |
| `img_035` | Tuk-tuk chở hàng chất cao | R-05, R-07 | Gán `tuk_tuk`, không bao trọn hàng hóa vào box xe. | Box mô tả phương tiện, không phải tải hàng. |
| `img_038` | Mô hình xe và búp bê trong tủ kính | R-02 | Không gán xe hoặc `person`. | Đồ chơi/vật trưng bày không thuộc đối tượng mục tiêu. |
| `img_040` | Giao thông hỗn hợp gồm xe buýt, taxi, tuk-tuk và xe máy | R-01, R-05, R-06 | Phân lớp từng phương tiện; tách người nhìn thấy khỏi xe; vật <20 px theo R-12. | Đây là ca áp dụng nhiều lớp và quy tắc đồng thời. |

Nếu chưa chốt được lớp hoặc biên box, bật `can_review`; nếu không thể tạo box hợp lệ, ghi ảnh, vị trí và lý do vào danh sách vấn đề ảnh. Không tự tạo ngoại lệ cho một ảnh.

## 5. QC và phiên bản

- Trước khi nộp, kiểm tra mọi ảnh được giao đều có trong COCO; đúng tên lớp và giá trị thuộc tính; box không vượt kích thước ảnh; box riêng không thấp hơn 20 px; không còn ca chưa ghi nhận.
- Dùng ngưỡng QC trong `submission/taxonomy.json`: `min_box_height_px=20`, `match_iou=0.50`, `iou_thr_large=0.85`, `iou_thr_small=0.70` (đối tượng lớn/nhỏ xác định theo cạnh dài nhất ≥/<50 px), `iaa_iou=0.85`, `iaa_class=0.95`, `error_rate_img_max=0.02`. Ngưỡng gán nhãn không thay thế bước reviewer kiểm tra bất đồng.
- Reviewer nhóm giải quyết ca có `can_review` và bất đồng theo rule ID; ghi quyết định, người chốt và phạm vi ảnh/lô vào nhật ký calibration. Chỉ cập nhật guideline sau khi đối chiếu bằng chứng ảnh.

| Phiên bản | Thay đổi | Lý do (ca nào) |
|---|---|---|
| v1.0-draft | Làm rõ taxonomy, thuộc tính, box và ca biên từ 40 ảnh. | Khởi tạo theo `TEMPLATE_guideline.md`; cần nhóm/reviewer xác nhận trước khi khóa. |
| v1.1 | Chưa phát hành. | Chỉ cập nhật sau calibration; ghi các ca và quyết định vào đây. |

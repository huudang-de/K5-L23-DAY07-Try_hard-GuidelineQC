# Guideline gán nhãn — v0.1 (bản cá nhân) — `102`

- Phiên bản: v0.1-cá-nhân (dựa trên v0 của giảng viên)
- Ngày: 2026-11-16
- Người viết: `102` · Người duyệt: (nhóm, Pha 2)
- Loại nhãn: bounding box trên ảnh 2D, CVAT, xuất COCO 1.0
- Taxonomy dùng: `submission/taxonomy.json` (6 lớp, giữ nguyên mục `rules`)

---

## 1. Mục tiêu

Nhãn dùng để huấn luyện mô hình **phát hiện người và phương tiện tham gia giao thông thật** trên ảnh đường phố
(camera giao thông, hỗ trợ lái). Hệ quả cần tránh:

| Lỗi | Hậu quả | Mức nghiêm trọng |
|---|---|---|
| Bỏ sót người / xe thật (`bo_sot`) | Mô hình không thấy người đi bộ, người đi xe máy → nguy cơ va chạm | **Cao nhất** |
| Gán box cho vật *không thật* (tượng, poster, ma-nơ-canh, phản chiếu, đồ chơi) | Mô hình phanh/né "ảo" | Cao |
| Sai lớp giữa các phương tiện (`car`/`van`/`truck`) | Ước lượng kích thước, quỹ đạo sai | Trung bình |
| Box lệch / sai thuộc tính | Giảm độ chính xác định vị | Thấp – trung bình |

Nguyên tắc chung: **chỉ gán vật thật, thuộc đúng 1 trong 5 lớp vật, nhìn thấy đủ để nhận ra lớp.**
Ca không chắc → vẫn vẽ box và tick `can_review`, không bỏ trống.

---

## 2. Lớp và thuộc tính

### 2.1 Lớp

| Lớp | Định nghĩa (một câu) | Ví dụ (ảnh) | Phản ví dụ / dễ nhầm với |
|---|---|---|---|
| `car` | Xe 4 bánh chở người cỡ nhỏ, ≤ 9 chỗ, khoang hành khách liền khối: sedan, hatchback, SUV, crossover, wagon, taxi. | img_012 (Mazda SUV), img_023 (sedan đen), img_015, img_018, img_024, img_040 (taxi xanh) | Pickup có thùng hàng hở → `truck` (R-03). Minivan/microbus thùng kín cao → `van`. Ô tô đồ chơi, mô hình → không gán (R-10). |
| `van` | Xe thùng kín, mui cao liền thân, 10–16 chỗ hoặc chở hàng cỡ nhỏ (minivan cao, microbus kiểu Toyota Hiace). | img_005 (van xám 51LD), img_016 (van trắng) | Xe khách ≥ 17 chỗ → bus, ngoài taxonomy (R-04). SUV/MPV thấp → `car`. |
| `truck` | Xe chở hàng có **cabin và khoang/thùng hàng tách riêng**: xe tải, xe tải nhẹ, đầu kéo + rơ-moóc, **pickup** (kể cả pickup có nắp thùng). | img_001, img_012, img_028, img_037 (pickup), img_002 (xe chở xe), img_007 (Isuzu ELF + pickup), img_036 (xe tải nhẹ Veryca) | Van thùng kín liền cabin → `van`. Tượng pickup bằng đá → không gán (img_027). |
| `motorcycle` | Xe 2 bánh có động cơ: xe máy, xe tay ga, mô tô, kể cả **mô tô kèm thùng bên (sidecar)** hoặc kèm thùng/rơ-moóc nhỏ gắn liền. | img_006, img_008, img_010, img_021, img_026, img_029, img_022 (sidecar) | Xe đạp, xích lô (không động cơ) → không có lớp, **không gán xe** (người lái vẫn là `person`, R-07). Xe 3 bánh có mui (tuk-tuk, auto-rickshaw) → R-05. |
| `person` | Con người thật, nhìn thấy được (đi bộ, đứng, ngồi, đang lái/ngồi sau xe hở). | img_014, img_034, img_001, img_032, mọi người ngồi xe máy | Tượng (img_017), ma-nơ-canh (img_033), người trên biển quảng cáo (img_018), tranh tường (img_026), linh vật/đồ hóa trang không có người bên trong (img_019) → không gán (R-10). Người ngồi **trong xe kín** sau kính → không gán (R-08). |
| `ignore_region` | Vùng **không phải một vật**: chứa vật thật nhưng không thể/không cần gán từng cái; bỏ sót trong vùng này không tính lỗi. | Đám đông dày img_019, bãi xe máy dày img_030, cụm xe kẹt xa img_021, xe buýt / tuk-tuk (R-04, R-05) | Không dùng `ignore_region` để "che" vật mình lười gán khi vật đó tách được rõ. |

**Vật không thuộc lớp nào** (xe đạp, xích lô, xe buýt, tuk-tuk, động vật, cọc tiêu, biển báo…):

- Không thuộc giao thông / không nguy hiểm khi bỏ sót (xe đạp, xích lô, cọc tiêu, ngựa tượng): **không gán gì**.
- Là phương tiện có động cơ cỡ lớn mà taxonomy thiếu lớp (xe buýt, tuk-tuk/auto-rickshaw): vẽ **`ignore_region`** bao vật đó để mô hình không bị phạt khi phát hiện nhầm thành `truck`/`car`, đồng thời **đề xuất thêm lớp** `bus`, `three_wheeler` trong họp Pha 2 (xem mục 5).

### 2.2 Thuộc tính

**`occluded_level`** — tỉ lệ diện tích *ước lượng của toàn bộ vật* (phần trong khung ảnh) bị vật khác che. Phần bị cắt mép ảnh **không** tính vào đây (tính ở `truncated`).

| Mức | Ý nghĩa | Ngưỡng |
|---|---|---|
| `0` | Thấy rõ | bị che < 10% |
| `1` | Che ít | 10% – < 40% |
| `2` | Che nhiều | 40% – 80% |
| `3` | Che gần hết / không ước lượng được kích thước thật | > 80%, **hoặc** không đoán được vật kéo dài tới đâu |

- Người ngồi trên xe máy **có** tính là vật che xe máy (và ngược lại xe che chân người).
- Hàng rào song sắt/lưới thưa (img_018): chỉ tính phần bị thanh rào che thật → thường mức `1`.
- Kính trong suốt (người sau kính xe hở như tuk-tuk) không tính là che.

**`truncated`** — vật bị mép ảnh cắt.

| Giá trị | Khi nào |
|---|---|
| `no` | Toàn bộ vật nằm trong khung. |
| `yes` | Một phần vật nằm ngoài khung ảnh (box chạm mép ảnh, cách mép ≤ 2 px, và vật rõ ràng còn tiếp ra ngoài). |

**`can_review`** — tick khi: (a) không chắc lớp, (b) không chắc vật có thật không (hóa trang/tượng), (c) không chắc biên box do mờ chuyển động/tối, (d) ca đã ghi là "cần hỏi reviewer" ở mục 4.

---

## 3. Quy tắc vẽ box

| ID | Quy tắc |
|---|---|
| **R-01** | **Box ôm sát phần nhìn thấy** của vật (không đoán phần bị che, không đoán phần ngoài mép ảnh). Mép box chạm điểm ngoài cùng nhìn thấy, sai số ≤ 2 px. |
| **R-02** | **Kích thước tối thiểu:** chỉ gán khi chiều cao box ≥ **20 px** (`min_box_height_px`). Vật nhỏ hơn: bỏ; nếu là cả một cụm nhỏ dày → có thể gộp vào 1 `ignore_region`. |
| **R-03** | **Pickup = `truck`** (có thùng hàng tách khỏi cabin), kể cả có nắp thùng (img_037) hay thùng chở đồ (img_012). |
| **R-04** | **Xe buýt / xe khách ≥ 17 chỗ** (img_009, img_031, img_037, img_040): taxonomy chưa có lớp → `ignore_region` bao xe, tick `can_review`. Không gán `truck` hay `van`. |
| **R-05** | **Xe 3 bánh có mui chở khách (tuk-tuk, auto-rickshaw)** (img_004, img_016, img_035, img_040): → `ignore_region` bao toàn bộ xe **kèm người ngồi trong/lái** (không gán `person` riêng bên trong). Đề xuất lớp mới ở Pha 2. |
| **R-06** | **Người đi xe máy = tách box:** mỗi người một box `person` (người lái và từng người ngồi sau, kể cả trẻ em kẹp giữa nếu thấy ≥ 20 px), **cộng** một box `motorcycle` cho xe. Box `person` bao từ đỉnh mũ bảo hiểm tới chân thấp nhất nhìn thấy (gồm cả phần chồng lên xe). Box `motorcycle` bao từ bánh xe tới tay lái/kính chắn gió, **không** kéo lên đầu/thân người. Hai box chồng nhau là bình thường. Ví dụ: img_026 → 2 `person` + 1 `motorcycle`. |
| **R-07** | **Xe đạp, xích lô, xe đẩy, gánh hàng:** không gán xe/vật; người điều khiển vẫn là `person` (img_003, img_006, img_023, img_029). |
| **R-08** | **Người trong xe kín** (ô tô, van, cabin xe tải) nhìn qua kính → **không gán**. Người trên xe **hở** (xe máy, xe mui trần img_019, thùng xe tải) → gán `person`. |
| **R-09** | **Phần nào thuộc box xe:** gồm thân xe + bộ phận gắn cố định (thùng sidecar, thùng hàng, nắp thùng pickup, khung/giá chở xe, bạt phủ, thùng/rơ-moóc nhỏ kéo theo). **Không** gồm: gương chiếu hậu, ăng-ten (phần mảnh nhô ra), và **hàng hóa nhô ra ngoài thân xe** (ống dài img_029, đống bao trên nóc img_035, tượng ngựa img_028). |
| **R-10** | **Vật không thật → không gán:** tượng, tượng đài, ma-nơ-canh, người/xe trên biển quảng cáo, poster, tranh tường, đồ chơi, mô hình thu nhỏ, linh vật, ảnh phản chiếu trong gương/kính/nước. (img_011?, img_017, img_018, img_020, img_024 xe vàng trên nóc, img_026 tranh tường, img_027, img_033, img_038). Hóa trang **có người thật bên trong** → `person` + `can_review`. |
| **R-11** | **Xe được chở trên xe khác** (img_002): xe đầu kéo + rơ-moóc chở xe = **1** box `truck` (bao cả khung chở); **mỗi ô tô thật** trên đó gán `car` riêng (vì là ô tô thật, có kích thước thật), `occluded_level` theo phần bị khung che. |
| **R-12** | **Xe chủ (ego vehicle)** và các bộ phận của nó (gương, nắp capo, khung cửa — img_020) → không gán. |
| **R-13** | **Đám đông / cụm xe dày:** vật ở hàng đầu, tách được biên và `occluded_level` ≤ 2 → box riêng. Phần phía sau mà ≥ 5 vật liền nhau, đa số che ≥ mức 2, không tách nổi biên → **một** `ignore_region` bao cụm đó (img_019 khán giả, img_021 đoạn kẹt xe phía xa, img_030 bãi xe máy, img_032 hàng người chờ xe nếu quá dày). |
| **R-14** | **Mờ chuyển động** (img_014, img_015, img_026): vật còn nhận ra biên → box theo **đường viền rõ nhất của thân vật**, không bao cả vệt mờ; tick `can_review`. Không nhận ra từng vật (img_014 xe máy nhòe nền) → `ignore_region`. |
| **R-15** | **Chỉ thấy phụ kiện** (ví dụ chỉ thấy gương xe máy ở mép img_040) và < 10% thân vật → không gán. |
| **R-16** | **Người đứng/ngồi trên xe đang đỗ** (img_010 cô gái ngồi xe tay ga đỗ): vẫn tách `person` + `motorcycle` như R-06. |
| **R-17** | Ảnh **không có vật nào** cần gán (vd. img_017, img_027, img_038) vẫn phải có trong file COCO (0 annotation). |

---

## 4. Ca khó

Cột **Loại**: `QT` = xử lý được bằng quy tắc ở trên; `HỎI` = cần reviewer/nhóm chốt, tạm thời gán theo quyết định ghi ở đây + tick `can_review`.

| Ảnh | Tình huống | Rule ID | Quyết định | Lý do | Loại |
|---|---|---|---|---|---|
| img_026 | Đôi nam nữ trên 1 xe máy, nền tranh tường cảng biển | R-06, R-10 | 2 `person` + 1 `motorcycle`; tranh tường không gán | Mỗi người là một đối tượng cần tránh riêng; tranh là hình in | QT |
| img_005 | 4–5 người trên xe máy chen sát, xe gần như bị người che hết | R-06, R-02 | Mỗi người 1 `person`; `motorcycle` chỉ gán chiếc thấy được thân/tay lái, `occluded_level`=2–3 | Thử lại R-06 trên ảnh rất dày — vẫn áp dụng được | QT |
| img_010 | Hai người trên 1 xe tay ga (người sau bị che phần lớn) + cô gái ngồi trên xe đỗ | R-06, R-16 | Người sau vẫn gán `person` `occluded_level`=2; cô gái + xe tách box | Không phân biệt xe đang chạy hay đỗ | QT |
| img_022 | Mô tô có sidecar, 2 người (ảnh đen trắng cũ) | R-09, R-06 | 1 `motorcycle` bao cả thùng sidecar; mỗi người 1 `person` | Sidecar gắn cố định, là một phương tiện | QT |
| img_002 | Đầu kéo chở 8–9 ô tô | R-11 | 1 `truck` (đầu kéo + khung); mỗi ô tô trên khung ≥ 20 px → `car`, che mức 1–2 | Ô tô thật; bỏ đi sẽ dạy mô hình rằng ô tô là nền | **HỎI** |
| img_029 | Xe tay ga chở ống nhựa dài, người đi xe đạp đỡ đầu ống | R-09, R-07 | `motorcycle` không bao ống; 2 `person`; xe đạp không gán | Hàng hóa nhô ra không phải thân xe; box sẽ phình vô nghĩa | QT |
| img_035 | Auto-rickshaw chất bao cao gấp đôi xe, kèm người lái mô tô | R-05, R-09, R-06 | Rickshaw → `ignore_region` (không gồm đống bao); người + mô tô gán bình thường | Taxonomy thiếu lớp xe 3 bánh | **HỎI** |
| img_004, img_016, img_040 | Tuk-tuk có tài xế ngồi trong | R-05 | `ignore_region` cả xe + người bên trong | Như trên; tránh ép thành `motorcycle`/`car` | **HỎI** |
| img_009, img_031, img_037, img_040 | Xe buýt / minibus 29 chỗ | R-04 | `ignore_region` | Taxonomy không có `bus`; minibus img_009 quá lớn để là `van` | **HỎI** |
| img_001, img_012, img_028, img_037 | Pickup (thùng hở, có hàng, có nắp thùng, chở tượng ngựa) | R-03, R-09, R-10 | `truck`; img_028 box không gồm tượng ngựa | Thùng hàng tách cabin | QT |
| img_036 | Xe tải nhẹ phủ bạt (Veryca) | R-03, R-09 | `truck`, bao cả khung bạt | Có thùng hàng riêng | QT |
| img_005, img_016 | Xe thùng kín mui cao | — | `van` | Thùng liền cabin, cỡ nhỏ | QT |
| img_017 | Tượng người cưỡi ngựa đội nón cọc tiêu | R-10, R-17 | Không gán; ảnh có 0 box | Tượng, không phải người thật | QT |
| img_027 | Tượng đài hình pickup ở nghĩa trang | R-10, R-17 | Không gán | Tượng đá | QT |
| img_033 | Ma-nơ-canh sau kính + phản chiếu xe trên kính | R-10 | Không gán | Ma-nơ-canh & phản chiếu | QT |
| img_018 | Biển quảng cáo có người; ô tô đỗ sau hàng rào | R-10, 2.2 | Người trên biển không gán; ô tô `car`, `occluded_level`=1 (rào thưa) | Hình in không phải người | QT |
| img_020 | Ô tô nhìn qua gương chiếu hậu của xe chủ | R-10, R-12 | Không gán gì | Phản chiếu + bộ phận xe chủ | QT |
| img_038 | Tủ kính trưng bày xe mô hình, búp bê | R-10, R-17 | Không gán | Đồ chơi | QT |
| img_024 | Bảo tàng xe; xe vàng nhỏ đặt trên nóc xe đen | R-10 | Xe thật đỗ → `car`; xe vàng trên nóc → không gán, ghi vấn đề cấp ảnh | Xe vàng có vẻ là mô hình/xe đồ chơi cỡ nhỏ, không chắc | **HỎI** |
| img_019 | Xe hoạt hình trong diễu hành, có người lái + linh vật chuột; khán giả đông | R-10, R-08, R-13 | Xe → `car` (xe chạy thật, có biển số); người lái → `person`; linh vật không gán; khán giả hàng đầu tách được → `person`, phía sau → `ignore_region` | Xe có hình dạng giả tưởng nhưng là phương tiện thật | **HỎI** |
| img_011 | Nhân vật "Predator" cỡ lớn cạnh dãy mô tô độ | R-10 | Nhân vật: không gán + `can_review` (nghi là tượng/đồ trang trí); mô tô độ thật → `motorcycle`; người xa ≥ 20 px → `person` | Không xác định được có người thật bên trong | **HỎI** |
| img_014 | Hai người đi bộ rõ nét, xe máy phía sau nhòe chuyển động | R-14 | 2 `person`; vùng xe/người nhòe → `ignore_region` | Không xác định được biên từng vật | QT |
| img_015 | Đường ướt, phản chiếu xe; xe giữa ảnh nhòe | R-14, R-10 | Xe thật → `car` (cả 2 xe cắt mép `truncated`=yes); phản chiếu trên mặt đường không gán | Phản chiếu không phải vật | QT |
| img_021, img_030 | Kẹt xe máy dày / bãi xe máy dày | R-13 | Hàng đầu box riêng; phần sau → 1–2 `ignore_region` | Không thể tách biên từng xe | QT |
| img_032 | Hàng người chờ xe buýt | R-13, R-02 | Người tách được ≥ 20 px → `person`; đoạn chồng dày → `ignore_region` | Phần lớn người đứng thành hàng tách được | QT |
| img_003 | Người dắt xe đạp, người gánh hàng | R-07 | `person`; không gán xe đạp, không bao quang gánh | Ngoài taxonomy | QT |
| img_023 | Xích lô có người đạp | R-07 | Người đạp → `person`; xích lô không gán | Xích lô không có động cơ | QT |
| img_007 | Kho tối, pickup + xe tải Isuzu | R-03 | 2 `truck` | — | QT |
| img_012 | Pickup đầy hàng sát camera, bị cắt mép | R-03, `truncated` | `truck`, `truncated`=yes | — | QT |

---

## 5. QC và phiên bản

**Ngưỡng QC** (lấy từ `taxonomy.json` → `rules`, không sửa):

- Ghép box giữa hai người gán khi IoU ≥ `match_iou` = 0.5.
- Box đạt khi IoU ≥ 0.85 (vật cao ≥ `large_px` 50 px) hoặc ≥ 0.70 (vật nhỏ hơn).
- IAA đạt khi IoU trung bình ≥ 0.85 và đồng thuận lớp ≥ 0.95.
- Lô được **Release** khi tỉ lệ ảnh lỗi ≤ 2%.

**Ai quyết ca mơ hồ:** người gán tick `can_review` → nhóm xem ở Pha 4–5 → người chốt ghi vào
`calibration_decisions.csv` kèm Rule ID. Ca "HỎI" ở mục 4 phải được nhóm chốt ở Pha 2 **trước** khi gán.

**Đề xuất mang vào họp Pha 2:**

1. Thêm lớp `bus` (R-04) và `three_wheeler` (R-05), hoặc chốt giữ `ignore_region`.
2. Chốt có gán ô tô đang được chở (img_002) hay không (R-11).
3. Chốt có tính người ngồi xe là "che" xe máy hay không (ảnh hưởng `occluded_level` gần như mọi xe máy).
4. Chốt gương chiếu hậu nằm trong hay ngoài box (R-09) — ảnh hưởng IoU vật nhỏ.

**Lịch sử thay đổi:**

| Phiên bản | Thay đổi | Lý do (ca nào) |
|---|---|---|
| v0 | Bản giảng viên: 6 lớp, 3 thuộc tính, 3 quy tắc | — |
| v0.1-`102` | Định nghĩa lớp + ví dụ/phản ví dụ; ngưỡng `occluded_level`; R-01…R-17; 29 dòng ca khó | Xem đủ 40 ảnh: người đi xe máy (img_005, img_026), xe chở xe (img_002), tượng/ma-nơ-canh/poster (img_017, img_027, img_033, img_018), tuk-tuk/bus (img_004, img_009) |

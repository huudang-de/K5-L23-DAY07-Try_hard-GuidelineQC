# Guideline gán nhãn — bản cá nhân (Pha 1) — 103

- Phiên bản: bản cá nhân Pha 1 (sửa từ v0)
- Ngày: 16/11/2026
- Người viết / duyệt: 103 / (chưa duyệt)
- Phạm vi: 40 ảnh `data/images/img_001` – `img_040`, box 2D trên CVAT, xuất COCO 1.0

## 1. Mục tiêu

Nhãn dùng để huấn luyện mô hình **phát hiện người và phương tiện giao thông thật** trên ảnh đường phố
(camera giao thông, xe tự lái). Mô hình cần biết **có vật thật nào trên đường và nó nằm ở đâu**.

Sai ở đâu thì nguy hiểm, theo thứ tự:

1. **Bỏ sót người thật** (người đi bộ, người lái hoặc ngồi sau xe máy). Mô hình học cách bỏ qua người, đây là lỗi an toàn nặng nhất.
2. **Gán vật không thật thành người/xe** (tượng, ma-nơ-canh, hình in, phản chiếu, đồ chơi). Mô hình báo động giả, có thể phanh gấp vô cớ.
3. **Box lệch hoặc lỏng**. Ước lượng vị trí và khoảng cách bị sai.

Ba nguyên tắc chung: chỉ gán **vật thật, kích thước thật**; box ôm **phần nhìn thấy**; không chắc thì tick `can_review`, **không đoán**.

## 2. Lớp và thuộc tính

### Lớp

| Lớp | Định nghĩa (một câu) | Ví dụ | Phản ví dụ / dễ nhầm với |
|---|---|---|---|
| `person` | Người thật ở mọi tư thế (đi, đứng, ngồi, lái xe hở), nhìn thấy trực tiếp chứ không qua kính xe kín. | img_014 (2 người qua đường), img_026 (2 người trên một xe máy), img_019 (người lái xe mui trần), img_004 (tài xế tuk-tuk) | Tượng người cưỡi ngựa img_017; ma-nơ-canh img_033; người trên biển quảng cáo img_018, poster img_030, hình dán trên kính taxi img_040; linh vật chuột img_019; nhân vật Predator khổng lồ img_011; khách ngồi sau kính minibus img_009 |
| `car` | Xe con thật chở người, cabin kín liền thân, tối đa 9 chỗ: sedan, hatchback, SUV, MPV, wagon, taxi, kể cả xe độ/giả tưởng nếu là xe chạy được trên đường. | img_023 (sedan đen), img_015 (xe đỏ, xe bạc), img_012 (SUV bạc), img_019 (xe Dangermouse có biển số và tài xế), img_024 (xe trưng bày trong bảo tàng) | Mô hình ô tô trong tủ kính img_038; xe vàng thu nhỏ trên nóc img_024; tượng bán tải img_027; xe Hiace thân cao → `van`; xe bán tải → `truck` |
| `truck` | Xe chủ yếu để chở hàng, có **thùng hàng tách khỏi cabin** (thùng hở, thùng kín, mui bạt, khung chở xe): xe tải, xe bán tải, xe tải nhỏ. | img_007 (xe tải Isuzu, bán tải có thùng cao), img_002 (xe chở ô tô), img_001, img_012, img_028 (bán tải), img_036 (xe tải nhỏ mui bạt), img_037 (bán tải có nắp thùng) | Xe mà cabin và khoang hàng là một khối liền → `van`; tượng bán tải img_027 → không gán |
| `van` | Xe một khối (cabin và khoang chở người/hàng chung một thân, đầu xe ngắn), cao hơn xe con, tối đa khoảng 16 chỗ. | img_005 (xe Hiace bạc phía sau dòng xe máy), img_016 (xe Hiace trắng) | Minibus/xe buýt dài, mỗi bên ≥ 4 ô cửa sổ khách, cửa lên xuống riêng (img_009) → đề xuất `bus`; bán tải có nắp thùng img_037 → `truck` |
| `motorcycle` | Xe hai bánh có động cơ (xe máy, xe tay ga, mô tô), đang chạy hay đang đỗ, kể cả loại gắn sidecar hoặc thùng bên hông. | img_006, img_021, img_034 (đang chạy), img_008, img_010 (đang đỗ), img_022 (mô tô sidecar), img_025 (xe máy kéo thùng bên hông) | Xe đạp img_006, img_029 → không phải lớp nào; tuk-tuk, xe lam 3 bánh img_004, img_016, img_035 → đề xuất `three_wheeler`; xích lô img_023 → không gán |
| `ignore_region` | Vùng có vật **thật, thuộc phạm vi** nhưng không vẽ được từng box (cao < 20 px, chồng chéo không tách được, nhòe trong suốt) hoặc phương tiện thật chưa có lớp. Bỏ sót trong vùng này không tính lỗi. | img_021 (các hàng xe máy phía xa), img_030 (bãi xe máy phía sau), img_019 (khán giả dày sau rào) | **Không** dùng để che vật không thật (tượng, hình in, phản chiếu): vật không thật thì không gán gì |

**Đề xuất thêm lớp (cần nhóm chốt ở Pha 2):**

| Lớp đề xuất | Định nghĩa | Ảnh | Lý do |
|---|---|---|---|
| `bus` | Xe chở khách từ khoảng 17 chỗ trở lên: minibus, xe buýt. | img_009 (minibus), img_031 (xe buýt số 82), img_037 (xe buýt GO bên trái), img_040 (xe buýt cam phía sau) | Xe buýt là phương tiện lớn, hay gặp; ép vào `van` hay `truck` sẽ làm sai nghĩa hai lớp đó. |
| `three_wheeler` | Xe ba bánh có động cơ chở người hoặc hàng: tuk-tuk, auto rickshaw, xe lam. | img_004 (2 tuk-tuk), img_016 (2 tuk-tuk), img_035 (2 auto rickshaw, một chiếc chở hàng cao), img_040 (tuk-tuk taxi) | Có ở 4/40 ảnh; hình dạng khác hẳn `car` và `motorcycle`, gán nhầm vào lớp nào cũng làm bẩn lớp đó. |

**Nếu nhóm không thêm hai lớp này:** vẽ `ignore_region` ôm phương tiện đó; người lái hoặc ngồi trên xe mà nhìn thấy trực tiếp vẫn gán `person`.

**Vật không thuộc lớp nào:**

- **Vật thật ngoài phạm vi** (xe đạp, xích lô, xe đẩy hàng, động vật, cọc tiêu, hàng hoá): không gán và không cần `ignore_region`. Người đi xe đạp/xích lô vẫn là `person`.
- **Vật không thật** (tượng, tượng đài, mô hình, đồ chơi, ma-nơ-canh, biển quảng cáo, tranh tường, hình dán, hình phản chiếu, hình trên màn hình): **không gán gì**. Gán vào đây là lỗi `thua`.
- **Phương tiện thật chưa có lớp** (`bus`, `three_wheeler` khi nhóm chưa thêm lớp): `ignore_region`.

### Thuộc tính

**`occluded_level`**: phần trăm diện tích vật bị **vật khác** che. Ước lượng theo đường bao đầy đủ của vật. **Không** tính phần bị mép ảnh cắt (phần đó là `truncated`).

| Mức | Bị che | Nhận biết nhanh | Ví dụ |
|---|---|---|---|
| `0` | ≤ 10% | Thấy gần như toàn bộ vật | img_023 sedan đen; img_014 người đàn ông áo trắng |
| `1` | > 10% đến 40% | Người lái xe máy bị thân xe che phần chân; xe đỗ sau hàng rào song sắt | img_006 người lái xe máy nhìn từ phía sau; img_018 ô tô sau hàng rào |
| `2` | > 40% đến 80% | Chỉ thấy khoảng nửa người hoặc nửa xe | img_005 xe Hiace bị đám người đi xe máy che; người ngồi sau ở img_005 |
| `3` | > 80% | Chỉ thấy đầu hoặc một góc xe, nhưng **vẫn nhận ra lớp** | img_021 người ở hàng giữa chỉ thấy mũ bảo hiểm |

- Cách ước lượng: chia người thành 4 đoạn bằng nhau (đầu vai, thân, đùi, cẳng chân), mỗi đoạn khoảng 25%. Chia xe làm đôi theo chiều dài rồi chia đôi tiếp.
- Kính trong suốt không tính là che. Phần chìm hẳn trong bóng tối tính như bị che.
- v0 ghi "3 là không rõ". Bản này bỏ nghĩa đó: **không rõ thì tick `can_review`**, không dùng mức 3 để thay.

**`truncated`**: `yes` khi vật bị **mép ảnh** cắt, tức box chạm mép ảnh (cách ≤ 2 px) và một phần vật nằm ngoài khung. Ngược lại là `no`.
Ví dụ `yes`: img_015 (xe đỏ bên trái, xe bạc bên phải), img_040 (taxi), img_032 (xe xanh bên trái), img_034 (người lái xe máy ở mép phải), img_026 (xe máy bị cắt ở mép dưới).

**`can_review`**: tick khi (a) không chắc lớp, (b) không chắc vật thật hay không, hoặc (c) không chắc ranh giới box (tối, nhòe). Ghi kèm vấn đề cấp ảnh nói lý do. Không tick cho ca đã có quy tắc rõ.

## 3. Quy tắc vẽ box

| Mã | Quy tắc |
|---|---|
| **R-01** | **Chỉ gán vật thật, kích thước thật.** Tượng, tượng đài, mô hình, đồ chơi, ma-nơ-canh, linh vật không thấy người bên trong, hình in, tranh tường, phản chiếu, hình trên màn hình: không gán gì. |
| **R-02** | **Box ôm sát phần nhìn thấy.** Mỗi cạnh cách mép vật ≤ 2 px. Không vẽ thêm phần bị che theo tưởng tượng. Gương chiếu hậu, cản, càng xe tính vào box xe; ăng-ten, dây buộc thì không. |
| **R-03** | **Vật bị che thành nhiều mảnh** (cột điện, người đứng trước, song sắt hàng rào): vẽ **một** box bao tất cả mảnh nhìn thấy của cùng một vật, kể cả phần thấy qua khe hàng rào. |
| **R-04** | **Vật bị mép ảnh cắt:** box dừng ở mép ảnh, `truncated = yes`. |
| **R-05** | **Người trên xe:** tách box. Mỗi người một box `person` (cả người lái và người ngồi sau), bao từ đỉnh đầu hoặc mũ bảo hiểm đến chân, kể cả chân đặt trên xe. Xe máy một box `motorcycle` bao toàn bộ phần xe nhìn thấy (bánh, thân, tay lái, gương). Hai box được phép chồng lên nhau. Người ngồi trên xe đang đỗ cũng tách như vậy. Xe đạp, xích lô: chỉ gán `person`. |
| **R-06** | **Đồ mang theo:** box `person` không gồm đồ người đó cầm hoặc gánh (đòn gánh img_003, ống thép img_029). Box xe gồm hàng chằng cố định trên thân xe (hàng trên nóc auto rickshaw img_035, thùng gắn bên hông xe máy img_025), nhưng không gồm vật dài do người cầm thò ra ngoài xe. |
| **R-07** | **Xe chở xe:** mỗi xe thật là một box riêng. Xe chở = `truck`, box bao cabin và khung chở của chính nó, không bao phần xe được chở nhô ra ngoài khung. Rơ-moóc nối sau (có cụm bánh riêng) là một box `truck` nữa. Mỗi xe được chở là một `car` riêng, có `occluded_level` và `truncated` như thường. Vật trên xe mà không thật (mô hình, tượng) thì theo R-01. |
| **R-08** | **Người trong xe:** chỉ gán `person` khi thấy người trực tiếp: xe máy, xe mui trần (img_019), tuk-tuk hở hông (img_004, img_016, img_035). Người ngồi sau kính của xe kín (img_009) thì không gán. |
| **R-09** | **Vật nhỏ:** vật có box cao < 20 px (`min_box_height_px`) thì không vẽ box riêng. Một cụm vật nhỏ gần nhau → một `ignore_region` bao cả cụm. Vật nhỏ đứng riêng → một `ignore_region` nhỏ. Ví dụ: người và xe máy ở cuối phố img_039, người trên bãi biển phía xa img_025. Vật cao 18–22 px: vẽ thử box rồi xem chiều cao CVAT hiển thị để quyết. |
| **R-10** | **Đám đông, xe dày đặc:** vật nào tách được đường bao và bị che ≤ 80% thì vẽ box riêng. Phần mà từ 5 vật cùng lớp trở lên chồng lên nhau không tách được → một `ignore_region` bao phần đó. `ignore_region` **không được phủ lên** vật đã có box riêng. |
| **R-11** | **Nhòe chuyển động:** vật nhòe nhưng còn đặc (không thấy nền xuyên qua) → box bao cả vệt nhòe của vật. Vật nhòe trong suốt như bóng mờ → `ignore_region`. |
| **R-12** | **Phản chiếu và xe chủ:** không gán vật trong gương, cửa kính, vũng nước (img_020, img_033, img_013). Không gán xe của chính người chụp (gương, cửa xe ở img_020). |
| **R-13** | **Ảnh đêm, tối:** chỉ vẽ khi thấy đường bao. Phần chìm trong tối tính vào `occluded_level`. Không chắc ranh giới → `can_review`. |
| **R-14** | **Đủ để nhận ra lớp:** chỉ vẽ khi phần nhìn thấy đủ để nói là lớp nào. Chỉ thấy một phụ tùng (ví dụ chiếc gương xe máy ở góc dưới img_040) thì không vẽ box xe. |

## 4. Ca khó

### Ca đã có quy tắc

| Ảnh | Tình huống | Rule ID | Quyết định | Lý do |
|---|---|---|---|---|
| img_017 | Tượng người cưỡi ngựa đội cọc tiêu | R-01 | Không gán gì | Tượng, không phải người. Dù cả 3 người cùng vẽ `person` (IAA cao) thì vẫn là lỗi `thua`. |
| img_027 | Tượng bán tải bằng đá ở nghĩa trang | R-01 | Không gán | Không phải xe thật. |
| img_033 | Ma-nơ-canh trong tủ kính, xe tải phản chiếu trên kính | R-01, R-12 | Không gán gì | Ma-nơ-canh không phải người; xe chỉ là hình phản chiếu. |
| img_038 | Tủ trưng bày đồ chơi, mô hình ô tô | R-01 | Không gán gì | Mô hình thu nhỏ. |
| img_018 | Người trên biển quảng cáo; ô tô đỗ sau hàng rào | R-01, R-03 | Không gán người trên biển; mỗi ô tô thật một box `car`, `occluded_level` 1 | Hình in không phải người. Xe nhìn qua song sắt vẫn là một vật, một box. |
| img_020 | Gương chiếu hậu phản chiếu đường và xe; cửa xe của người chụp | R-12 | Không gán gì | Phản chiếu và xe chủ đều ngoài phạm vi. |
| img_002 | Xe chở ô tô, khoảng 10 xe được chở, một người đứng gần đuôi xe | R-07, R-04 | Xe chở: `truck` (cabin và khung); rơ-moóc phía sau: `truck` riêng; mỗi xe được chở: `car`; người: `person` | Xe được chở là xe thật. Nếu đổi R-07 thì các ảnh img_024, img_028, img_012 cũng phải xét lại. |
| img_024 | Bảo tàng xe; xe vàng thu nhỏ đặt trên nóc xe Pacer đen | R-01, R-07 | Xe Pacer và các xe trưng bày: `car` (xe ở mép trái dưới: `truncated = yes`); xe vàng trên nóc: không gán | Xe vàng nhỏ hơn xe thật, bánh nhỏ, không có nội thất: là mô hình. |
| img_028 | Bán tải chở tượng ngựa; SUV đỏ phía sau | R-01, lớp `truck` | Bán tải: `truck`; tượng ngựa: không gán; SUV đỏ: `car`, bị che khoảng 30% → `occluded_level` 1 | Bán tải có thùng tách cabin. |
| img_019 | Xe giả tưởng Dangermouse, linh vật chuột, khán giả đông | R-08, R-10, R-04 | Xe: `car`; tài xế: `person`; linh vật: không gán; khối khán giả sau rào: `ignore_region`; người đứng tách riêng: `person`; đầu người sát mép dưới: `person`, `truncated = yes` | Xe có biển số, có người lái, chạy trên đường nên là xe thật. |
| img_005 | Nhiều người đi xe máy chen sát, bị che và bị mép ảnh cắt | R-05, R-03, R-04 | Mỗi người một `person`; mỗi xe máy nhìn thấy đủ để nhận ra một `motorcycle`; xe Hiace phía sau: `van`, `occluded_level` 2; xe ở mép trái: `car`, `truncated = yes` | Tách người và xe để đếm được người. Xe máy chỉ thấy tay lái mà không nhận ra được thì theo R-14. |
| img_026 | Hai người trên một xe máy trong hầm, xe bị cắt ở mép dưới | R-05, R-04 | 2 `person` + 1 `motorcycle` (`truncated = yes`) | Người ngồi sau che một phần người lái → người lái `occluded_level` 1. |
| img_029 | Xe máy chở ống thép dài; người đi xe đạp | R-05, R-06 | 2 `person` + 1 `motorcycle` (box không gồm ống thép); xe đạp không gán | Ống do người cầm, không phải thân xe. |
| img_022 | Mô tô sidecar trong ảnh đen trắng | Lớp `motorcycle`, R-05 | 1 `motorcycle` bao cả sidecar + `person` | Ảnh đen trắng vẫn là ảnh chụp vật thật. |
| img_037 | Bán tải có nắp thùng kín; xe buýt GO; người đi bộ | Lớp `truck`, đề xuất `bus` | Bán tải: `truck`; xe buýt: `bus` (hoặc `ignore_region` nếu không thêm lớp); người: `person` | Cabin và thùng tách nhau nên vẫn là bán tải. |
| img_009 | Minibus khoảng 29 chỗ, người đi xe máy ở xa | Đề xuất `bus`, R-08, R-09 | Minibus: `bus` (hoặc `ignore_region`); khách trong xe: không gán; xe máy ở xa cao < 20 px: `ignore_region` | Xe dài, nhiều ô cửa khách, không phải `van`. |
| img_004, img_016, img_035 | Tuk-tuk, auto rickshaw (img_035 chở hàng cao trên nóc) | Đề xuất `three_wheeler`, R-06, R-08 | Xe 3 bánh: `three_wheeler` (hoặc `ignore_region`), box gồm hàng trên nóc; tài xế: `person`; ở img_016 người phụ nữ sát mép phải dưới: `person`, `truncated = yes` | Xe hở hông nên thấy tài xế trực tiếp. |
| img_023 | Xích lô có người đạp; sedan đen | R-05 | Người đạp: `person`; xích lô: không gán; sedan: `car` | Xích lô không có động cơ, ngoài phạm vi. |
| img_021, img_030 | Dòng xe máy dày đặc; bãi xe máy đỗ chen chúc | R-10, R-09 | Xe và người phía trước tách được: box riêng; phần phía sau chồng chéo: `ignore_region` | Không thể vẽ từng chiếc nhất quán giữa 3 người gán. |
| img_032 | Khoảng 30 người đứng chờ ở trạm xe buýt; xe xanh bị cắt mép trái | R-10, R-04 | Người tách được: `person`; nhóm đứng chồng lên nhau: `ignore_region`; xe xanh: `car`, `truncated = yes` | Giữ box riêng cho người rõ, chỉ bỏ phần không tách được. |
| img_014 | Hai người qua đường rõ nét; người đi xe máy nhòe phía sau | R-11 | 2 `person` (`occluded_level` 0); vệt xe máy nhòe trong suốt: `ignore_region` | Vệt nhòe thấy nền xuyên qua, không vẽ được ranh giới. |
| img_015 | Đường mưa; xe trắng nhòe mờ giữa đường; xe đỏ và xe bạc bị cắt mép | R-11, R-04 | Xe nhòe mờ: `ignore_region`; xe đỏ, xe bạc: `car`, `truncated = yes`; các xe đỗ bên kia đường: `car` | Xe nhòe trong suốt như bóng mờ. |
| img_040 | Người đội mũ bảo hiểm, chỉ thấy gương xe máy; taxi; tuk-tuk; hình dán mặt người trên kính taxi | R-14, R-04, R-01 | Người: `person`, `truncated = yes`; không vẽ xe máy; taxi: `car`, `truncated = yes`; SUV đen: `car`; hình dán: không gán | Chỉ một chiếc gương thì chưa đủ để vẽ box xe. |
| img_013 | Đường đêm, vũng nước phản chiếu, xe ở cuối đường rất nhỏ | R-13, R-12, R-09 | 2 người đi bộ: `person`; hình phản chiếu: không gán; xe ở xa cao < 20 px: `ignore_region` | Ảnh tối nhưng hai người vẫn rõ đường bao. |
| img_010 | Người ngồi trên xe máy đang đỗ; xe đẩy bán rau | R-05 | `person` + `motorcycle` tách riêng; xe đẩy: không gán | Xe đỗ vẫn là `motorcycle`. |
| img_003 | Người gánh hàng; người dắt xe đạp; xe máy đỗ | R-06, R-05 | Người: `person` (không gồm đòn gánh, thúng); xe đạp: không gán; xe máy đỗ tách được: `motorcycle` | Đồ gánh không phải thân người. |

### Ca cần hỏi reviewer (tick `can_review` và ghi vấn đề cấp ảnh)

| Ảnh | Tình huống | Rule ID | Tạm quyết định | Vì sao chưa chắc |
|---|---|---|---|---|
| img_011 | Xe máy độ trang trí bằng kim loại trưng bày ở lễ hội; nhân vật Predator khổng lồ | R-01, R-14 | Xe: `motorcycle` + `can_review`; Predator: không gán | Không rõ là xe chạy được hay tác phẩm điêu khắc. Predator to gấp nhiều lần người thật nên là tượng. |
| img_019 | Linh vật chuột ngồi trong xe | R-01 | Không gán + ghi vấn đề cấp ảnh | Có thể là người mặc đồ linh vật; ảnh không thấy phần người nào. |
| img_007 | Bán tải có thùng cao gắn đèn trong kho tối | Lớp `truck`, R-13 | `truck` + `can_review` nếu không thấy rõ khe giữa cabin và thùng | Thiếu sáng, khó thấy thùng có tách cabin không. |

## 5. QC và phiên bản

- **Ngưỡng QC** (giữ nguyên mục `rules` trong `taxonomy.json`):
  - vật cao < 20 px → `ignore_region` (R-09);
  - khớp box: IoU ≥ 0,85 với box cao ≥ 50 px, IoU ≥ 0,7 với box nhỏ hơn;
  - IAA: IoU trung bình ≥ 0,85 và cùng lớp ≥ 95%;
  - lỗi theo ảnh ≤ 2% mới Release.
- **Ai quyết ca mơ hồ:** người gán tick `can_review` và ghi vấn đề cấp ảnh, không tự đoán. Ca được chốt ở buổi calibration (Pha 5) theo đa số nhóm; hoà thì reviewer QC độc lập của nhóm quyết. Quyết định ghi vào guideline v1.1, kèm Rule ID và ảnh.
- **Đổi phiên bản:** mọi thay đổi làm đổi nhãn đều tăng phiên bản. Ghi rõ các ảnh bị ảnh hưởng (ví dụ đổi R-07 thì phải xét lại img_002, img_024, img_028, img_012).
- **Câu hỏi mang ra họp Pha 2:**
  1. Có thêm `bus` và `three_wheeler` không?
  2. Có cần lớp cho xe đạp không?
  3. Người ngồi sau kính xe kín có gán không?
  4. Rơ-moóc có tách thành box riêng không?

| Phiên bản | Thay đổi | Lý do (ca nào) |
|---|---|---|
| v0 | Bản khởi đầu của giảng viên | — |
| 103 · Pha 1 | Định nghĩa lớp có ví dụ và phản ví dụ; `occluded_level` theo % đo được (bỏ "3 là không rõ"); thêm R-01 đến R-14; đề xuất `bus`, `three_wheeler` | img_017, img_027, img_033 (vật không thật); img_002 (xe chở xe); img_005, img_026 (người đi xe máy); img_009, img_004 (phương tiện chưa có lớp) |

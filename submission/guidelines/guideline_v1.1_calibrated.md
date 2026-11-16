# Guideline gán nhãn — v1.1 (sau Calibration) — Nhóm Try_hard

- Phiên bản: **v1.1**
- Ngày: 2026-11-16
- Nguồn: hợp nhất 5 bản cá nhân `guideline_101.md` … `guideline_105.md` (giữ nguyên các bản cá nhân để đối chiếu)
- Biên soạn: 102 · Thư ký: 103 · Điều phối: 101 · Reviewer độc lập: 105
- Trạng thái: **cập nhật sau Calibration Pha 5** (editorial, giữ nguyên `submission/taxonomy.json` v1.0, phạm vi rà sửa 37 ảnh bị cờ trong `flagged_images.txt`).
- Loại nhãn: bounding box 2D, CVAT, xuất COCO 1.0. Phạm vi: `data/images/img_001.jpg` – `img_040.jpg`

---

## 1. Mục tiêu

Nhãn dùng để huấn luyện mô hình **phát hiện người và phương tiện giao thông thật** trên ảnh đường phố
(camera giao thông, hỗ trợ lái / xe tự lái). Mô hình cần biết **có vật thật nào trên đường và nó nằm ở đâu**.

Sai ở đâu thì nguy hiểm, xếp theo mức độ:

1. **Bỏ sót người thật** (người đi bộ, người lái hoặc ngồi sau xe). Mô hình học cách bỏ qua người. Đây là lỗi an toàn nặng nhất.
2. **Gán vật không thật** (tượng, ma-nơ-canh, hình in, phản chiếu, đồ chơi) thành người hoặc xe. Mô hình báo động giả và có thể phanh gấp vô cớ.
3. **Sai lớp phương tiện** (`car` / `van` / `truck` / `bus`…). Ước lượng kích thước và quỹ đạo bị sai.
4. **Box lệch, lỏng hoặc sai thuộc tính.** Vị trí và khoảng cách bị ước lượng sai.

**Ba nguyên tắc chung:**
- Chỉ gán **vật thật, kích thước thật**.
- Box ôm **phần nhìn thấy**.
- Không chắc thì tick `can_review`, **không đoán**.

---

## 2. Lớp và thuộc tính

### 2.1 Lớp (9 lớp, gồm `ignore_region`)

Thứ tự để xếp lớp một phương tiện:
1. Có **bàn đạp, không động cơ** → `bicycle`.
2. Có **3 bánh, động cơ, có mui/cabin che người lái** → `tuk_tuk`.
3. Có **2 bánh, động cơ** (kể cả khi gắn sidecar/thùng kéo, không mui) → `motorcycle`.
4. Còn lại là xe 4 bánh trở lên: xếp theo **hình dạng thân** vào `car` / `van` / `truck` / `bus`.

| Lớp | Định nghĩa (một câu) | Ví dụ | Phản ví dụ / dễ nhầm với |
|---|---|---|---|
| `car` | Ô tô con chở người tối đa 9 chỗ, thân thấp: sedan, hatchback, SUV, crossover, wagon, coupe, MPV gia đình, taxi. Gồm cả xe độ / giả tưởng nếu là xe chạy được trên đường (có biển số, có người lái). | img_023 (sedan đen), img_012 (SUV bạc), img_015 (xe đỏ, xe bạc), img_019 (xe Dangermouse), img_024 (xe trưng bày), img_040 (taxi, SUV đen) | Pickup → `truck` (R-07). Xe thân hộp mui cao (Hiace) → `van`. Mô hình / đồ chơi → không gán (R-01). |
| `van` | Xe **một khối** (cabin và khoang sau chung một thân, đầu xe ngắn), mui cao hơn xe con, tối đa 16 chỗ hoặc chở hàng cỡ nhỏ. | img_005 (Hiace bạc phía sau dòng xe máy), img_016 (Hiace trắng) | MPV gia đình thân thấp → `car`. Pickup có nắp thùng (img_037) → `truck`. Xe ≥ 17 chỗ (img_009) → `bus`. |
| `truck` | Xe chở hàng có **thùng / khoang hàng tách khỏi cabin** (thùng hở, thùng kín, mui bạt, khung chở xe): xe tải, đầu kéo, xe chở xe, **pickup** (kể cả pickup có nắp thùng). | img_002 (xe chở ô tô), img_007 (Isuzu ELF + pickup), img_036 (xe tải nhỏ mui bạt), img_001, img_012, img_028, img_037 (pickup) | Xe một khối → `van`. Tượng pickup bằng đá (img_027) → không gán. |
| `bus` | Xe chở khách **từ 17 chỗ trở lên**, thân dài, mỗi bên ≥ 4 ô cửa sổ khách, có cửa lên xuống riêng: xe buýt, xe khách, minibus cỡ lớn. | img_009 (minibus), img_031 (xe buýt số 82), img_037 (xe buýt GO bên trái), img_040 (xe buýt phía sau) | Hiace ≤ 16 chỗ → `van`. Không suy lớp chỉ từ màu sơn hoặc chữ trên xe. |
| `tuk_tuk` | Xe **3 bánh có động cơ, có mui / cabin che người lái**, chở khách hoặc hàng: tuk-tuk, auto-rickshaw, xe lam. | img_004 (2 tuk-tuk), img_016 (2 tuk-tuk), img_035 (2 auto-rickshaw, một chiếc chất hàng cao), img_040 (tuk-tuk taxi) | Xe máy kéo thùng / ba gác máy **không mui** → `motorcycle`. Xích lô, ba gác đạp → `bicycle`. |
| `motorcycle` | Xe **2 bánh có động cơ**: xe số, xe tay ga, mô tô, xe máy điện, xe đạp điện; kể cả xe gắn **sidecar** hoặc **kéo thùng / ba gác máy** không mui; đang chạy hay đang đỗ đều tính. | img_006, img_021, img_034 (đang chạy), img_008, img_010 (đang đỗ), img_022 (sidecar), img_025 (xe máy kéo thùng bên hông) | Xe đạp → `bicycle`. Tuk-tuk → `tuk_tuk`. Chỉ thấy gương xe (img_040) → không gán (R-14). |
| `bicycle` | Xe **do người đạp, không động cơ**: xe đạp, xe đạp chở hàng, **xích lô**, xe ba gác đạp; đang chạy, đang dắt hay đang đỗ. | img_006 (xe đạp trong dòng xe), img_023 (xích lô), img_029 (người đi xe đạp đỡ ống), img_003 (xe đạp được dắt), img_039 | Xe đạp điện → `motorcycle`. Xe đẩy bán hàng không có bàn đạp (img_010) → không gán. |
| `person` | Người thật ở mọi tư thế (đi, đứng, ngồi, lái hoặc ngồi trên xe hở), **nhìn thấy trực tiếp**, không qua kính xe kín. | img_014 (2 người qua đường), img_026 (2 người trên một xe máy), img_019 (tài xế xe mui trần), img_004 (tài xế tuk-tuk), img_032 (người chờ xe) | Tượng (img_017), ma-nơ-canh (img_033), người trên biển quảng cáo (img_018), poster (img_030), hình dán trên kính taxi (img_040), linh vật chuột (img_019), nhân vật Predator khổng lồ (img_011), khách sau kính minibus (img_009). |
| `ignore_region` | Vùng chứa **vật thật thuộc taxonomy** nhưng không vẽ được từng box (quá nhỏ, chồng chéo không tách được, nhòe trong suốt). Bỏ sót trong vùng này không tính lỗi. | img_021 (hàng xe máy phía xa), img_030 (bãi xe máy phía sau), img_019 (khán giả dày sau rào) | **Không** dùng cho vật không thật (tượng, hình in, phản chiếu): những vật đó không gán gì. Không dùng thay box cho vật rõ, cao ≥ 20 px. |

**Vật không thuộc lớp nào:**

| Loại | Xử lý |
|---|---|
| Vật thật ngoài phạm vi: xe đẩy hàng, động vật, cọc tiêu, hàng hóa rời, cơ sở hạ tầng | **Không gán**, không cần `ignore_region`. |
| Vật không thật: tượng, tượng đài, mô hình, đồ chơi, ma-nơ-canh, linh vật không thấy người bên trong, biển quảng cáo, tranh tường, hình dán, phản chiếu, hình trên màn hình | **Không gán gì** (R-01). Gán vào là lỗi `thua`. |

### 2.2 Thuộc tính

**`occluded_level`**: phần trăm diện tích vật bị **vật khác** che, ước lượng theo đường bao đầy đủ của vật.
**Không** tính phần bị mép ảnh cắt (phần đó thuộc `truncated`).

| Mức | Bị che | Nhận biết nhanh | Ví dụ |
|---|---|---|---|
| `0` | ≤ 10% | Thấy gần như toàn bộ vật | img_023 sedan đen; img_014 người đàn ông áo trắng |
| `1` | > 10% – 30% | Người lái xe máy bị thân xe che phần chân; xe đỗ sau hàng rào song sắt | img_006 người lái nhìn từ phía sau; img_018 ô tô sau hàng rào |
| `2` | > 30% – 70% | Chỉ thấy khoảng nửa người hoặc nửa xe | img_005 Hiace bị đám người đi xe máy che; người ngồi sau ở img_005 |
| `3` | > 70% | Chỉ thấy đầu hoặc một góc xe, nhưng **vẫn nhận ra lớp** | img_021 người ở hàng giữa chỉ thấy mũ bảo hiểm |

- **Cách ước lượng thống nhất:**
  - Người: chia thành 4 đoạn bằng nhau (đầu–vai, thân, đùi, cẳng chân), mỗi đoạn khoảng 25%.
  - Xe: chia đôi theo chiều dài, rồi chia đôi tiếp, mỗi phần khoảng 25%.
- **Mọi vật che đều tính**, kể cả người ngồi trên xe che xe và xe che chân người.
- Song sắt / lưới thưa: chỉ tính phần thanh rào che thật, thường là mức `1`.
- **Kính trong suốt không tính là che.** Phần chìm hẳn trong bóng tối tính như bị che.
- Không nhận ra lớp → **không vẽ box** (R-14). Mức `3` **không** có nghĩa "không rõ": không chắc thì tick `can_review`.

**`truncated`**:

| Giá trị | Khi nào |
|---|---|
| `yes` | Vật bị **mép ảnh** cắt: box chạm mép ảnh (cách ≤ 2 px) **và** một phần vật rõ ràng nằm ngoài khung. Ví dụ: img_015 (xe đỏ trái, xe bạc phải), img_040 (taxi), img_032 (xe xanh trái), img_034 (người lái xe máy mép phải), img_026 (xe máy cắt ở mép dưới). |
| `no` | Toàn bộ vật nằm trong khung, hoặc chỉ chạm mép mà không bị cắt mất phần nào. |

**`can_review`**: tick khi (a) không chắc lớp, (b) không chắc vật thật hay không, hoặc (c) không chắc biên box (tối, nhòe).
Ghi kèm vấn đề cấp ảnh nêu lý do. **Không** tick cho ca đã có quy tắc rõ trong guideline này.

---

## 3. Quy tắc vẽ box

### Nhóm A — Chọn vật nào để gán

| Mã | Quy tắc |
|---|---|
| **R-01** | **Chỉ gán vật thật, kích thước thật.** Tượng, tượng đài, mô hình, đồ chơi, ma-nơ-canh, linh vật không thấy người bên trong, hình in, biển quảng cáo, tranh tường, hình dán, hình trên màn hình: **không gán gì**. Người thật mặc trang phục hóa trang (thấy được kích thước người, tay chân, cử động) → `person`. Hình dạng người nhưng **lớn hơn hẳn người thật** hoặc gắn cố định trên bệ → coi là tượng. |
| **R-02** | **Phản chiếu và xe chủ:** không gán vật trong gương, cửa kính, vũng nước. Không gán xe của chính người chụp (gương, capo, khung cửa). |
| **R-03** | **Người trong xe:** chỉ gán `person` khi thấy người **trực tiếp**: xe máy, xe đạp, xích lô, tuk-tuk hở hông, xe mui trần, thùng xe tải. Người ngồi **sau kính** của xe kín (ô tô, van, bus, cabin tải): không gán. |
| **R-04** | **Kích thước tối thiểu:** box cao **< 20 px** (`min_box_height_px`) thì không vẽ box riêng. Vật cao 18–22 px: vẽ thử rồi xem chiều cao CVAT hiển thị để quyết. Vật nhỏ đứng riêng lẻ → bỏ qua. Cụm **≥ 5** vật nhỏ liền nhau → một `ignore_region` (R-13). |
| **R-14** | **Đủ để nhận ra lớp:** chỉ vẽ khi phần nhìn thấy đủ để nói là lớp nào. Chỉ thấy một phụ tùng (ví dụ chiếc gương xe máy ở góc dưới img_040) → không vẽ box xe. |

### Nhóm B — Vẽ box thế nào

| Mã | Quy tắc |
|---|---|
| **R-05** | **Box ôm sát phần nhìn thấy:** mỗi cạnh cách mép vật ≤ 2 px. Không vẽ thêm phần bị che theo tưởng tượng. Không bao bóng đổ. Box luôn nằm trong ảnh: `0 ≤ x`, `0 ≤ y`, `x + w ≤ W`, `y + h ≤ H`. |
| **R-06** | **Vật bị che thành nhiều mảnh** (cột điện, người đứng trước, song sắt): vẽ **một** box bao tất cả mảnh nhìn thấy của cùng một vật, kể cả phần thấy qua khe hàng rào. |
| **R-08** | **Người trên xe — tách box.** Mỗi người một box `person` (cả người lái và từng người ngồi sau, kể cả trẻ em kẹp giữa nếu cao ≥ 20 px), bao từ đỉnh đầu / mũ bảo hiểm đến chân thấp nhất nhìn thấy, kể cả phần chồng lên xe. Xe một box riêng bao **toàn bộ phần xe** nhìn thấy, không kéo lên đầu / thân người. Hai box chồng nhau là bình thường. Áp dụng cho **mọi** loại xe hở: `motorcycle`, `bicycle` (gồm xích lô), `tuk_tuk`, xe mui trần. Xe đang đỗ có người ngồi cũng tách như vậy. |
| **R-09** | **Phần nào thuộc box xe.** **Gồm** thân xe và mọi bộ phận gắn cố định: gương chiếu hậu, cản, sidecar, thùng kéo, nắp thùng pickup, khung chở xe, khung mui bạt, giá nóc, ghế khách của xích lô. **Không gồm** ăng-ten, dây buộc, và **hàng hóa nhô ra ngoài đường bao thân xe** (ống dài, đống bao chất trên nóc, tượng chở trên thùng). Hàng nằm gọn trong thùng thì tự nhiên đã ở trong box. |
| **R-10** | **Đồ người mang theo:** box `person` gồm đồ đeo sát người (túi, ba lô), **không** gồm đồ cầm hoặc gánh thò ra (đòn gánh, thúng, ống dài). |
| **R-11** | **Mép ảnh:** box dừng ở mép ảnh, `truncated = yes`. |
| **R-12** | **Nhòe chuyển động:** vật nhòe nhưng còn **đặc** (không thấy nền xuyên qua) → box bao toàn bộ phần đặc của vật, kể cả vệt nhòe đặc. Vật nhòe **trong suốt** như bóng mờ → `ignore_region`. Không đoán lớp từ vệt nhòe. |
| **R-15** | **Ảnh đêm, tối:** chỉ vẽ khi thấy đường bao. Phần chìm trong tối tính vào `occluded_level`. Không chắc ranh giới → `can_review`. |

### Nhóm C — Ca đặc biệt

| Mã | Quy tắc |
|---|---|
| **R-07** | **Pickup = `truck`**, kể cả khi có nắp thùng kín / canopy (img_037), thùng chở đồ (img_012) hay chở tượng (img_028). Lý do: thùng tách khỏi cabin. Đây là tiêu chí nhìn thấy được, không phụ thuộc mục đích sử dụng. |
| **R-13** | **Đám đông, xe dày đặc:** vật nào tách được đường bao và bị che ≤ 70% → box riêng. Phần **≥ 5 vật** liền nhau chồng lên nhau không tách được → **một** `ignore_region` bao phần đó. `ignore_region` **không được phủ lên** vật đã có box riêng. Script đếm vùng ignore phủ lên box của người khác là bất đồng (`bat_dong_ignore`). |
| **R-16** | **Xe chở xe** (img_002): **một** box `truck` cho xe chở, bao cabin, khung xe và khung chở xe, **không** kéo lên theo phần xe được chở nhô ra ngoài khung (theo R-09). **Mỗi ô tô thật** trên khung, cao ≥ 20 px → một box `car` riêng, gán `occluded_level` theo phần bị khung che. Xe được chở là vật thật, có kích thước thật. Vật được chở mà không thật (mô hình, tượng) thì theo R-01. |
| **R-17** | **Xe 3 bánh:** xếp theo **hình dạng phần đầu xe**, không theo công dụng (chở khách hay chở hàng). Có mui / cabin che người lái → `tuk_tuk`. Đầu là xe máy nguyên vẹn, người lái ngồi lộ trên yên → `motorcycle` (box gồm cả thùng). Có bàn đạp → `bicycle`. |
| **R-18** | **Ảnh không có vật nào cần gán** (img_017, img_020, img_027, img_033, img_038) vẫn phải có mặt trong file COCO với 0 annotation. |

---

## 4. Ca khó

### 4.1 Ca đã có quy tắc

| Ảnh | Tình huống | Rule ID | Quyết định | Lý do |
|---|---|---|---|---|
| img_017 | Tượng người cưỡi ngựa đội cọc tiêu | R-01, R-18 | Không gán gì | Tượng. Dù cả 3 người cùng vẽ `person` (IAA cao) thì vẫn là lỗi `thua`. |
| img_027 | Tượng pickup bằng đá ở nghĩa trang | R-01, R-18 | Không gán | Không phải xe thật. |
| img_033 | Ma-nơ-canh trong tủ kính; xe phản chiếu trên kính | R-01, R-02, R-18 | Không gán gì | Ma-nơ-canh và phản chiếu. |
| img_038 | Tủ kính đồ chơi, mô hình ô tô, búp bê | R-01, R-18 | Không gán gì | Mô hình thu nhỏ. |
| img_018 | Người trên biển quảng cáo; ô tô đỗ sau hàng rào | R-01, R-06 | Người trên biển: không gán. Mỗi ô tô thật: một `car`, `occluded_level` 1 | Hình in không phải người. Xe nhìn qua song sắt vẫn là một vật, một box. |
| img_020 | Gương chiếu hậu phản chiếu đường và xe; cửa xe người chụp | R-02, R-18 | Không gán gì | Phản chiếu và xe chủ. |
| img_002 | Xe chở ô tô 2 tầng; một người đứng gần đuôi xe | R-16, R-11 | 1 `truck` (cabin + khung chở). Mỗi ô tô trên khung cao ≥ 20 px: `car`. Người: `person` | Ô tô được chở là xe thật. Bỏ đi sẽ dạy mô hình rằng hình dạng ô tô là nền. |
| img_024 | Bảo tàng xe; xe vàng thu nhỏ trên nóc xe Pacer đen | R-01 | Pacer và các xe trưng bày: `car` (xe góc trái dưới: `truncated = yes`). Xe vàng trên nóc: không gán | Xe vàng nhỏ hơn xe thật, bánh nhỏ, không có nội thất: là mô hình. |
| img_028 | Pickup chở tượng ngựa; SUV đỏ phía sau | R-07, R-09, R-01 | Pickup: `truck`, box **không** gồm tượng ngựa. Tượng: không gán. SUV đỏ: `car`, `occluded_level` 1 | Thùng tách cabin. Tượng là hàng chở, nhô ra ngoài thân xe. |
| img_001, img_012 | Pickup thùng hở chở hàng | R-07, R-09 | `truck`. img_012 bị cắt mép: `truncated = yes` | Thùng tách cabin. |
| img_037 | Pickup có nắp thùng kín; xe buýt GO; người đi bộ | R-07 | Pickup: `truck`. Xe buýt: `bus`. Người: `person` | Nắp thùng gắn cố định, không biến pickup thành `van`. |
| img_036 | Xe tải nhỏ mui bạt (Veryca) | R-09 | `truck`, box gồm cả khung bạt | Có thùng hàng riêng. |
| img_007 | Kho tối: pickup có khung cao + xe tải Isuzu ELF | R-07, R-15 | 2 `truck` | Cả hai có thùng tách cabin. |
| img_005, img_016 | Xe Hiace thân hộp | lớp `van` | `van`. img_005: `occluded_level` 2 | Một khối, mui cao, ≤ 16 chỗ. |
| img_009 | Minibus khoảng 29 chỗ; xe máy ở xa | lớp `bus`, R-03, R-04 | Minibus: `bus`. Khách sau kính: không gán. Xe máy cao < 20 px: bỏ qua (cụm ≥ 5 thì `ignore_region`) | Xe dài, nhiều ô cửa khách. |
| img_031 | Đường đêm mưa: xe buýt số 82, ô tô, xe máy | lớp `bus`, R-15, R-02 | Xe buýt: `bus`. Xe khác theo lớp. Phản chiếu trên mặt đường: không gán | Phản chiếu không phải vật. |
| img_004, img_016 | Tuk-tuk có tài xế, hở hông | R-17, R-08, R-03 | Xe: `tuk_tuk`. Tài xế và khách nhìn thấy trực tiếp: `person` riêng. img_016: người phụ nữ sát mép phải dưới: `person`, `truncated = yes` | Hở hông nên thấy người trực tiếp. |
| img_035 | Auto-rickshaw chất bao cao gấp đôi xe; người lái mô tô | R-17, R-09, R-08 | Rickshaw: `tuk_tuk`, box **không** gồm đống bao trên nóc. Người lái mô tô: `person` + `motorcycle` | Hàng nhô ra ngoài đường bao thân xe. |
| img_040 | Người đội mũ bảo hiểm, chỉ thấy gương xe máy; taxi; tuk-tuk; xe buýt; hình dán mặt người trên kính taxi | R-14, R-11, R-01 | Người: `person`, `truncated = yes`. Không vẽ xe máy. Taxi: `car`, `truncated = yes`. SUV đen: `car`. Tuk-tuk: `tuk_tuk`. Xe buýt: `bus`. Hình dán: không gán | Một chiếc gương chưa đủ để vẽ box xe. |
| img_026 | Hai người trên một xe máy trong hầm, nền tranh tường | R-08, R-11, R-01 | 2 `person` + 1 `motorcycle` (`truncated = yes`). Người lái bị người sau che: `occluded_level` 1. Tranh tường: không gán | Mỗi người là một đối tượng cần bảo vệ. |
| img_005 | Nhiều người trên xe máy chen sát, bị che và cắt mép | R-08, R-06, R-11, R-14 | Mỗi người một `person`. Mỗi xe máy nhận ra được: một `motorcycle`. Xe máy chỉ thấy tay lái: không vẽ. Xe mép trái: `car`, `truncated = yes` | Kiểm thử R-08 trên ảnh dày nhất. |
| img_010 | Hai người trên xe tay ga; cô gái ngồi trên xe đỗ; xe đẩy bán rau | R-08 | Người và xe tách box (kể cả xe đỗ). Xe đẩy: không gán | Xe đỗ có người vẫn tách như xe đang chạy. |
| img_022 | Mô tô sidecar, ảnh đen trắng cũ | R-09, R-08 | 1 `motorcycle` bao cả sidecar + mỗi người một `person` | Sidecar gắn cố định. Ảnh đen trắng vẫn là ảnh chụp vật thật. |
| img_025 | Đường dạo biển; xe máy kéo thùng bên hông; người đi dạo; người xa | R-17, R-09, R-04 | Xe máy + thùng: 1 `motorcycle`. Người ≥ 20 px: `person`. Người xa < 20 px: bỏ qua, cụm ≥ 5 thì `ignore_region` | Đầu xe là xe máy, không mui. |
| img_029 | Xe tay ga chở ống dài; người đi xe đạp đỡ đầu ống | R-08, R-09, R-10 | 2 `person` + 1 `motorcycle` + 1 `bicycle`. Không box nào gồm ống | Ống là hàng nhô ra, không phải thân xe hay thân người. |
| img_023 | Xích lô có người đạp; sedan đen | R-08, lớp `bicycle` | Người đạp: `person`. Xích lô: `bicycle` (gồm ghế khách). Sedan: `car` | Xích lô không động cơ. |
| img_006 | Dòng xe máy và xe đạp nhìn từ phía sau; taxi | R-08 | Mỗi người: `person`. Xe máy: `motorcycle`. Xe đạp: `bicycle`. Taxi: `car`, `truncated = yes` | Cùng một quy tắc tách box cho mọi xe hở. |
| img_003 | Người gánh hàng; người dắt xe đạp; xe máy đỗ | R-10, R-08 | `person` (không gồm đòn gánh, thúng). Xe đạp đang dắt: `bicycle`. Xe máy đỗ tách được: `motorcycle` | Đồ gánh không phải thân người. |
| img_019 | Xe hoạt hình diễu hành có tài xế và linh vật chuột; khán giả đông sau rào | R-01, R-03, R-13 | Xe: `car` (có biển số, có người lái). Tài xế: `person`. Linh vật: không gán. Khán giả hàng đầu tách được: `person`. Khối phía sau: `ignore_region`. Đầu người sát mép dưới: `person`, `truncated = yes` | Xe hình dạng giả tưởng nhưng là phương tiện thật. |
| img_021, img_030 | Dòng xe máy dày đặc; bãi xe máy đỗ chen chúc | R-13, R-04 | Hàng đầu tách được: box riêng. Phần chồng chéo phía sau: `ignore_region`, không phủ lên box riêng | Không thể vẽ từng chiếc nhất quán giữa 3 người gán. |
| img_032 | Khoảng 30 người chờ ở trạm; xe xanh cắt mép trái | R-13, R-11 | Người tách được: `person`. Nhóm đứng chồng nhau: `ignore_region`. Xe xanh: `car`, `truncated = yes` | Giữ box cho người rõ, chỉ bỏ phần không tách được. |
| img_008 | Chợ đêm: xe máy đỗ thành hàng, người mặc áo phản quang | R-08, R-13, R-15 | Người và xe tách được: box riêng. Hàng xe đỗ sát nhau không tách được: `ignore_region` | Không bỏ qua đối tượng tiền cảnh rõ. |
| img_014 | Hai người qua đường rõ nét; xe máy nhòe phía sau | R-12 | 2 `person` (`occluded_level` 0). Vệt xe nhòe trong suốt: `ignore_region` | Thấy nền xuyên qua, không vẽ được ranh giới. |
| img_015 | Đường mưa; xe nhòe mờ giữa đường; xe đỏ và xe bạc cắt mép | R-12, R-11, R-02 | Xe nhòe trong suốt: `ignore_region`. Xe đỏ, xe bạc: `car`, `truncated = yes`. Xe đỗ bên kia đường: `car`. Phản chiếu trên mặt đường: không gán | Nhòe trong suốt như bóng mờ. |
| img_013 | Đường đêm, vũng nước phản chiếu; xe ở cuối đường | R-15, R-02, R-04 | 2 người đi bộ: `person`. Phản chiếu: không gán. Xe xa < 20 px: bỏ qua | Ảnh tối nhưng hai người vẫn rõ đường bao. |

### 4.2 Ca cần theo dõi ở calibration (tạm quyết định, tick `can_review`)

| Ảnh | Tình huống | Rule ID | Tạm quyết định | Vì sao chưa chắc |
|---|---|---|---|---|
| img_011 | Nhân vật Predator khổng lồ; dãy mô tô độ trang trí bằng kim loại | R-01, R-14 | Predator: **không gán** (lớn hơn hẳn người thật, đứng cố định → tượng). Mô tô độ: `motorcycle` + `can_review`. Khách tham quan ≥ 20 px: `person` | 104 và 105 cho rằng có người thật bên trong. Ảnh không thấy bộ phận người nào. Chưa rõ mô tô là xe chạy được hay tác phẩm điêu khắc. |
| img_019 | Linh vật chuột ngồi trong xe | R-01 | Không gán + ghi vấn đề cấp ảnh | Có thể là người mặc đồ linh vật, nhưng ảnh không thấy phần người nào. |
| img_007 | Pickup có khung cao gắn đèn trong kho tối | R-07, R-15 | `truck`. Tick `can_review` nếu không thấy rõ khe giữa cabin và thùng | Thiếu sáng. |

---

## 5. QC và phiên bản

### 5.1 Ngưỡng QC (từ `submission/taxonomy.json` → `rules`, **không sửa**)

| Ngưỡng | Giá trị | Ý nghĩa |
|---|---|---|
| `min_box_height_px` | 20 | Box riêng cao < 20 px → không vẽ (R-04). |
| `match_iou` | 0.5 | Hai box của hai người được ghép cặp khi IoU ≥ 0.5. |
| `large_px` | 50 | Vật "lớn" khi **chiều cao** trung bình của cặp box ≥ 50 px. |
| `iou_thr_large` / `iou_thr_small` | 0.85 / 0.70 | Box đạt khi IoU ≥ 0.85 (vật lớn) hoặc ≥ 0.70 (vật nhỏ). |
| `iaa_iou` / `iaa_class` | 0.85 / 0.95 | IAA đạt khi IoU trung bình ≥ 0.85 và cùng lớp ≥ 95%, tính trên các cặp ghép được. |
| `error_rate_img_max` | 0.02 | Release khi tỉ lệ ảnh lỗi ≤ 2%. Lô 40 ảnh: chỉ 1 ảnh lỗi (2.5%) là đã vượt ngưỡng. |

### 5.2 Ai quyết ca mơ hồ

1. **Lúc gán (Pha 3):** người gán tick `can_review` hoặc ghi vấn đề cấp ảnh. Không hỏi nhau, không xem nhãn người khác.
2. **Calibration (Pha 5):** nhóm chốt theo đa số. Hòa thì **105** (reviewer độc lập) quyết. **103** ghi vào `calibration_decisions.csv` kèm Rule ID, ảnh và `pham_vi`.
3. **Đổi quy tắc làm đổi nhãn** → tăng phiên bản (v1.1), ghi các ảnh bị ảnh hưởng. Đổi nghĩa quy tắc (`semantic`) → `pham_vi = lo`, kiểm lại cả 40 ảnh. Ví dụ đổi R-16 phải xét lại img_002, img_024, img_028.

### 5.3 Các bất đồng giữa 5 bản cá nhân và cách chốt

| Vấn đề | 101 | 102 | 103 | 104 | 105 | **Chốt v1.0** | Lý do |
|---|---|---|---|---|---|---|---|
| Lớp mới | đề xuất `bus`, `bicycle`, `three_wheeler` | đề xuất `bus`, `three_wheeler` | đề xuất `bus`, `three_wheeler` | không thêm; tuk-tuk → `motorcycle` | `bus`, `tuk_tuk` | **`bus`, `tuk_tuk`, `bicycle`** | 4/5 bản muốn tách bus và xe 3 bánh. Tên `tuk_tuk` khớp ca tham chiếu. Có `bicycle` thì xích lô và xe đạp có lớp đúng thay vì bị bỏ qua. |
| Pickup | `car` (img_037 → `van`) | `truck` | `truck` | `car` | `truck` | **`truck`** (R-07) | Tiêu chí nhìn thấy được (thùng tách cabin), không cần đoán mục đích sử dụng. |
| Xe chở xe img_002 | 1 `truck` gồm cả ô tô | `truck` + từng `car` | `truck` + rơ-moóc riêng + từng `car` | `truck` + từng `car` | `truck` + từng `car` | **1 `truck` + từng `car`** (R-16) | Ô tô được chở là xe thật. Không tách rơ-moóc vì khó thống nhất ranh giới đầu kéo / rơ-moóc. |
| Hàng trên nóc tuk-tuk img_035 | — | không bao | bao | — | không bao | **Không bao** (R-09) | Cùng một quy tắc với ống dài img_029 và tượng ngựa img_028. |
| Người trong tuk-tuk | `person` | không gán (gộp `ignore_region`) | `person` | — | — | **`person`** (R-03, R-08) | Hở hông nên thấy trực tiếp. Bỏ sót người là lỗi nặng nhất. |
| Gương chiếu hậu trong box xe | gồm | không gồm | gồm | — | — | **Gồm** (R-09) | Là bộ phận gắn cố định. "Bao mọi thứ gắn vào xe" dễ áp dụng đồng nhất hơn là liệt kê ngoại lệ. |
| Ngưỡng `occluded_level` | 10/30/70/85 | 10/40/80 | 10/40/80 | 0/30/70 | 0/30/70/90 | **10 / 30 / 70** | Trùng gợi ý của tài liệu lab. Mức 0 có dung sai 10% để vết che nhỏ không gây lệch thuộc tính. |
| Mức `3` = "không rõ"? | — | có (> 80% hoặc không ước lượng được) | không, dùng `can_review` | — | — | **Không** | Một mức chỉ nên có một nghĩa. |
| Nhân vật Predator img_011 | không gán + `can_review` | không gán + `can_review` | không gán (tượng) | `person` | `person` | **Không gán**, theo dõi ở calibration | Kích thước lớn hơn hẳn người, không thấy bộ phận người. Vẫn đưa vào 4.2 vì nhóm chia 3–2. |
| Nhòe chuyển động | — | box theo viền rõ nhất | đặc → bao cả vệt; trong suốt → ignore | — | box nếu nhận ra + `can_review` | **Theo 103** (R-12) | "Đặc hay trong suốt" dễ kiểm hơn "viền rõ nhất". |
| Vật < 20 px đứng riêng | bỏ qua | bỏ qua | `ignore_region` nhỏ | bỏ qua | ignore | **Bỏ qua**; chỉ cụm ≥ 5 → `ignore_region` | Ít vùng ignore thì ít bất đồng `bat_dong_ignore`. |
| Ngưỡng vật "lớn" | chiều cao | — | — | chiều cao | cạnh dài | **Chiều cao** | Đúng với code `scripts/common.py` (trung bình chiều cao hai box). |

### 5.4 Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do (ca nào) |
|---|---|---|
| v0 | Bản giảng viên: 6 lớp, 3 thuộc tính, 3 quy tắc | — |
| 101–105 · Pha 1 | 5 bản cá nhân độc lập | Rà 40 ảnh |
| **v1.0** | Thêm lớp `bus`, `tuk_tuk`, `bicycle` (9 lớp). Ngưỡng `occluded_level` 10/30/70 kèm cách ước lượng. R-01 → R-18. 35 dòng ca khó + 3 ca theo dõi. Bảng chốt bất đồng (5.3). | img_002 (xe chở xe), img_004 / img_035 (xe 3 bánh), img_009 / img_031 (bus), img_023 (xích lô), img_037 (pickup nắp thùng), img_017 / img_027 / img_033 (vật không thật) |
| **v1.1** | Làm rõ cách áp dụng `R-07` (`img_012`, `img_028`), `R-17` (`img_004`, `img_016`, `img_023`, `img_034`), `bus`/`van` (`img_009`, `img_037`), `R-01`/`R-18` (`img_017`, `img_027`), ước lượng `occluded_level` và ranh giới `ignore_region` (`R-13`); giải trình 29 ca `can_review`. | Phân tích 1.449 ca bất thường từ `iaa_summary.md` (`IoU TB = 0.848`, `occluded_level = 47.3%`) trên 37 ảnh |

### 5.5 Tổng kết chỉnh sửa khi Calibration (Pha 5)

Từ kết quả IAA trước calibration (`40` ảnh, `706` cặp ghép, `IoU TB = 0.848`, `Cùng lớp = 96.3%`, `Cùng occluded_level = 47.3%`, `1,449` ca trên `37` ảnh):
1. **Nhầm lớp phương tiện (`26` ca `sai_lop`):** giữ chặt quy tắc đã chốt trong v1.0 — pickup luôn là `truck` (`R-07`: `img_012`, `img_028`), xe 3 bánh có mui che người lái là `tuk_tuk` (`R-17`: `img_004`, `img_016`), xích lô không động cơ là `bicycle` (`img_023`), minibus/xe buýt lớn ≥ 17 chỗ là `bus` (`img_009`, `img_037`).
2. **Thuộc tính `occluded_level` và `truncated` (`414` ca `sai_thuoc_tinh`):** áp dụng thống nhất phép chia 4 đoạn 25% cho người/xe; không để mặc định `0` cho box bị che; bật `truncated = yes` khi vật bị mép ảnh cắt (`R-11`).
3. **Vật giả / ảnh rỗng & `can_review` (`29` ca `co_can_review`):** không gán bất kỳ box hay `ignore_region` nào lên tượng/phản chiếu/mô hình (`R-01`, `R-18`: `img_017`, `img_027` phải để `0` annotation); dãy mô tô độ ở `img_011` giữ `motorcycle` và hạ `can_review = false` sau khi chốt calibration.
4. **Ranh giới box riêng và `ignore_region` (`498` ca `bo_sot_thua`, `220` ca `bat_dong_ignore`):** chỉ tạo box riêng khi chiều cao ≥ 20 px và tách được biên (`R-04`); không đặt `ignore_region` phủ lên đối tượng đã vẽ box riêng (`R-13`).

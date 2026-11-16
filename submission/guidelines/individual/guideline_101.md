# Guideline gán nhãn — v0.1 (cá nhân) — Thành viên 101

- Phiên bản: v0.1 (bản đề xuất cá nhân từ v0)
- Ngày: 2026-11-16
- Người viết: 101

---

## 1. Mục tiêu

Xây dựng bộ nhãn 2D Bounding Box trên ảnh đường phố để huấn luyện mô hình **Object Detection cho hệ thống camera giao thông và nhận thức xe tự hành (AD/ADAS)**.

- **Ứng dụng:** Nhận diện chính xác **người** và **phương tiện đang hoặc có khả năng tham gia giao thông** trên đường.
- **Rủi ro nếu gán sai:**
  - *Dương tính giả (nhận nhầm vật tĩnh thành vật thật):* Gán tượng, ma-nơ-canh, đồ chơi, ảnh biển quảng cáo hay ảnh phản chiếu trong gương thành `person` / `car` khiến mô hình phanh khẩn cấp sai (phantom braking).
  - *Nhầm ranh giới/chồng lấn:* Gộp người lái vào xe máy hoặc kéo box ôm luôn kiện hàng dài quá khổ khiến mô hình ước lượng sai vị trí và kích thước vật cản.
  - *Bỏ sót (False Negative):* Bỏ sót người/xe bị che khuất một phần làm tăng nguy cơ va chạm.

---

## 2. Lớp và thuộc tính

### 2.1. Lớp (Classes)

> **Lưu ý đề xuất sửa Taxonomy ở Pha 2:** Bộ 40 ảnh xuất hiện nhiều **xe buýt** (`img_032.jpg`), **xe 3 bánh / tuk-tuk** (`img_004.jpg`, `img_035.jpg`), **xích lô / xe đạp** (`img_006.jpg`, `img_009.jpg`, `img_029.jpg`). Dưới đây là định nghĩa làm rõ trên bộ lớp của `v0` (kèm phần đề xuất mở rộng lớp khi họp nhóm).

| Lớp | Định nghĩa (một câu) | Ví dụ hợp lệ | Phản ví dụ / Dễ nhầm với |
|---|---|---|---|
| `car` | Ô tô chở khách 4–9 chỗ (sedan, hatchback, SUV, MPV gia đình) và xe bán tải cabin 2 hàng ghế thùng hở không gắn thùng kín thương mại. | Xe tuần tra cảnh sát, ô tô du lịch (`img_013.jpg`, `img_040.jpg`); xe diễu hành có biển số và người lái thật (`img_019.jpg`); xe bán tải thùng hở (`img_001.jpg`, `img_012.jpg`). | Tượng đài xe bán tải (`img_027.jpg`); xe mô hình đồ chơi (`img_038.jpg`); xe phản chiếu qua gương (`img_020.jpg`); xe tải nhẹ có thùng rời (`img_036.jpg`). |
| `truck` | Xe tải chở hàng có khoang/thùng hàng tách biệt với cabin lái (tải nhẹ, tải trung, xe bồn, xe chở xe). | Xe chở ô tô 2 tầng Renault (`img_002.jpg`); xe tải giao hàng trong kho (`img_007.jpg`); xe tải đầu bằng thùng rời (`img_036.jpg`). | Xe bán tải cá nhân chở đồ nhỏ ở thùng sau (`img_012.jpg` $\to$ `car`); xe thùng kín liền khối nhỏ (`van`). |
| `van` | Xe thùng kín liền khối giữa cabin và khoang chở (xe giao hàng cỡ nhỏ, minibus $< 16$ chỗ, bán tải gắn nắp thùng cao kín liền nóc). | Xe bán tải gắn nắp thùng kín `Master 75 Truck Cap` (`img_037.jpg`); xe van/minivan trong dòng xe kẹt (`img_040.jpg`). | Xe buýt cỡ lớn công cộng (`img_032.jpg` — đề xuất thêm lớp `bus`); xe tải thùng vuông tách rời cabin (`truck`). |
| `motorcycle` | Phương tiện cơ giới 2 bánh (xe số, xe ga, xe mô tô phân khối lớn), kể cả xe máy gắn thùng bên (sidecar). | Xe máy chở người trên đường (`img_005.jpg`, `img_026.jpg`); dàn xe máy đỗ/di chuyển (`img_008.jpg`, `img_030.jpg`); mô tô gắn sidecar (`img_022.jpg`). | Xe đạp thường (`img_029.jpg` phải); xích lô 3 bánh đạp chân (`img_006.jpg`); xe tuk-tuk 3 bánh (`img_004.jpg`, `img_035.jpg`). |
| `person` | Con người bằng xương bằng thịt (người đi bộ, người đứng bên đường, người lái xe hoặc ngồi trên xe). | Người đi bộ băng qua đường (`img_014.jpg`), người bán hàng (`img_003.jpg`, `img_010.jpg`), người ngồi trên xe máy/xích lô/tuk-tuk (`img_004.jpg`, `img_026.jpg`, `img_034.jpg`). | Ma-nơ-canh trong tủ kính (`img_033.jpg`); tượng đồng (`img_017.jpg`); ảnh người trên billboard (`img_018.jpg`); mascot chuột (`img_019.jpg`); tượng/hóa trang Predator (`img_011.jpg` cần chốt). |
| `ignore_region` | Vùng tập trung nhiều người/xe quá nhỏ ($< 20\text{ px}$ chiều cao) hoặc chồng chéo dày đặc không thể tách box từng cá thể. | Đám đông khán giả đứng xa sau hàng rào (`img_019.jpg`), hàng trăm xe máy/người nhỏ tít hậu cảnh (`img_014.jpg`, `img_030.jpg`, `img_040.jpg`). | Vật thể đơn lẻ rõ ràng có chiều cao $\ge 20\text{ px}$. |

**Vật không thuộc lớp nào của `v0`:**
- **Xe buýt (`img_032.jpg`), Xe 3 bánh Tuk-tuk/Auto-rickshaw (`img_004.jpg`, `img_035.jpg`), Xích lô & Xe đạp (`img_006.jpg`, `img_009.jpg`, `img_023.jpg`, `img_029.jpg`, `img_034.jpg`, `img_039.jpg`):**
  - *Đề xuất cá nhân cho Pha 2:* Bổ sung thêm `bus` và `bicycle` (hoặc `three_wheeler`) vào `taxonomy.json`. Nếu giữ nguyên 5 lớp `v0`, **không gán ép** vào lớp khác; ghi lại vào `image_issues.csv` hoặc thống nhất một quy ước duy nhất (vd không gán xe đạp/xích lô, nhưng **vẫn gán `person`** cho người lái ngồi trên đó).

### 2.2. Thuộc tính (Attributes)

| Thuộc tính | Các mức cố định | Định nghĩa & Ngưỡng đo cụ thể |
|---|---|---|
| `occluded_level` | `0`, `1`, `2`, `3` | • `0`: Thấy rõ toàn bộ hoặc che $< 10\%$.<br>• `1`: Che khuất ít (**$10\% \text{ – } 30\%$** diện tích vật).<br>• `2`: Che khuất nhiều (**$31\% \text{ – } 70\%$** diện tích, vẫn nhận rõ lớp).<br>• `3`: Che rất nặng (**$71\% \text{ – } 85\%$** diện tích, chỉ thấy 1 phần nhận diện được; nếu che $> 85\%$ và không rõ hình dạng $\to$ bỏ qua). |
| `truncated` | `no`, `yes` | • `no`: Vật nằm trọn bên trong khung ảnh (hoặc hở dưới $2\text{ px}$ sát viền).<br>• `yes`: Vật bị **cắt mất một phần bởi cạnh khung ảnh** (box chạm sát mép ảnh). |
| `can_review` | `false` / `true` | • Tick `true` khi gặp tình huống mơ hồ chưa có quy tắc dứt khoát (ví dụ: ranh giới `van`/`truck`, người mặc đồ hóa trang `img_011.jpg`) để đưa vào danh sách họp calibration. |

---

## 3. Quy tắc vẽ box

- **`R-01` (Ôm sát phần nhìn thấy):** Box chữ nhật chạm sát biên ngoài cùng nhìn thấy được của vật thể (bánh xe, gương, mái tóc, ba-đờ-sốc), khe hở không quá $3\text{–}5\text{ px}$. Không bao gồm bóng đổ trên mặt đường hay phần bị vật khác che khuất (không đoán phần bị ẩn).
- **`R-02` (Cắt mép ảnh):** Vật chạm cạnh ảnh thì vẽ box chạm đúng mép ảnh (không vượt ra ngoài $[0, W]$ và $[0, H]$) và đặt `truncated = yes`.
- **`R-03` (Tách người và phương tiện):** Khi người ngồi trên xe máy, xích lô, tuk-tuk hoặc trong ô tô mui trần (`img_005.jpg`, `img_019.jpg`, `img_026.jpg`), **tách riêng**: vẽ 1 box `person` cho từng người nhìn thấy được (từ đầu đến phần thân/chân lộ ra) và 1 box riêng cho phương tiện. Không gộp người lái vào hộp xe.
- **`R-04` (Vật giả / Phi thực thể — Không gán):** Tuyệt đối **không gán** tượng người/ngựa (`img_017.jpg`, `img_028.jpg`), tượng đài xe (`img_027.jpg`), ma-nơ-canh (`img_033.jpg`), đồ chơi/mô hình trong tủ kính (`img_038.jpg`), hình người in trên billboard (`img_018.jpg`), nhân vật mascot (`img_019.jpg`) và **ảnh phản chiếu** trong gương chiếu hậu hoặc mặt kính (`img_020.jpg`, `img_033.jpg`).
- **`R-05` (Xe chủ Ego & Xe chở xe):** Không gán cho bộ phận của xe chụp ảnh (gương/nắp ca-pô của xe chủ như `img_020.jpg`). Với xe tải chở nhiều ô tô (`img_002.jpg`), chỉ vẽ 1 box `truck` bao trùm toàn bộ xe tải + hàng ô tô chở trên sàn.
- **`R-06` (Ngưỡng kích thước tối thiểu & `ignore_region`):** Khớp với `taxonomy.json`: bỏ qua vật thể đơn lẻ có chiều cao **$< 20\text{ px}$**. Khi có nhóm từ **$\ge 5$ người hoặc xe** ở hậu cảnh xa cao $< 20\text{ px}$ hoặc đứng dính nhau không thể tách riêng (`img_019.jpg`, `img_025.jpg`, `img_030.jpg`, `img_040.jpg`), vẽ 1 box `ignore_region` bao cụm đó.
- **`R-07` (Hàng hóa cồng kềnh gắn trên xe):** Với xe máy/xe chở hàng dài nhô quá xa khỏi thân xe (`img_029.jpg` chở thanh nhôm dài hàng mét), box `motorcycle` chỉ ôm khung xe và phần hàng gắn trực tiếp trong giới hạn thân xe, không kéo box dài ra hết thanh kim loại thừa ra ngoài.

---

## 4. Ca khó (Phát hiện khi rà 40 ảnh)

| Ảnh | Tình huống | Rule ID | Quyết định (cá nhân 101) | Lý do |
|---|---|---|---|---|
| `img_002.jpg` | Xe đầu kéo Renault chở 7 ô tô con trên 2 tầng sàn | `R-05` | Vẽ **1 box `truck`** trùm cả xe tải và 7 ô tô trên sàn; không vẽ 7 box `car` con. | 7 ô tô là hàng hóa tĩnh đang được vận chuyển, không phải phương tiện tự lưu thông độc lập. |
| `img_004.jpg`, `img_035.jpg` | Xe 3 bánh Tuk-tuk / Auto-rickshaw chở khách và hàng | *(Đề xuất)* | Đề xuất thêm lớp `three_wheeler` ở Pha 2. Nếu giữ taxonomy `v0`, gán vào `ignore_region` hoặc bỏ qua + tick `can_review` (cần nhóm thống nhất). | `v0` chỉ có `car/truck/van/motorcycle`, ép Tuk-tuk vào `car` hay `motorcycle` sẽ làm nhiễu đặc trưng lớp. |
| `img_006.jpg`, `img_009.jpg`, `img_023.jpg`, `img_034.jpg` | Xe xích lô 3 bánh có người đạp và hành khách | `R-03` | Đề xuất thêm `bicycle`/`cyclo` ở Pha 2. Với `v0`: **vẫn vẽ `person`** cho người đạp và khách, không gán xích lô vào `motorcycle`. | `person` vẫn là đối tượng cần phát hiện dù ngồi trên phương tiện phi cơ giới. |
| `img_011.jpg` | Mô hình/người hóa trang Predator khổng lồ bên cạnh dàn mô tô độ | `R-04` | Không gán `person` cho tượng/hóa trang Predator (tick `can_review` nếu nhóm phân vân người thật mặc giáp); vẫn gán `motorcycle` cho các xe độ có bánh/khung rõ và `person` cho khách tham quan. | Ngoại hình giáp quái vật không mang đặc trưng `person` giao thông chuẩn, dễ làm nhiễu phân lớp. |
| `img_017.jpg`, `img_027.jpg`, `img_028.jpg` | Tượng đài người cưỡi ngựa (`img_017`), bia mộ xe bán tải (`img_027`), xe bán tải gắn tượng ngựa trên thùng (`img_028`) | `R-04` | **Không gán** các bức tượng; ở `img_028.jpg` chỉ vẽ box `car` (hoặc `truck`) cho chiếc xe bán tải thật bên dưới (không kéo hộp lên tới đỉnh tượng ngựa trừ khi coi là hàng chở). | Tượng đá/đồng là vật tĩnh vô tri, không phải người hoặc xe tham gia giao thông. |
| `img_018.jpg`, `img_033.jpg`, `img_038.jpg` | Biển quảng cáo có mặt người (`img_018`), ma-nơ-canh & phản chiếu xe trên kính (`img_033`), đồ chơi ô tô/búp bê tủ kính (`img_038`) | `R-04` | **Tuyệt đối không gán** toàn bộ ảnh này (ảnh trống hợp lệ trong COCO). | Đều là hình in 2D, tượng nhựa, mô hình thu nhỏ hoặc ảnh phản chiếu — phản ví dụ điển hình cần loại trừ. |
| `img_019.jpg` | Xe diễu hành màu vàng hình siêu xe Dangermouse có 1 người lái và 1 mascot chuột | `R-03`, `R-04` | Vẽ `car` cho xe vàng (`truncated=no`), vẽ `person` cho tài xế thật, **không gán** mascot chuột; đám đông sau rào vẽ `ignore_region`. | Xe chạy thật có biển số `YR04 NUM`, còn mascot là mô hình bông/nhựa. |
| `img_020.jpg` | Gương chiếu hậu xe chủ và phản chiếu 1 ô tô trong gương | `R-04`, `R-05` | **Không gán** cả gương xe chủ lẫn ảnh ô tô phản chiếu trong gương. | Gương xe chủ là phần gắn cố định với camera; xe trong gương là ảnh ảo ngược chiều/méo hình. |
| `img_022.jpg` | Xe mô tô gắn thùng bên (sidecar) | `R-01` | Vẽ **1 box `motorcycle`** bao trùm cả xe máy lẫn phần thùng sidecar gắn liền, tách riêng `person` ngồi trên xe. | Sidecar và xe máy tạo thành một phương tiện gắn kết cứng trên mặt đường. |
| `img_029.jpg` | Xe máy chở thanh nhôm dài vắt ngang vai + người đi xe đạp ngay bên cạnh | `R-03`, `R-07` | Tách riêng `person` (người lái xe máy) và `motorcycle` (chỉ ôm thân xe, **không kéo dài theo thanh nhôm**); người đi xe đạp vẽ `person`. | Kéo box theo thanh nhôm dài tạo box rỗng > 70% nền đường, giảm mạnh độ chính xác IoU của detector. |
| `img_032.jpg` | Xe buýt công cộng ở trạm chờ đông người | *(Đề xuất)* | Đề xuất thêm lớp `bus` vào `taxonomy.json` ở Pha 2 (nếu giữ `v0`, cần nhóm thống nhất có gán tạm `van`/`truck` hay bỏ qua). | Xe buýt xuất hiện rõ nét và khác biệt hoàn toàn kích thước với `van` 7-16 chỗ. |
| `img_036.jpg`, `img_037.jpg` | Xe tải nhỏ đầu bằng `Veryca` (`img_036`) vs Xe bán tải gắn nắp thùng cao `Truck Cap` (`img_037`) | *(Taxonomy)* | `img_036.jpg` gán **`truck`** (thùng hàng rời cabin); `img_037.jpg` gán **`van`** (hoặc `car` tùy nhóm chốt vì thân bán tải gắn nắp kín liền khối). | Phân biệt rạch ròi giữa `car` – `van` – `truck` là nguyên nhân gây bất đồng cao nhất khi đo IAA. |

---

## 5. QC và phiên bản

- **Ngưỡng QC áp dụng (lấy từ `submission/taxonomy.json`):**
  - `min_box_height_px`: **20 px**
  - `match_iou`: **0.50**
  - `iaa_iou`: **0.85** (`iou_thr_large = 0.85` cho vật $\ge 50\text{ px}$; `iou_thr_small = 0.70` cho vật $< 50\text{ px}$)
  - `iaa_class`: **0.95** (tối thiểu 95% các cặp box ghép khớp phải trùng lớp)
  - `error_rate_img_max`: **0.02** ($\le 2\%$ số ảnh lỗi khi QC bàn giao)
- **Người quyết định ca mơ hồ:** Họp nhóm thống nhất ở Pha 2 (`v1.0`) và Calibration ở Pha 5 (`v1.1`); mã `103` cập nhật tài liệu, mã `101` điều phối, mã `105` kiểm lại chéo độc lập ở Pha 6.

| Phiên bản | Thay đổi | Lý do (ca nào) |
|---|---|---|
| v0 | Bản khởi đầu của giảng viên | Thiếu ngưỡng che khuất, thiếu quy tắc người+xe, chưa xử lý ca ngoại lệ |
| v0.1 (`101`) | Bổ sung ngưỡng `occluded_level`, `R-01`–`R-07`, 12 nhóm ca khó từ 40 ảnh, đề xuất bổ sung lớp `bus`/`bicycle`/`three_wheeler` | Từ rà soát 40 ảnh `img_001`–`img_040` |

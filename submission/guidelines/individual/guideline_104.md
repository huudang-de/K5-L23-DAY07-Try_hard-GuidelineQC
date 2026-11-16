# Guideline gán nhãn — v1.0 — 104

- **Phiên bản:** v1.0
- **Ngày:** 16/11/2026
- **Người thực hiện:** Thành viên 104
- **Công cụ:** CVAT (xuất định dạng COCO 1.0)

---

## 1. Mục tiêu
- **Mục đích:** Cung cấp dữ liệu gán nhãn 2D Bounding Box chuẩn xác để huấn luyện mô hình phát hiện người và phương tiện giao thông trên đường phố đô thị.
- **Rủi ro an toàn:** 
  - Bỏ sót (`False Negative`) đối tượng dễ bị tổn thương như người đi bộ hoặc người đi xe máy dẫn đến nguy cơ va chạm gây chấn thương nghiêm trọng hoặc tử vong.
  - Bounding box vẽ lỏng lẻo, lệch tâm hoặc gán nhầm lớp (`Misclassification`) làm sai lệch ước lượng khoảng cách và thời gian va chạm (Time-to-Collision - TTC), khiến hệ thống điều khiển phanh khẩn cấp kích hoạt sai thời điểm hoặc không kích hoạt.

---

## 2. Lớp và thuộc tính

### 2.1. Lớp (Classes)

| Lớp | Định nghĩa (một câu) | Ví dụ | Phản ví dụ / Dễ nhầm với |
|---|---|---|---|
| `car` | Phương tiện cơ giới 4 bánh chở người cỡ nhỏ/trung bình (sedan, hatchback, SUV, coupe, xe bán tải pickup). | Xe taxi, sedan 4 chỗ, SUV 7 chỗ, xe bán tải (`img_001`, `img_037`). | Xe thùng kín nhỏ (`van`), xe tải thùng hàng riêng (`truck`), mô hình đồ chơi (`img_038`). |
| `truck` | Phương tiện cơ giới chuyên dụng chở hàng có cabin riêng biệt tách rời với thùng hàng hoặc rơ-moóc phía sau. | Xe tải chở hàng, xe đầu kéo chuyên chở ô tô (`img_002`), xe tải thùng kín (`img_007`). | Xe bán tải nhỏ (`car`), xe tải van nhỏ liền một khối cabin (`van`). |
| `van` | Phương tiện cơ giới chở người hoặc chở hàng cỡ trung có thân xe dạng hộp kín một khối thống nhất (không tách rời cabin và khoang sau). | Xe cứu thương, Ford Transit, minivan, xe giao hàng dạng hộp kín liền khối. | Xe con SUV (`car`), xe tải chở hàng có thùng rời (`truck`). |
| `motorcycle` | Phương tiện cơ giới hai hoặc ba bánh gắn động cơ. | Xe số, xe tay ga, mô tô phân khối lớn, xe ba bánh máy / tuk-tuk (`img_004`, `img_035`). | Xe đạp thường, xe xích lô đạp chân chở khách (`cyclo` — không động cơ). |
| `person` | Cơ thể người thật đang sống (người đi bộ, đứng, ngồi, lái xe hoặc hành khách). | Người đi bộ qua đường (`img_014`), người lái xe máy (`img_005`), người ngồi sau xe máy (`img_026`). | Ma-nơ-canh cửa hàng (`img_033`), tượng người bằng đồng/đá (`img_017`), hình ảnh người in trên poster quảng cáo (`img_018`). |
| `ignore_region` | Vùng ảnh không yêu cầu gán nhãn chi tiết từng vật đơn lẻ; việc bỏ sót vật thể trong vùng này không tính là lỗi QC. | Đám đông người hoặc xe quá xa/dày đặc không thể tách rời, phương tiện ngoài taxonomy xuất hiện lớn trong ảnh. | Phương tiện đơn lẻ, rõ nét (bắt buộc phải gán box riêng). |

> **Quy tắc xử lý vật thể ngoài Taxonomy (`R-00`):**
> - Các phương tiện như xe buýt (`bus`), xe xích lô (`cyclo`), xe đạp (`bicycle`) không nằm trong 5 lớp đối tượng chính của taxonomy.
> - **Xử lý:** Gán vào vùng `ignore_region` nếu xuất hiện rõ ràng hoặc cản trở tầm nhìn, nhằm tránh mô hình bị phạt lỗi `False Positive` (nhận nhầm) hoặc `False Negative` (bỏ sót).

---

### 2.2. Thuộc tính (Attributes)

| Thuộc tính | Kiểu dữ liệu | Các mức cố định | Định nghĩa & Ngưỡng định lượng cụ thể |
|---|---|---|---|
| `occluded_level` | Select | `0`, `1`, `2`, `3` | **`0`**: Nhìn thấy toàn vẹn **100%** diện tích trực diện của vật thể (không bị che).<br/>**`1`**: Bị che nhẹ từ **1% đến 30%** (vẫn nhận diện đầy đủ và dễ dàng mọi đặc trưng chính).<br/>**`2`**: Bị che vừa từ **31% đến 70%** (khuyết nhiều bộ phận nhưng vẫn suy luận được qua ngữ cảnh).<br/>**`3`**: Bị che nặng **> 70%** (chỉ còn lộ một phần rất nhỏ cơ thể hoặc thân xe). |
| `truncated` | Select | `no`, `yes` | **`no`**: Toàn bộ thân vật thể nằm trọn vẹn bên trong khung hình.<br/>**`yes`**: Ít nhất một cạnh của vật thể bị cắt ngang bởi mép viền ảnh. |
| `can_review` | Checkbox | `true`, `false` | **`true`**: Annotator không chắc chắn về lớp, ranh giới box hoặc gặp ca biên mới, cần gắn cờ để Reviewer/nhóm xem xét.<br/>**`false`**: Tự tin đúng theo quy tắc đã ban hành. |

---

## 3. Quy tắc vẽ box

- **`R-01` (Visible Boundary — Ôm sát phần nhìn thấy):** Bounding box chỉ bao quanh phần pixel nhìn thấy được của vật thể (`visible bounding box`), không cố mở rộng ra vùng không gian bị che khuất ngầm. Cả 4 cạnh của box phải tiếp xúc chính xác với mép ngoài cùng của vật.
- **`R-02` (Rider & Vehicle — Tách riêng người và xe):** Đối với xe máy/mô tô có người ngồi trên:
  - Gán **1 box `person`** ôm từ đỉnh đầu (hoặc mũ bảo hiểm) đến điểm thấp nhất của người (chân/giày).
  - Gán **1 box `motorcycle`** ôm trọn toàn bộ xe (từ bánh trước, tay lái đến đuôi xe).
  - Hai box này đè lên nhau là hoàn toàn hợp lệ. Nếu có người ngồi sau (hành khách), gán thêm 1 box `person` riêng cho người ngồi sau (`img_026`).
- **`R-03` (Car Carrier — Xe chở xe):** Xe đầu kéo chở ô tô được gán là `truck`. Các xe ô tô con nằm trên rơ-moóc/giá chở nếu còn nguyên vẹn và nhìn thấy được thì gán từng box `car` riêng biệt, gán thuộc tính `occluded_level` tương ứng với độ che khuất thực tế (`img_002`).
- **`R-04` (Cargo & Luggage — Hàng hóa trên phương tiện):**
  - Hàng hóa nằm trên thùng xe tải hoặc xe bán tải: Box `truck`/`car` bao gồm cả thùng xe và hàng hóa gắn liền trên xe (`img_012`).
  - Hàng hóa cồng kềnh trên xe máy (`img_029`): Tuyệt đối không gộp hàng hóa vào box `person`. Box `motorcycle` ôm phần xe nhìn thấy được; nếu kiện hàng quá khổ che khuất xe nghiêm trọng, gán vùng hàng hóa đó là `ignore_region`.
- **`R-05` (Non-real Objects — Vật thể không có thực):** Tượng đài bằng đá/đồng (`img_017`, `img_027`), ma-nơ-canh trong tủ kính (`img_033`), tranh ảnh chân dung/billboard quảng cáo (`img_018`), mô hình đồ chơi thu nhỏ (`img_038`): **TUYỆT ĐỐI KHÔNG GÁN BOX** (hoặc đánh dấu `ignore_region` nếu nằm sát mặt đường dễ gây nhiễu). Lý do: Tránh mô hình xe tự lái nhận nhầm và kích hoạt phanh giả.
- **`R-06` (Reflection — Hình ảnh phản chiếu):** Hình phản chiếu người hoặc xe qua gương chiếu hậu (`img_020`), mặt kính cửa sổ hoặc vũng nước trên mặt đường: **KHÔNG GÁN BOX**.
- **`R-07` (Minimum Size — Ngưỡng kích thước tối thiểu):** Vật thể có **chiều cao < 20 pixels** (`min_box_height_px = 20` theo taxonomy): Bỏ qua không gán nhãn đơn lẻ; nếu là cụm đông đúc ở hậu cảnh xa thì khoanh vào vùng `ignore_region` (`img_025`).
- **`R-08` (3-Wheel Vehicles — Xe ba bánh & Tuk-tuk):** Xe tuk-tuk hoặc xe ba bánh máy (`img_004`, `img_035`): Gán nhãn lớp `motorcycle` vì cùng thuộc nhóm phương tiện cơ giới cỡ nhỏ gọn gàng, có cơ cấu điều khiển ghi-đông và động cơ xe máy.
- **`R-09` (Costume / Mascot — Nhân vật hóa trang):** Người mặc trang phục hóa trang/mascot đi lại trên đường (`img_011`): Gán nhãn `person` vì đây là người thật đang tham gia không gian công cộng, có khả năng di chuyển và gây nguy cơ va chạm.
- **`R-10` (Ego Vehicle — Phương tiện gắn camera):** Nắp capo hoặc gương chiếu hậu của chính xe gắn camera xuất hiện ở phần dưới ảnh: Không gán box hoặc đánh dấu `ignore_region`.

---

## 4. Ca khó

| Ảnh | Tình huống | Rule ID | Quyết định | Lý do |
|---|---|---|---|---|
| `img_002.jpg` | Xe tải đầu kéo chở nhiều ô tô con trên giá | `R-03` | 1 box `truck` cho xe tải lớn; mỗi ô tô con trên giá gán 1 box `car` riêng biệt. | Ô tô trên giá có đầy đủ đặc trưng thị giác của `car`, hệ thống cần nhận diện để phòng tránh va chạm trên cao. |
| `img_004.jpg` | Xe Tuk-tuk xanh đậu trên vỉa hè | `R-08` | Gán lớp `motorcycle`. | Xe 3 bánh máy chia sẻ chung cơ cấu điều khiển và động cơ tương đương mô tô trong taxonomy. |
| `img_005.jpg` | Đám đông xe máy lưu thông tại TP.HCM | `R-02`, `R-01` | Tách rời: mỗi người ngồi trên xe gán 1 box `person`, mỗi xe gán 1 box `motorcycle`. | Tách biệt giúp mô hình học phát hiện người đi bộ độc lập với phát hiện phương tiện. |
| `img_011.jpg` | Nhân vật Predator cosplay đứng cạnh hàng xe máy | `R-09` | Gán `person` cho nhân vật Predator; gán `motorcycle` cho xe máy. | Có người thật bên trong trang phục, có hành vi di chuyển và rủi ro va chạm như người thật. |
| `img_012.jpg` | Xe bán tải chở thùng hàng nông sản đầy ắp | `R-04`, `R-01` | Gán `car` (pickup), bounding box bao gồm cả phần hàng hóa nằm trên thùng. | Hàng nằm gọn trên thùng xe, kích thước box đại diện cho toàn bộ khối tích cản trở không gian di chuyển. |
| `img_017.jpg` | Tượng Duke of Wellington cưỡi ngựa bằng đồng | `R-05` | **Không gán box** (bỏ qua). | Tượng điêu khắc bất động, không tham gia giao thông; tránh mô hình báo động giả (False Positive). |
| `img_018.jpg` | Billboard áp phích chân dung chính trị gia | `R-05` | **Không gán box** (bỏ qua). | Hình ảnh in 2D trên biển bảng quảng cáo phẳng, không phải người thật. |
| `img_020.jpg` | Cầu vồng và ngoại cảnh phản chiếu trong gương xe | `R-06` | **Không gán box** (bỏ qua). | Phản xạ quang học, không tồn tại vật thể vật lý thực tại vị trí phản chiếu. |
| `img_026.jpg` | Cặp đôi nam nữ cùng ngồi trên một xe máy | `R-02` | Gán 2 box `person` (người lái và người ngồi sau), 1 box `motorcycle`. | Cả hai đối tượng đều là người tham gia giao thông cần được hệ thống bảo vệ. |
| `img_027.jpg` | Tượng đài xe bán tải tạc bằng đá hoa cương | `R-05` | **Không gán box** (bỏ qua). | Tượng đá cố định trong công viên tưởng niệm, không phải phương tiện vận tải cơ giới. |
| `img_029.jpg` | Xe máy chở bó hàng cồng kềnh chiếm diện tích lớn | `R-04`, `R-02` | 1 box `person`, 1 box `motorcycle` ôm phần xe nhìn thấy được. | Bounding box `person` không được ôm hàng hóa; giữ ranh giới phần cơ giới xe máy chuẩn xác. |
| `img_033.jpg` | Ma-nơ-canh trưng bày quần áo trong tủ kính | `R-05` | **Không gán box** (bỏ qua). | Ma-nơ-canh bất động bên trong nhà, không phải người thật tham gia giao thông. |
| `img_038.jpg` | Tủ kính trưng bày mô hình ô tô đồ chơi và búp bê | `R-05` | **Không gán box** (bỏ qua). | Đồ chơi mô hình thu nhỏ trong nhà, không thuộc phạm vi đối tượng đường phố. |

> *Ghi chú:* Nếu gặp ca biên chưa được quy định rõ trong bảng trên, annotator tích chọn `can_review = true` trên box hoặc ghi chú cấp ảnh để đưa ra phiên Calibration của nhóm.

---

## 5. QC và phiên bản

### 5.1. Ngưỡng QC áp dụng (Theo chuẩn `taxonomy.json` & Lab Rubric)
- **IoU chấp nhận:** 
  - Vật thể lớn (chiều cao box $\ge 50\text{ px}$): $\text{IoU} \ge 0.85$.
  - Vật thể nhỏ (chiều cao box $20\text{ px} \le h < 50\text{ px}$): $\text{IoU} \ge 0.70$.
- **Độ đồng thuận liên annotator (IAA):**
  - $\text{Mean IoU} \ge 0.85$ (trên các cặp box ghép 1–1 với $\text{IoU} \ge 0.5$).
  - Tỉ lệ trùng khớp lớp (Class Agreement) $\ge 95\%$.
- **Ngưỡng nghiệm thu Release:** Tỉ lệ ảnh có lỗi $\le 2\%$ trên tổng số 40 ảnh đã duyệt (tối đa 0 ảnh lỗi trong lô 40 ảnh để đạt chuẩn Release ngay từ vòng 1).

### 5.2. Lịch sử thay đổi phiên bản
| Phiên bản | Thay đổi chính | Lý do / Căn cứ |
|---|---|---|
| `v0` | Bản khởi đầu từ giảng viên | Bản sơ khởi, chưa định nghĩa ca biên, chưa phân biệt rider/xe, chưa có định lượng thuộc tính. |
| `v1.0` | Hoàn thiện định lượng 4 mức che khuất (`occluded_level`), bổ sung 10 quy tắc từ `R-01` đến `R-10`, xử lý triệt để các ca biên đặc biệt (tượng, billboard, đồ chơi, xe chở xe, tuk-tuk). | Phân tích 40 bức ảnh thực tế và các tình huống giao thông đô thị phức tạp. |

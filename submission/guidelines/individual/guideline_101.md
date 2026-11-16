# Guideline gán nhãn — v0.2 (bản cá nhân) — 102

- Phiên bản: v0.2 (cá nhân, viết độc lập ở Pha 1)
- Ngày: 2026-11-16
- Người viết / duyệt: 101 / chưa duyệt
- Loại nhãn: bounding box 2D trên ảnh đường phố, công cụ CVAT, xuất COCO 1.0

## 1. Mục tiêu

Nhãn dùng để huấn luyện mô hình phát hiện **người và phương tiện thật đang ở ngoài đường** (camera giao thông, xe tự lái).

Hai loại sai nguy hiểm nhất:
- **Dương tính giả**: gán nhãn cho tượng, đồ chơi, hình in, phản chiếu. Mô hình sẽ học "tượng ngựa = người", "xe mô hình = ô tô".
- **Âm tính giả**: bỏ sót người đi xe máy hoặc xe nhỏ ở xa, nhất là trong đám đông xe máy.

Nguyên tắc chung: **chỉ gán vật thật, 3D, nằm trong cảnh, nhìn thấy được ít nhất một phần**. Không đoán phần không thấy.

## 2. Lớp và thuộc tính

### Lớp (mỗi vật đúng một lớp)

| Lớp | Định nghĩa (một câu) | Ví dụ | Phản ví dụ / dễ nhầm |
|---|---|---|---|
| `car` | Ô tô chở người có mui kín, gầm thấp: sedan, hatchback, SUV, taxi 4 chỗ, bán tải cabin kép chở người. | img_015 (xe đỏ, xe bạc), img_023 (Honda đen), img_031 | Xe mô hình (img_038), xe gắn trên nóc xe (img_024) → xem R-04. |
| `truck` | Xe có thùng hàng hở/riêng phía sau cabin, hoặc xe chở hàng nặng: xe tải, xe bán tải thùng hở, xe chở xe, xe ba bánh chở hàng. | img_002 (xe chở xe), img_001 (Suzuki thùng hở), img_036 (xe tải xanh), img_035 (ba bánh hàng) | Van (thùng liền cabin, nhỏ). |
| `van` | Xe thùng kín liền khối, cỡ nhỏ-vừa, chở người hoặc hàng nhẹ: minivan, xe 16 chỗ, xe buýt mini. | img_005 (van bạc), img_009 (buýt mini) | `car` (nhỏ hơn, không có thùng dài); `truck` (thùng tách cabin). |
| `motorcycle` | Xe hai bánh có động cơ (xe máy, mô tô, xe tay ga), kể cả xe có thùng bên hông (sidecar). | img_006, img_008, img_022 | **Xe đạp** (không có lớp, xem dưới). |
| `person` | Người thật: đi bộ, đứng, ngồi, hoặc ngồi/lái trên xe. | img_014, img_034, img_032 | Ma-nơ-canh, tượng, hình người in (R-04). |
| `ignore_region` | Vùng có nhiều vật nhỏ/dày đặc không thể gán từng vật chính xác. | img_030 (bãi xe), img_032 (đám đông xa) | Không dùng để "né" vật rõ ràng. |

Xe buýt cỡ lớn (img_031, img_037 xe buýt xanh, img_040 buýt phía sau) chưa có lớp riêng. **Quyết định ban đầu:** gán `truck` cho buýt cỡ lớn, `van` cho buýt mini; đánh `can_review` và hỏi reviewer có cần thêm lớp `bus` không.

### Vật không thuộc lớp nào

| Vật | Cách xử lý |
|---|---|
| Xe đạp, xích lô, xe ba gác đạp (img_003, img_023, img_029) | **Không gán** xe. Người đạp xe vẫn gán `person`. Đề xuất thêm lớp `bicycle` — đánh `can_review`. |
| Xe ba bánh có động cơ (tuk-tuk, xe lam: img_004, img_016, img_035) | Gán `truck` nếu chở hàng (img_035); gán `car` nếu chở người (img_004). Đánh `can_review`. |
| Vật không phải người/xe (cọc tiêu, biển, cây, thùng rác) | Không gán. |
| Vật trông giống người/xe nhưng không thật (R-04) | Không gán. |

### Thuộc tính

**`occluded_level`** — tính theo **% diện tích vật bị che**, ước lượng so với toàn bộ vật (kể cả phần bị che):

| Mức | Nghĩa | Ngưỡng |
|---|---|---|
| `0` | Thấy rõ | che < 10% |
| `1` | Che ít | 10–40% |
| `2` | Che nhiều | 40–75% |
| `3` | Che gần hết, chỉ thấy một phần nhỏ | > 75% |

Ví dụ: img_005 người ngồi sau trong hàng xe máy che ~50% → `2`; img_030 xe máy đỗ trong bãi che chồng nhau → `2–3`.

**`truncated`** — `yes` nếu vật bị **mép ảnh** cắt mất một phần; `no` nếu không. Che bởi vật khác thì **không** tính là truncated, dùng `occluded_level`. Ví dụ img_015 (hai ô tô sát mép trái/phải) → `yes`; img_005 (người trái dưới) → `yes`.

**`can_review`** — tick khi: lớp mơ hồ, mức che nằm sát ngưỡng, vật nghi ngờ thật/giả, hoặc vật nhỏ sát `min_box_height_px`. Tick còn hơn đoán bừa.

## 3. Quy tắc vẽ box

- **R-01 Box ôm sát phần nhìn thấy.** Hộp vừa chạm mép ngoài của phần thấy được; không chừa lề, không cắt vào vật. Không suy đoán phần bị che hoặc bị cắt.
- **R-02 Người đi xe máy: hai box riêng.** Một box `motorcycle` ôm xe (không gồm người), một box `person` ôm người (gồm đầu, thân, chân thấy được). Hai box chồng lên nhau là bình thường. Người ngồi sau cũng một box `person` riêng. Áp dụng cho img_005, img_006, img_008, img_021, img_026, img_035.
- **R-03 Hàng hóa và vật mang theo không tính vào box.** Giỏ hàng, bao tải, ống dài (img_001, img_012, img_029, img_035) nằm ngoài box xe/người. Riêng thùng xe tải là một phần của xe nên được bao vào.
- **R-04 Vật không thật thì không gán.** Gồm: tượng (img_017), ma-nơ-canh (img_033), xe mô hình/đồ chơi (img_038), xe điêu khắc (img_027 mộ hình xe tải), mô hình xe trên nóc xe thật (img_024), điêu khắc kim loại (img_011), hình người in trên biển quảng cáo (img_018) và áp phích (img_030). Nếu vật vẫn **di chuyển được trên đường như xe thật** (xe hoa lễ hội có người lái, img_019) thì gán `car` và đánh `can_review`.
- **R-05 Phản chiếu thì không gán.** Xe/người thấy qua gương chiếu hậu (img_020), kính cửa hàng (img_033, img_038), vũng nước (img_013, img_031). Ngoại lệ: không có.
- **R-06 Xe/người bị che.** Vẫn gán nếu còn thấy ≥ 10% diện tích và đủ lớn (R-08). Box chỉ ôm phần thấy. Dưới 10% hoặc không nhận ra loại → bỏ, không đoán.
- **R-07 Bị cắt mép ảnh.** Box vẽ đến sát mép ảnh, `truncated = yes`. Cắt quá nửa nhưng vẫn nhận ra loại → vẫn gán, tick `can_review`.
- **R-08 Vật quá nhỏ.** Box cao dưới `min_box_height_px` (20 px) thì **không gán**, không đưa vào `ignore_region`. Nếu từ 20 px trở lên nhưng mờ, nhìn không rõ loại → gán lớp đoán tốt nhất và tick `can_review`.
- **R-09 Đám đông / hàng xe dày.** Gán riêng từng vật nếu thấy rõ ranh giới và đủ lớn. Phần xa, dày đặc, không phân tách được (img_032 đám người đứng chờ xa, img_021 phía sau cùng, img_030 bãi xe) → một hoặc vài `ignore_region` ôm cụm đó. Ranh giới: người/xe ở **tiền cảnh và trung cảnh** luôn gán riêng.
- **R-10 Xe chở xe (img_002).** Gán **một box `truck`** cho cả xe đầu kéo và khung chở. Các xe nhỏ đang chở trên đó **gán riêng `car`** (đánh `can_review`, `occluded_level` theo thực tế), vì chúng là xe thật có thể nhận diện độc lập. Nếu nhóm quyết ngược lại thì sửa ở v1.0.
- **R-11 Xe chủ (ego).** Nếu thấy phần xe của camera (đầu/mui/gương xe quay phim) thì **không gán**. Gương và khung cửa xe ở img_020 không gán.
- **R-12 Xe đỗ / xe trong nhà.** Vẫn gán như xe đang chạy (img_007, img_008, img_030). Mục tiêu là nhận diện vật, không phải trạng thái.
- **R-13 Ảnh trắng đen / ban đêm / mờ chuyển động** (img_022, img_014, img_031, img_013). Gán như ảnh thường. Vật bị nhòe vì chuyển động: box ôm vùng nhòe; không rõ → `can_review`.
- **R-14 Xích lô, xe ba gác đạp.** Không gán xe, chỉ gán `person` (img_023).

## 4. Ca khó

| Ảnh | Tình huống | Rule ID | Quyết định | Lý do |
|---|---|---|---|---|
| img_005 | Nhiều người đội mũ bảo hiểm trên xe máy, tiền cảnh, xếp chồng nhau | R-02, R-06, R-07 | Mỗi người một `person`, mỗi xe một `motorcycle` (chỉ thấy một phần → che 2–3), người sát mép trái dưới `truncated=yes`. | Chồng lấn, vật ở tiền cảnh phải gán riêng. |
| img_006 | Hai làn xe máy, xe đạp nhìn từ sau, đèn hậu sáng | R-02, R-08 | Mỗi xe máy + người một cặp box; xe đạp không gán, người đạp gán `person`; vật xa < 20 px bỏ. | Xe đạp ngoài phạm vi; vật xa quá nhỏ. |
| img_010 | Người ngồi xe máy cạnh người bán hàng đứng | R-02 | Xe + người lái + người đứng đều có box riêng. | Nhìn thấy rõ, không loại bỏ ai. |
| img_003 | Gánh hàng rong, xe đạp, xe máy phía sau | R-03, R-14 | Gán `person` người gánh, không gán xe đạp và gánh hàng. Xe máy phía sau gán `motorcycle` + `person`. | Gánh hàng là vật mang theo; xe đạp ngoài phạm vi. |
| img_002 | Xe chở xe | R-10 | 1 `truck` + từng `car` trên xe + người đứng cạnh `person`. | Xem R-10. |
| img_009 | Buýt mini đi giữa phố, xe máy nhỏ phía sau | R-08 | `van` cho buýt mini (`can_review`); xe máy xa ≥ 20 px gán, còn lại bỏ. | Chưa có lớp `bus`. |
| img_017 | Tượng người cưỡi ngựa | R-04 | Không gán gì. | Tượng không phải người/ngựa thật. |
| img_033 | Ma-nơ-canh, hình in, gương xe phản chiếu | R-04, R-05 | Không gán. | Vật giả + phản chiếu. |
| img_038 | Kệ đồ chơi, xe mô hình sau kính | R-04, R-05 | Không gán. | Mô hình và kính phản chiếu. |
| img_027 | Mộ hình xe tải | R-04 | Không gán. | Công trình điêu khắc. |
| img_028 | Xe bán tải cổ chở tượng ngựa | R-04, R-03 | Xe gán `truck` (bán tải thùng hở); tượng ngựa không gán. | Tượng không phải vật thật. |
| img_024 | Ô tô thật chở mô hình xe trên nóc | R-04 | Ô tô đen `car`, mô hình vàng không gán. | Mô hình là vật trang trí. |
| img_019 | Xe hoa lễ hội hình xe thể thao có người lái | R-04 | Gán `car` + `person` người lái, tick `can_review`; hình chuột trắng không gán. | Xe vẫn di chuyển trên đường nhưng giống đạo cụ. |
| img_020 | Gương chiếu hậu có xe phản chiếu | R-05, R-11 | Không gán xe trong gương; không gán xe chủ. | Phản chiếu + xe ego. |
| img_011 | Điêu khắc kim loại hình xe máy, đầu tượng lớn | R-04 | Không gán xe điêu khắc; người ở xa gán `person` (nhỏ, tick `can_review` nếu sát 20 px). | Vật nghệ thuật. |
| img_018 | Biển quảng cáo có mặt người, bãi đậu xe sau hàng rào | R-04, R-06 | Không gán mặt trên biển; ô tô sau hàng rào gán `car`, `occluded_level` 1–2. | Hình in vs xe thật bị che bởi hàng rào. |
| img_025 | Nhiều người, ô tô, xe máy xa trên đường biển | R-08, R-09 | Gán vật đủ ≥ 20 px; cụm xa nhỏ không gán. | Quá nhỏ. |
| img_021 | Cả biển xe máy dày đặc | R-09 | Hàng đầu gán từng xe + người; phía sau xa → `ignore_region`. | Không thể gán hết. |
| img_030 | Bãi xe máy dồn đống | R-09, R-06 | Xe tiền cảnh gán riêng, phần dày phía sau → `ignore_region`; người mặc đồng phục xanh gán `person`. | Xe che nhau nặng. |
| img_032 | Đám đông đứng chờ xa, vài xe sau | R-09 | Người rõ nhất gán `person`; phần còn lại → `ignore_region`. | Dày đặc, nhỏ. |
| img_035 | Xe ba bánh chở hàng chất cao, xe máy bên cạnh | R-03, R-02 | `truck` cho xe ba bánh (thùng hàng nằm trong box); `motorcycle` + `person` cho xe máy. | Hàng hóa chất cao là một phần xe chở. |
| img_004 | Hai xe tuk-tuk chở người đậu | R-12 | Gán `car` (kèm `can_review`) và `person` tài xế thấy rõ. | Chưa có lớp riêng cho tuk-tuk. |
| img_012 | Xe bán tải thùng đầy hàng, ô tô phía trước | R-03 | Bán tải `truck` (hàng trong thùng nằm trong box), ô tô `car` phía trước. | Hàng hóa thuộc xe. |
| img_029 | Hai người, một người vác ống dài | R-03, R-14 | `person` x2, `motorcycle` x1 (xe có người ngồi); xe đạp không gán; ống không tính vào box. | Ống kim loại là vật mang theo. |
| img_001 | Người nữ đứng sát xe bán tải thùng hở, nhóm người sau | R-06 | `truck`, `person` đứng trước (rõ), từng người ngồi sau gán riêng, mức che 1–2. | Chồng lấn nhẹ. |
| img_007 | Xe tải trắng Isuzu và bán tải trong nhà xe tối | R-12 | `truck` cho Isuzu, `truck` (hoặc `car` nếu bán tải cabin kép, `can_review`) cho xe trắng giữa; thùng rác, rào không gán. | Xe đỗ vẫn gán. |
| img_008 | Ban đêm, xe máy đỗ dày, hai người mặc áo cam | R-02, R-09 | Xe + người rõ gán riêng; cụm xa sau cùng → `ignore_region`; biển số không tính. | Đêm tối, mật độ cao. |
| img_014 | Hai người đi bộ nhìn từ sau, xe máy mờ chuyển động | R-13 | `person` x2 rõ; xe máy mờ → gán nếu nhận ra, tick `can_review`. | Mờ chuyển động. |
| img_031 | Đêm, phố ướt có phản chiếu ánh đèn | R-05, R-13 | Gán xe/người thật; vệt sáng trên đường không gán. | Phản chiếu mặt đường. |
| img_036 | Xe tải xanh chở hàng, phía sau có xe máy | R-06 | Gán `truck`; xe máy ở xa gán nếu ≥ 20 px. | — |
| img_040 | Người đội mũ bảo hiểm trước taxi, tuk-tuk, buýt | R-06, R-01 | `person` người đội mũ (cắt mép → `truncated=yes`), taxi `car`, SUV `car`, tuk-tuk `car` + `can_review`. | Che chồng nhiều lớp. |
| img_034 | Người đi bộ giữa đường, ô tô đen, xe máy hai bên | R-01, R-07 | `person` người đi bộ; `car`; người lái xe máy cắt mép phải → `truncated=yes`. | — |
| img_037 | Bán tải trắng có thùng chụp kín, hai buýt xanh phía sau | R-12 | `truck` (`can_review`) cho bán tải; buýt → `truck` + `can_review`; người đi bộ `person`. | Chưa có lớp `bus`. |
| img_026 | Đêm, hai người trên xe máy trước tranh tường in cảnh cảng | R-02, R-04 | `motorcycle` + hai `person`; tranh tường không gán. | Hình in. |

*(Một số ảnh ngoài bảng — img_013, img_015, img_016, img_022, img_023, img_039 — áp dụng thẳng R-01…R-14.)*

## 5. QC và phiên bản

- Ngưỡng QC lấy từ `taxonomy.json`: `min_box_height_px = 20`, IoU box lớn ≥ 0,85 (≥ 50 px), box nhỏ ≥ 0,7, đồng thuận nhãn–lớp ≥ 0,95, tỉ lệ lỗi tối đa mỗi ảnh 2%.
- Ca mơ hồ: tick `can_review`, ghi mã `R-xx` và `img_0xx` vào nhật ký QC, reviewer quyết.
- Ai quyết ca không có quy tắc: reviewer; nếu lặp lại ≥ 2 lần thì đưa lên họp nhóm để thêm quy tắc.
- Mọi thay đổi làm đổi nhãn đều tăng phiên bản.

| Phiên bản | Thay đổi | Lý do (ca nào) |
|---|---|---|
| v0 | Bản giảng viên. | — |
| v0.2 (102) | Thêm định nghĩa lớp có ví dụ; ngưỡng `occluded_level`; quy tắc R-01…R-14; bảng ca khó. | Người đi xe máy (img_006), xe chở xe (img_002), vật giả/phản chiếu (img_017, img_033, img_037, img_020), đám đông (img_021, img_032). |

## Câu hỏi cần hỏi reviewer / nhóm

1. Có thêm lớp `bicycle` (xe đạp, xích lô) và `bus` không? (img_003, img_023, img_031, img_037)
2. Xe ba bánh (tuk-tuk) xếp `car` hay `truck`? (img_004, img_016, img_035)
3. Xe nhỏ đang chở trên xe chở xe có gán riêng không? (img_002)
4. Xe hoa lễ hội hình xe thể thao có tính là xe thật không? (img_019)

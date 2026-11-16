# Guideline gán nhãn — v1.0 (thống nhất, khóa cho lượt gán độc lập) — nhóm 101 + 102

- Phiên bản: v1.0
- Ngày: 2026-11-16
- Người viết / duyệt: 101, 102 (hợp nhất từ `individual/guideline_101.md` và `individual/guideline_102.md`)
- Loại nhãn: bounding box 2D trên ảnh đường phố, CVAT, xuất COCO 1.0
- Taxonomy: `submission/taxonomy.json` v1.0 (8 lớp, thêm `bus` và `three_wheeler`; `rules` giữ nguyên)
- Ghi chú cho nhóm: cột "Chốt" ở mục 0 là **đề xuất hợp nhất**. Hai bạn xác nhận hoặc sửa lại khi họp Pha 2 trước khi khóa.

---

## 0. Các điểm hai bản cá nhân khác nhau và cách chốt

| # | Chủ đề | Bản 101 | Bản 102 | **Chốt v1.0** | Lý do / phương án dự phòng |
|---|---|---|---|---|---|
| 1 | Xe buýt (img_009, 031, 037, 040) | buýt lớn → `truck`, buýt mini → `van`, `can_review` | ngoài taxonomy → `ignore_region` | **Thêm lớp `bus`** | Cả hai bản đều thấy thiếu lớp. Nhãn `truck`/`van` cho buýt làm sai lớp; `ignore_region` bỏ mất vật to, rõ. Dự phòng: nếu nhóm không thêm lớp thì dùng `ignore_region`. |
| 2 | Tuk-tuk, xe lam (img_004, 016, 035, 040) | `truck` nếu chở hàng, `car` nếu chở người | ngoài taxonomy → `ignore_region` | **Thêm lớp `three_wheeler`** | Cùng một loại xe không nên đổi lớp theo việc chở gì (làm hai người gán lệch). Dự phòng: `ignore_region` + `person`. |
| 3 | Xe đạp, xích lô (img_003, 006, 023, 029) | không gán xe, đề xuất lớp `bicycle` | không gán xe | **Không gán xe, chỉ gán `person`** | Hai bản đã giống nhau. Chỉ thêm `bicycle` ở v1.1 nếu calibration cho thấy thiếu. |
| 4 | Bán tải cabin kép (img_007, 037) | bảng lớp ghi `car`, bảng ca khó ghi `truck` (tự mâu thuẫn) | `truck` | **`truck`** | Tiêu chí duy nhất: có **thùng hở/riêng phía sau cabin**. Khớp định nghĩa `truck` gốc trong taxonomy v0. |
| 5 | Vật cao < 20 px | không gán, **không** `ignore_region` | không box; phủ `ignore_region` | **Phủ `ignore_region`** | `QC_CHEATSHEET.md` ghi "vật nhỏ cao < 20 px → `ignore_region`". `validate_labels.py` cũng cảnh báo box không-ignore cao < 20 px. |
| 6 | Vật ≥ 20 px nhưng mờ | gán lớp đoán tốt nhất + `can_review` | không đoán lớp, `ignore_region` | **Nhận ra lớp: gán + `can_review`; không nhận ra: `ignore_region`** | Gộp hai cách, tách bằng một câu hỏi duy nhất: "nhận ra lớp chưa?". |
| 7 | Mức che tối thiểu | che > 90% thì bỏ | mức 3 (che > 75%) vẫn gán nếu nhận ra lớp | **Gán khi còn thấy ≥ 10% và nhận ra lớp và cao ≥ 20 px**; còn lại bỏ | 101 chặt ở ngưỡng 10%, 102 không nêu số. Chọn số có thể đo. |
| 8 | Ngưỡng `occluded_level` | % **bị che**: <10 / 10–40 / 40–75 / >75 | % **nhìn thấy**: ≥90 / 60–90 / 25–60 / <25 | **Giữ nguyên, ghi bằng % bị che** | Hai bản tương đương hoàn toàn. Chỉ thống nhất cách diễn đạt. |
| 9 | Hàng hóa trên xe (img_012, 029, 035) | quy tắc nói hàng ngoài box, nhưng bảng ca khó cho hàng trong thùng vào box (img_012, 035) | hàng không thuộc box | **Box ôm thân xe + thùng; hàng nhô ra khỏi đường bao thân/thùng thì không tính** | Sửa chỗ tự mâu thuẫn của 101: hàng nằm gọn trong thùng thì đã nằm trong đường bao; hàng chất cao hoặc nhô ngang thì loại. |
| 10 | Người trong xe kín | không nêu | không vẽ `person` | **Không vẽ** (xe kín, có kính); xe hở thì vẽ | Lấp chỗ trống của 101. |
| 11 | Đám đông (img_019, 021, 030, 032) | tiền cảnh + trung cảnh luôn gán riêng | gán riêng nếu thấy ≥ 25%, ≥ 20 px, tách được | **Dùng ba điều kiện của 102** | Có số đo được; ý "tiền cảnh/trung cảnh" của 101 thường đã thỏa ba điều kiện. |
| 12 | Xe chở xe (img_002) | 1 `truck` + `car` từng xe, `can_review` | giống, không `can_review` | **Giống nhau; không cần `can_review`** | Đã đồng thuận. Đây là quy tắc có thể ảnh hưởng ảnh khác (img_001, 012, 035): kiểm cả lô nếu đổi. |
| 13 | Xe giả chạy được (img_019) | `car` + `can_review` | giống | **Giống nhau** | Đã đồng thuận. |
| 14 | Vật không thật, phản chiếu, xe ego | không box | không box | **Không box (mẫu âm)** | Đã đồng thuận. |

---

## 1. Mục tiêu

Nhãn huấn luyện mô hình **phát hiện vật 2D** cho người và phương tiện **thật** trên đường (camera giao thông, hỗ trợ lái). Nhãn dạy mô hình: *"có một người / xe thật ở đây, và nó nằm trong hộp này"*.

Sai nguy hiểm nhất, theo thứ tự:
1. **Dương tính giả**: box trên tượng, hình in, đồ chơi, phản chiếu → mô hình học "tượng ngựa = người".
2. **Âm tính giả**: bỏ sót người đi bộ, người đi xe máy, xe nhỏ.
3. **Sai lớp** (xe tải vs xe con, xe ba bánh vs xe máy).
4. **Box lỏng** (IoU ghép ≥ 0.5; đạt khi IoU ≥ 0.85 với vật cao ≥ 50 px, ≥ 0.7 với vật nhỏ hơn).

**Phép thử chung** cho ca chưa có quy tắc: *"Mô hình có phải đoán ra điều này không?"* Nếu có → không ép nhãn: nhận ra lớp thì gán + `can_review` (R-20), không nhận ra thì `ignore_region`.

---

## 2. Lớp và thuộc tính

### Lớp (mỗi vật đúng một lớp)

| Lớp | Định nghĩa (một câu) | Ví dụ | Phản ví dụ / dễ nhầm |
|---|---|---|---|
| `person` | Người thật đang đi, đứng, ngồi, đạp xe hoặc ngồi trên xe, thấy được ít nhất đầu hoặc thân. | img_014, 034 người đi bộ; mỗi người trên xe ở img_006 | Tượng (img_017), ma-nơ-canh (img_033), mặt người trên biển (img_018), linh vật bông (img_019) |
| `motorcycle` | Xe hai bánh có động cơ, kể cả xe ga và xe có sidecar. | img_006, 008, 010 xe ga; img_022 sidecar | Xe đạp, xích lô (không gán xe), tuk-tuk (ba bánh) |
| `car` | Ô tô chở người, mui kín, gầm thấp: sedan, hatchback, SUV, wagon, taxi, xe thể thao. | img_015, 023, 031; img_024 xe Pacer | Bán tải thùng hở (`truck`); xe mô hình (img_038) |
| `truck` | Xe có thùng hàng riêng / thùng hở: xe tải, **bán tải kể cả cabin kép**, đầu kéo + rơ-moóc, xe chở xe. | img_001, 036 xe tải nhỏ; img_012, 028, 037 bán tải; img_002 | SUV (`car`); van kín (`van`) |
| `van` | Xe thùng kín liền khối cỡ nhỏ (cỡ 16 chỗ), **tối đa 4 ô cửa sổ bên mỗi bên**. | img_005, 016 | Minibus dài (`bus`); SUV (`car`) |
| `bus` | Xe buýt, xe khách, minibus lớn: thân dài, **từ 5 ô cửa sổ bên liên tiếp** hoặc trên 16 chỗ. | img_009 (buýt mini), 031, 037, 040 | Van nhỏ (`van`) |
| `three_wheeler` | Xe ba bánh có động cơ: tuk-tuk, xe lam, auto-rickshaw, chở người hoặc hàng. | img_004, 016, 035, 040 | Xích lô đạp (không gán xe, R-04); xe máy |
| `ignore_region` | Vùng không gán từng vật; bỏ sót trong vùng này **không tính lỗi** (R-08, R-09, R-10, R-20). | cụm xe máy dày ở img_021; bãi xe img_030 | Dùng để né vật rõ, lớn chỉ vì lười vẽ (bị tính là lỗi) |

**Vật không thuộc lớp nào:** xe đạp, xích lô đạp, xe ba gác đạp (R-04); cọc tiêu, biển, cây, thùng rác (không gán gì).

### Thuộc tính

**`occluded_level`** — tính theo **% diện tích vật bị che** (ước lượng so với toàn bộ vật, kể cả phần khuất):

| Mức | Nghĩa | Bị che | (nhìn thấy) |
|---|---|---|---|
| `0` | Thấy rõ | < 10% | ≥ 90% |
| `1` | Che ít | 10% – 40% | 60% – 90% |
| `2` | Che nhiều | 40% – 75% | 25% – 60% |
| `3` | Che gần hết, còn nhận ra lớp | > 75% | < 25% |

- Vật che **mảnh** (hàng rào lưới, dây, cành thưa, kính): nếu vẫn nhận ra vật thì **không tính là che** (img_018 xe sau hàng rào: mức 0–1).
- Người ngồi sau người khác trên xe máy (img_005): thường mức 2–3. Xe máy đậu chồng nhau (img_030): 2–3.
- Cờ `occluded` có sẵn của CVAT (phím `Q`) **bỏ qua**.

**`truncated`** — `yes` khi vật bị **mép ảnh cắt** (cạnh box chạm mép ảnh, sai số ≤ 2 px) và phần ngoài ảnh là thân thật của vật. Vật bị vật khác che **không** là truncated, dùng `occluded_level`. Ví dụ: img_015 (hai ô tô sát mép), img_005 (người góc trái dưới), img_026 (xe máy cắt mép dưới), img_040 (người đội mũ).

**`can_review`** — tick khi thật sự phân vân giữa hai quyết định có quy tắc khác nhau: lớp ranh giới, nghi vật thật/giả, mức che sát ngưỡng, vật sát 20 px, vật mờ (R-10). Không tick cho box chỉ hơi lỏng. Ghi ảnh, box và lý do để họp.

---

## 3. Quy tắc vẽ box

| ID | Quy tắc |
|---|---|
| **R-01** | **Box ôm sát phần nhìn thấy.** Không đoán phần khuất hoặc phần bị cắt. |
| **R-02** | **Độ sát:** mục tiêu sai số ≤ 2–3 px hoặc ≤ 5% cạnh. Người: đầu → chân hoặc phần thấy. Xe: thân + bánh + gương. **Không gồm**: bóng đổ, ăng-ten, dây, vệt phản chiếu, vật nhô ra xa. |
| **R-03** | **Người đi xe máy: hai loại box riêng, chồng lên nhau.** Một `motorcycle` ôm xe (không gồm người), một `person` ôm mỗi người (cả người ngồi sau, ngồi sidecar). Không dùng một box chung. Áp dụng img_005, 006, 008, 010, 021, 022, 026, 029, 035. |
| **R-04** | **Xe đạp, xích lô đạp, xe ba gác đạp:** chỉ vẽ `person` cho người đạp. Không gán xe (img_003, 006, 023, 029). |
| **R-05** | **Hàng hóa và vật mang theo:** box xe ôm **thân xe và thùng xe**; hàng nằm gọn trong đường bao thùng thì tự nhiên nằm trong box. Hàng **nhô ra khỏi đường bao** (chất cao, nhô ngang, ống dài img_029, gánh hàng img_003, hàng chất cao img_035) **không tính vào box**. Mui che (canopy img_037), thùng xe, nắp thùng thuộc xe. |
| **R-06** | **Xe chở xe (img_002):** xe đầu kéo + khung đã ghép = **một** `truck`; **mỗi xe con nhìn thấy trên đó một `car`** (chồng trong box `truck`, mức che theo thực tế, không cần `can_review`). Nếu nhóm đổi quy tắc này phải kiểm lại ca chở hàng khác (img_001, 012, 035). |
| **R-07** | **Người trong xe kín, có kính** (taxi, ô tô): **không** vẽ `person`. Người ở xe **hở** (xe máy, tuk-tuk, xích lô, xe mui trần img_019) vẽ nếu thấy rõ. |
| **R-08** | **Vật nhỏ:** chiều cao box < `min_box_height_px` (20 px, đo trên ảnh gốc trong CVAT) → **không vẽ box**, phủ `ignore_region` (ôm vật hoặc cả cụm). Ví dụ: người xa ở img_011, 013, 025, 039; xe xa ở img_009. |
| **R-09** | **Đám đông / cụm dày:** vẽ box riêng cho vật thỏa **cả ba**: (a) thấy ≥ 25% (mức ≤ 2), (b) cao ≥ 20 px, (c) tách được bằng mắt khỏi vật kề. Phần còn lại phủ `ignore_region`. Ca: img_019, 021, 030, 032, 008, 036. |
| **R-10** | **Mờ chuyển động, ban đêm, ảnh trắng đen** (img_014, 015, 022, 013, 031): gán như ảnh thường. Nhận ra lớp và ranh giới đủ rõ → gán (mờ nhiều thì tick `can_review`). Không nhận ra lớp → `ignore_region`. |
| **R-11** | **Vật không thật → không box, không ignore** (mẫu âm cho mô hình): tượng (img_017, 027, 028 ngựa), ma-nơ-canh (img_033), hình người/xe in trên biển hoặc tranh tường (img_018, 026, 030), xe mô hình và đồ chơi (img_038, nóc xe img_024), tác phẩm điêu khắc (img_011), linh vật bông (img_019). Ghi vào danh sách ca. |
| **R-12** | **Phản chiếu → không box:** gương chiếu hậu (img_020), kính cửa hàng (img_033, 038), mặt đường ướt (img_013, 031). Không ngoại lệ. |
| **R-13** | **Xe chủ (ego):** phần xe của người quay (mui, gương, nắp capô) **không gán** (img_020). |
| **R-14** | **Xe đậu, xe không người** gán như xe đang chạy (img_003, 007, 008, 010, 012, 030, 036); xe trong nhà kho / bảo tàng vẫn gán nếu là xe thật (img_007, 024). |
| **R-15** | **Phân lớp xe** theo mục 2. Điểm dễ lệch: bán tải thùng hở (kể cả cabin kép) → `truck`; SUV / wagon → `car`; minibus ≥ 5 ô cửa sổ bên → `bus`; van ≤ 4 ô cửa sổ → `van`; xe ba bánh → `three_wheeler` bất kể chở người hay hàng. |
| **R-16** | **Xe giả chạy được trên đường có người lái thật** (img_019 xe Danger Mouse có biển số): xem là xe thật → `car` + `can_review`; người lái `person`. |
| **R-17** | **Ảnh không có vật theo phạm vi** (img_017, 033, 038): vẫn nằm trong Task, không box. |
| **R-18** | **Che tối thiểu:** vật còn thấy **≥ 10%**, nhận ra lớp, cao ≥ 20 px thì gán (mức 3 nếu < 25%). Che hơn 90% hoặc không nhận ra lớp thì bỏ (không box, không ignore). |
| **R-19** | **Cắt mép ảnh:** box dừng đúng mép ảnh, `truncated = yes`. Cắt quá nửa nhưng nhận ra lớp thì vẫn gán, tick `can_review`. |
| **R-20** | **Không chắc lớp:** chỉ thấy mảnh không nhận ra (ví dụ chỉ thấy gương xe máy ở img_040) → `ignore_region` nếu ≥ 20 px, bỏ nếu nhỏ hơn. Nhận ra lớp nhưng phân vân giữa hai lớp → gán lớp tốt nhất + `can_review`. |

---

## 4. Ca khó

| Ảnh | Tình huống | Rule | Quyết định | Lý do |
|---|---|---|---|---|
| img_017 | Tượng người cưỡi ngựa | R-11, R-17 | Không box, ảnh rỗng | Tượng không phải người; hai người cùng vẽ `person` vẫn sai dù IAA cao |
| img_002 | Xe chở xe | R-06 | 1 `truck` + từng `car` thấy được + người đứng cạnh `person` | Xe được chở là xe thật; tránh hai cách vẽ khác nhau |
| img_005 | Cận cảnh nhiều người trên xe máy, chồng lấn, xe van nền | R-03, R-09, R-18, R-19 | Mỗi người một `person`; xe máy khi thấy tay lái/gương rõ; chen chúc quá mức → `ignore_region`; người góc trái dưới `truncated=yes`; van nền `van` | Che mức 2–3, chỉ vẽ phần thấy |
| img_006 | Xe máy, xe đạp nhìn từ sau, taxi trái | R-03, R-04, R-08 | Mỗi xe máy + người một cặp box; xe đạp chỉ `person`; taxi `car`; vật xa < 20 px → `ignore_region` | Xe đạp ngoài lớp |
| img_010 | Người ngồi xe máy cạnh người bán hàng | R-03 | Xe, người lái, người đứng đều một box riêng | Thấy rõ |
| img_026 | Đêm, hai người trên xe máy cắt mép dưới, tranh tường cảng | R-03, R-11, R-19 | 2 `person` + 1 `motorcycle` (`truncated=yes`); tranh tường không box | Tranh là hình in |
| img_003 | Gánh hàng rong, xe đạp, xe máy đậu | R-04, R-05, R-14 | `person` người gánh (gánh không vào box); xe đạp không gán; xe máy đậu `motorcycle` (+ `person` nếu có người) | Gánh là vật mang theo |
| img_004 | Hai tuk-tuk đậu | R-07, R-15 | 2 `three_wheeler` + `person` tài xế nếu thấy rõ | Xe hở, có lớp riêng |
| img_016 | Van, xe ga có người, hai tuk-tuk, người cắt mép phải | R-03, R-15, R-19 | `van`; `motorcycle` + `person`; 2 `three_wheeler`; `person` (`truncated=yes`) | |
| img_023 | Xích lô + Honda đen | R-04 | `car`; `person` người đạp; không gán xích lô | |
| img_035 | Xe ba bánh chất hàng cao, xe máy có người, ô tô | R-05, R-03, R-15 | 2 `three_wheeler` (hàng nhô cao không vào box); `motorcycle` + `person`; `car` | |
| img_009 | Buýt mini giữa phố, xe máy nhỏ xa | R-15, R-08 | `bus`; xe máy ≥ 20 px gán, còn lại `ignore_region` | ≥ 5 ô cửa sổ bên |
| img_031 | Đêm, phố ướt, buýt, xe ô tô | R-12, R-10, R-15 | `bus`, `car`, `motorcycle` thật; vệt sáng đường không box | Phản chiếu |
| img_037 | Bán tải trắng mui chụp, hai buýt xanh, người xa | R-05, R-15 | `truck` (canopy thuộc xe); 2 `bus`; `person` | |
| img_040 | Người đội mũ sát camera, taxi, SUV, tuk-tuk, buýt nền, chỉ thấy gương xe máy | R-19, R-20 | `person` (`truncated=yes`); taxi `car` (cắt mép phải); SUV `car`; `three_wheeler`; `bus`; gương xe máy → không box | Gương không đủ nhận ra xe |
| img_028 | Bán tải cổ chở tượng ngựa | R-11, R-05 | `truck`; ngựa không box | Tượng |
| img_027 | Bia mộ hình xe tải | R-11, R-17 | Không box | Không phải xe thật |
| img_024 | Ô tô Pacer đen, mô hình xe vàng trên nóc, xe cổ nền | R-11, R-14 | `car` (box không gồm vật trên nóc); xe nền thật gán nếu thỏa R-08/R-09 | Mô hình là vật trang trí |
| img_019 | Xe Danger Mouse diễu hành + đám đông sau rào | R-16, R-09, R-11 | `car` + `can_review`; `person` người lái; người hàng đầu gán; đám sau `ignore_region`; linh vật không box | |
| img_020 | Gương chiếu hậu có xe phản chiếu | R-12, R-13 | Không box | Phản chiếu + xe chủ |
| img_033 | Ma-nơ-canh, hình in, kính | R-11, R-12, R-17 | Không box | |
| img_038 | Kệ đồ chơi sau kính | R-11, R-12, R-17 | Không box | |
| img_018 | Biển có mặt người; ô tô sau hàng rào | R-11, mục 2 | Không `person` cho biển; `car` cho xe thật (mức 0–1) | Hình in vs xe thật |
| img_011 | Tượng Predator, xe máy bằng sắt, người xa | R-11, R-08 | Không box điêu khắc; người < 20 px → `ignore_region` | |
| img_029 | Xe ga chở ống dài + xe đạp | R-05, R-04, R-03 | `motorcycle` + `person`; `person` người đạp; ống ngoài box | |
| img_022 | Ảnh trắng đen, xe có sidecar | R-03, R-10 | 1 `motorcycle` (gồm sidecar) + `person` ngồi sidecar | Sidecar là phần của xe |
| img_021 | Biển xe máy ùn tắc | R-09 | Xe + người tách được gán riêng; phía sau → `ignore_region` | Không thể gán hết |
| img_030 | Bãi xe máy dồn đống, người mặc xanh | R-09, R-18, R-14 | Xe tiền cảnh gán riêng; phần dày → `ignore_region`; `person` | Chồng lấn nặng |
| img_032 | Đám đông đứng chờ xa | R-09 | Người rõ nhất `person`; còn lại `ignore_region` | |
| img_025 | Người, xe nhỏ xa trên đường biển | R-08, R-09 | Gán vật ≥ 20 px; còn lại `ignore_region` | |
| img_008 | Đêm, xe máy đậu dày, áo cam | R-03, R-09 | Xe + người rõ gán riêng; cụm xa → `ignore_region` | |
| img_014 | Hai người rõ, xe máy mờ | R-10 | 2 `person`; xe mờ nhận ra thì gán + `can_review`, không nhận ra → `ignore_region` | |
| img_013 | Đêm, đường ướt, người nhỏ | R-12, R-08 | Vật thật ≥ 20 px gán; còn lại `ignore_region`; phản chiếu không box | |
| img_007 | Xe tải Isuzu + bán tải trắng trong nhà xe tối | R-14, R-15 | 2 `truck`; thùng rác, rào không gán | |
| img_012 | Bán tải đầy hàng, SUV phía trước | R-05, R-15 | `truck` (hàng gọn trong thùng nằm trong box); SUV `car` | |
| img_001 | Người đứng cạnh bán tải thùng hở, nhóm người sau | R-18 | `truck`; từng `person`, mức che 1–2 | |
| img_036 | Xe tải xanh, xe máy đậu sau | R-15, R-09 | `truck`; xe máy ≥ 20 px gán, còn lại `ignore_region` | |
| img_034 | Người đi bộ giữa đường, ô tô, xe máy cắt mép phải | R-19 | `person`; `car`; người lái xe máy `truncated=yes` | |

*(img_015, 039: áp dụng thẳng R-01…R-20; img_015 hai ô tô sát mép → `truncated=yes`.)*

---

## 5. QC và phiên bản

- **Ngưỡng QC** (`taxonomy.json`, không sửa): IoU ghép ≥ 0.5; đạt IoU 0.85 (vật cao ≥ 50 px) / 0.7 (nhỏ hơn); IAA: IoU TB ≥ 0.85, cùng lớp ≥ 95%; vật < 20 px → `ignore_region`; Release khi lỗi theo ảnh ≤ 2% trên toàn lô.
- **Ca mơ hồ:** tick `can_review`, ghi `R-xx` và `img_0xx` vào nhật ký QC.
- **Ai quyết ca chưa có quy tắc:** người chốt ghi trong `calibration_decisions.csv` (một dòng mỗi ca). Ca lặp lại ≥ 2 lần → thêm quy tắc mới và tăng phiên bản.
- **Khóa:** v1.0 và `taxonomy.json` v1.0 khóa cho lượt gán độc lập. Mọi thay đổi làm đổi cách chọn vật, lớp, box hay thuộc tính là `semantic` (xem GUIDE.md Pha 5).

| Phiên bản | Thay đổi | Lý do (ca nào) |
|---|---|---|
| v0 | Bản giảng viên | |
| v0.x (101, 102) | Hai bản cá nhân viết độc lập | Pha 1 |
| **v1.0** | Hợp nhất hai bản; thêm lớp `bus`, `three_wheeler`; chốt 14 điểm bất đồng (mục 0); R-01…R-20 | img_002, 005, 009, 004, 016, 035, 017, 019, 012, 029 |

## Câu hỏi còn mở cho reviewer / coach

1. Có chấp nhận thêm `bus` và `three_wheeler` vào taxonomy không? (nếu không: `ignore_region` + `person`)
2. Có cần lớp `bicycle` cho v1.1 không? (img_003, 006, 023, 029)
3. Ngưỡng "5 ô cửa sổ" để tách `van` / `bus` có hợp lý không? (img_009, 016, 005)

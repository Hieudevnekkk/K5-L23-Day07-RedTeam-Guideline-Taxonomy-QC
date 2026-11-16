# Báo cáo QC — RedTeam

## 1. Mục đích, dữ liệu và phiên bản

- Mục đích và giới hạn sử dụng: Thực hành đặc tả và QC nhãn bounding box 2D (COCO 1.0) trên 40 ảnh đường phố. Tập ảnh được tuyển chọn để luyện ca biên, độ đại diện cho dữ liệu vận hành chưa được đánh giá; không dùng để huấn luyện hay đánh giá mô hình thật.
- Nguồn và phép sử dụng dữ liệu: `data/ATTRIBUTION.csv`, 40 ảnh, giấy phép CC BY-SA 4.0 (15), CC BY 2.0 (7), CC BY-SA 2.0 (6), CC BY 4.0 (5), CC0 (3), CC BY-SA 3.0 (3), Public domain (1). Ảnh CC BY và CC BY-SA cần ghi tác giả khi tái sử dụng.
- Manifest và checksum nhãn cuối: `submission/qc/batch_manifest.json`; `labels_calibrated.json` SHA-256 `68269b381d06309babd51091e2d8c9b09ce83956a423a95d7f54e29b7a0ca103`.
- Guideline và taxonomy khi gán / khi bàn giao: gán theo guideline v1.0 và taxonomy v1.0; bàn giao guideline v1.1 (`guideline_change.json` là `editorial`, không đổi cách chọn vật, lớp, box, thuộc tính), taxonomy giữ v1.0.

## 2. Phân công và đồng thuận trước calibration

- Lượt hoàn thành / được giao: 80/80 lượt (2 người gán 101 và 102, mỗi người 40 ảnh, 40/40 ảnh có đủ 2 lượt).
- Số cặp box ghép, IoU TB, đồng thuận lớp và mẫu số: 240 cặp box ghép (IoU ≥ 0.5, tổng cặp người gán trên 40 ảnh chung); IoU TB 0.861 (≥ 0.85); cùng lớp 99.2% (≥ 95%); cùng `occluded_level` 68.3%; cùng `truncated` 99.2%. Đạt ngưỡng IAA, nhưng chỉ tính trên cặp đã ghép.
- Box chưa ghép, loại do ignore, bất đồng vùng ignore: chưa ghép 22 (101) / 32 (102); 8 box bỏ qua do `ignore_region`; 60 ca `bat_dong_ignore` trong 277 ca bất thường.
- Ca hai người có thể cùng sai sau khi đối chiếu tham chiếu: chưa đối chiếu với ca tham chiếu của coach. Cần kiểm các ảnh vật không thật (img_017, img_033, img_038) và img_019, vì IAA cao không chứng minh nhãn đúng quy tắc R-11/R-16. Cách kiểm: so box của ảnh đó trong `labels_snapshot.json` với bảng tham chiếu của coach.

## 3. Calibration và kiểm lại

- Ca đã xử lý, rule ID, bản sửa và người xác nhận: 277 ca trên 32 ảnh cờ trong `submission/qc/calibration_decisions.csv`, người chốt 101, phạm vi `anh`. Rule dùng: R-01, R-02, R-03, R-08, R-09, R-15, R-18, R-19, R-20. Người kiểm lại cột `kiem_lai` còn ghi "Chờ 102 kiểm lại", chưa có xác nhận của người thứ hai.
- Tác động v1.1 lên nhãn và ảnh: sửa câu chữ, không đổi nghĩa; phạm vi 32 ảnh cờ, 8 ảnh còn lại giữ nguyên. Nhãn cuối `labels_calibrated.json` gộp 40 ảnh, 306 annotation.
- Tham chiếu bằng chứng khi Rework: chưa chạy vòng 2. Vòng 1 ở `submission/qc/reviews/round-01/`; vùng cần sửa là img_040, box `#301`.

## 4. Kết quả duyệt

- Vòng QC: 1
- Người review và ngày: 102, 2026-11-16
- Đã duyệt / cả lô: 40/40
- Ảnh lỗi / ảnh đã duyệt: 1/40 = 2.5% (vượt ngưỡng 2%)
- Annotation đối tượng hiện có bị lỗi / annotation đối tượng đã duyệt: 1/292 = 0.3%
- Vật bỏ sót, ảnh và vùng: 0 vật bỏ sót ghi nhận
- Ca khó / phần còn lại, lỗi lặp và ví dụ: ca khó 32/32 đã duyệt, 1 ảnh có lỗi (3.1%); phần còn lại 8/8 đã duyệt, 0 lỗi. Chỉ một loại lỗi (`thua`), chưa thấy lỗi lặp. Ví dụ: `qc_review.csv`, dòng img_040 `#301`.
- Một lỗi máy chặn được và một lỗi cần người đọc ảnh: máy chặn được: `validate_labels.py` chặn lớp lạ và thiếu thuộc tính khi export COCO. Cần người đọc ảnh: img_040 `#301 three_wheeler` trùng `#298` trên cùng một tuk-tuk, cả hai box cùng mép trái x≈304.

## 5. Quyết định bàn giao

- Quyết định: Hold
- Lý do, người nhận và phạm vi sử dụng: số liệu IAA đạt ngưỡng nhưng tỉ lệ ảnh lỗi vòng 1 là 2.5% (> 2%), nhãn cuối còn box đánh dấu `can_review`, và `kiem_lai` của 277 ca chưa có xác nhận người thứ hai. Phiếu QC do một người điền (mã 102) và chưa được người review bản cuối trong `TEAM.md` xem lại. Người nhận: coach. Phạm vi sử dụng: chỉ để chấm bài thực hành, không dùng cho mô hình.
- Nếu còn việc: (1) xóa hoặc xác nhận lại box `#301` ở img_040 (101); (2) chốt và bỏ cờ `can_review` ở các box đã quyết định; (3) 102 xác nhận cột `kiem_lai`; (4) đối chiếu ca tham chiếu của coach; (5) chạy `build_final.py`, `qc_sample.py --round 2`, duyệt và `qc_report.py --round 2`, giữ nguyên `round-01/`.
- Kiểm ngược từ báo cáo tới ảnh, rule và nhãn: ca img_040 → phiếu `submission/qc/reviews/round-01/qc_review.csv` (dòng `#301`, `thua`, R-01) → `work/review/round-01/img_040.jpg.png` → annotation id 301 trong `submission/qc/labels_calibrated.json`, bbox [304, 79, 229, 321], trùng id 298 bbox [305, 70, 534, 338].

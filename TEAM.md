# Thông tin nhóm

- Tên nhóm: RedTeam
- Mã thành viên (ba chữ số): 101, 102
- Người điều phối: Nguyễn Minh Hiếu
- Người review bản cuối: Nguyễn Hoàn Thiện
- Commit chốt bài (điền ở kênh nộp sau khi commit):

## Kết quả tóm tắt (điền cuối bài)

- Lượt gán hoàn tất / được giao: 80/80 (2 người, mỗi người 40 ảnh)
- IAA trước calibration (kèm mẫu số): IoU TB 0.861, cùng lớp 99.2% trên 240 cặp box ghép (40 ảnh); chưa ghép 22/32
- Số ảnh phải calibration: 32 / 40
- Vòng QC và số ảnh đã duyệt / cả lô: vòng 1, 40/40
- Tỉ lệ lỗi: 2.5% theo ảnh (1/40), 0.3% theo đối tượng (1/292)
- **Quyết định** (Release / Hold / Rework) và lý do một dòng: Hold: ảnh lỗi 2.5% vượt 2% (img_040 box `#301` trùng `#298`), còn `can_review` và 277 ca calibration chưa có người thứ hai xác nhận.

# Thông tin nhóm

- Tên nhóm: Try_hard
- Mã thành viên (ba chữ số): 101, 102, 103, 104, 105
- Người điều phối: 101
- Người review bản cuối: 105
- Commit chốt bài (điền ở kênh nộp sau khi commit):

## Kết quả tóm tắt (điền cuối bài)

- Lượt gán hoàn tất / được giao: 120/120
- IAA trước calibration (kèm mẫu số): IoU TB 0.848, cùng lớp 96.3% (706 cặp box ghép trên 120 lượt ảnh)
- Số ảnh phải calibration: 37 / 40
- Vòng QC và số ảnh đã duyệt / cả lô: Vòng 1 — 40/40
- Tỉ lệ lỗi: 25.0% theo ảnh (10/40), 3.9% theo đối tượng (20/514; thêm 1 bỏ sót ghi riêng)
- **Quyết định** (Release / Hold / Rework) và lý do một dòng: Rework — 10/40 (25.0%) ảnh trong lô còn lỗi vượt ngưỡng 2% (9 sai lớp, 6 sai vị trí/kích thước, 4 thừa, 1 bỏ sót, 1 sai thuộc tính).

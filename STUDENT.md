# Thông tin học viên

- Họ và tên: Phạm Đức Thọ
- MSHV: 2a202603182
- Lớp (L2 / L3 / khác): K5-L23
- Tài khoản GitHub: phamductho (hoặc cập nhật username GitHub của bạn)
- Repo bài nộp (URL): https://github.com/VinUni-AI20k/K5-L23-Day08-AI-Assisted-Labeling-PhamDucTho-2a202603182
- Commit chốt bài (hash): cập nhật sau khi commit chốt bài
- Máy: Windows 11, CPU (không có GPU)
- CVAT (Docker): v2.81.0

## Kết quả tóm tắt (điền cuối bài; số lấy từ `submission/qc/*`, kèm mẫu số)

- Máy: accepted / edited / deleted: 588/620 (94.8%) / 9/620 (1.5%) / 14/620 (2.3%) (mẫu số: 620 box máy); added: 10/607 (1.6%) (mẫu số: 607 đối tượng sau duyệt)
- So với gold: precision / recall của **máy**: 63.5% (47/74) / 83.9% (47/56)
- Bắt bẫy: 4/12 = 33.3%
- Thời gian: gán tay 5.0 phút/ảnh (20 phút / 4 ảnh); duyệt có pre-label 1.9 phút/ảnh (45 phút / 24 ảnh)
- Lỗi QC: 80.0% theo ảnh (4/5), 5.1% theo đối tượng (6/118)
- **Quyết định:** Rework — lý do một dòng: Ảnh có lỗi QC 4/5 (80.0% > 2%), lỗi lặp `thua` (4/6 lỗi) và tỷ lệ bắt bẫy 4/12 (33.3% < 80%) nên cần duyệt lại theo guideline v1.1.

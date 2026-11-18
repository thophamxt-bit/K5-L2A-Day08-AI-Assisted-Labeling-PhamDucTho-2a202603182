# Báo cáo QC tự động (nháp)

> **Gợi ý** do script tính; bạn xác nhận. Script chỉ đếm theo phiếu kiểm bằng mắt người.

## Mẫu
- 5 ảnh (0 ca khó + 5 ngẫu nhiên), 118 đối tượng.
- taxonomy v1.0

## Số đo
| Chỉ số | Giá trị | Mẫu số |
|---|---|---|
| Lỗi **theo đối tượng** | 6/118 = 5.1% | 118 đối tượng trong mẫu |
| **Ảnh có lỗi** (dùng cho ngưỡng) | 4/5 = 80.0% | 5 ảnh trong mẫu |
| Máy: accepted / edited / deleted | 588/620 = 94.8% / 9/620 = 1.5% / 14/620 = 2.3% | 620 box máy |
| Máy: added | 10/607 = 1.6% | 607 đối tượng sau duyệt |
| Bắt lỗi bẫy | 4/12 = 33.3% | 12 bẫy |

## Top 3 loại lỗi
| Loại lỗi | Số lỗi | Ảnh ví dụ |
|---|---|---|
| thua | 4 | img_003.jpg, img_014.jpg, img_031.jpg |
| sai_vi_tri_kich_thuoc | 2 | img_003.jpg, img_031.jpg |

**Lỗi hệ thống**: 'thua' lặp lại -> sửa quy tắc / recipe, không chỉ sửa từng ảnh.

## Giới hạn thống kê
- 4 ảnh lỗi trong 5 ảnh mẫu: cận trên 95% (Wilson) của tỉ lệ ảnh lỗi thật là **96%**. Ngưỡng 2% không thể được **chứng minh** bằng mẫu nhỏ như vậy: hãy nói rõ giới hạn này trong báo cáo.

## Gợi ý quyết định
**REWORK**

- ảnh có lỗi 80.0% > 2%
- lỗi hệ thống: 'thua' chiếm 4/6 lỗi
- bắt bẫy 33% < 80% -> lô duyệt chưa đáng tin

Cây quyết định: Hold (guideline chưa rõ / dữ liệu hỏng) -> Rework (> ngưỡng, lỗi lặp, bắt bẫy / nhất quán thấp) -> Release.

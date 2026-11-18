# Số đo review (det)

- Ảnh tính: 24 (batch,gold); box máy: 620; đối tượng sau duyệt: 607; ngưỡng accepted: IoU >= 0.9
- Đã **loại 25 đối tượng nằm quanh các bẫy** khỏi bảng này: lỗi nhân tạo không phản ánh chất lượng máy.

## 1. Bốn trạng thái (AI error profile)
| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted (giữ nguyên) | 588 | 588/620 = 94.8% | 620 box máy |
| edited (sửa) | 9 | 9/620 = 1.5% | 620 box máy |
| deleted (xoá) | 14 | 14/620 = 2.3% | 620 box máy |
| added (vẽ thêm) | 10 | 10/607 = 1.6% | 607 đối tượng sau duyệt |

Trong số deleted + added, **2 trường hợp là đổi lớp** (máy gán sai lớp, box đúng chỗ): đó không phải máy bỏ sót. Số vật máy **sót thật** ≈ added − đổi lớp = 8.

### Theo lớp
| Lớp | accepted | edited | deleted | added |
|---|---|---|---|---|
| person | 267 | 0 | 2 | 2 |
| bicycle | 6 | 0 | 1 | 1 |
| motorbike | 236 | 2 | 1 | 0 |
| car | 51 | 0 | 0 | 0 |
| bus | 3 | 0 | 0 | 2 |
| truck | 5 | 0 | 0 | 0 |
| three_wheeler | 20 | 7 | 10 | 5 |

## 2. So với gold (nhãn gán tay từ đầu)
Ảnh gold: 4; đối tượng gold: 56 (mẫu số của recall)
| | TP | FP | FN | Precision | Recall |
|---|---|---|---|---|---|
| Máy (pre-label) | 47 | 27 | 9 | 63.5% (KTC 95%: 52–74%) | 83.9% (KTC 95%: 72–91%) |

*Lạc quan: ngưỡng được chọn trên chính các ảnh gold này (in-sample), và chỉ có 4 ảnh. Đọc khoảng tin cậy, đừng đọc con số đơn.*


(Ảnh gold không nằm trong nhãn sau duyệt: không tính residual / new error. Đó là chủ ý: duyệt lại ảnh bạn vừa tự gán tay sẽ đo trí nhớ, không đo thiên kiến.)

## 3. Ảnh bẫy
- Bắt được 4/12 = 33.3% (KTC 95%: 14–61%) (ngưỡng đạt: 80%). Chỉ 12 bẫy nên khoảng tin cậy rất rộng: đọc cùng danh sách bẫy lọt, đừng kết luận chỉ từ %.
  - missing: 1/3
  - extra: 2/3
  - wrong_class: 0/3
  - loose: 1/3
  - lọt: img_010.jpg (extra), img_008.jpg (wrong_class), img_026.jpg (missing), img_027.jpg (missing), img_025.jpg (wrong_class), img_029.jpg (loose), img_022.jpg (wrong_class), img_014.jpg (loose)

## Gợi ý đọc số
- `deleted` cao → ngưỡng thấp hoặc ánh xạ nhãn sai; `added` cao → máy bỏ sót (xem lớp nào);
- `accepted` thấp (< 50%) → sửa mỗi box lâu hơn vẽ mới: đổi model/ngưỡng hoặc vẽ tay;
- bắt bẫy < 80% → duyệt lại lô, siết checklist (quét ảnh TRƯỚC khi nhìn nhãn máy).

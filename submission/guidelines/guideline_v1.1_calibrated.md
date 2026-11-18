# Label contract — Phố châu Á — BỔ SUNG v1.1 — 2a202603182 - Phạm Đức Thọ

> **Chỉ viết phần thay đổi so với v1.0** (không copy cả file). Chốt các câu hỏi mở ở mục 6 của v1.0 (hoặc nói rõ câu nào chưa chốt, vì sao)
> và thêm quy tắc mới rút ra từ lỗi của máy / lỗi của bạn. Mỗi ca khó **có ảnh ví dụ** (`img_0xx`) và khớp một dòng trong `qc/calibration_decisions.csv`.

## 1. Câu hỏi mở đã chốt (từ mục 6 của v1.0)

| # | Câu hỏi | Quyết định | Ảnh ví dụ | `case_id` |
|---|---|---|---|---|
| 1 | Xe đạp thồ chở hàng cồng kềnh vs xe đạp điện: `bicycle` hay `motorbike`? | Xe đạp thồ không động cơ gán `bicycle` (box bao trọn cả giỏ hàng); xe đạp điện có động cơ/bình điện gán `motorbike`. | `img_031.jpg` | `CAL-01` |
| 2 | Mask `three_wheeler` có gồm người lái / hành khách ngồi bên trong không? | Không: `person` và `three_wheeler` tách riêng; khi dùng SAM phải thêm 1–3 click âm lên người lái/hành khách. | `img_013.jpg` | `CAL-03` |

## 2. Ca khó mới (từ lỗi của máy, bẫy lọt, box `can_review`)

| Ảnh | Tình huống | Máy làm gì (model nào) | Quyết định của bạn | Quy tắc mới (vào mục nào của v1.0) |
|---|---|---|---|---|
| `img_003.jpg` | Xe ba gác có mái che và chở hàng | YOLOE + YOLO26 sinh 3 box `three_wheeler` chồng nhau (#3 nửa dưới, #27 toàn xe, #10 mái che); tương tự `img_035.jpg` có 2 box (#626, #627) | Giữ duy nhất 1 box bao trọn thân xe + mái che, xoá các box con/trùng | Mục 3.1: Một đối tượng `three_wheeler` chỉ có 1 box thống nhất (`CAL-02`) |
| `img_031.jpg` | Vật tĩnh màu nổi (mũ lưỡi trai cam trên kệ) & người bán hàng bị cắt chân | YOLO26 nhận nhầm mũ thành `person` (#526, conf 0.34) và box `person` (#527) cắt ngang hông | Xoá box giả #526; kéo box #527 đủ toàn thân người bán hàng | Mục 5.1: Ẩn pre-label 10–15s đầu mỗi ảnh và kiểm tra đủ 4 cạnh box (`CAL-04`) |

## 3. Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do (ca nào, `case_id`) |
|---|---|---|
| v1.0 | Bản giảng viên | — |
| v1.1 | Chốt quy tắc `bicycle`/`motorbike` cho xe chở hàng, tách người lái khỏi mask SAM `three_wheeler`, chống trùng box `three_wheeler` và siết checklist ẩn pre-label | `CAL-01`, `CAL-02`, `CAL-03`, `CAL-04` |

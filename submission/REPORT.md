# Báo cáo Day 08 — 2a202603182 - Phạm Đức Thọ

> Một file duy nhất cho cả bài (đã gộp recipe, QC và ghi chú cá nhân). Mọi con số kèm **mẫu số** và nguồn (file nào).

## 1. Model và recipe (Part 2–3)

| Lớp | Model + cỡ | Ngưỡng | Vì sao (dẫn `run_id` trong `model_log.csv`) |
|---|---|---|---|
| person / bicycle / motorbike / car / bus / truck | `yolo26-m` | `motorbike=0.1`, các lớp khác mặc định `0.3` (từ `threshold_sweep.md`) | `r01`, `r02`: YOLO26m bắt chặt các lớp COCO; `motorbike=0.1` đạt recall 92% trên gold (13 vật), các lớp `person`/`bicycle`/`car` không đạt 90% ở `<=0.5` và `bus` (0 vật)/`truck` (1 vật) < 5 vật nên giữ mặc định `0.3`. |
| three_wheeler | `yoloe-s` (prompt mặc định từ `taxonomy.json`: tuk-tuk, cyclo, cargo tricycle) | `three_wheeler=0.15` (từ `threshold_sweep.md`) | `r03`: YOLO26 không có lớp `three_wheeler` (gán nhầm tuk-tuk thành `truck`/`car`). YOLOE-s ở `0.15` đạt recall 100%, precision 78% (7 vật gold) và được ưu tiên đè `car`/`truck` khi `IoU >= 0.6`. |

- Lệnh `merge_prelabels.py` đã dùng (dán nguyên): `python scripts/merge_prelabels.py work/prelabel/yoloe_all/pre.coco.json work/prelabel/y26_all/pre.coco.json --thr "motorbike=0.1,three_wheeler=0.15" --conf 0.3 --iou 0.6 --split "batch,gold" --out work/prelabel/merged_review/pre.coco.json`
- **Phiên bản SAM bạn chọn** (SAM 1 / 2.1 / 3) và lý do: Chọn **SAM 2.1 (`sam2.1_b.pt`)** (`r04`) vì cân bằng tốt giữa chất lượng biên mask và tốc độ chạy trên CPU, không cần xin quyền truy cập (gated) như SAM 3 (`facebook/sam3`).
- Lỗi nào **không sửa được bằng đổi ngưỡng** (ví dụ tuk-tuk bị gọi `car`)? Sửa thế nào? Lỗi **mù lớp / nhầm lớp** (như tuk-tuk ở `img_011.jpg` bị YOLO26 gọi `truck` với `conf=0.77`, hay `img_004.jpg` gọi tuk-tuk là `car`) và lỗi **box lỏng / trùng nhiều box** (`img_003.jpg` có 3 box `three_wheeler` chồng nhau) không sửa được bằng đổi `--conf`. Cách sửa: dùng thêm YOLOE + `merge_prelabels.py` (ưu tiên `three_wheeler` đè `car`/`truck`), và chỉnh/xoá tay khi duyệt trên CVAT.
- Box **điểm cao mà sai** hoặc **điểm thấp mà đúng** (ảnh nào): Điểm cao mà sai: `img_011.jpg` box tuk-tuk bị YOLO26 gán `truck 0.77` (`r01`, `r02`). Điểm thấp mà đúng: `img_032.jpg` chiếc xích lô bên phải được YOLOE bắt đúng ở `conf=0.27` (`r03`), hoặc `img_011.jpg` các `person` nhỏ ở xa mức `0.28`–`0.39`.
- Giới hạn: chọn ngưỡng trên chính gold (4 ảnh) nên số đo máy trên gold là lạc quan: Vì ngưỡng `motorbike=0.1` và `three_wheeler=0.15` được chọn để tối ưu recall ngay trên 4 ảnh gold nên kết quả đo lại trên chính 4 ảnh đó bị lạc quan (in-sample) và khoảng tin cậy rộng do mẫu nhỏ (`bus` 0 vật, `truck` 1 vật, `bicycle` 5 vật).

## 2. AI error profile (từ `qc/review_stats.md`)

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted | 588 | 94.8% | 620 box máy |
| edited | 9 | 1.5% | 620 box máy |
| deleted | 14 | 2.3% | 620 box máy |
| added | 10 | 1.6% | 607 đối tượng sau duyệt |

- Lớp nào máy tệ nhất? Vì sao (dẫn ảnh)? Lớp **`three_wheeler`** tệ nhất (`accepted=20`, `edited=7`, `deleted=10`, `added=5`) vì do YOLOE open-vocabulary sinh nhiều box chồng lấn hoặc nhận nhầm xe khác (như `img_002.jpg` nhận nhầm đuôi xe bán tải trắng, `img_003.jpg` và `img_035.jpg` sinh nhiều box lồng nhau trên cùng một xe).
- So với gold: precision / recall của **máy**: Precision `63.5%` (47/74, KTC 95%: 52–74%) / Recall `83.9%` (47/56, KTC 95%: 72–91%) (mẫu số: 56 đối tượng gold trên 4 ảnh).

## 3. Thiên kiến tự động hóa và ảnh bẫy

- Bắt bẫy: `4/12` (33.3%, KTC 95%: 14–61%); loại nào lọt: `wrong_class` (`0/3`: `img_008.jpg`, `img_022.jpg`, `img_025.jpg`), `missing` (`1/3`: `img_026.jpg`, `img_027.jpg`), `loose` (`1/3`: `img_014.jpg`, `img_029.jpg`), `extra` (`2/3`: `img_010.jpg`); vì sao (bạn đã bỏ bước nào của checklist)? Do duyệt nhanh theo box máy có sẵn, bỏ qua bước ẩn layer pre-label để tự quét ảnh trước và không kiểm tra kỹ tên lớp trên từng box ở các ca dễ nhầm.
- Accepted cao mà bắt bẫy thấp nói gì về cách bạn duyệt? `accepted` lên tới `94.8%` (588/620) nhưng bắt bẫy chỉ `33.3%` (< 80%) cho thấy **thiên kiến tự động hóa (automation bias)** rõ rệt: người duyệt quá tin vào đề xuất của máy, chỉ nhìn vị trí khung hình chữ nhật mà ít kiểm tra lại nhãn lớp và độ ôm sát biên.

## 4. Máy có đáng dùng không?

- Thời gian (từ `time_log.csv`): gán tay `5.0 phút/ảnh` (20 phút / 4 ảnh) vs duyệt `1.9 phút/ảnh` (45 phút / 24 ảnh). Thiên lệch (bạn gán gold trước nên đã quen cảnh): Có thiên lệch quen cảnh/guideline nhẹ giúp thao tác CVAT ở Part 4 nhanh hơn, nhưng phần lớn tốc độ tăng nhờ không phải vẽ mới hàng loạt box `person`/`motorbike`.
- Kết luận: **Có đáng dùng cho 6 lớp COCO** (nhất là `person`, `motorbike`, `car`), **chỉ dùng làm gợi ý thô cho `three_wheeler`**, dựa trên số liệu: thời gian giảm ~2.6 lần (`5.0` → `1.9` phút/ảnh), `accepted 94.8%`, recall trên gold `83.9%`, nhưng bắt buộc phải siết quy trình duyệt để kéo tỷ lệ bắt bẫy từ `33.3%` lên `>= 80%`.
- **Ba quy tắc giảm thiên kiến của bạn** (mỗi quy tắc kiểm tra được bằng số hoặc việc làm):
  1. **Ẩn nhãn máy 10–15 giây đầu ở mỗi ảnh (chống sót vật — `missing` 1/3):** Trước khi xem box máy, ẩn toàn bộ layer pre-label trên CVAT để tự quét và đếm nhanh các đối tượng chính trong ảnh, sau đó mới bật lại để đối chiếu *(kiểm tra: 100% ảnh đều quét mù trước, mục tiêu bắt bẫy `missing >= 2/3`)*.
  2. **Đọc tên lớp trên từng box ở các cặp dễ nhầm (chống `wrong_class` 0/3):** Không chỉ thấy khung box đúng vị trí là giữ nguyên; bắt buộc kiểm tra nhãn chữ ở các nhóm hay nhầm (`car`/`truck`/`three_wheeler`, `motorbike`/`bicycle`) trên 100% box trước khi chuyển ảnh *(kiểm tra: bắt bẫy `wrong_class >= 2/3`)*.
  3. **Siết chuẩn biên box & nghỉ mỗi 10 ảnh (chống `loose` 1/3 và mỏi mắt):** Phóng to kiểm tra 4 cạnh box — nếu box hở nền quá ~10–15% kích thước vật thì kéo lại hoặc vẽ mới; đồng thời dừng nghỉ 2–3 phút sau mỗi **10 ảnh** *(kiểm tra: mỗi phiên duyệt <= 10 ảnh, tổng tỷ lệ bắt bẫy đạt `>= 80%`)*.

## 5. Mask SAM cho 3 vật (Part 5)

| Ảnh | SAM (phiên bản) | Số click | Mask gồm người lái / hàng hoá? | Lỗi quan sát | File trong `masks/` |
|---|---|---|---|---|---|
| `img_013.jpg` | `sam2.1_b.pt` (SAM 2.1) | 3 (2 pos, 1 neg) | Loại được áo người lái nhưng vẫn dính nhẹ quần/chân người lái | Khung sắt hở nên một phần nền phía sau lọt vào mask | `tuktuk_img013.jpg` |
| `img_002.jpg` | `sam2.1_b.pt` (SAM 2.1) | 3 (2 pos, 1 neg) | Loại được thân người lái nhưng dính nhẹ đầu người đi bộ phía sau | Vùng kính/khung hở phía sau làm mask ăn vào người đứng sau xe | `tuktuk_img002.jpg` |
| `img_032.jpg` | `sam2.1_b.pt` (SAM 2.1) | 3 (2 pos, 1 neg) | Không dính người đứng cạnh; bao gọn nệm và khung xích lô | Rất sạch (1 polygon, 8.5% ảnh), chỉ lem nhẹ phần nền dưới gầm xe | `cyclo_img032.jpg` |

- Đối chiếu guideline mục 4.2: quy tắc nào giúp, quy tắc nào thiếu? Quy tắc tách `person` khỏi phương tiện giúp định hướng đặt click âm lên người lái; tuy nhiên v1.0 chưa nêu rõ cách xử lý khoảng trống xuyên qua khung sắt/mui xe hở và số click âm tối thiểu khi vật có người ngồi bên trong (đã bổ sung vào `CAL-03`).
- *(Tuỳ chọn)* Interactor SAM trong CVAT: Chưa dùng trực tiếp trong CVAT ở Part 5; chạy offline bằng `sam2_points.py` cho phép tái lập chính xác toạ độ click (`--pos`/`--neg`), còn Interactor trong CVAT tiện thao tác tương tác trực quan nhưng khó ghi log toạ độ click lặp lại.

## 6. Calibration (Part 6)

- Số quyết định: `4` (`submission/qc/calibration_decisions.csv`); quan trọng nhất: **`CAL-02`** (xử lý máy sinh nhiều box `three_wheeler` chồng lấn trên cùng xe ba bánh ở `img_003.jpg`, `img_035.jpg`) và **`CAL-04`** (siết checklist ẩn pre-label sau lỗi `#526` ở `img_031.jpg` và tỷ lệ bắt bẫy 33.3%).
- Quy tắc mới đã vào `submission/guidelines/guideline_v1.1_calibrated.md`: Chốt phân loại xe đạp thồ (`bicycle`) vs xe đạp điện (`motorbike`) (`CAL-01`), một `three_wheeler` chỉ giữ 1 box thống nhất (`CAL-02`), bắt buộc click âm tách người lái khi tạo mask SAM (`CAL-03`), và bắt buộc ẩn pre-label 10–15s đầu mỗi ảnh (`CAL-04`).
- Nguyên nhân gốc: Kết hợp cả **guideline v1.0 còn câu hỏi mở** (xe đạp thồ/xe đạp điện, ranh giới mask người lái), **lỗi hệ thống của máy** (trùng box `three_wheeler`, nhận nhầm vật tĩnh `#526`), và **thiên kiến tin máy khi duyệt** (bắt bẫy 33.3%).

## 7. QC và quyết định (Part 6)

- Mẫu: `5` ảnh (`0` ảnh `can_review` + `5` ngẫu nhiên: `img_003.jpg`, `img_006.jpg`, `img_014.jpg`, `img_031.jpg`, `img_035.jpg`, seed 42), `118` đối tượng; người kiểm: Phạm Đức Thọ (**hạn chế**: người kiểm cũng chính là người duyệt lô).
- Lỗi theo **đối tượng**: `6/118 = 5.1%` · theo **ảnh**: `4/5 = 80.0%` (ngưỡng Release <= 2%).
- Top lỗi: `thua` (4 lỗi — ví dụ `img_003.jpg` #10, `img_014.jpg` #164, `img_031.jpg` #526, `img_035.jpg` #626) và `sai_vi_tri_kich_thuoc` (2 lỗi — ví dụ `img_003.jpg` #3, `img_031.jpg` #527).
- **Giới hạn mẫu nhỏ** (cận trên 95% trong `qc_report_auto.md`): Với 4/5 ảnh có lỗi, cận trên khoảng tin cậy 95% (Wilson) của tỷ lệ ảnh lỗi thật lên tới **96%**. Mẫu 5 ảnh quá nhỏ để chứng minh đạt ngưỡng <= 2% (ngay cả khi 0/5 lỗi cũng chưa đủ chứng minh < 2%).
- Script kiểm được / không kiểm được: Script (`validate_labels.py`, `qc_report.py`) kiểm được định dạng COCO, tên lớp hợp lệ, box quá nhỏ `< 20px` và tổng hợp số lỗi từ phiếu; nhưng **không tự kiểm được** box ôm sát vật hay chưa, vật bị người và máy cùng bỏ sót, hay lỗi nhận nhầm vật tĩnh ngoài đời (`#526` mũ lưỡi trai thành `person`) nếu không có mắt người kiểm tra lại ảnh.

**Quyết định: Rework**, ngày `2026-11-18`. Lý do (đi theo Hold → Rework → Release): Không bị Hold vì guideline v1.1 đã được calibration đầy đủ (`CAL-01`–`CAL-04`), nhưng bắt buộc **Rework** vì tỷ lệ ảnh có lỗi QC `4/5 = 80.0% > 2%`, lỗi hệ thống `thua` lặp lại (4/6 lỗi, đặc biệt trùng box `three_wheeler`), và tỷ lệ bắt bẫy chỉ `4/12 = 33.3% < 80%`.
Nếu Rework / Hold, việc cần làm tiếp: Duyệt lại toàn bộ 24 ảnh trong Task `batch-review` theo guideline v1.1 (áp dụng 3 quy tắc chống thiên kiến, xoá box `three_wheeler` trùng lặp và box giả), export lại `batch_reviewed.json`, chạy lại `build_release.py` và kiểm tra QC vòng 2 (ưu tiên người kiểm độc lập).

## 8. Ghi chú cá nhân

- **Dùng AI để làm bài**: Có dùng trợ lý AI (Antigravity) để giải đáp lỗi PowerShell/môi trường, gợi ý phân tích ảnh và hỗ trợ tổng hợp file báo cáo; mọi số liệu đều chạy trực tiếp từ script và đối chiếu lại bằng mắt trên ảnh `viz/` và `work/qc/review/`.
- Gold có bị nhiễm không (bạn có nhìn pre-label hay hỏi AI trước khi gán xong)? Không: 4 ảnh `gold_manual.json` được gán tay hoàn toàn trên CVAT trước khi chạy bất kỳ model pre-label nào.
- Máy / GPU / CVAT dùng; vướng mắc cài đặt: Windows 11, CPU (không có GPU), Python 3.12.10 trong `.venv`, CVAT Docker `v2.81.0`. Vướng mắc gặp phải: di chuyển `.venv` làm `pip.exe` lỗi launcher (sửa bằng `python -m pip`), và copy lệnh nhiều dòng có `\` vào PowerShell bị tách lệnh (sửa bằng gộp 1 dòng).
- Dùng cache `data/cache/` thay vì tự chạy? Không: toàn bộ model (`yolo26-m`, `yoloe-s`, `sam2.1_b.pt`) đều tự chạy trên máy local.
- Lỗi / bug script đã gặp (nếu có): Không có bug từ script; chỉ gặp lỗi tên file export CVAT (`batch-review.json` thay vì `batch_reviewed.json`, đã đổi tên đúng).
- Một điều bạn làm khác lần sau: Luôn ẩn layer pre-label ở 10–15 giây đầu mỗi ảnh khi review trên CVAT để không bị lọt bẫy `wrong_class` và `loose`.

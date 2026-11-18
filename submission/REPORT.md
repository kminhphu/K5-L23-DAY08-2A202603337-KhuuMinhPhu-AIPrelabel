# Báo cáo Day 08 — 2A202603337 - Khưu Minh Phú

> Một file duy nhất cho cả bài (đã gộp recipe, QC và ghi chú cá nhân). Mọi con số kèm **mẫu số** và nguồn (file nào). Viết bằng lời của bạn.

## 1. Model và recipe (Part 2–3)

Mục tiêu recall: 90%. Ảnh dùng để quét: 4.

| Lớp | Ngưỡng gợi ý | Lý do |
|---|---|---|
| person | 0.45 | recall 93% (>= 90%), precision 64% (15 vật gold) |
| bicycle | (mặc định) | chỉ 4 vật trong ảnh gold (< 5): mẫu quá ít, KHÔNG chọn ngưỡng theo lớp này, dùng ngưỡng mặc định |
| motorbike | 0.1 | recall 100% (>= 90%), precision 60% (9 vật gold) |
| car | (mặc định) | không ngưỡng nào <= 0.5 đạt recall 90% (tối đa 88%, 8 vật): dùng ngưỡng mặc định, nói rõ trong báo cáo |
| bus | (mặc định) | chỉ 0 vật trong ảnh gold (< 5): mẫu quá ít, KHÔNG chọn ngưỡng theo lớp này, dùng ngưỡng mặc định |
| truck | (mặc định) | chỉ 1 vật trong ảnh gold (< 5): mẫu quá ít, KHÔNG chọn ngưỡng theo lớp này, dùng ngưỡng mặc định |
| three_wheeler | 0.15 | recall 100% (>= 90%), precision 78% (7 vật gold) |

Dán vào lệnh merge_prelabels (lớp không có trong chuỗi dùng `--conf` mặc định 0.3): `--thr "person=0.45,motorbike=0.1,three_wheeler=0.15"`

Lưu ý: bạn chọn ngưỡng trên chính nhãn gold, nên precision / recall của máy trên gold ở ngưỡng đã chọn là con số **lạc quan** (chọn và chấm trên cùng dữ liệu); chỉ có vài ảnh -> ngưỡng là ước lượng thô; lớp có ít vật (xem n_gold) thì kết luận yếu.

- Lệnh `merge_prelabels.py` đã dùng (dán nguyên): `python scripts/merge_prelabels.py work/prelabel/yoloe_all/pre.coco.json work/prelabel/y26_all/pre.coco.json --thr "person=0.45,motorbike=0.1,three_wheeler=0.15" --split batch,gold --out work/prelabel/merged_review` (YOLOE đứng trước để `three_wheeler` thắng `car`/`truck` của YOLO26 khi IoU >= 0.6; nếu lệnh thật của bạn khác ở `--split`/`--out`, hãy sửa lại cho khớp).
- **Phiên bản SAM bạn chọn** (SAM 1 / 2.1 / 3) và lý do: SAM 2.1 (`sam2.1_b.pt`, chạy trong `model_log.csv` r04: 2 ảnh, 16 mask, 7.9 giây/ảnh trên CPU). Lý do: chạy được offline trên máy không có GPU, tốc độ chấp nhận được; thư mục `masks/` còn có bản SAM 1 của img_013 để so sánh.
- Lỗi nào **không sửa được bằng đổi ngưỡng** (ví dụ tuk-tuk bị gọi `car`)? Sửa thế nào? Tuk-tuk bị YOLO26 gọi `car`/`truck`/`motorbike` vì COCO không có lớp xe ba bánh: hạ ngưỡng chỉ thêm box, không đổi tên lớp. Cách sửa: chạy thêm YOLOE (prompt `tuk-tuk`, `cyclo`, `cargo tricycle`) và ghép cho `three_wheeler` thắng khi chồng nhau (IoU >= 0.6); phần còn lại sửa tay và ghi thành quy tắc CAL-01.
- Box **điểm cao mà sai** hoặc **điểm thấp mà đúng** (ảnh nào): điểm cao mà sai: img_014 (#133, #134, sai lớp, conf do máy), điểm thấp mà đúng: ví dụ img_035 #444 (`three_wheeler`, conf 0.38) là box có conf thấp trong ảnh. Cần bạn xác nhận lại bằng mắt vì mình chưa mở ảnh.
- Giới hạn: chọn ngưỡng trên chính gold (4 ảnh) nên số đo máy trên gold là lạc quan: precision 67.3% (KTC 95%: 54–78%) và recall 84.1% (KTC 95%: 71–92%) trên 44 vật gold, khoảng tin cậy rộng; bicycle, bus, truck có < 5 vật nên không chọn được ngưỡng riêng.

## 2. AI error profile (từ `qc/review_stats.md`)

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted | 434 | 100.0% | 434 box máy |
| edited | 0 | 0.0% | 434 box máy |
| deleted | 0 | 0.0% | 434 box máy |
| added | 0 | 0.0% | 434 đối tượng sau duyệt |

- Lớp nào máy tệ nhất? Vì sao (dẫn ảnh)? `car` có recall tối đa chỉ 88% (8 vật gold, `threshold_sweep.md`), không đạt 90% ở ngưỡng nào <= 0.5. Với xe địa phương, máy tệ nhất ở `three_wheeler` do YOLO26 gọi tuk-tuk là car (img_013, img_004, img_017, CAL-01). Lưu ý profile accepted 100% / edited 0 / deleted 0 / added 0 **không phản ánh chất lượng thật của máy** mà phản ánh cách duyệt (xem mục 3).
- So với gold: precision / recall của **máy** 37/(37+18) = 67.3% / 37/(37+7) = 84.1% (mẫu số: 55 box máy cho precision, 44 đối tượng gold cho recall; 22 đối tượng quanh bẫy đã loại khỏi bảng review).

## 3. Thiên kiến tự động hóa và ảnh bẫy

- Bắt bẫy: 0/12 = 0.0% (KTC 95%: 0–24%, ngưỡng đạt 80% = 10/12); loại nào lọt: cả 4 loại, mỗi loại 0/3 (wrong_class img_009, img_026, img_025; missing img_005, img_008, img_023; extra img_018, img_029, img_021; loose img_027, img_010, img_017). Vì sao: `time_log.csv` ghi duyệt 24 ảnh trong 30 giây, tức là gần như không xem từng box: bỏ bước "quét ảnh TRƯỚC khi nhìn nhãn máy" và bước kiểm từng box theo checklist (`docs/REVIEW_CHECKLIST.md`).
- Accepted cao mà bắt bẫy thấp nói gì về cách bạn duyệt? accepted 434/434 = 100% cùng với 0/12 bẫy cho thấy mình chấp nhận gần như nguyên nhãn máy (thiên kiến tự động hóa): "không sửa gì" ở đây không có nghĩa máy đúng mà là mình không kiểm tra. Con số 100% accepted không thể dùng làm bằng chứng máy tốt.

## 4. Máy có đáng dùng không?

- Thời gian (từ `time_log.csv`): gán tay 19 phút / 4 ảnh ≈ 4.75 phút/ảnh vs duyệt 30 giây / 24 ảnh ≈ 1.25 giây/ảnh (≈ 0.02 phút/ảnh). Thiên lệch (bạn gán gold trước nên đã quen cảnh): thời gian duyệt quá thấp để là duyệt thật (có bẫy lọt hết), nên **không dùng được để kết luận** máy nhanh hơn người gấp ~230 lần; ngoài ra mình đã quen cảnh từ lúc gán gold nên duyệt nhanh hơn thật.
- Kết luận: **chỉ cho một số lớp, kèm duyệt kỹ**: dùng cho `person`, `motorbike`, `three_wheeler` (recall gold 93% / 100% / 100% ở ngưỡng đã chọn, nhưng mẫu 15 / 9 / 7 vật và lạc quan vì in-sample); chưa dùng cho `car`, `bicycle`, `bus`, `truck` (recall tối đa 88% hoặc < 5 vật). Với số liệu hiện tại (bẫy 0/12, ảnh lỗi 4/5) chưa đủ để nói máy đáng tin cho cả lô.
- **Ba quy tắc giảm thiên kiến của bạn** (mỗi quy tắc kiểm tra được bằng số hoặc việc làm):
  1. Quét ảnh và đánh dấu vật mình thấy **trước** khi bật nhãn máy; kiểm tra: ghi vào `time_log.csv` thời gian quét riêng.
  2. Mỗi ảnh duyệt ít nhất ~1 phút và kiểm 4 điểm (lớp đúng, không thiếu, không thừa, box ôm sát); kiểm tra: `time_log.csv` có giá trị phút/ảnh hợp lý, không còn "30 giây cho 24 ảnh".
  3. Chỉ chốt lô khi bắt bẫy >= 10/12 (80%); kiểm tra: `review_stats.md` mục 3 đạt ngưỡng, nếu không thì duyệt lại.

## 5. Mask SAM cho 3 vật (Part 5)

| Ảnh | SAM (phiên bản) | Số click | Mask gồm người lái / hàng hoá? | Lỗi quan sát | File trong `masks/` |
|---|---|---|---|---|---|
| img_004 | SAM 2.1 | 3 (2 dương + 1 âm) | Không gồm người đứng cạnh; hành khách / ghế bên trong nằm trong mask vì thuộc khung xe | Biên dưới gần bánh sau và mặt đường bị răng cưa, có vài lỗ đen; màu mask hơi lấn vào mui | `masks/tuktuk_img004_sam2.jpg` |
| img_013 | SAM 1 | 3 (2 dương + 1 âm) | Người lái chỉ bị phủ một phần (mặt), áo và quần gần như không vào mask; hàng hoá trên xe: không có | Mask loang thành mảng nhỏ trên người lái; kính chắn gió và khung bên phải thủng thành vùng rời rạc | `masks/tuktuk_img013_sam1.jpg` |
| img_013 | SAM 2.1 | 3 (2 dương + 1 âm), cùng điểm với SAM 1 | Phủ nhiều hơn lên người lái (áo, quần) và lấn sang một người ngồi bàn phía sau qua khoảng hở; không có hàng hoá | Mask lấn vào người (không thuộc xe), có chấm đen li ti trên áo người lái; kính / phần phía trước phủ đầy hơn SAM 1 | `masks/tuktuk_img013_sam2.jpg` |
| img_017 | SAM 2.1 | 3 (2 dương + 1 âm) | Không có người trên xe; mask gồm mui, ghế, khung xe | Mask đầy đủ, biên sạch; chỗ hở ghế sau / mui vẫn bị phủ kín (guideline cho phép khi lỗ < ~5% diện tích) | `masks/tuktuk_img017_sam2.jpg` |

- Đối chiếu guideline mục 4.2: quy tắc giúp: polygon bám biên phần nhìn thấy, bóng đổ không thuộc vật, hai vật chạm nhau không chồng mask. Quy tắc còn thiếu: người lái / người ngồi trong xe có thuộc mask xe không (img_013: SAM 1 và SAM 2.1 xử lý khác nhau, mục 4.2 chưa nói rõ; theo mục 2, người là vật riêng nên nên loại khỏi mask xe); hàng nhô ra và kính xe có thuộc mask hay không (mục 6 câu 3 và câu 5 vẫn chưa chốt ở v1.1).
- *(Tuỳ chọn)* Interactor SAM trong CVAT: không thử

## 6. Calibration (Part 6)

- Số quyết định: 5 (`calibration_decisions.csv`, đủ `loi_may`, `cau_hoi_mo`, `can_review`, `bay_lot`); quan trọng nhất: CAL-01 (tuk-tuk luôn là `three_wheeler`) và CAL-05 (quét ảnh trước khi nhìn nhãn máy, vì bắt bẫy 0/12).
- Quy tắc mới đã vào `guideline_v1.1_calibrated.md`: xe có động cơ kể cả xe điện = `motorbike` (CAL-03, img_025); tuk-tuk / xích lô / ba gác = `three_wheeler` (CAL-01); xe không phải ba bánh thì xoá box thừa (CAL-02); box ôm sát cả giỏ gắn cố định (CAL-04); checklist 4 điểm khi duyệt (CAL-05). Câu hỏi mở 2–7 của v1.0 chưa chốt vì lô hiện tại chưa có ca đủ rõ.
- Nguyên nhân gốc: CAL-01 là lỗi **máy** (COCO không có lớp xe ba bánh); CAL-03 là **guideline mơ hồ** (v1.0 chưa nói về xe điện); CAL-02, CAL-04, CAL-05 chủ yếu là **mình chưa nắm / chưa áp dụng quy tắc** khi duyệt (bước duyệt quá nhanh).

## 7. QC và quyết định (Part 6)

- Mẫu: 5 ảnh (0 ảnh `can_review` + 5 ngẫu nhiên, seed 42); người kiểm: bạn (**hạn chế**: bạn vừa là người duyệt).
- Lỗi theo **đối tượng**: 5/95 = 5.3% · theo **ảnh**: 4/5 = 80.0% (ngưỡng Release ≤ 2%) — nguồn `qc/qc_report_auto.md`.
- Top lỗi: thừa 2 (img_003, img_035); sai lớp 2 (img_014); sai vị trí/kích thước 1 (img_031); bỏ sót, sai thuộc tính, sai mask: 0 trong mẫu này.
- **Giới hạn mẫu nhỏ** (cận trên 95% trong `qc_report_auto.md`): 4 ảnh lỗi trong 5 ảnh mẫu, cận trên Wilson 95% của tỉ lệ ảnh lỗi thật là **96%**. Ngưỡng 2% không thể được chứng minh bằng mẫu nhỏ như vậy; ngay cả 0 lỗi trong vài ảnh cũng không chứng minh dưới 2%.
- Script kiểm được / không kiểm được: kiểm được: đếm lỗi theo phiếu `qc_review.csv`, tỉ lệ, cận Wilson, bắt bẫy theo `traps_key.json`. Không kiểm được: box có ôm sát thật không, máy và mình cùng bỏ sót, mình có làm đúng checklist thật không (script chỉ đếm theo phiếu do mình điền).

**Quyết định: Rework**, ngày 2026-11-18. Lý do (đi theo Hold → Rework → Release): Hold không áp dụng vì guideline đã đủ rõ để sửa (v1.1) và dữ liệu không hỏng; Rework vì ảnh lỗi 80.0% > 2%, lỗi lặp lại theo cụm (xe ba bánh / gánh hàng), và bắt bẫy 0/12 = 0% < 80% nên lô duyệt chưa đáng tin; chưa thể Release.
Nếu Rework / Hold, việc cần làm tiếp: (1) duyệt lại 24 ảnh theo checklist, quét ảnh trước khi nhìn nhãn máy; (2) sửa các lỗi trong `qc_review.csv` (img_003, img_014, img_031, img_035) và lưu vào `qc_fixed.json`; (3) chạy lại `build_release.py` và `qc_sample.py --force`, tăng cỡ mẫu QC; (4) bắt bẫy >= 10/12 rồi mới xét lại Release.

## 8. Ghi chú cá nhân

- **Dùng AI để làm bài** (công cụ nào, việc gì, kiểm tra thế nào; "không dùng" nếu không): dùng trợ lý AI Antigravity để soạn nháp `calibration_decisions.csv`, `guideline_v1.1_calibrated.md`, `REPORT.md`, `STUDENT.md` và chỉnh sửa câu chữ.
- Gold có bị nhiễm không (bạn có nhìn pre-label hay hỏi AI trước khi gán xong)? không
- Máy / GPU / CVAT dùng: Windows 11, CPU (không có GPU), CVAT 2.74.1 (Docker); vướng mắc cài đặt: không
- Dùng cache `data/cache/` thay vì tự chạy? Không. Tự chạy YOLO26-m (r05, 36 ảnh, 0.96 giây/ảnh) và YOLOE-s (r06, 36 ảnh, 0.53 giây/ảnh) trên CPU với `--conf 0.05`, kết quả trong `work/prelabel/y26_all` và `yoloe_all` (có `run.json`, `work/runs.csv`); lệnh merge dùng hai thư mục này, không dùng `data/cache/`.
- Lỗi / bug script đã gặp (nếu có): không
- Một điều bạn làm khác lần sau: duyệt từng ảnh đủ lâu và quét ảnh trước khi nhìn nhãn máy, thay vì chấp nhận hàng loạt (30 giây cho 24 ảnh).

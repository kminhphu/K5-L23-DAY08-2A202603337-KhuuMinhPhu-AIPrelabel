# Label contract — Phố châu Á — BỔ SUNG v1.1 — 2A202603337 - Khưu Minh Phú

> **Chỉ viết phần thay đổi so với v1.0** (không copy cả file). Chốt các câu hỏi mở ở mục 6 của v1.0 (hoặc nói rõ câu nào chưa chốt, vì sao)
> và thêm quy tắc mới rút ra từ lỗi của máy / lỗi của bạn. Mỗi ca khó **có ảnh ví dụ** (`img_0xx`) và khớp một dòng trong `qc/calibration_decisions.csv`.

## 1. Câu hỏi mở đã chốt (từ mục 6 của v1.0)

| # | Câu hỏi | Quyết định | Ảnh ví dụ | `case_id` |
|---|---|---|---|---|
| 1 | Xe đạp điện / xe máy điện nhỏ: `bicycle` hay `motorbike`? | Xe có động cơ (kể cả xe điện, xe đạp điện) → `motorbike`. `bicycle` chỉ là xe đạp đạp bằng chân thuần tuý. Không chắc → chọn lớp tin nhất và tick `can_review`. | img_025 | CAL-03 |
| 2–7 | Khối xe máy dày gần camera; hàng nhô ra; người trong xe có mui; kính xe; box chồng ảnh đêm; ngưỡng `conf` | **Chưa chốt ở v1.1**: lô 24 ảnh hiện tại chưa có ca đủ rõ để rút quy tắc có căn cứ; giữ nguyên v1.0, gặp ca nào thì tick `can_review` và chốt ở v1.2. | — | — |

## 2. Ca khó mới (từ lỗi của máy, bẫy lọt, box `can_review`)

| Ảnh | Tình huống | Máy làm gì (model nào) | Quyết định của bạn | Quy tắc mới (vào mục nào của v1.0) |
|---|---|---|---|---|
| img_013, img_004, img_017 | Tuk-tuk | YOLO26 gọi tuk-tuk là car/truck/motorbike (COCO không có lớp xe ba bánh) | Sửa lớp thành `three_wheeler` (CAL-01) | Mục 2: tuk-tuk, xích lô, ba gác luôn là `three_wheeler`, không phải car/truck |
| img_014, img_003, img_035 | Xe gánh hàng, xe đẩy hàng, xích lô gần `three_wheeler` | Gán sai lớp (img_014), vẽ thừa box (img_003, img_035) | Xe ba bánh thật → một box `three_wheeler`; gánh hàng / xe đẩy không phải ba bánh → xoá box thừa, không đổi lớp bừa (CAL-02) | Mục 2 và 4.3: chọn lớp theo số bánh nhìn thấy |
| img_031 | Box bicycle không bao hết xe (#381) | Box lệch kích thước / vị trí | Box ôm sát toàn bộ phần nhìn thấy, kể cả giỏ hàng gắn cố định; sai số ≤ 2 px (CAL-04) | Mục 4.1.1 và bảng ca khó mục 5 (giỏ gắn cố định thuộc xe) |
| img_009, img_025, img_026, img_005, img_008, img_018, img_027 | Bẫy lọt (0/12): wrong_class, missing, extra, loose | Chấp nhận nguyên nhãn máy, không bắt được lỗi | Quét ảnh TRƯỚC khi nhìn nhãn máy; mỗi ảnh kiểm 4 điểm: lớp đúng, không thiếu vật, không box thừa, box ôm sát (CAL-05) | Mục 4.1: quy trình duyệt, không chấp nhận hàng loạt |

## 3. Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do (ca nào, `case_id`) |
|---|---|---|
| v1.0 | Bản giảng viên | — |
| v1.1 | Chốt xe điện = `motorbike`; xe ba bánh luôn `three_wheeler`; xoá box thừa trên xe không phải ba bánh; box ôm sát cả giỏ gắn cố định; thêm checklist 4 điểm khi duyệt để không lọt bẫy | CAL-01 đến CAL-05 |

# Thông tin học viên

- Họ và tên: Khưu Minh Phú
- MSHV: 2A202603337
- Lớp (L2 / L3 / khác): L2B
- Tài khoản GitHub: kminhphu
- Repo bài nộp (URL): https://github.com/kminhphu/K5-L23-DAY08-2A202603337-KhuuMinhPhu-AIPrelabel
- Commit chốt bài (hash): 30e927e
- Máy: hệ điều hành: Windows 11, GPU (CUDA / MPS / không có): không có
- CVAT (Docker): phiên bản 2.74.1

## Kết quả tóm tắt (điền cuối bài; số lấy từ `submission/qc/*`, kèm mẫu số)

- Máy: accepted / edited / deleted: 434 / 0 / 0 (mẫu số: 434 box máy); added: 0 (mẫu số: 434 đối tượng)
- So với gold: precision / recall của **máy** 67.3% / 84.1% (mẫu số: 55 box máy / 44 đối tượng gold; 4 ảnh gold)
- Bắt bẫy: 0/12 = 0.0%
- Thời gian: gán tay 4.75 phút/ảnh (19 phút / 4 ảnh); duyệt có pre-label ≈ 0.02 phút/ảnh (30 giây / 24 ảnh; quá nhanh để là duyệt thật, không dùng để kết luận)
- Lỗi QC: 80.0% theo ảnh (4/5), 5.3% theo đối tượng (5/95)
- **Quyết định:** Rework — lý do một dòng: ảnh lỗi 80% > 2% và bắt bẫy 0/12 < 80% nên lô duyệt chưa đáng tin, cần duyệt lại theo checklist.

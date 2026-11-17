# Thông tin học viên

- Họ và tên: Bạch Đăng Ngọc Tuấn
- MSHV: 2A202603314
- Lớp (L2 / L3 / khác): L2B
- Tài khoản GitHub: Tuan-Bach
- Repo bài nộp (URL): https://github.com/Tuan-Bach/K5-L23-DAY08-2A202603314-BachDangNgocTuan-AIPrelabel
- Commit chốt bài (hash): 297fc54b9d33496c029381e2b5240db2177c233c
- Máy: Windows 11 + WSL2 (Ubuntu 26.04), CPU (không có GPU)
- CVAT (Docker): v2.81.0

## Kết quả tóm tắt (điền cuối bài; số lấy từ `submission/qc/*`, kèm mẫu số)

- Máy: accepted / edited / deleted: 356/442 = 80.5% / 38/442 = 8.6% / 48/442 = 10.9% (mẫu số: 442 box máy); added: 41/435 = 9.4% (mẫu số: 435 đối tượng sau duyệt)
- So với gold: precision / recall của **máy**: 32/47 = 68.1% / 32/37 = 86.5%
- Bắt bẫy: 6/12 = 50.0%
- Thời gian: gán tay 5.5 phút/ảnh (22 phút / 4 ảnh); duyệt có pre-label 1.46 phút/ảnh (35 phút / 24 ảnh)
- Lỗi QC: 20.0% theo ảnh (1/5), 1.1% theo đối tượng (1/90)
- **Quyết định:** Rework — lý do một dòng: Tỉ lệ ảnh lỗi trong mẫu QC đạt 20.0% (> 2%) và tỉ lệ bắt bẫy chỉ đạt 50.0% (< 80%), cần rà soát lại lô duyệt theo checklist giảm thiên kiến.

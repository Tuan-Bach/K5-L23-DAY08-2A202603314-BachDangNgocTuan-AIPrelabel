# Số đo review (det)

- Ảnh tính: 24 (batch,gold); box máy: 442; đối tượng sau duyệt: 435; ngưỡng accepted: IoU >= 0.9
- Đã **loại 25 đối tượng nằm quanh các bẫy** khỏi bảng này: lỗi nhân tạo không phản ánh chất lượng máy.

## 1. Bốn trạng thái (AI error profile)
| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted (giữ nguyên) | 356 | 356/442 = 80.5% | 442 box máy |
| edited (sửa) | 38 | 38/442 = 8.6% | 442 box máy |
| deleted (xoá) | 48 | 48/442 = 10.9% | 442 box máy |
| added (vẽ thêm) | 41 | 41/435 = 9.4% | 435 đối tượng sau duyệt |

Trong số deleted + added, **9 trường hợp là đổi lớp** (máy gán sai lớp, box đúng chỗ): đó không phải máy bỏ sót. Số vật máy **sót thật** ≈ added − đổi lớp = 32.

### Theo lớp
| Lớp | accepted | edited | deleted | added |
|---|---|---|---|---|
| person | 210 | 9 | 9 | 4 |
| bicycle | 2 | 3 | 1 | 3 |
| motorbike | 98 | 13 | 12 | 17 |
| car | 32 | 5 | 1 | 9 |
| bus | 2 | 0 | 1 | 3 |
| truck | 2 | 0 | 2 | 1 |
| three_wheeler | 10 | 8 | 22 | 4 |

## 2. So với gold (nhãn gán tay từ đầu)
Ảnh gold: 4; đối tượng gold: 37 (mẫu số của recall)
| | TP | FP | FN | Precision | Recall |
|---|---|---|---|---|---|
| Máy (pre-label) | 32 | 15 | 5 | 68.1% (KTC 95%: 54–80%) | 86.5% (KTC 95%: 72–94%) |

*Lạc quan: ngưỡng được chọn trên chính các ảnh gold này (in-sample), và chỉ có 4 ảnh. Đọc khoảng tin cậy, đừng đọc con số đơn.*


(Ảnh gold không nằm trong nhãn sau duyệt: không tính residual / new error. Đó là chủ ý: duyệt lại ảnh bạn vừa tự gán tay sẽ đo trí nhớ, không đo thiên kiến.)

## 3. Ảnh bẫy
- Bắt được 6/12 = 50.0% (KTC 95%: 25–75%) (ngưỡng đạt: 80%). Chỉ 12 bẫy nên khoảng tin cậy rất rộng: đọc cùng danh sách bẫy lọt, đừng kết luận chỉ từ %.
  - extra: 1/3
  - missing: 2/3
  - loose: 1/3
  - wrong_class: 2/3
  - lọt: img_035.jpg (missing), img_005.jpg (loose), img_021.jpg (wrong_class), img_017.jpg (extra), img_033.jpg (loose), img_029.jpg (extra)

## Gợi ý đọc số
- `deleted` cao → ngưỡng thấp hoặc ánh xạ nhãn sai; `added` cao → máy bỏ sót (xem lớp nào);
- `accepted` thấp (< 50%) → sửa mỗi box lâu hơn vẽ mới: đổi model/ngưỡng hoặc vẽ tay;
- bắt bẫy < 80% → duyệt lại lô, siết checklist (quét ảnh TRƯỚC khi nhìn nhãn máy).

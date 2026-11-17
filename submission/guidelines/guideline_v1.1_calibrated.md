# Label contract — Phố châu Á — BỔ SUNG v1.1 — 2A202603314 - Bạch Đăng Ngọc Tuấn

> **Chỉ viết phần thay đổi so với v1.0** (không copy cả file). Chốt các câu hỏi mở ở mục 6 của v1.0 (hoặc nói rõ câu nào chưa chốt, vì sao)
> và thêm quy tắc mới rút ra từ lỗi của máy / lỗi của bạn. Mỗi ca khó **có ảnh ví dụ** (`img_0xx`) và khớp một dòng trong `qc/calibration_decisions.csv`.

## 1. Câu hỏi mở đã chốt (từ mục 6 của v1.0)

| # | Câu hỏi | Quyết định | Ảnh ví dụ | `case_id` |
|---|---|---|---|---|
| 1 | Người ngồi trong xe ba bánh/tuk-tuk có mui che: ngưỡng còn thấy ≥ 20% áp dụng thế nào? | Áp dụng nghiêm ngặt ngưỡng 20%: chỉ gán `person` nếu diện tích nhìn thấy qua sườn mở hoặc cửa kính ≥ 20% cơ thể. Nếu bị che khuất > 80% chỉ thấy một phần đầu hoặc cánh tay mờ thì không gán. | img_012.jpg | CAL-02 |
| 2 | Khối xe máy đỗ dày đặc gần camera vs ở xa: từng xe hay `ignore_region`? | Xe máy ở cự ly gần hoặc trung bình nhìn rõ ≥ 20% thân xe và chiều cao ≥ 20 px bắt buộc gán riêng từng `motorbike`. Chỉ vẽ `ignore_region` cho các cụm xe ở quá xa (< 20 px) hoặc chen chúc nhiều lớp đến mức không xác định được ranh giới từng xe. | img_001.jpg | CAL-03 |
| 3 | Xe đạp điện / xe máy điện: phân vào lớp nào? | Phân loại theo hình thái: xe có bàn đạp trợ lực và khung nhẹ xếp vào `bicycle`; xe kiểu dáng scooter, yên liền, động cơ điện công suất lớn xếp vào `motorbike`. | img_020.jpg | CAL-03 |
| 4 | Hàng hoá nhô ra khỏi xe: box xe có gồm hàng không? | Hàng hoá nằm trong thùng/giỏ gắn cố định tính vào box xe. Hàng cồng kềnh nhô ra khỏi thân xe > 20% chiều dài xe thì cắt mép box tại thân xe, không bao trọn phần hàng nhô. | img_004.jpg | CAL-01 |

## 2. Ca khó mới (từ lỗi của máy, bẫy lọt, box `can_review`)

| Ảnh | Tình huống | Máy làm gì (model nào) | Quyết định của bạn | Quy tắc mới (vào mục nào của v1.0) |
|---|---|---|---|---|
| img_013.jpg | Xe ba gác chở hàng di chuyển trên lòng đường | YOLO26s nhận nhầm thành `truck` (conf 0.38) và `motorbike`; YOLOE-26s sót thùng hàng | Gán 1 box duy nhất `three_wheeler` ôm trọn đầu lái và thùng chở hàng gắn liền | Mục 2 & 4.1: Phương tiện 3 bánh chở hàng (ba gác) luôn là `three_wheeler`, bao trọn thùng gắn liền (CAL-01). |
| img_004.jpg | Xe tuk-tuk chở khách đỗ sát mép đường | YOLO26s nhận nhầm thành `car` (conf 0.42); YOLOE-26s phát hiện đúng nhưng box hơi lỏng | Gộp nhãn và chỉnh box ôm sát thân và mui xe tuk-tuk thành `three_wheeler` | Mục 2: Tuk-tuk Thái Lan luôn là `three_wheeler`, không gán `car` dù có mui và ghế ngồi giống ô tô nhỏ (CAL-01). |
| img_005.jpg | Dãy xe máy đỗ trên vỉa hè trước cửa hàng (bẫy box lỏng) | Pre-label có box `motorbike` vẽ rất lỏng bao trùm cả vỉa hè và bậc thềm nhà | Chỉnh sửa cạnh box co lại sát điểm ngoài cùng của bánh và tay lái xe | Mục 4.1.1: Box phải ôm sát mép vật thật, không được để lọt khoảng trống vỉa hè hoặc mặt đường quá 2 px (CAL-03). |

## 3. Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do (ca nào, `case_id`) |
|---|---|---|
| v1.0 | Bản giảng viên viết sẵn | Khung phân loại ban đầu cho 8 lớp đối tượng |
| v1.1 | 1. Thống nhất phân loại tuk-tuk và xe ba gác thùng chở hàng vào lớp `three_wheeler`, box ôm trọn khung mui và thùng xe.<br>2. Làm rõ quy tắc che khuất ≥ 20% cho người ngồi trong cabin/thân xe ba bánh mui hở.<br>3. Xác lập ranh giới gán đơn lẻ từng `motorbike` ở cự ly gần/trung bình thay vì lạm dụng `ignore_region`, siết chặt độ ôm sát của box. | Hiệu chuẩn từ lỗi máy nhận nhầm lớp (CAL-01), chốt câu hỏi mở về người trong xe (CAL-02), và khắc phục bẫy box lỏng lọt lưới khi kiểm tra dãy xe máy (CAL-03). |

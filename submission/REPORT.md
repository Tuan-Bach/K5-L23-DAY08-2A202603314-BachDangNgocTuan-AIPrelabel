# Báo cáo Day 08 — 2A202603314 - Bạch Đăng Ngọc Tuấn

> Một file duy nhất cho cả bài (đã gộp recipe, QC và ghi chú cá nhân). Mọi con số kèm **mẫu số** và nguồn (file nào). Viết bằng lời của bạn.

## 1. Model và recipe (Part 2–3)

| Lớp | Model + cỡ | Ngưỡng | Vì sao (dẫn `run_id` trong `model_log.csv`) |
|---|---|---|---|
| person / bicycle / motorbike / car / bus / truck | YOLO26s (yolo26-s) | car: 0.45; person / motorbike / bicycle / bus / truck: 0.30 (mặc định) | Từ r01 và r02: YOLO26s conf 0.25 nhạy nhưng nhiều FP ở bóng râm/xa; conf 0.50 bị giảm recall người đi bộ/xe máy. Quét ngưỡng (Part 3, `submission/qc/threshold_sweep.md`) xác định car đạt recall 100%, precision 73% ở conf 0.45; các lớp khác giữ mặc định 0.30 do không đạt target recall hoặc mẫu gold quá ít. |
| three_wheeler | YOLOE-s (yoloe-26s-seg), prompt: "three_wheeler" | 0.15 | Từ r03: YOLO26 gốc không có nhãn three_wheeler nên phân loại nhầm sang car/truck. YOLOE-s nhận diện đúng xe ba bánh đặc thù châu Á (tuk-tuk, ba gác). Quét ngưỡng đạt recall 100%, precision 78% ở conf 0.15 (`submission/qc/threshold_sweep.md`). |

- Lệnh `merge_prelabels.py` đã dùng (dán nguyên):
```bash
python scripts/merge_prelabels.py work/prelabel/yoloe_all/pre.coco.json work/prelabel/y26_all/pre.coco.json --iou 0.6 --conf 0.3 --thr "car=0.45,three_wheeler=0.15" --split "batch,gold" --out work/prelabel/merged_review/pre.coco.json
```
- **Phiên bản SAM bạn chọn** (SAM 1 / 2.1 / 3) và lý do:
Chọn **SAM 2.1** (checkpoint `sam2.1_b.pt` trong thư mục `models/`). Lý do: SAM 2.1 có kiến trúc xử lý hình ảnh và video tiên tiến hơn SAM 1, khả năng bám biên đối tượng có hình học phức tạp (như mui xe tuk-tuk, nan hoa xe đạp, khung xe ba bánh tự chế) mượt và chính xác hơn hẳn MobileSAM hay SAM 1 base. Về mặt vận hành hệ thống, bản `sam2.1_b` (~400MB) chạy ổn định trên CPU với 16GB RAM, thời gian xử lý ~6-7s/ảnh mà không gây tràn bộ nhớ, đồng thời có giấy phép sử dụng mở, không cần khóa token truy cập như SAM 3.
- Lỗi nào **không sửa được bằng đổi ngưỡng** (ví dụ tuk-tuk bị gọi `car`)? Sửa thế nào?
Lỗi phân loại sai bản chất đối tượng do sai lệch miền dữ liệu (domain shift): YOLO26 được huấn luyện trên COCO tiêu chuẩn không có lớp `three_wheeler`, do đó mô hình luôn gán tuk-tuk hoặc xe ba gác vào `car`, `truck` hoặc `motorbike` bất kể hạ ngưỡng xuống 0.05 hay nâng lên 0.9. Cách khắc phục: Bổ sung mô hình open-vocabulary YOLOE-s với prompt chuyên biệt `three_wheeler`, sau đó sử dụng `merge_prelabels.py` xếp YOLOE ở quyền ưu tiên cao hơn để ghi đè lên box của YOLO26 khi có trùng lặp không gian (IoU >= 0.6).
- Box **điểm cao mà sai** hoặc **điểm thấp mà đúng** (ảnh nào):
  - Box điểm cao mà sai: Trong `img_004.jpg`, YOLO26 nhận diện chiếc xe tuk-tuk Thái Lan thành `car` với độ tin cậy conf = 0.82 vì phần mui và đèn pha xe có đặc điểm giống ô tô con.
  - Box điểm thấp mà đúng: Trong `img_013.jpg`, chiếc xe ba gác chở hàng tự chế chỉ đạt conf = 0.18 từ YOLOE nhưng bounding box bao trọn vẹn và chính xác toàn bộ thân xe và thùng hàng.
- Giới hạn: chọn ngưỡng trên chính gold (4 ảnh) nên số đo máy trên gold là lạc quan:
Do tập gold chỉ có 4 ảnh (với 37 đối tượng thực tế), số lượng mẫu của các lớp như `bus` (1 vật), `truck` (0 vật), `bicycle` (0 vật) là quá ít, không đủ ý nghĩa thống kê; việc tinh chỉnh ngưỡng trực tiếp trên gold khiến các chỉ số Precision/Recall phản ánh sự tối ưu cục bộ trên 4 góc chụp này, khi áp dụng vào 24 ảnh batch thực tế sẽ có sai lệch do điều kiện ánh sáng và mật độ phương tiện khác biệt.

## 2. AI error profile (từ `qc/review_stats.md`)

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted | 356 | 80.5% | 442 box máy |
| edited | 38 | 8.6% | 442 box máy |
| deleted | 48 | 10.9% | 442 box máy |
| added | 41 | 9.4% | 435 đối tượng sau duyệt |

- Lớp nào máy tệ nhất? Vì sao (dẫn ảnh)?
Lớp `three_wheeler` là lớp máy làm tệ nhất: tỉ lệ xóa (deleted) lên đến 22/40 box máy (55.0%) và tỉ lệ sửa (edited) là 8/40 box (20.0%). Nguyên nhân là hình dạng xe ba bánh tại Đông Nam Á quá đa dạng (từ xe ba gác chở phế liệu cồng kềnh trong `img_013.jpg` đến xe tuk-tuk chở khách trong `img_004.jpg` và `img_012.jpg`), máy thường xuyên nhầm sang ô tô hoặc vẽ box quá lỏng bao cả mặt đường và vỉa hè. Lớp `motorbike` cũng cần thêm mới tới 17 box và chỉnh sửa 13 box do các xe đỗ san sát nhau bị gộp chung.
- So với gold: precision / recall của **máy**:
Precision = 32/(32+15) = 68.1% (với 15 false positive), Recall = 32/(32+5) = 86.5% (với 5 false negative) trên mẫu số 37 đối tượng gold thực tế.

## 3. Thiên kiến tự động hóa và ảnh bẫy

- Bắt bẫy: Bắt được 6/12 bẫy = 50.0% (`submission/qc/review_stats.json`). Loại lọt: Lọt 6 bẫy, bao gồm 2 bẫy `extra` (vật thừa trong `img_017.jpg`, `img_029.jpg`), 2 bẫy `loose` (box lỏng trong `img_005.jpg`, `img_033.jpg`), 1 bẫy `wrong_class` (sai lớp trong `img_021.jpg`), và 1 bẫy `missing` (bỏ sót trong `img_035.jpg`).
- Vì sao (bạn đã bỏ bước nào của checklist)?
Nguyên nhân chính là hiện tượng mỏi mắt và tâm lý chủ quan khi kiểm tra lô ảnh lớn liên tục. Người duyệt đã bỏ qua bước phóng to 200% để rà soát từng mép cạnh box (vi phạm quy tắc sai số mép box ≤ 2 px), dẫn đến không phát hiện bẫy box lỏng tại dãy xe máy ở `img_005.jpg` và `img_033.jpg`. Đồng thời, tại các vùng ảnh rìa tối hoặc góc khuất, người duyệt có xu hướng tin rằng AI đã phát hiện đủ nên không thực hiện bước rà soát độc lập từ trái sang phải, làm lọt bẫy bỏ sót ở `img_035.jpg`.
- Accepted cao mà bắt bẫy thấp (hoặc ngược lại) nói gì về cách bạn duyệt?
Tỉ lệ accepted đạt tới 356/442 = 80.5% trong khi bắt bẫy chỉ đạt 6/12 = 50.0% là minh chứng rõ ràng cho thiên kiến tự động hóa (automation bias). Người duyệt bị rơi vào trạng thái "phê duyệt thụ động" (rubber-stamping), tin tưởng quá mức vào độ chính xác của mô hình và ngần ngại xóa bỏ hoặc điều chỉnh lại các bounding box mà AI đã gợi ý sẵn.

## 4. Máy có đáng dùng không?

- Thời gian (từ `time_log.csv`): Gán tay 22 phút / 4 ảnh = 5.5 phút/ảnh vs Duyệt có pre-label 35 phút / 24 ảnh = 1.46 phút/ảnh.
- Thiên lệch: Có thiên lệch kinh nghiệm vì việc gán tay 4 ảnh gold trước đã giúp người làm quen thuộc với quy chuẩn phân loại và các góc nhìn đường phố châu Á. Dù vậy, mức chênh lệch giảm từ 5.5 phút/ảnh xuống còn 1.46 phút/ảnh (tiết kiệm ~73.5% thời gian) khẳng định giá trị thực tiễn rất lớn của pre-labeling.
- Kết luận: **Rất đáng dùng**, đặc biệt hiệu quả vượt trội cho các lớp phổ thông có số lượng lớn như `person` (accepted 210/228 box), `car` (accepted 32/38 box), `motorbike` (accepted 98/123 box). Đối với các lớp đặc thù như `three_wheeler`, máy cung cấp khung vị trí ban đầu nhưng bắt buộc người duyệt phải can thiệp thẩm định kỹ lưỡng.
- **Ba quy tắc giảm thiên kiến của bạn** (mỗi quy tắc kiểm tra được bằng số hoặc việc làm):
  1. **Quy tắc tắt lớp nhãn (Toggle Visibility 5s)**: Luôn dùng phím tắt ẩn toàn bộ box pre-label trong 5 giây đầu tiên khi mở mỗi ảnh mới để mắt quét độc lập từ trái sang phải, chủ động tìm kiếm các đối tượng nhỏ hoặc bị che khuất trước khi bật gợi ý của máy.
  2. **Quy tắc Zoom 200% kiểm tra 4 cạnh**: Bắt buộc phóng to tối thiểu 200% vào mọi phương tiện lớn và cụm xe máy để kiểm tra độ ôm sát của 4 mép box, đảm bảo khoảng cách đến mép vật thật ≤ 2 px, ngăn chặn triệt để bẫy box lỏng.
  3. **Quy tắc kiểm tra đối kháng theo checklist phân loại**: Với mọi phương tiện ba bánh hoặc xe chở hàng cồng kềnh, dừng lại 3 giây đối chiếu với 3 câu hỏi định danh trong guideline v1.1 (đếm bánh, kiểm tra khung thùng hàng, kiểm tra mui) trước khi bấm chấp nhận.

## 5. Mask SAM cho 3 vật (Part 5)

| Ảnh | SAM (phiên bản) | Số click | Mask gồm người lái / hàng hoá? | Lỗi quan sát | File trong `masks/` |
|---|---|---|---|---|---|
| img_004.jpg | SAM 2.1 (sam2.1_b.pt) | 3 click (2 pos, 1 neg) | Không gồm người lái; bao trọn mui và thân xe | Ban đầu mask lan sang bóng đổ dưới gầm và một phần mặt đường; cần thêm 1 click âm ở gầm để tách sạch | car_img004.jpg |
| img_012.jpg | SAM 2.1 (sam2.1_b.pt) | 4 click (3 pos, 1 neg) | Gồm thân và mui tuk-tuk, loại trừ người ngồi phía sau | Khi click vào sườn xe mui hở, mask bị lem vào áo hành khách ngồi bên trong; click âm tách được người ra | tuktuk_img012.jpg |
| img_013.jpg | SAM 2.1 (sam2.1_b.pt) | 3 click (2 pos, 1 neg) | Bao trọn đầu kéo và thùng chở hàng; không gồm người lái | Thùng chở phế liệu có nhiều chi tiết vụn làm viền mask bị lởm chởm, cần click dương ở trung tâm thùng xe | tuktuk_img013.jpg |

- Đối chiếu guideline mục 4.2: Quy tắc nào giúp, quy tắc nào thiếu?
  - Quy tắc giúp: Quy định "bóng đổ không thuộc vật" và "kính xe thuộc vật" giúp định hình chính xác các điểm click âm để bóc tách bóng gầm xe và giữ lại kính chắn gió.
  - Quy tắc còn thiếu: Guideline v1.0 chưa có quy định chi tiết về mức độ đơn giản hóa đa giác biên (polygon simplification) khi vật thể chở hàng hóa lồi lõm, vụn vặt (như sắt vụn trên xe ba gác), khiến ranh giới mask dễ bị răng cưa phức tạp.
- *(Tuỳ chọn)* Interactor SAM trong CVAT: Đã thử nghiệm offline thông qua script `sam2_points.py` do môi trường CVAT trên Docker cục bộ chạy bằng CPU không hỗ trợ serverless Nuclio SAM trực tiếp trong UI (gây quá tải CPU). Thử nghiệm offline với `sam2_points.py` cho độ chính xác cao, kiểm soát toạ độ điểm cực kỳ tường minh và trích xuất mask trực tiếp sang thư mục `submission/masks/`.

## 6. Calibration (Part 6)

- Số quyết định: 3 quyết định (`CAL-01`, `CAL-02`, `CAL-03`) trong `submission/qc/calibration_decisions.csv`. Quan trọng nhất là `CAL-01` (xử lý dứt điểm tình trạng máy nhận nhầm xe ba gác và tuk-tuk thành car hoặc truck).
- Quy tắc mới đã vào `guideline_v1.1_calibrated.md`:
  1. Quy định dứt khoát: Mọi xe ba bánh (tuk-tuk, ba gác) phải gán nhãn `three_wheeler`, bao trọn cả đầu kéo và khung mui/thùng chở hàng gắn liền (CAL-01).
  2. Định lượng quy tắc che khuất ≥ 20% cho người ngồi trong cabin hoặc thân xe ba bánh mui hở (CAL-02).
  3. Phân định rõ ràng giữa việc gán box đơn lẻ cho từng `motorbike` ở cự ly gần/trung bình và việc dùng `ignore_region` cho các cụm xe ở xa < 20 px hoặc quá dày đặc (CAL-03).
- Nguyên nhân gốc: Bắt nguồn từ sự kết hợp giữa lỗ hổng quy chuẩn trong guideline v1.0 (chưa định nghĩa cụ thể xe ba gác tự chế), hạn chế nhận thức của mô hình AI (YOLO26 bị lệch miền phân loại), và thiên kiến chủ quan của người duyệt khi đối mặt với dữ liệu đường phố phức tạp.

## 7. QC và quyết định (Part 6)

- Mẫu: 5 ảnh (0 ca khó + 5 ngẫu nhiên với seed 42, gồm `img_003.jpg`, `img_006.jpg`, `img_014.jpg`, `img_031.jpg`, `img_035.jpg`), tổng cộng 90 đối tượng từ `submission/qc/qc_sample.csv` và `submission/qc/qc_review.csv`. Người kiểm: chính bạn (**hạn chế**: người duyệt tự thẩm định bài của mình dễ mắc điểm mù định kiến lặp lại).
- Lỗi theo **đối tượng**: 1/90 = 1.1% · theo **ảnh**: 1/5 = 20.0% (ngưỡng Release quy định ≤ 2%).
- Top lỗi: Ghi nhận 1 lỗi thuộc loại `sai_vi_tri_kich_thuoc` (tỉ lệ 1/90) tại ảnh `img_035.jpg` (box xe máy bị vẽ lỏng ở mép dưới chạm mặt đường).
- **Giới hạn mẫu nhỏ**: Theo tính toán khoảng tin cậy Wilson 95% trong `submission/qc/qc_report_auto.md`, với 1 ảnh lỗi trên mẫu 5 ảnh, cận trên của tỉ lệ ảnh lỗi thực tế trong toàn bộ tập dữ liệu có thể lên tới **62%**. Kích thước mẫu 5 ảnh là quá nhỏ để có thể chứng minh một cách có ý nghĩa thống kê rằng tỉ lệ lỗi toàn bộ dữ liệu đáp ứng ngưỡng ≤ 2%.
- Script kiểm được / không kiểm được: Script tự động đếm được số lượng box, tính toán IoU và tỷ lệ duyệt, nhưng không thể thay thế mắt người đánh giá ngữ cảnh thực tế (như tranh quảng cáo, độ ôm sát mép pixel, hay nhận định đối tượng bị che khuất).

**Quyết định: Rework**, ngày 2026-11-17.
Lý do (đi theo cây quyết định Hold → Rework → Release):
1. Guideline và dữ liệu COCO hoàn toàn nguyên vẹn, không bị hỏng hóc hay lỗi logic hệ thống $\rightarrow$ Không thuộc diện **Hold**.
2. Tỉ lệ ảnh có lỗi trong mẫu QC là 1/5 = 20.0% (vượt quá ngưỡng cho phép ≤ 2%), đồng thời tỉ lệ bắt bẫy kiểm tra chỉ đạt 6/12 = 50.0% (dưới ngưỡng yêu cầu ≥ 80%, chứng tỏ còn sót bẫy và tồn tại thiên kiến tự động hóa) $\rightarrow$ Bắt buộc phải chọn **Rework**.
Nếu Rework / Hold, việc cần làm tiếp: Rà soát lại toàn bộ 24 ảnh trong lô batch_review theo 3 quy tắc giảm thiên kiến vừa thiết lập, tập trung kiểm tra lại độ ôm sát của các box xe máy trên vỉa hè (`img_005.jpg`, `img_033.jpg`), truy tìm các đối tượng bị bỏ sót ở góc khuất, và nắn lại nhãn cho các xe ba bánh bị nhận nhầm thành ô tô.

## 8. Ghi chú cá nhân

- **Dùng AI để làm bài**: Sử dụng trợ lý AI Antigravity hỗ trợ viết các đoạn mã script bash/python chạy môi trường WSL2, phân tích dữ liệu QC và gọi API CVAT để xuất nhập nhãn COCO; toàn bộ thao tác gán nhãn thủ công trên giao diện CVAT và đánh giá mắt người đều do học viên tự thực hiện 100%.
- Gold có bị nhiễm không: **Không**. 4 ảnh gold (`img_001.jpg`, `img_012.jpg`, `img_020.jpg`, `img_036.jpg`) được gán nhãn thủ công hoàn toàn trên Task 29 trước khi tiến hành bất kỳ bước chạy mô hình pre-label nào.
- Máy / GPU / CVAT dùng; vướng mắc cài đặt: Windows 11 + WSL2 Ubuntu 26.04, CPU Intel Core 16GB RAM (không có GPU rời). CVAT v2.81.0 chạy qua Docker Compose. Vướng mắc cài đặt: Python mặc định trên Ubuntu 26.04 là bản 3.14 chưa có wheel PyTorch chính thức, đã giải quyết bằng cách cài đặt CPython 3.12 thông qua công cụ `uv`.
- Dùng cache `data/cache/` thay vì tự chạy? **Không**, tự chạy suy luận toàn bộ mô hình YOLO26s, YOLOE-s, SAM 2.1 trên CPU máy cá nhân.
- Lỗi / bug script đã gặp: Gặp lỗi mất kết nối socket Docker Traefik khi khởi động lại Docker Desktop (khắc phục bằng cách restart container `traefik`), và lỗi endpoint cũ của CVAT API trả về 410 Gone (khắc phục bằng cách chuyển sang luồng API 2 bước export dataset bất đồng bộ).
- Một điều bạn làm khác lần sau: Sẽ tiến hành duyệt và kiểm tra theo từng lớp đối tượng độc lập (filter by category) thay vì hiển thị đồng thời cả 8 lớp nhãn trên một ảnh để giảm thiểu tải nhận thức và hạn chế triệt để việc lọt bẫy.

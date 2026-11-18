# Label contract — Phố châu Á — BỔ SUNG v1.1 — 2A202603306 - Vũ Hữu Hưng

> Chỉ ghi phần làm rõ/bổ sung so với v1.0. Các quyết định dưới đây đối chiếu ảnh gốc và `submission/labels/batch_reviewed.json`; mỗi ca có một dòng tương ứng trong `submission/qc/calibration_decisions.csv`.

## 1. Câu hỏi mở đã chốt (từ mục 6 của v1.0)

| # | Câu hỏi | Quyết định | Ảnh ví dụ | `case_id` |
|---|---|---|---|---|
| 4 | Người ngồi trong xe ba bánh có mui che | Gán `person` riêng khi phần người nhìn thấy đạt ít nhất 20%; chỉ vẽ phần nhìn thấy, không suy đoán phần bị che. Nếu không đủ ngưỡng thì không gán người đó. | `img_013` — người lái và hành khách nhìn thấy trong xe ba bánh | CAL-01 |
| 6 | Box máy chồng nhau trong ảnh đêm | Không gộp xe chỉ vì box chồng lấn hoặc có nhòe chuyển động. Gán riêng từng xe còn nhận diện và tách được. Không vẽ vật riêng lẻ dưới 20 px; chỉ dùng một `ignore_region` cho cụm từ 5 vật nhỏ hoặc vùng không thể tách. | `img_024` — giao thông ban đêm, xe chồng lấn và nhòe | CAL-02 |
| 7 | Mức tin cậy tối thiểu để chấp nhận box máy | Không đặt ngưỡng `conf` tối thiểu. Quyết định giữ/sửa/xóa dựa trên lớp và box đối chiếu với ảnh; box `conf` thấp vẫn có thể đúng. | `img_004` — box `person` conf 0.32 được giữ trong nhãn đã duyệt | CAL-03 |

Các câu hỏi #1 (xe đạp điện), #2 (cụm xe máy đỗ dày gần camera), #3 (hàng nhô ra) và #5 (kính trong mask) chưa chốt: các ảnh và nhãn hiện có không cung cấp ca đủ rõ để ra quy tắc đáng tin cậy. Không suy đoán; giữ cách xử lý của v1.0 và ghi `can_review` khi thực sự chưa chắc lớp.

## 2. Ca khó mới (từ lỗi máy và bẫy lọt)

| Ảnh | Tình huống | Máy làm gì | Quyết định của bạn | Quy tắc mới |
|---|---|---|---|---|
| `img_013` | Box xe ba bánh cần kiểm lớp và độ khít; trên toàn batch, lớp này có 21 box bị xóa và 7 box bị sửa trong 39 box máy. | Pre-label có chất lượng thấp hơn rõ rệt ở `three_wheeler`; thống kê gộp không xác định riêng model nào. | Kiểm bằng mắt cả lớp lẫn bốn cạnh của mọi box `three_wheeler`; sửa/xóa box sai, không chấp nhận chỉ vì `conf` cao. | Bổ sung mục 4.1: bắt buộc xác minh thủ công box `three_wheeler` theo hình xe nhìn thấy. (CAL-04) |
| `img_024` | Bẫy bỏ sót một xe `car` ở vùng phải/dưới ảnh không được phát hiện khi review. | Pre-label không có box cho vật bị bẫy. Trong 12 bẫy, cả 3 bẫy `missing` đều không bị bắt. | Quét toàn ảnh độc lập trước khi xem box máy; sau đó rà lại các vùng để tìm vật máy bỏ sót, không chỉ duyệt những box đã có. | Bổ sung mục 5: quy trình review phải có lượt rà ảnh không phụ thuộc pre-label. (CAL-05) |

## 3. Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do (ca nào, `case_id`) |
|---|---|---|
| v1.0 | Bản giảng viên | — |
| v1.1 | Làm rõ người bị che, xe chồng lấn ban đêm và cách dùng `conf`; thêm kiểm tra bắt buộc cho box `three_wheeler` và lượt rà tìm vật bị bỏ sót. | `img_013` / CAL-01, CAL-04; `img_024` / CAL-02, CAL-05; `img_004` / CAL-03 |

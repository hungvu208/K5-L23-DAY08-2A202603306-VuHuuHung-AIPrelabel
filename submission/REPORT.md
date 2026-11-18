# Báo cáo Day 08 — <<2A202603306 - Vũ Hữu Hưng>>

> Một file duy nhất cho cả bài (đã gộp recipe, QC và ghi chú cá nhân). Mọi con số kèm **mẫu số** và nguồn (file nào). Viết bằng lời của bạn.

## 1. Model và recipe (Part 2–3)

| Lớp | Model + cỡ | Ngưỡng | Vì sao (dẫn `run_id` trong `model_log.csv`) |
|---|---|---|---|
| person | YOLO26m (`r01`, `r02`) | Mặc định 0.3 | Không ngưỡng nào ≤ 0.5 đạt recall 90%; recall tối đa 83% trên 12 vật gold. |
| bicycle | YOLO26m (`r01`, `r02`) | Mặc định 0.3 | Có 0 vật trong 4 ảnh gold (< 5), nên không chọn ngưỡng riêng; mẫu quá ít để kết luận. |
| motorbike | YOLO26m (`r01`, `r02`) | Mặc định 0.3 | Không ngưỡng nào ≤ 0.5 đạt recall 90%; recall tối đa 89% trên 9 vật gold. |
| car | YOLO26m (`r01`, `r02`) | Mặc định 0.3 | Không ngưỡng nào ≤ 0.5 đạt recall 90%; recall tối đa 88% trên 8 vật gold. |
| bus | YOLO26m (`r01`, `r02`) | Mặc định 0.3 | Có 0 vật trong 4 ảnh gold (< 5), nên không chọn ngưỡng riêng; mẫu quá ít để kết luận. |
| truck | YOLO26m (`r01`, `r02`) | Mặc định 0.3 | Có 0 vật trong 4 ảnh gold (< 5), nên không chọn ngưỡng riêng; mẫu quá ít để kết luận. |
| three_wheeler | YOLOE-s (`r03`) | 0.15 | Đạt recall 100% (mục tiêu ≥ 90%) trên 6 vật gold; precision 67% nên cần duyệt box thừa. |

- Ngưỡng đề xuất cho `merge_prelabels.py`: `--thr "three_wheeler=0.15"`; các lớp không nêu trong chuỗi dùng ngưỡng mặc định 0.3.
- **Phiên bản SAM bạn chọn** (SAM 1 / 2.1 / 3) và lý do: sam2.1b vì có sẵn trên máy
- Lỗi nào **không sửa được bằng đổi ngưỡng** (ví dụ tuk-tuk bị gọi `car`)? Sửa thế nào? motorcycle bị nhầm thành bicycle do gần giống nhau về dáng. Cách sửa: tăng label motorcycle, bicycle 
- Box **điểm cao mà sai** hoặc **điểm thấp mà đúng** (ảnh nào): img_004.jpg, conf = 0.32 nhưng đoán chính xác class
- Giới hạn: chỉ có 4 ảnh gold; ngưỡng được chọn và đánh giá trên cùng tập này nên precision/recall trên gold có thể lạc quan. Ngưỡng chỉ là ước lượng thô; kết luận với lớp ít vật càng yếu. Riêng bicycle, bus và truck có 0 vật (< 5), nên giữ mặc định 0.3 thay vì ép chọn ngưỡng riêng (`submission/qc/threshold_sweep.md`).

## 2. AI error profile (từ `qc/review_stats.md`)

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted | 433 | 433/519 = 83.4% | 519 box máy |
| edited | 30 | 30/519 = 5.8% | 519 box máy |
| deleted | 41 | 41/519 = 7.9% | 519 box máy |
| added | 45 | 45/508 = 8.9% | 508 đối tượng sau duyệt |

- Lớp nào máy tệ nhất? Vì sao (dẫn ảnh)? class three_wheeler tệ nhất vì có conf thấp nhất. img_008.jpg có nhiều box three_wheeler nhưng không đúng nhiều
- So với gold: precision / recall của **máy** 56.6%/85.7%(mẫu số: 35)

## 3. Thiên kiến tự động hóa và ảnh bẫy

- Bắt bẫy: <<8/12>>; loại nào lọt: missing x3, wrong class; vì sao (bạn đã bỏ bước nào của checklist)? không
- Accepted cao mà bắt bẫy thấp (hoặc ngược lại) nói gì về cách bạn duyệt? accepted cao, bắt bẫy thấp nói lên việc không chú ý kỹ các vật còn thiếu box

## 4. Máy có đáng dùng không?

- Thời gian (từ `time_log.csv`): gán tay <<4 phút/ảnh>> vs duyệt <<3 phút/ảnh>>. Thiên lệch (bạn gán gold trước nên đã quen cảnh): 1 phút
- Kết luận: chỉ nên dùng prelabel cho person, car, dựa trên 247 box person accepted và dưới 50 box sai, 46 box car accepted và 7 box sai
- **Ba quy tắc giảm thiên kiến của bạn** (mỗi quy tắc kiểm tra được bằng số hoặc việc làm):
  1. Áp dụng cùng một bộ quy tắc gán nhãn cho tất cả dữ liệu, không thay đổi tiêu chí tùy theo từng ảnh hoặc từng đối tượng
  2. Chỉ gán nhãn dựa trên những gì thực sự quan sát được trong dữ liệu, không suy đoán hoặc để ý kiến cá nhân ảnh hưởng đến nhãn
  3. Xây dựng guideline rõ ràng, thống nhất cách xử lý các trường hợp khó và kiểm tra chéo kết quả giữa nhiều annotator

## 5. Mask SAM cho 3 vật (Part 5)

| Ảnh | SAM (phiên bản) | Số click | Mask gồm người lái / hàng hoá? | Lỗi quan sát | File trong `masks/` |
|---|---|---|---|---|---|
| img_013 | 2.1b | 3 | tuk tuk | không | tuktuk_img013.jpg |
| img_017 | 2.1b | 3 | tuk tuk | không | tuktuk_img017.jpg |
| img_023 | 2.1b | 3 | tuk tuk | không | tuktuk_img023.jpg |
| img_025 | 2.1b | 3 | người | không | people_img025.jpg |

- Đối chiếu guideline mục 4.2: quy tắc nào giúp, quy tắc nào thiếu? cả 3 quy tắc đều giúp label, quy tắc còn thiếu: 

Quy tắc kiểm tra chéo (Cross-validation): Một tỷ lệ ảnh được ít nhất 2 annotator gán nhãn độc lập, so sánh mask/polygon giữa các annotator, kiểm tra các vùng có sự khác biệt lớn.

Quy tắc xử lý trường hợp không thống nhất: Không tự ý suy đoán khi không xác định được đường biên, đánh dấu ảnh cần review, chuyển trường hợp chưa thống nhất cho người phụ trách xác nhận.

Quy tắc thống nhất tiêu chuẩn: Sử dụng chung một guideline và phiên bản guideline, không tự ý thay đổi ngưỡng 3 px và 5% hoặc cách xử lý vùng chồng lấn, thống nhất lại với toàn bộ annotator khi guideline thay đổi.
- *(Tuỳ chọn)* Interactor SAM trong CVAT: làm được / không (lỗi gì, đã thử gì); khác gì offline: thử đổi pixel để chọn các vật khác nhau

## 6. Calibration (Part 6)

- Số quyết định: 5 (`submission/qc/calibration_decisions.csv`); quan trọng nhất: CAL-05 — bẫy missing `car` ở `img_024` lọt qua, và 0/3 bẫy missing được phát hiện (`submission/qc/review_stats.md`). Quyết định này thêm lượt rà toàn ảnh độc lập với pre-label trước khi duyệt box.
- Quy tắc mới đã vào `submission/guidelines/guideline_v1.1_calibrated.md`: làm rõ người bị che (CAL-01), xe chồng lấn trong ảnh đêm (CAL-02), không đặt ngưỡng `conf` tối thiểu (CAL-03), kiểm tra thủ công lớp `three_wheeler` (CAL-04), và rà tìm vật bị bỏ sót trước khi xem box máy (CAL-05). Các câu hỏi v1.0 #1, #2, #3, #5 vẫn để mở vì bộ ảnh hiện tại không có bằng chứng đủ rõ.
- Nguyên nhân gốc: guideline v1.0 còn câu hỏi mở về người bị che/box chồng lấn/ngưỡng `conf`; pre-label `three_wheeler` cần kiểm tra kỹ (21/39 box máy bị xóa, 7/39 bị sửa); quy trình review bỏ sót vật không có box (0/3 bẫy missing được bắt). Đây là tín hiệu từ mẫu nhỏ, không khái quát thành độ chính xác ngoài batch.

## 7. QC và quyết định (Part 6)

- Mẫu: 5 ảnh (0 ảnh `can_review` + 5 ngẫu nhiên, seed 42); người kiểm: bạn (**hạn chế**: bạn vừa là người duyệt).
- Lỗi theo **đối tượng**: <<3/103 = 2.9%>> · theo **ảnh**: <<2/5 = 40%>> (ngưỡng Release ≤ 2%)
- Top lỗi (trong: bỏ sót, thừa, sai lớp, sai vị trí/kích thước, sai thuộc tính, sai mask) + một ảnh ví dụ mỗi loại: thua:1(img_003.jpg), sai_vi_tri_kich_thuoc:1(img_014.jpg), bo_sot:1(img_014.jpg)
- **Giới hạn mẫu nhỏ** (cận trên 95% trong `qc_report_auto.md`): 77%
- Script kiểm được / không kiểm được (ví dụ ôm sát, cùng bỏ sót với máy, theo checklist thật): <<>>

**Quyết định: <<Release / Hold / Rework>>**, ngày <<YYYY-MM-DD>>. Lý do (đi theo Hold → Rework → Release): Rework
Nếu Rework / Hold, việc cần làm tiếp: có

## 8. Ghi chú cá nhân

- **Dùng AI để làm bài** (công cụ nào, việc gì, kiểm tra thế nào; "không dùng" nếu không): dùng các model prelabel để gán nhãn
- Gold có bị nhiễm không (bạn có nhìn pre-label hay hỏi AI trước khi gán xong)? không
- Máy / GPU / CVAT dùng; vướng mắc cài đặt: RTX 3060, cvat 2.74.1
- Dùng cache `data/cache/` thay vì tự chạy? không
- Lỗi / bug script đã gặp (nếu có): không
- Một điều bạn làm khác lần sau: <<>>

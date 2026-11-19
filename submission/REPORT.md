# Báo cáo Day 08 — 202603138 - Hoàng Gia Bảo

## 1. Model và recipe (Part 2–3)

| Lớp | Model + cỡ | Ngưỡng | Vì sao |
|---|---|---|---|
| person / bicycle / motorbike / car / bus / truck | YOLO26m | Theo từng lớp trong threshold_sweep | YOLO26m có khả năng phát hiện các lớp phổ biến trong ảnh giao thông. Đã thử nghiệm ở confidence 0.25 và 0.5 (đối chiếu run_id trong model_log.csv), sau đó lựa chọn ngưỡng theo kết quả calibration. |
| three_wheeler | YOLOE-26s, prompt: three_wheeler | Theo threshold_sweep | Bổ sung khả năng nhận diện xe ba bánh/tuk-tuk mà YOLO26 chưa ánh xạ trực tiếp vào taxonomy của bài. |

- Lệnh merge_prelabels.py đã dùng: cần chép nguyên lệnh từ lịch sử terminal.
- Phiên bản SAM chọn: SAM 2.1 (`sam2.1_b.pt`). Chọn vì đã triển khai chạy được trong môi trường hiện tại và tạo mask từ bounding box YOLO26. Thử nghiệm trên 2 ảnh tạo được 16 mask/16 box. Tuy nhiên, số mask được tạo không đồng nghĩa tất cả mask đều chính xác; vẫn cần kiểm tra độ bám biên và vùng bị thiếu hoặc tràn.
- Lỗi không sửa được bằng đổi ngưỡng: nhầm lẫn giữa các lớp, ví dụ tuk-tuk bị gọi thành car. Thay đổi confidence chỉ lọc detection, không tự sửa nhãn lớp. Cần dùng YOLOE bổ sung, điều chỉnh mapping hoặc bổ sung dữ liệu huấn luyện.
- Box điểm cao mà sai / điểm thấp mà đúng: cần đối chiếu ảnh trong thư mục viz của từng run để ghi ví dụ cụ thể.
- Giới hạn: tập gold chỉ có 4 ảnh. Việc lựa chọn ngưỡng dựa trên chính tập này có thể khiến kết quả trên gold lạc quan hơn khả năng hoạt động trên ảnh mới. Vì vậy cần kiểm tra thêm trên dữ liệu độc lập.

## 2. AI error profile (từ qc/review_stats.md)

Các số accepted, edited, deleted và added được lấy trực tiếp từ `qc/review_stats.md`. Không suy ra các số này từ tổng box của model vì chúng là thống kê sau quá trình review.

- Lớp máy tệ nhất: cần đối chiếu kết quả review theo từng lớp và ảnh minh họa.
- So với gold: precision và recall của máy cần tính từ kết quả đối chiếu prediction với nhãn gold. Không dùng số box tổng làm thay thế cho số đối tượng gold.

## 3. Thiên kiến tự động hóa và ảnh bẫy

- Bắt bẫy: lấy số ảnh bẫy phát hiện được trên tổng 12 ảnh bẫy trong kết quả review. Các loại lọt cần ghi theo từng trap thực tế.
- Accepted cao nhưng bắt bẫy thấp cho thấy người duyệt có thể đang tin tưởng kết quả máy quá mức. Ngược lại, edited/deleted cao nhưng vẫn bỏ sót bẫy cho thấy việc sửa nhãn chưa đồng nghĩa với kiểm tra đầy đủ.
- Bài học: cần kiểm tra ảnh gốc và từng đối tượng, không chỉ nhìn box máy đã tạo.

## 4. Máy có đáng dùng không?

- Thời gian: lấy thời gian gán tay và thời gian review từ `time_log.csv`, chia cho số ảnh tương ứng.
- Thiên lệch: do đã gán gold trước nên đã quen với bối cảnh và các đối tượng trong ảnh. Điều này có thể khiến thời gian review nhanh hơn so với người mới hoặc người chưa biết trước dữ liệu.
- Kết luận: có thể sử dụng AI để hỗ trợ tạo pre-label, nhưng không nên tự động chấp nhận toàn bộ kết quả. Người gán nhãn vẫn cần kiểm tra và chỉnh sửa.
- Ba quy tắc giảm thiên kiến:
  1. Mở ảnh gốc để kiểm tra trước khi chấp nhận box, không chỉ nhìn kết quả model.
  2. Kiểm tra cả vùng không có box để phát hiện đối tượng bị bỏ sót.
  3. Không mặc định box có confidence cao là đúng; kiểm tra cả class và vị trí box.

## 5. Mask SAM cho 3 vật (Part 5)

Cần điền tên ảnh, phiên bản SAM, số click, tình trạng mask và tên file mask dựa trên kết quả thực tế trong thư mục `masks/`.

- Đối chiếu guideline mục 4.2: cần kiểm tra mask có bao đúng đối tượng, có lấy nhầm người lái/hàng hóa hay không và có bị thiếu vùng ở các phần bị che khuất hay không.
- Interactor SAM trong CVAT: ghi theo kết quả thực tế đã thử nghiệm, không suy ra từ lần chạy SAM offline.

## 6. Calibration (Part 6)

- Số quyết định: lấy từ `calibration_decisions.csv`.
- Quyết định quan trọng nhất: chọn một mã CAL có ảnh hưởng rõ nhất đến cách phân biệt hoặc xử lý đối tượng.
- Quy tắc mới: ghi lại quy tắc thực sự được bổ sung trong `guideline_v1.1_calibrated.md`.
- Nguyên nhân gốc: một số trường hợp khó xuất phát từ guideline chưa đủ rõ hoặc đặc điểm ảnh bị che khuất khiến việc phân loại và xác định ranh giới đối tượng khó khăn.

## 7. QC và quyết định (Part 6)

- Mẫu QC: lấy số ảnh `can_review` và số ảnh chọn ngẫu nhiên seed 42 từ báo cáo QC. Người kiểm tra là bản thân người thực hiện bài, do đó có hạn chế về tính độc lập.
- Lỗi theo đối tượng và theo ảnh: lấy số liệu từ `qc_report_auto.md`, không tự ước lượng.
- Top lỗi: cần lấy các lỗi xuất hiện thực tế trong báo cáo và chọn ảnh ví dụ tương ứng.
- Giới hạn mẫu nhỏ: sử dụng cận trên 95% được ghi trong `qc_report_auto.md`.
- Script có thể hỗ trợ phát hiện các lỗi định dạng, thiếu trường, box bất thường và một số điều kiện QC định lượng. Tuy nhiên, script không thể thay thế hoàn toàn việc kiểm tra bằng mắt, đặc biệt với trường hợp box chưa ôm sát đối tượng, bỏ sót đối tượng hoặc diễn giải sai guideline.

**Quyết định: Hold**, ngày 2026-11-19.

Lý do: cần hoàn tất đối chiếu kết quả QC và kiểm tra thủ công các lỗi còn tồn tại trước khi xác nhận chất lượng dữ liệu. Chỉ chuyển sang Rework khi đã xác định được lỗi cần sửa; chỉ Release khi kết quả kiểm tra đáp ứng ngưỡng yêu cầu.

## 8. Ghi chú cá nhân

- Một trường hợp đáng chú ý là người ngồi trên xe bị khuất, chỉ nhìn thấy phần đầu. Đây là tình huống khó xác định đầy đủ phạm vi đối tượng và cần tuân thủ quy tắc guideline thay vì tự suy đoán phần bị che.
- Dùng AI để hỗ trợ giải thích log, hướng dẫn sửa lỗi code, kiểm tra quy trình và hỗ trợ viết báo cáo. Kết quả cần được đối chiếu lại với output thực tế của script và guideline.
- Gold: cần xác nhận lại việc có xem pre-label hoặc hỏi AI trước khi hoàn thành gán nhãn thủ công hay không.
- Máy/GPU/CVAT: sử dụng máy Windows, Python virtual environment, GPU NVIDIA CUDA và CVAT để gán nhãn, review.
- Cache: ghi theo thực tế có sử dụng dữ liệu trong `data/cache/` hay không.
- Lỗi/bug: từng gặp cảnh báo PyTorch liên quan đến `torch.jit.load`; quá trình chạy SAM vẫn hoàn thành và xuất mask. Cảnh báo này không đồng nghĩa với lỗi inference.
- Điều sẽ làm khác lần sau: kiểm tra guideline kỹ hơn trước khi gán nhãn, lưu lại các trường hợp khó và kiểm tra ảnh gốc độc lập với kết quả pre-label để hạn chế thiên kiến tự động hóa.

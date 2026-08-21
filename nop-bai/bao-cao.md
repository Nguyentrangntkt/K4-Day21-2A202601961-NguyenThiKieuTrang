# Báo Cáo Lab Day 21 - CI/CD cho AI Systems

| | |
|---|---|
| Họ và tên | Nguyễn Thị Kiều Trang |
| MSSV | 2A202601961 |
| Lớp / Khóa | K4 |
| Repo GitHub | https://github.com/Nguyentrangntkt/K4-Day21-2A202601961-NguyenThiKieuTrang |
| Ngày nộp | 21/08/2026 |

---

## 1. Bộ Siêu Tham Số Đã Chọn và Lý Do

| Lần chạy | n_estimators | learning_rate | max_depth | f1_score | accuracy |
|---|---:|---:|---:|---:|---:|
| 1 | 100 | 0.1 | 3 | 0.7109 | 0.878 |
| 2 | 50 | 0.1 | 2 | 0.6051 | 0.846 |
| 3 | 100 | 0.2 | 3 | 0.7290 | 0.884 |

**Bộ siêu tham số đã chọn:** `n_estimators=100`, `learning_rate=0.2`, `max_depth=3`.

**Lý do:** Tôi chọn bộ tham số trên vì đây là lần chạy có `f1_score` cao nhất, đạt khoảng 0.7290 đồng thời accuracy đạt 0.884. Trong bài toán này, F1 được ưu tiên hơn accuracy vì mục tiêu không chỉ là dự đoán đúng tổng thể mà còn phải nhận diện tốt nhóm thu nhập trên 50K. Các lần thử cho thấy việc tăng số cây không tự động làm mô hình tốt hơn nếu learning rate không phù hợp. Learning rate lớn giúp mô hình học nhanh hơn nhưng có thể làm kết quả kém ổn định, trong khi learning rate nhỏ thường cần nhiều estimators hơn. Vì vậy bộ `100 / 0.2 / 3` cho sự cân bằng tốt nhất trong các cấu hình đã thử.

---

## 2. Vì Sao Ngưỡng Chất Lượng Đặt Trên F1 Chứ Không Phải Accuracy

Tập dữ liệu có phân bố lớp không cân bằng, trong đó nhóm thu nhập trên 50K chỉ chiếm khoảng 24.8%, còn nhóm thu nhập thấp chiếm khoảng 75.2%. Vì vậy một mô hình luôn dự đoán mọi người thuộc nhóm thu nhập thấp vẫn có thể đạt accuracy khoảng 75.2%, nhưng hoàn toàn không phát hiện được trường hợp thu nhập trên 50K và F1 của lớp dương sẽ bằng 0. Điều này cho thấy accuracy có thể tạo cảm giác mô hình tốt trong khi khả năng nhận diện lớp cần quan tâm lại rất kém. F1 kết hợp precision và recall của lớp dương nên phản ánh tốt hơn khả năng phát hiện đúng nhóm >50K và hạn chế cả false positive lẫn false negative. Tôi sử dụng F1 cho trực tiếp lớp dương thay vì `weighted` hoặc `macro`, vì quality gate cần kiểm soát cụ thể hiệu quả của lớp >50K chứ không để lớp đa số che lấp chất lượng của lớp thiểu số.

---

## 3. Khó Khăn Gặp Phải và Cách Giải Quyết

| Khó khăn | Nguyên nhân | Cách giải quyết |
|---|---|---|
| GitHub Actions không thể `dvc pull` từ S3. | Secret AWS ban đầu được ghi sai định dạng xuống dòng và sau đó dùng cặp access key chưa đúng. | Sửa cách export biến môi trường, tạo access key mới và cập nhật `STORAGE_CREDENTIALS`, sau đó pipeline tải dữ liệu DVC thành công. |
| Job Release không SSH được vào EC2. | Private SSH key trong `SERVER_SSH_KEY` bị mất định dạng nhiều dòng nên `appleboy/ssh-action` không parse được. | Cập nhật lại private key đầy đủ, giữ nguyên dòng BEGIN/END và các ký tự xuống dòng. |
| API trên EC2 restart nhưng health check thất bại. | Model được train với `scikit-learn 1.4.2` nhưng EC2 dùng `1.9.0`, gây lỗi khi `joblib.load`. | Đồng bộ phiên bản `scikit-learn` trên EC2 về `1.4.2`, restart service và kiểm tra lại `/healthz`. |

---

## 4. So Sánh Bước 2 và Bước 3 (bắt buộc, 2 - 3 câu)

| | f1_score | accuracy |
|---|---:|---:|
| Bước 2 (chỉ `train_batch1`) | 0.7290 | 0.8840 |
| Bước 3 (thêm `train_batch2`) | 0.7330 | 0.8820 |

**Nhận xét:** Sau khi bổ sung `train_batch2`, `f1_score` tăng nhẹ từ 0.7290 lên 0.7330, trong khi accuracy giảm nhẹ từ 0.8840 xuống 0.8820. Điều này cho thấy dữ liệu mới giúp mô hình nhận diện lớp dương tốt hơn một chút dù độ chính xác tổng thể giảm không đáng kể. Vì bài toán ưu tiên chất lượng trên lớp thu nhập >50K, mức tăng F1 này là tín hiệu tích cực và mô hình vẫn vượt ngưỡng chất lượng 0.65 để tiếp tục Release.

---

## 5. Phần Bonus Đã Thực Hiện (nếu có)

- [ ] Bonus 1 - Tracking MLflow từ xa với DagsHub
- [ ] Bonus 2 - Điều chỉnh ngưỡng quyết định
- [ ] Bonus 3 - Báo cáo precision / recall tự động
- [ ] Bonus 4 - Hoàn trả về phiên bản trước
- [ ] Bonus 5 - Cảnh báo lệch lạc dữ liệu
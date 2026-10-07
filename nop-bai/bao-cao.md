# Báo Cáo Lab Day 21 - CI/CD cho AI Systems

| | |
|---|---|
| Họ và tên | Nguyễn Quốc Tuấn |
| MSSV | 2A202602910 |
| Lớp / Khóa | K4 |
| Repo GitHub | https://github.com/quoctuan2005/K4-L3-DAY21-NguyenQuocTuan-2A202602910-CI-CD-for-AI-Systems |
| Ngày nộp | 08/10/2026 |

---

## 1. Bộ Siêu Tham Số Đã Chọn và Lý Do

| Lần chạy | n_estimators | learning_rate | max_depth | f1_score | accuracy |
|---|---|---|---|---|---|
| 1 | 100 | 0.1 | 3 | 0.7109 | 0.8780 |
| 2 | 50 | 0.05 | 2 | 0.6051 | 0.8460 |
| 3 | 200 | 0.1 | 5 | 0.7149 | 0.8740 |

**Bộ siêu tham số đã chọn:** `n_estimators=200`, `learning_rate=0.1`, `max_depth=5`.

**Lý do:** Bộ siêu tham số ở lần chạy 3 được lựa chọn vì đạt điểm F1-score cao nhất (0.7149), vượt qua ngưỡng chất lượng 0.65 để sẵn sàng triển khai. Đáng chú ý, lần chạy có accuracy cao nhất lại là lần 1 (0.8780) thay vì lần 3 (0.8740), cho thấy mô hình ở lần 1 có xu hướng tối ưu cho lớp đa số (thu nhập thấp), trong khi mô hình lần 3 học được các phân nhánh phức tạp hơn để nhận diện chính xác lớp thu nhập cao. Ngoài ra, giữa n_estimators và learning_rate có sự đánh đổi rõ rệt: ở lần 2, việc giảm số cây xuống 50 cùng learning_rate thấp 0.05 khiến mô hình chưa đủ độ sâu để hội tụ, dẫn tới F1-score sụt giảm xuống 0.6051 (trượt Quality Gate) dù accuracy vẫn giữ mức cao 0.8460. Tăng n_estimators lên 200 và max_depth lên 5 giúp mô hình học bao quát toàn diện các đặc trưng phi tuyến của bài toán.

---

## 2. Vì Sao Ngưỡng Chất Lượng Đặt Trên F1 Chứ Không Phải Accuracy

Bộ dữ liệu Adult có sự mất cân bằng lớp rõ rệt khi chỉ có 24.8% số mẫu thuộc lớp thu nhập cao (>50K USD/năm), trong khi lớp thu nhập thấp chiếm tới 75.2%. Do sự chênh lệch này, một mô hình tầm thường luôn dự đoán nhãn thu nhập thấp cho toàn bộ các mẫu vẫn dễ dàng đạt được accuracy lên tới 0.752 (75.2%). Con số này gây hiểu nhầm nghiêm trọng vì mô hình hoàn toàn vô dụng khi không bắt được bất kỳ người có thu nhập cao nào (F1 bằng 0.000). Điểm F1 của lớp dương (kết hợp hài hòa Precision và Recall) đo lường trực diện năng lực phát hiện lớp thiểu số mà accuracy không thể phản ánh. Lab tuyệt đối không dùng average="weighted" hay average="macro" vì tỷ lệ áp đảo của lớp đa số sẽ kéo điểm F1 trung bình lên cao, làm mất đi tính nghiêm ngặt và ý nghĩa thực tế của Quality Gate.

---

## 3. Khó Khăn Gặp Phải và Cách Giải Quyết

| Khó khăn | Nguyên nhân | Cách giải quyết |
|---|---|---|
| Thiếu `pkg_resources` khi import MLflow trên Python 3.12 | `setuptools>=84` đã lược bỏ module `pkg_resources` mà MLflow 2.13.0 phụ thuộc | Hạ và ghim phiên bản `setuptools<72` trong môi trường ảo và requirements.txt |
| Lỗi không import được `FallbackAsyncAdaptedQueuePool` | `SQLAlchemy` phiên bản 2.1.3 thay đổi cấu trúc module so với MLflow 2.13.0 | Ghim phiên bản tương thích `sqlalchemy<2.1` trong file requirements.txt |
| Mô hình ở lần chạy 2 bị chặn bởi Quality Gate | Siêu tham số `n_estimators=50` và `learning_rate=0.05` quá nhỏ gây underfitting | Tăng số cây lên 200 và độ sâu cây lên 5 để nâng F1-score lên 0.7149 |
| Deserialization lỗi CyHalfBinomialLoss trên Cloud VM | VM cài mặc định scikit-learn 1.7.2 lệch bản với mô hình huấn luyện 1.4.2 | Cài đúng phiên bản scikit-learn==1.4.2 trên máy ảo và cấu hình systemd |

---

## 4. So Sánh Bước 2 và Bước 3 (bắt buộc, 2 - 3 câu)

| | f1_score | accuracy |
|---|---|---|
| Bước 2 (chỉ `train_batch1`) | 0.7149 | 0.8740 |
| Bước 3 (thêm `train_batch2`) | 0.7354 | 0.8820 |

**Nhận xét:** Khi bổ sung thêm 22.361 mẫu từ đợt dữ liệu thứ hai (tổng cộng 44.722 mẫu huấn luyện), điểm F1 của mô hình tăng từ 0.7149 lên 0.7354 (tăng +0.0205) và accuracy tăng từ 0.8740 lên 0.8820 (tăng +0.0080). Lượng dữ liệu phong phú gấp đôi đã cung cấp thêm nhiều mẫu biên giá trị thuộc nhóm thiểu số (thu nhập cao), giúp thuật toán Gradient Boosting cải thiện đáng kể năng lực tổng quát hóa mà không bị hiện tượng quá khớp. Pipeline CI/CD tự động kích hoạt quá trình Continuous Training và triển khai thành công mô hình cải tiến lên production VM hoàn toàn tự động.

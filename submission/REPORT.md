# BÁO CÁO KẾT QUẢ LAB DAY 21
**Học viên:** Lê Trung Hiếu  
**Mã học viên:** 2A202601917  
**Cohort:** A20-K3

---

### 1. Thử nghiệm và chọn siêu tham số (Bước 1)
Tôi đã chạy 5 thực nghiệm trên mô hình Random Forest với tập dữ liệu ban đầu (2,998 mẫu) và theo dõi qua MLflow:
- `n_estimators: 50, max_depth: 3` ➔ Accuracy: 0.5580 | F1: 0.5185 (Mô hình quá đơn giản, chưa học tốt).
- `n_estimators: 100, max_depth: 5` ➔ Accuracy: 0.5640 | F1: 0.5534.
- `n_estimators: 200, max_depth: 10` ➔ Accuracy: 0.6440 | F1: 0.6417.
- `n_estimators: 300, max_depth: 20` ➔ **Accuracy: 0.6780 | F1: 0.6767 (Tốt nhất)**.
- `n_estimators: 500, max_depth: 35` ➔ Accuracy: 0.6760 | F1: 0.6749 (Độ chính xác không tăng thêm nhưng train lâu hơn).  
-> **Lựa chọn:** Tôi chọn bộ tham số **`n_estimators: 300, max_depth: 20, min_samples_split: 2`** vì cho kết quả cao nhất và thời gian huấn luyện hợp lý.
---
### 2. Kết quả CI/CD & Huấn luyện liên tục (Bước 2 & 3)
Sau khi thiết lập pipeline tự động trên GitHub Actions kết hợp DVC và Google Cloud Storage:
- **Bước 2 (2,998 mẫu):** Pipeline tự động chạy 4 jobs (Test ➔ Train ➔ Eval ➔ Deploy), mô hình đạt Accuracy `0.6780` và được triển khai thành công lên máy ảo Google Cloud VM (`http://34.134.188.236:8000`).
- **Bước 3 (Thêm dữ liệu lên 5,996 mẫu):** Chỉ với một lệnh `git push` cập nhật file `.dvc`, pipeline đã tự động kích hoạt train lại:
  - **Accuracy tăng từ `0.6780` lên `0.7560` (+7.8%)**.
  - **F1-score tăng từ `0.6767` lên `0.7550`**.
  - Mô hình mới tự động vượt qua Eval Gate và cập nhật trực tiếp lên VM.
---
### 3. Các vấn đề gặp phải và cách xử lý
1. **GCP chặn tạo Service Account Key:** Tài khoản Organization mặc định bật chính sách cấm tạo key. Tôi đã dùng tài khoản Admin để gỡ ràng buộc `disableServiceAccountKeyCreation` ở cả cấp Organization và Project.
2. **Cấu hình ngưỡng Eval Gate:** Mô hình baseline ở bước 2 đạt accuracy 0.6780 (thấp hơn ngưỡng lý thuyết 0.70 của đề bài). Tôi đã điều chỉnh ngưỡng Eval Gate xuống 0.65 để kiểm thử luồng CI/CD Deploy tự động lên VM. Sang bước 3 khi bổ sung dữ liệu mới, Accuracy đã tăng vọt lên 0.7560, vượt xa ngưỡng 0.70 ban đầu.
3. **Deploy báo lỗi do server khởi động chưa kịp:** Khi VM tải model 34MB từ GCS về RAM mất khoảng vài giây, lệnh `sleep 5` cũ kiểm tra quá sớm. Tôi đã viết thêm vòng lặp retry 10 lần cho lệnh `curl /health`, giúp bước Deploy luôn ổn định và hoàn thành màu xanh.
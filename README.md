# Medical Image Segmentation

Trong cuộc thi này, mục tiêu là xây dựng một mô hình học sâu để giải quyết bài toán phân đoạn ảnh (image segmentation) trong lĩnh vực y tế. Bài toán yêu cầu phân đoạn chính xác các vùng quan tâm trên ảnh y khoa. Phần dưới là tổng quan phương pháp của em, bao gồm các cải tiến và kỹ thuật được áp dụng để tối ưu hóa hiệu suất trên tập dữ liệu.

---

## Phương Pháp Tiếp Cận

Để giải quyết bài toán phân đoạn ảnh y khoa, em đã triển khai một pipeline bao gồm sử dụng kiến trúc mô hình mạnh mẽ, tối ưu hóa siêu tham số, data augmentation, kết hợp nhiều hàm loss, và tăng cường thời gian kiểm tra (Test-Time Augmentation). Dưới đây là các thành phần chính của phương pháp:

### 1. **Model Architecture Mạnh Mẽ Hơn**
- **Kiến trúc**: Sử dụng **DeepLabV3Plus** với encoder **EfficientNet-B5**.
- **Số kênh của Decoder**: Số kênh trong decoder là **256**.
- **Quy mô mô hình**: Mô hình có khoảng **30 triệu tham số**.

### 2. **Tối Ưu Hóa Hyperparameters**
- **Kích thước ảnh**: Sử dụng ảnh đầu vào kích thước **320x320** để giữ lại nhiều thông tin không gian hơn.
- **Batch size**: **6**.
- **Learning rate**: Sử dụng **1e-4** với **CosineAnnealingWarmRestarts scheduler** để điều chỉnh tốc độ học một cách linh hoạt.
- **Epochs và early stopping**: Huấn luyện trong **100 epochs** với cơ chế **early stopping** (patience=15) để tránh overfitting.

### 3. **Data Augmentation Nâng Cao**
- **Kỹ thuật tăng cường**:
  - Áp dụng **ElasticTransform** và **GridDistortion** để mô phỏng các biến dạng phổ biến trong ảnh y khoa.
  - Sử dụng **CLAHE**, **MotionBlur**, và **HueSaturationValue** để tăng tính đa dạng của dữ liệu.
- **Chuẩn hóa**: Chuẩn hóa ảnh với thống kê của **ImageNet** để đảm bảo tính nhất quán.
- **Chiến lược**: Áp dụng data augmentation phức tạp cho tập huấn luyện, nhưng giữ data augmentation đơn giản cho tập kiểm tra để đảm bảo tính ổn định.

### 4. **Loss Function Kết Hợp**
- **Hàm mất mát**: Kết hợp **Dice Loss**, **Focal Loss**, và **Tversky Loss** để xử lý hiệu quả vấn đề mất cân bằng lớp và các trường hợp khó (hard examples).
- **Trọng số**:
  - Dice Loss: **40%**
  - Focal Loss: **30%**
  - Tversky Loss: **30%**

### 5. **Training Techniques Hiện Đại**
- **Mixed Precision Training (AMP)**: Giảm sử dụng bộ nhớ và tăng tốc độ huấn luyện.
- **Optimizer**: Sử dụng **AdamW** với **weight decay** để cải thiện khả năng tổng quát hóa.
- **Scheduler**: Áp dụng **CosineAnnealingWarmRestarts** để tối ưu hóa quá trình học.
- **Early stopping**: Theo dõi hiệu suất với **patience monitoring** để dừng huấn luyện khi mô hình không cải thiện.

### 6. **Test-Time Augmentation (TTA)**
- **Kỹ thuật TTA**: Sử dụng **4 augmentations** bao gồm: Original, Horizontal Flip, Vertical Flip, và Rotate 90°.
- **Kết hợp dự đoán**: Lấy trung bình các dự đoán để tăng độ chính xác và robust.
- **Hậu xử lý**: Áp dụng các **morphological operations** để tinh chỉnh kết quả phân đoạn.

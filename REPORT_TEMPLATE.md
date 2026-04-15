# CSC4005 – Lab 1 Report Template

## 1. Mục tiêu

### Mục tiêu học tập
Sau bài lab này, sinh viên xây dựng một pipeline huấn luyện hoàn chỉnh cho bài toán phân loại ảnh lỗi bề mặt thép bằng mạng nơ-ron truyền thẳng (MLP). Mục tiêu chính của buổi học không phải là chạy thật nhiều mô hình để đạt điểm số cao nhất, mà là **hiểu đúng quy trình huấn luyện một mô hình học sâu**.

Cụ thể, sinh viên sẽ:
- Chuẩn bị dữ liệu đúng cách
- Chọn hàm mất mát phù hợp
- Hiểu vai trò của train/validation/test set
- Theo dõi hiện tượng overfitting và underfitting
- So sánh các optimizer khác nhau
- Áp dụng regularization
- Ghi lại toàn bộ quá trình bằng Weights & Biases

### Bối cảnh thực tế
Chúng ta đóng vai nhóm kỹ sư AI hỗ trợ bộ phận QA trong dây chuyền sản xuất thép. Mỗi ảnh đầu vào là một ảnh grayscale chụp bề mặt thép. Mô hình cần dự đoán ảnh đó thuộc loại lỗi nào, giúp tự động hóa quá trình kiểm chất lượng.

### Bộ dữ liệu
Bộ dữ liệu sử dụng là **NEU Surface Defect Database**, một bộ dữ liệu công khai chứa các ảnh lỗi bề mặt thép với 6 loại lỗi chính:
- **Crazing** (mạng nứt)
- **Inclusion** (tạp chất)
- **Patches** (vùng mảng)
- **Pitted_Surface** (bề mặt có rãnh)
- **Rolled-in_Scale** (tầng oxy cuộn vào)
- **Scratches** (vết xước)


## 2. Cấu hình thí nghiệm

Ba cấu hình chính được thí nghiệm:

### Run A: Baseline AdamW
- **Optimizer**: AdamW
- **Learning Rate**: 0.001
- **Weight Decay**: 0.0001
- **Dropout**: 0.3
- **Hidden Dimensions**: [512, 256, 64]
- **Batch Size**: 32
- **Epochs**: 20
- **Scheduler**: None

### Run B: SGD with Higher Learning Rate
- **Optimizer**: SGD
- **Learning Rate**: 0.01 (10× cao hơn baseline)
- **Dropout**: 0.3
- **Hidden Dimensions**: [512, 256, 64]
- **Batch Size**: 32
- **Epochs**: 20

### Run C: Strong Regularization (Lower LR)
- **Optimizer**: AdamW
- **Learning Rate**: 0.0005 (50% giảm so với baseline)
- **Weight Decay**: 0.0001
- **Dropout**: 0.3
- **Hidden Dimensions**: [512, 256, 64]
- **Batch Size**: 32
- **Epochs**: 20

## 3. Kết quả

### Bảng so sánh hiệu suất

| Configuration | Val Loss | Val Acc | Test Loss | Test Acc |
|---|---|---|---|---|
| Run A (AdamW LR=0.001) | 1.4993 | 41.85% | 1.4957 | 38.15% |
| Run B (SGD LR=0.01) | 1.4447 | **46.30%** | 1.3950 | **45.56%** |
| Run C (AdamW LR=0.0005) | 1.6286 | 32.96% | 1.6021 | 34.44% |

### Learning Curves

**Run A (AdamW baseline)**: 
- Training loss giảm từ 1.814 → 1.732 
- Validation loss giảm từ 1.785 → 1.727
- Validation accuracy cải thiện từ 14.07% → 24.44%
- Học chậm và ổn định, không có dấu hiệu overfitting rõ ràng

**Run B (SGD với LR cao)**:
- Training loss giảm từ 1.789 → 1.650
- Validation loss giảm từ 1.770 → 1.641
- Validation accuracy cải thiện từ 17.41% → 29.63%
- Học tập nhanh hơn, tiến triển tốt nhất trong 3 run

**Run C (AdamW với LR thấp)**:
- Training loss giao động từ 1.830 → 1.760
- Validation loss giao động từ 1.778 → 1.761
- Validation accuracy biến động từ 16.67% → 24.07%
- Học chậm, không có sự cải thiện rõ rệt, dấu hiệu underfitting

### Phân tích chi tiết từng lớp (Run B - tốt nhất)

- **Patches**: Precision=48.8%, Recall=91.1%, F1=63.6% (tốt nhất)
- **Pitted_Surface**: Precision=55.9%, Recall=42.2%, F1=48.1%
- **Scratches**: Precision=45.7%, Recall=46.7%, F1=46.2%
- **Crazing**: Precision=45.5%, Recall=55.6%, F1=50.0%
- **Rolled-in_Scale**: Precision=33.3%, Recall=37.8%, F1=35.4%
- **Inclusion**: Precision=0%, Recall=0%, F1=0% (không được nhận diện)

## 4. Phân tích

### Cấu hình nào tốt nhất?
**Run B (SGD với Learning Rate = 0.01)** có hiệu suất tốt nhất với:
- Test Accuracy: 45.56% (cao hơn 7.41% so với Run A, 11.12% so với Run C)
- Test Loss: 1.3950 (thấp nhất)
- Validation Loss: 1.4447 (thấp nhất)
- Khả năng tổng quát hóa tốt hơn, gap validation-test không lớn

### Dấu hiệu Overfitting / Underfitting

**Run A (Baseline AdamW)**:
- Gap entre validation và training loss khá nhỏ (~0.01 từ epoch 5 onwards)
- Validation accuracy không tăng lên nhiều sau epoch 5
- Dấu hiệu **underfitting nhẹ**: Model không học được đủ tính năng của dữ liệu

**Run B (SGD LR cao)**:
- Loss curves validation và training chặt chẽ
- Cải thiện ổn định qua các epoch
- **Fit khác nhất**: Sự cân bằng tốt giữa học tập và tổng quát hóa

**Run C (Strong regularization)**:
- Validation loss dao động, không có xu hướng giảm rõ ràng
- Training loss vẫn không giảm đáng kể
- Dấu hiệu **underfitting nặng**: Learning rate quá thấp, model không thể học tập hiệu quả

### Sự khác biệt giữa AdamW và SGD

Trong thí nghiệm của chúng tôi:

1. **AdamW (Run A vs C)**:
   - Adaptive learning rate giúp tự điều chỉnh bước học
   - Nhưng cần learning rate phù hợp (0.001 tốt hơn 0.0005)
   - Khi LR quá thấp → underfitting nặng
   - Momentum mặc định giúp học ổn định nhưng chậm

2. **SGD (Run B)**:
   - Với learning rate cao hơn (0.01), hội tụ nhanh hơn
   - Cho metrics tốt hơn cả AdamW baseline (45.56% vs 38.15%)
   - Học tập tuyến tính nhưng hiệu quả hơn cho bài toán này
   - Có thể SGD với momentum phù hợp hơn với kiến trúc/dữ liệu này

**Kết luận**: SGD vượt trội AdamW trong thí nghiệm này nhờ learning rate được tinh chỉnh tốt hơn, chứng minh tầm quan trọng của hyperparameter tuning.

## 5. Kết luận

### Best Configuration: Run B (SGD với LR = 0.01)

**Lý do chọn**:
1. **Hiệu suất cao nhất**: Test Accuracy 45.56% là cao nhất trong 3 run
2. **Loss thấp nhất**: Test Loss 1.3950, cho thấy prediction confidence tốt hơn
3. **Tổng quát hóa tốt**: Validation-Test gap nhỏ (46.30% → 45.56%), model không overfit
4. **Học tập ổn định**: Learning curves mượt mà, không biến động
5. **Khám phá được pattern tốt**: Phát hiện tốt nhất lớp Patches (91.1% recall) và các lớp khác

### Nhận xét chung
- Bài toán phân loại NEU lỗi bề mặt vẫn còn khó (45.56% accuracy cho 6 lớp chỉ hơi tốt hơn random)
- Lớp **Inclusion** không được mô hình nhận diện được - cần thêm dữ liệu hoặc cải thiện đặc trưng
- **Data augmentation** giúp, nhưng model vẫn cần tối ưu hóa thêm (có thể dùng deeper network, pretrained models, v.v.)
- SGD đơn giản nhưng hiệu quả, cho thấy tầm quan trọng của lựa chọn optimizer phù hợp

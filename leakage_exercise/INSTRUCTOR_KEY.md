# Đáp án riêng — bài kiểm tra mù về data leakage

## Phạm vi và cách sử dụng
Đây là thí nghiệm có lỗi được cài có chủ đích để giảng viên kiểm toán. Chỉ gửi `blind_audit.ipynb` trong lượt kiểm tra ban đầu, giữ tệp đáp án này để đối chiếu sau khi thầy hoàn thành. Không dùng điểm A để tuyên bố cải thiện mô hình nghiên cứu. Notebook nghiên cứu `financal_analyze.ipynb` không thay đổi.

## Đáp án
- **A có leakage**. `fit_validation` gồm train cộng 25% các dòng validation và 40% các dòng test. Tập `fit_test` gồm development cộng 40% các dòng test. Validation và test vẫn giữ nguyên các dòng này khi đánh giá.
- **B là đối chứng**. Fit trên train/development với gap 5 phiên trước các mốc đánh giá; không đưa dòng validation/test tương ứng vào fit.
- `extra_v` và `extra_t` được lấy bằng RNG seed 42. Các dòng được sắp theo thứ tự chỉ mục sau khi ghép; việc sort không loại bỏ leakage.
- Cùng lỗi áp dụng cho mô hình hồi quy và phân loại. ExtraTrees có `min_samples_leaf=1` nên có thể ghi nhớ các dòng đã được học. Mức tăng điểm phải đọc từ kết quả thực chạy, không được giả định hoặc sửa tay.
- Rủi ro thứ hai: nhãn 5 ngày của các dòng bổ sung có thể đưa thông tin vượt mốc thời gian fit; trùng dòng không phải vấn đề duy nhất.

## Kiểm tra nên bổ sung
```python
assert set(fitting).isdisjoint(evaluating)
assert X.index[fitting].max() < X.index[evaluating].min()
assert label_end.iloc[fitting].max() < X.index[evaluating].min()
```
Áp dụng riêng cho cả hai bước validation và test. Thất bại ở A là kết quả mong đợi của bài tập. Với protocol B các điều kiện đều phải đạt.

## Diễn giải
Điểm A là hiệu năng trên một tập có các quan sát đã tham gia fit, không phải ước lượng out-of-sample hợp lệ. Điểm cao hơn minh họa nguy cơ đánh giá sai. B là kết quả đối chứng, không được so trực tiếp với notebook nghiên cứu như một cải thiện vì notebook nghiên cứu có danh sách mô hình và cách lựa chọn khác.

## Kết quả thực chạy (mẫu lịch sử đã ghim)

Hồi quy:
```csv
Case,Stage,RMSE,MAE,R2
A,Validation,0.040676957294327576,0.023706835997210268,0.6315426091309244
A,Test,0.06473267370810612,0.025983567426955564,0.8416516336650112
B,Validation,0.05396335570974619,0.03792677807086537,0.35153244717812415
B,Test,0.10137388296682748,0.05238110809367152,0.6116541577044377
```

Phân loại:
```csv
Case,Stage,Accuracy,Balanced accuracy,AUC,Brier
A,Validation,0.6213333333333333,0.6164348420932619,0.7203853608482499,0.19461374814814814
A,Test,0.7116402116402116,0.7049389815347262,0.8232736282381672,0.1554730158730159
B,Validation,0.5013333333333333,0.4958243073765819,0.5038265306122449,0.2598423703703704
B,Test,0.503968253968254,0.5053031755159415,0.5181742025713657,0.2585405643738977
```

SHA-256 notebook bài tập: `cc2146f2aa40d4db3e792068df53f2d23861ea954fdbaa19ecc9b340a7c99fc0`.

---

title : "Đánh giá và hiệu quả"
date: 2025-07-16
weight : 4
chapter : false
pre : " <b> 4. </b> "
---------------------

## 📈 Đánh giá mô hình và hiệu quả pipeline
Sau khi hoàn thành pipeline tạo dữ liệu tổng hợp, chúng ta cần đánh giá cả **chất lượng dữ liệu sinh ra** và **hiệu quả tổng thể** của hệ thống. 
Các tiêu chí đánh giá gồm:

* Mức độ tương đồng phân phối giữa dữ liệu thật và dữ liệu tổng hợp
* Khả năng loại bỏ thông tin định danh (PII)
* Khả năng tái sử dụng pipeline với dataset khác
* Thời gian và chi phí triển khai so với cách làm thủ công

---

## 1. Đánh giá chất lượng dữ liệu tổng hợp

### 🎯 Tiêu chí chính:
- **Histogram Similarity**: Độ tương đồng phân phối giữa dữ liệu thật và synthetic cho từng cột numeric.
- **Correlation Similarity**: So sánh ma trận tương quan của dữ liệu thật và synthetic.
- **Outlier Proportion**: Tỷ lệ giá trị ngoại lai trong synthetic so với dữ liệu thật.
- **Diversity**: Độ đa dạng của dữ liệu synthetic (không bị lặp lại quá nhiều giá trị).

### 🧪 Ví dụ kiểm tra histogram similarity:
Dùng khoảng cách Wasserstein (Earth Mover’s Distance) để đo độ lệch phân phối:

```python
from scipy.stats import wasserstein_distance

def histogram_similarity(real_series, synthetic_series):
    real_series = real_series.dropna().astype(float)
    synthetic_series = synthetic_series.dropna().astype(float)
    distance = wasserstein_distance(real_series, synthetic_series)
    return max(0, 1 - distance) * 100  # % tương đồng

similarity_amount = histogram_similarity(df['amount'], synthetic_df['amount'])
print(f"Histogram similarity (amount): {similarity_amount:.2f}%")
```
### 🎯 Kỳ vọng:
* Histogram similarity: ≥ 90% cho các cột quan trọng (amount, step…)
* Correlation similarity: ≥ 85%
* 100% bản ghi vượt qua kiểm tra regex PII và Macie
---

## 2. Đánh giá khả năng loại bỏ PII
### ✅ Mục tiêu:
Đảm bảo dữ liệu sinh ra không chứa thông tin nhạy cảm từ dữ liệu gốc.

### 🔍 Cách kiểm tra:
* Kiểm tra tự động bằng regex cho email, số tài khoản, số điện thoại.
* Quét Macie trên output synthetic để xác nhận.

```python
import re

patterns = {
    "email": r"[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}",
    "phone": r"\b\d{9,11}\b",
    "acct": r"\b\d{6,20}\b"
}

for col in synthetic_df.columns:
    for name, pat in patterns.items():
        matches = synthetic_df[col].astype(str).str.contains(pat, regex=True, na=False).sum()
        if matches > 0:
            print(f"⚠️ Found {matches} {name} in column {col}")
```
### 🎯 Kỳ vọng:
* Không có match nào xuất hiện ở bất kỳ cột nào.
---

## 3. Khả năng tái sử dụng pipeline
### 🔄 Tiêu chí:
- Thay dataset → chỉ cần thay đường dẫn S3 hoặc mapping schema trong Glue.
- Không cần viết lại code xử lý/học máy từ đầu.
- Pipeline có thể chạy cho ≥ 2 dataset khác nhau.

### 📌 Cách kiểm tra:
- Thử chạy lại pipeline với dataset thứ 2 (cùng định dạng tabular).
- Đo thời gian từ upload data → dữ liệu synthetic output.

### 🎯 Kỳ vọng:
- Thời gian setup lại < 15 phút.
- Code core (huấn luyện TVAE, đánh giá) giữ nguyên > 90%.
---

## 4. Hiệu quả thời gian và chi phí
### 4.1. So sánh với quy trình thủ công
| Tiêu chí                 | Thủ công (ẩn danh + tạo mẫu) | Pipeline tự động     |
| -------------------------| -----------------------------| ---------------------|
| Thời gian xử lý 1 dataset| 1–2 ngày                     | 2–3 giờ              |
| Nguy cơ sót PII          | Cao                          | Thấp (Macie + regex) |
| Độ tương đồng dữ liệu    | Không ổn định                | ≥ 90% thống kê       |
| Chi phí                  | Cao (nhân sự)                | ~0.45 USD (Free Tier)|

### 4.2. Tính tiết kiệm thời gian
Ví dụ:
- Lần đầu chạy pipeline: 3 giờ (bao gồm setup dịch vụ).
- Lần sau (reuse): < 1 giờ.
→ Tiết kiệm ~70% thời gian mỗi lần chạy lại.

### 4.3. Chi phí dự kiến (AWS Free Tier)
- S3: 0.10 USD
- Glue: 0.15 USD
- Macie: 0.20 USD
- SageMaker Studio Lab: 0 USD
→ Tổng: ~0.45 USD / 1 dataset


## 5. Tổng kết lợi ích thu được

### 🎯 Từ góc nhìn kỹ thuật:
* ✅ Đảm bảo dữ liệu an toàn, không PII.
* ✅ Dữ liệu synthetic giữ nguyên tính hữu ích cho ML/AI.
* ✅ Pipeline dễ bảo trì, dễ mở rộng.

### 💼 Từ góc nhìn doanh nghiệp:
* ⏱️ Tăng tốc time-to-insight.
* 💰 Giảm chi phí xử lý dữ liệu.
* 📊 Tăng khả năng hợp tác giữa các nhóm mà không vi phạm bảo mật.
---



---
title : "Các bước chuẩn bị"
date: 2025-06-18
weight : 2
chapter : false
pre : " <b> 2. </b> "
---

## Tổng quan Chuẩn bị

Để thực hiện workshop thành công, bạn cần chuẩn bị đầy đủ các điều kiện tiên quyết về tài khoản **AWS**, môi trường làm việc, và kiến thức cơ bản. Phần này sẽ hướng dẫn chi tiết từng bước chuẩn bị.

## 1. Tài khoản AWS

### Yêu cầu Tài khoản:
- **AWS Account** đã kích hoạt và xác thực đầy đủ
- **Credit card** đã đăng ký (chỉ dùng cho xác minh, không bị tính phí trong Free Tier)
- **S3 & SageMaker** phải hoạt động ở Region hỗ trợ (ví dụ: `ap-southeast-1`)

### AWS Free Tier Coverage:
Workshop này được thiết kế để chạy hoàn toàn trong **AWS Free Tier**:

| Dịch vụ                | AWS Free Tier                 | Sử dụng trong workshop      |
|------------------------|-------------------------------|-----------------------------|
| **S3**                 | 5GB, 20.000 GET/PUT           | < 500MB, vài trăm requests  |
| **Training Job**       | Không Free Tier               | Dùng instance nhỏ + Spot    |
| **Glue**               | 1 triệu requests ETL/month    | 1–2 job nhỏ, < 10 phút/job  |
| **Macie**              | Dùng thử 30 ngày miễn phí     | Quét vài trăm MB dữ liệu    |
| **SageMaker Studio**   | 250 giờ t3.medium/tháng       | ~2–3 giờ huấn luyện mô hình |

{{% notice info %}}
**Ước tính chi phí**: ~$0 nếu dùng Notebook, < $1–2 USD nếu huấn luyện ít hoặc lưu dữ liệu nhỏ..
{{% /notice %}}

### Thiết lập IAM và Quyền truy cập :

Để thực hiện workshop, tài khoản **AWS** của bạn cần có các permissions sau:

#### **Quyền IAM tối thiểu:**
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "sagemaker:*",
        "s3:*",
        "glue:*",
        "macie2:*",
        "iam:PassRole"
      ],
      "Resource": "*"
    }
  ]
}}
```

#### **Khuyến nghị:**
- Dùng AdministratorAccess trong môi trường học/lab.
- Không dùng root account để thao tác.
- Gắn IAM Role vào SageMaker Studio để tránh quản lý key thủ công.

## 2. Môi trường Làm việc

### Trình duyệt:
- **Modern browser**: Chrome, Firefox, Safari bản mới nhất
- **JavaScript enabled**
- **Cookies enabled** cho **AWS Console**
- **Pop-up blocker disabled** cho AWS domains

### Kết nối mạng:
- **Stable internet connection** ( ≥ 2 Mbps)
- **Access to AWS endpoints** (không bị firewall block)
- **HTTPS support** (TLS 1.2+)

### Công cụ cần thiết:
| Công cụ | Yêu cầu tối thiểu                       |
| ------- | --------------------------------------- |
| Python  | >= 3.10                                 |
| pip     | Cài gói: `pandas`, `boto3`, `sagemaker`, `sdv[all]`, `seaborn` |
| Jupyter | Khuyến khích dùng  SageMaker Studio      |
| AWS CLI | Để thao tác S3, Glue, Macie qua terminal (tuỳ chọn) |

```
pip install pandas boto3 sdv[all] matplotlib seaborn

```
AWS CLI (tuỳ chọn):
```
aws configure
```

## 3. Kiến thức Cơ bản

### 3.1 Tổng quan về AWS cho ML

#### **AWS Services Overview:**
| Dịch vụ          | Vai trò chính trong workshop |
|------------------|------------------------------|
| **Amazon S3**    | Lưu trữ dữ liệu gốc và dữ liệu tổng hợp(.csv)|
| **AWS Glue**     | Tiền xử lý và chuẩn hóa dữ liệu |
| **Amazon Macie** | 	Phát hiện thông tin nhạy cảm (PII) |
| **SageMaker Studio** | Huấn luyện mô hình sinh dữ liệu |
| **IAM**          | Quản lý quyền truy cập tài nguyên AWS |

#### **AWS Regions và Availability Zones:**
- **Region**: Geographical area với multiple data centers
- **AZ**: Isolated data center trong một region
- **Workshop region**: **ap-southeast-1 (Singapore)**

### 3.2 Kiến thức Machine Learning cơ bản

#### Khái niệm chính:
| Thuật ngữ                   | Mô tả                                                        |
| --------------------------- | ------------------------------------------------------------ |
| **Synthetic Data**          | Dữ liệu giả lập mô phỏng phân phối thống kê của dữ liệu thật |
| **PII**                     | Thông tin định danh cá nhân (email, số tài khoản...)         |
| **TVAE/CTGAN**              | Mô hình sinh dữ liệu tabular dựa trên deep learning          |
| **Train/Test Split**        | Chia dữ liệu để huấn luyện và kiểm thử                       |
| **Histogram & Correlation** | Công cụ đánh giá tương đồng dữ liệu                          |


## 4. Dữ liệu & Thư mục
### Dataset yêu cầu:
- BankSim Dataset từ Kaggle (mô phỏng giao dịch ngân hàng).
- Định dạng: .csv
- Có thể tải về và upload lên S3.

### Cấu trúc thư mục gợi ý:

```
synthetic-data-workshop/
├── data/
│   ├── bs140513_032310.csv
│   └── bsNET140513_032310.csv
├── notebooks/
│   └── 01_pipeline.ipynb
├── scripts/
│   └── generate_synthetic.py
└── README.md
```

## 5. Tài liệu Tham khảo

### **AWS Documentation:**
- [AWS Glue Documentation](https://docs.aws.amazon.com/glue/)
- [Amazon Macie Documentation](https://docs.aws.amazon.com/macie/)
- [SageMaker Python SDK](https://sagemaker.readthedocs.io/en/stable/)
- [AWS Boto3 S3 Docs](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/s3.html)
- [Amazon SageMaker Documentation]()
- [SDV Documentation]()
- [Pandas Documentation](https://pandas.pydata.org/docs/)



{{% notice tip %}}
Nếu gặp vấn đề trong quá trình chuẩn bị, hãy tham khảo **AWS Support** hoặc **AWS Community Forums** để được hỗ trợ.
Không cần chuyên gia ML để tham gia workshop này. Chỉ cần hiểu các khái niệm cơ bản và làm theo hướng dẫn, bạn sẽ có thể xây dựng pipeline tạo dữ liệu tổng hợp an toàn và tái sử dụng được.
{{% /notice %}}
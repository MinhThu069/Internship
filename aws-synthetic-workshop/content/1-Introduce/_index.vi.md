---
title : "Giới thiệu"
date: 2025-07-16
weight : 1 
chapter : false
pre : " <b> 1. </b> "
---

## Tổng quan Workshop

Workshop này hướng dẫn cách xây dựng **Nền tảng Tạo Dữ liệu Tổng hợp với Bảo toàn Quyền riêng tư** trên AWS.  
Bạn sẽ học cách:
- Tải dữ liệu tài chính mẫu từ S3
- Tiền xử lý và chuẩn hóa dữ liệu bằng AWS Glue
- Kiểm tra và loại bỏ thông tin định danh (PII) với Amazon Macie
- Huấn luyện mô hình sinh dữ liệu (TVAE) bằng SageMaker Studio Lab
- Đánh giá chất lượng và phân phối của dữ liệu tổng hợp

## Vì sao cần tạo dữ liệu tổng hợp?

Trong nhiều tổ chức, đặc biệt là **ngân hàng và tài chính**, dữ liệu thật thường chứa thông tin nhạy cảm và không thể chia sẻ cho nhóm kỹ thuật hoặc cộng tác bên ngoài. Việc ẩn danh thủ công:
- **Tốn nhiều thời gian** và dễ sai sót
- **Nguy cơ rò rỉ PII** nếu làm chưa chuẩn
- **Cản trở hợp tác** giữa các nhóm
- **Giảm tốc độ nghiên cứu và phát triển AI**

### Lợi ích của dữ liệu tổng hợp:
- **Loại bỏ PII** nhưng vẫn giữ phân phối thống kê gốc
- **An toàn chia sẻ** cho Data Scientist, Developer, Researcher
- **Tăng tốc phát triển mô hình** mà không vi phạm quy định
- **Tiết kiệm chi phí** (có thể triển khai trên AWS Free Tier)

## Kiến trúc Hệ thống

### Luồng xử lý dữ liệu:
CSV → Amazon S3 → AWS Glue → Amazon Macie → SageMaker Studio Lab (TVAE) → Dữ liệu tổng hợp → Đánh giá thống kê & trực quan hóa

![Architecture](/images/draw.png)

### Các dịch vụ chính:

#### 1. **Amazon S3**
- Lưu trữ dữ liệu gốc và dữ liệu tổng hợp
- Hỗ trợ kết nối trực tiếp với Glue, Macie và Studio Lab

#### 2. **AWS Glue**
- Tiền xử lý dữ liệu: chuẩn hóa schema, xử lý giá trị null, đổi tên cột
- Xuất dữ liệu dạng CSV hoặc Parquet

#### 3. **Amazon Macie**
- Quét dữ liệu để phát hiện thông tin nhạy cảm (email, số tài khoản…)
- Xuất báo cáo kiểm tra PII

#### 4. **SageMaker Studio Lab + SDV (TVAE)**
- Huấn luyện mô hình sinh dữ liệu
- Sinh dữ liệu tổng hợp có phân phối tương tự dữ liệu thật
- Trực quan hóa và so sánh histogram, correlation

## Mục tiêu cuối cùng

Workshop sẽ giúp bạn:
- Tạo ra dữ liệu tổng hợp **không chứa PII**
- Đảm bảo phân phối dữ liệu sinh ra ≥ 90% giống dữ liệu gốc
- Xây dựng pipeline có thể **tái sử dụng cho nhiều dataset**
- Sẵn sàng tích hợp vào các pipeline AI/ML thực tế

## Chuẩn bị Workshop

Trước khi bắt đầu, hãy đảm bảo bạn có:
- **AWS Account** với quyền sử dụng:
  - Amazon S3
  - AWS Glue
  - Amazon Macie
  - Amazon SageMaker
- Nếu là tài khoản mới: nên **kích hoạt Free Tier**
- **Quyền IAM tối thiểu**:
  - `AmazonS3FullAccess` (hoặc quyền với bucket cụ thể)
  - `AWSGlueServiceRole`
  - `AmazonMacieFullAccess`
  - `AmazonSageMakerFullAccess`

{{% notice info %}}
Workshop này được thiết kế để chạy trong **AWS Free Tier**. Tổng chi phí ước tính **< $1 USD** nếu thực hiện đúng hướng dẫn.
{{% /notice %}}

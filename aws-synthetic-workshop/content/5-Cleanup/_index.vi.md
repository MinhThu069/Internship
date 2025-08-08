---
title : "Dọn dẹp tài nguyên"
date: 2025-07-16
weight : 5
chapter : false
pre : " <b> 5. </b> "
---

Sau khi hoàn thành workshop, bạn cần dọn dẹp các tài nguyên **AWS** để tránh phát sinh chi phí không mong muốn.

## 1. Xoá dữ liệu trên Amazon S3 (và bucket)
### Cách 1: Console
- Truy cập **AWS Console** → **S3**
- Tìm bucket bạn đã tạo, ví dụ:  
  - `synthetic-data-demo-`
- Thực hiện:
  - Nhấn **Empty** → Xác nhận
  - Sau đó **Delete bucket** → Xác nhận

### Cách 2: CLI
```
aws s3 rm s3://$BUCKET --recursive
aws s3 rb s3://$BUCKET
```

{{% notice warning %}}
**Bucket** phải trống hoàn toàn trước khi có thể xoá. Hãy đảm bảo các thư mục raw/, cleaned/, synthetic/, reports/, scripts/ đã bị xoá.
{{% /notice %}}

## 2. Xoá tài nguyên AWS Glue (Job, script, Crawler/Data Catalog nếu có)
### Console 
- Vào **AWS Glue** → **ETL jobs** → chọn job bạn tạo (ví dụ Glue-BankSim-Clean) → Delete.
### CLI
```
# Xoá Glue Job
aws glue delete-job --job-name Glue-BankSim-Clean

# (Tuỳ chọn) Nếu bạn có upload script lên S3, xoá file script
aws s3 rm s3://$BUCKET/scripts/glue_job_banksim.py
```
## 3. Macie — dọn dẹp job & kết quả quét
### Console
- Mở **Amazon Macie** → **Jobs**.
- Nếu **job** là ONE_TIME thì sẽ hoàn tất và dừng; bạn có thể **Delete job** để gọn tài nguyên.
- Vào **Findings** → **chọn tất cả** → Archive (lưu trữ kết quả cho sạch dashboard).

### CLI
```
import os, boto3
region = os.environ.get("AWS_REGION","ap-southeast-1")
macie = boto3.client("macie2", region_name=region)

# Liệt kê job rồi xoá (ví dụ xoá job theo tên)
jobs = macie.list_classification_jobs()["items"]
for j in jobs:
    if j.get("name") == "macie-scan-cleaned":
        macie.delete_classification_job(jobId=j["jobId"])
        print("Deleted:", j["jobId"])
```

{{% notice tip %}}
Bạn không bắt buộc phải Disable toàn bộ Macie của tài khoản/region. Chỉ cần xoá job và archive findings là đủ để không phát sinh chi phí thêm (nhất là khi còn trong thời gian trial).
{{% /notice %}}

## 4. SageMaker: dừng/xoá tài nguyên Studio hoặc Studio Lab

### 4.1 Nếu bạn dùng SageMaker Studio
- **SageMaker** → **Studio** → **Domains** → **Users**:
- Chọn **User profile** của bạn → **Delete apps** (KernelGateway/JupyterServer…) nếu còn đang chạy.
- (Tuỳ chọn, cân nhắc kỹ) **Delete Domain** và **User profile** nếu không dùng nữa.

{{% notice warning %}}
Xoá Domain sẽ loại bỏ toàn bộ apps/metadata liên quan đến Studio. Chỉ làm nếu chắc chắn không dùng lại.
{{% /notice %}}

### 4.2 Nếu bạn dùng SageMaker Studio Lab
- Chỉ cần **Shutdown kernel/notebook** hiện tại.
- **Studio Lab** không tạo **training job/endpoints** trên tài khoản AWS, nên không có gì để xoá ngoài file local của workspace (nếu muốn).

## 5. Xoá IAM Role/Policy tạm dùng cho Glue
### Console
- Truy cập **IAM** → **Roles**
- Tìm các Role được SageMaker tạo:
  - Ví dụ: `AWSGlueServiceRole-SyntheticDemo.`
- Chỉ nên xóa nếu không sử dụng cho mục đích khác

{{% notice tip %}}
Bạn có thể giữ lại **IAM Role** nếu có kế hoạch sử dụng lại trong tương lai.
{{% /notice %}}

### CLI
```
aws iam detach-role-policy --role-name AWSGlueServiceRole-SyntheticDemo --policy-arn arn:aws:iam::aws:policy/service-role/AWSGlueServiceRole
aws iam detach-role-policy --role-name AWSGlueServiceRole-SyntheticDemo --policy-arn arn:aws:iam::aws:policy/AmazonS3FullAccess
aws iam delete-role --role-name AWSGlueServiceRole-SyntheticDemo
```

## 6. Kiểm tra Billing

- Truy cập **Billing Dashboard** → **Cost Explorer**
- Lọc theo các dịch vụ: **S3**, **Glue**,**Macie**, **SageMaker** v.v.
- Đảm bảo không còn tài nguyên chạy nền (Glue job pending, Studio app đang chạy…).
- Tạo AWS Budget nhỏ (ví dụ 5–10 USD) để nhận cảnh báo nếu có phát sinh.

## 7. Checklist hoàn tất ✅

✅Đã xóa **S3 Bucket**  
✅Đã xóa **Glue Job/Script**  
✅Đã xóa kết quả **Macie jobs & findings**  
✅Đã xóa **Studio Apps** (nếu có)  
✅Đã xóa **IAM Roles** (nếu cần)  
✅Đã kiểm tra **Billing** và không phát sinh chi phí  

{{% notice info %}}
Một số tài nguyên có thể mất vài phút để biến mất khỏi AWS Console sau khi xóa.
{{% /notice %}}

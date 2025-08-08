---
title : "Các bước thực hiện"
date: 2025-07-16
weight : 3
chapter : false
pre : " <b> 3. </b> "
---

## 🛠️ Các bước thực hiện

Phần này hướng dẫn từng bước cụ thể để triển khai **pipeline tạo dữ liệu tổng hợp có bảo toàn quyền riêng tư** trên AWS.
Pipeline sẽ bao gồm: **lưu trữ – tiền xử lý – kiểm tra PII – huấn luyện mô hình TVAE – đánh giá dữ liệu**.

## 1. Upload dữ liệu lên Amazon S3
🌟 Mục tiêu:
Đưa tập dữ liệu CSV gốc lên S3 để để làm nguồn đầu vào.

📂 Thực hiện:
- Truy cập **AWS Console** → chọn **Amazon S3**.
- Tạo một bucket mới, ví dụ: `synthetic-data-demo`.
- Upload tập `bs140513_032310.csv` lên thư mục raw/
Hoặc sử dụng:
```
aws s3 mb s3://synthetic-data-demo
aws s3 cp bs140513_032310.csv s3://synthetic-data-demo/raw/
```

💬 Lưu ý:
Bucket nên cùng Region với SageMaker (ví dụ ap-southeast-1)
Dữ liệu gốc không nên chứa thông tin định danh thật

## 2. Tiền xử lý dữ liệu với AWS Glue
🌟 Mục tiêu:
Làm sạch dữ liệu, chuẩn hóa schema, xử lý giá trị thiếu trước khi huấn luyện.
```python
# glue_job_banksim.py
import sys
from awsglue.transforms import ApplyMapping
from awsglue.utils import getResolvedOptions
from pyspark.context import SparkContext
from awsglue.context import GlueContext
from awsglue.job import Job

args = getResolvedOptions(sys.argv, ["JOB_NAME", "SOURCE_BUCKET", "TARGET_BUCKET"])
sc = SparkContext()
glueContext = GlueContext(sc)
spark = glueContext.spark_session
job = Job(glueContext)
job.init(args["JOB_NAME"], args)

source_path = f"s3://{args['SOURCE_BUCKET']}/raw/"
target_path = f"s3://{args['TARGET_BUCKET']}/cleaned/"

# Đọc CSV (header auto)
dyf = glueContext.create_dynamic_frame.from_options(
    connection_type="s3",
    connection_options={"paths": [source_path]},
    format="csv",
    format_options={"withHeader": True}
)

mappings = []
mappings.append(("step", "long", "step", "int"))
mappings.append(("amount", "double", "amount", "double"))

try:
    dyf_cleaned = ApplyMapping.apply(frame=dyf, mappings=mappings)
except Exception:
    dyf_cleaned = dyf

# Ghi ra Parquet
glueContext.write_dynamic_frame.from_options(
    frame=dyf_cleaned,
    connection_type="s3",
    connection_options={"path": target_path},
    format="parquet"
)

job.commit()

```
💬 Lưu ý:
- Giữ schema nhất quán để tránh lỗi khi huấn luyện.


## 3. Quét và loại bỏ thông tin định danh (PII) với Amazon Macie
🌟 Mục tiêu:
Đảm bảo dữ liệu đã làm sạch không còn chứa email, số tài khoản, địa chỉ…
```python
import boto3, json, time, os
region = os.environ.get("AWS_REGION","ap-southeast-1")
bucket = os.environ.get("BUCKET") or "synthetic-data-demo-<your-unique-suffix>"

macie = boto3.client("macie2", region_name=region)

resp = macie.create_classification_job(
    jobType="ONE_TIME",
    name="macie-scan-cleaned",
    s3JobDefinition={
        "bucketDefinitions":[{"accountId": boto3.client('sts').get_caller_identity()['Account'], "buckets":[bucket]}],
        "scoping":{
            "includes":{
                "and":[{"simpleScopeTerm":{"comparator":"STARTS_WITH","key":"OBJECT_KEY","values":["cleaned/"]}}]
            }
        }
    },
    samplingPercentage=100
)
print("Created Macie job:", resp["jobId"])

```
💬 Lưu ý:
- Nếu vẫn còn PII → quay lại bước Glue để loại bỏ.

## 4. Huấn luyện mô hình TVAE trong SageMaker Studio Lab
🌟 Mục tiêu:
Sinh dữ liệu tổng hợp giữ nguyên phân phối thống kê nhưng không chứa PII.
- 🔄 Các bước:
1. Mở Jupyter Notebook trong SageMaker Studio Lab.
2. Cài các thư viện cần thiết:
``` 
pip install sdv[all] pandas matplotlib seaborn boto3 
```
3. Tải Parquet đã làm sạch từ S3 & đọc vào Pandas
```python
import boto3, os, pandas as pd

AWS_REGION = os.environ.get("AWS_REGION","ap-southeast-1")
BUCKET = os.environ.get("BUCKET") or "synthetic-data-demo-<your-unique-suffix>"
CLEANED_PREFIX = "cleaned/"

s3 = boto3.client("s3", region_name=AWS_REGION)

# Tải file parquet đầu tiên về (demo)
resp = s3.list_objects_v2(Bucket=BUCKET, Prefix=CLEANED_PREFIX)
keys = [o["Key"] for o in resp.get("Contents", []) if o["Key"].endswith(".parquet")]
assert len(keys) > 0, "Không tìm thấy file Parquet trong cleaned/"

local_pq = "cleaned.parquet"
s3.download_file(BUCKET, keys[0], local_pq)

# Đọc parquet => pandas
df = pd.read_parquet(local_pq)
df.head()
```

4. Huấn luyện TVAE & sinh dữ liệu tổng hợp
```python
from sdv.tabular import TVAE
import numpy as np
import random

SEED = 42
np.random.seed(SEED); random.seed(SEED)

model = TVAE(epochs=300, batch_size=500, cuda=False)  # cuda=True nếu có GPU
model.fit(df)

synthetic_df = model.sample(len(df))
synthetic_df.head()
```

## 5. Đánh giá dữ liệu tổng hợp
🌟 Mục tiêu:
So sánh phân phối dữ liệu tổng hợp với dữ liệu gốc.
```python
import matplotlib.pyplot as plt

def compare_hist(col):
    if col not in df.columns or col not in synthetic_df.columns:
        return
    plt.figure(figsize=(6,4))
    df[col].dropna().astype(float).hist(bins=30, alpha=0.5, label="Real")
    synthetic_df[col].dropna().astype(float).hist(bins=30, alpha=0.5, label="Synthetic")
    plt.legend(); plt.title(f"Distribution: {col}"); plt.show()

# Thử vài cột numeric “quen thuộc”
for col in ["amount", "step"]:
    if col in df.columns:
        compare_hist(col)

# Correlation (nếu là numeric)
real_corr = df.select_dtypes(include="number").corr()
syn_corr  = synthetic_df.select_dtypes(include="number").corr()
print("Real Corr shape:", real_corr.shape, "Synthetic Corr shape:", syn_corr.shape)
```

## 6. Lưu trữ và tái sử dụng dữ liệu tổng hợp lên S3
🌟 Mục tiêu:
Lưu dữ liệu tổng hợp để chia sẻ hoặc huấn luyện các mô hình khác.
```
synthetic_df.to_csv("synthetic_banksim.csv", index=False)
s3.upload_file("synthetic_banksim.csv", "synthetic-data-demo", "synthetic/synthetic_banksim.csv")

```

## 7. Cleanup (tuỳ chọn)

```bash
# Xoá mọi object trong bucket
aws s3 rm s3://$BUCKET --recursive

# Xoá bucket
aws s3 rb s3://$BUCKET

# Xoá Glue Job
aws glue delete-job --job-name Glue-BankSim-Clean

# (Nếu muốn) Detach & xoá role Glue
aws iam detach-role-policy --role-name AWSGlueServiceRole-SyntheticDemo --policy-arn arn:aws:iam::aws:policy/service-role/AWSGlueServiceRole
aws iam detach-role-policy --role-name AWSGlueServiceRole-SyntheticDemo --policy-arn arn:aws:iam::aws:policy/AmazonS3FullAccess
aws iam delete-role --role-name AWSGlueServiceRole-SyntheticDemo

```

{{% notice tip %}}
Lưu ý quan trọng:
- Giữ nguyên Region (ví dụ ap-southeast-1) cho S3/Glue/Macie/Studio để tránh lỗi quyền & định vị.
- Tên bucket phải unique toàn cầu; sửa biến BUCKET ngay từ đầu.
- Macie: enable service trong region trước khi tạo job. Bản dùng thử thường miễn phí 30 ngày.
- SDV/TVAE: lần đầu cài có thể hơi lâu (kéo theo torch/sklearn). Nhớ Restart Kernel sau khi pip install.
- Schema BankSim có thể khác nhau (bản PaySim/BankSim biến thể). Mình đã tránh “cứng” schema; nếu bạn muốn ép kiểu nhiều cột hơn trong Glue, chỉnh mappings tương ứng (hoặc đọc schema từ crawler rồi export).
{{% /notice %}}
---
title : "Implementation Steps"
date: 2025-07-16
weight : 3
chapter : false
pre : " <b> 3. </b> "
---

## 🛠️ Implementation Steps

This section walks you through each step of building a **privacy-preserving synthetic data generation pipeline** on AWS.  
The pipeline will cover: **storage – preprocessing – PII scanning – TVAE model training – data evaluation**.

---

## 1. Upload Data to Amazon S3
🌟 **Goal**:  
Upload the original CSV dataset to S3 as the pipeline’s data source.

📂 **Steps**:
- Go to **AWS Console** → **Amazon S3**.
- Create a new bucket, e.g., `synthetic-data-demo`.
- Upload `bs140513_032310.csv` into the `raw/` folder.  
  Or via AWS CLI:
```bash
aws s3 mb s3://synthetic-data-demo
aws s3 cp bs140513_032310.csv s3://synthetic-data-demo/raw/
```

💬 Note:  
- The bucket should be in the same Region as SageMaker (e.g., ap-southeast-1)  
- Original data should not contain real PII.

## 2. Preprocess Data with AWS Glue
🌟 Goal:  
Clean the data, standardize the schema, handle missing values before training.
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

# Read CSV (header auto-detected)
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

# Write to Parquet
glueContext.write_dynamic_frame.from_options(
    frame=dyf_cleaned,
    connection_type="s3",
    connection_options={"path": target_path},
    format="parquet"
)

job.commit()

```
💬 Note:
- Keep schema consistent to avoid training errors.

## 3. Scan and Remove PII with Amazon Macie
🌟 Goal:  
Ensure the cleaned data no longer contains sensitive fields such as emails, account numbers, or addresses.  
- 🔄 Steps:
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
💬 Note:
- If **PII** is detected → return to the **Glue** step for removal.


## 4. Train TVAE Model in SageMaker Studio Lab
🌟 Goal: 
Generate synthetic data with the same statistical distribution as the original, without PII.  
- 🔄 Steps:
1. Open Jupyter Notebook in SageMaker Studio Lab.
2. Install dependencies:
```
pip install sdv[all] pandas matplotlib seaborn boto3
```

3. Download cleaned Parquet data from S3 & load into Pandas:
```python
import boto3, os, pandas as pd

AWS_REGION = os.environ.get("AWS_REGION","ap-southeast-1")
BUCKET = os.environ.get("BUCKET") or "synthetic-data-demo-<your-unique-suffix>"
CLEANED_PREFIX = "cleaned/"

s3 = boto3.client("s3", region_name=AWS_REGION)

# Download first parquet file (demo)
resp = s3.list_objects_v2(Bucket=BUCKET, Prefix=CLEANED_PREFIX)
keys = [o["Key"] for o in resp.get("Contents", []) if o["Key"].endswith(".parquet")]
assert len(keys) > 0, "No Parquet file found in cleaned/"

local_pq = "cleaned.parquet"
s3.download_file(BUCKET, keys[0], local_pq)

# Load parquet into Pandas
df = pd.read_parquet(local_pq)
df.head()
```

4. Train TVAE & generate synthetic data:
```python
from sdv.tabular import TVAE
import numpy as np
import random

SEED = 42
np.random.seed(SEED); random.seed(SEED)

model = TVAE(epochs=300, batch_size=500, cuda=False)  # Set cuda=True if GPU is available
model.fit(df)

synthetic_df = model.sample(len(df))
synthetic_df.head()
```

## 5. Evaluate Synthetic Data
🌟 Goal:
Compare the distribution of synthetic data with the original dataset.

```python
import matplotlib.pyplot as plt

def compare_hist(col):
    if col not in df.columns or col not in synthetic_df.columns:
        return
    plt.figure(figsize=(6,4))
    df[col].dropna().astype(float).hist(bins=30, alpha=0.5, label="Real")
    synthetic_df[col].dropna().astype(float).hist(bins=30, alpha=0.5, label="Synthetic")
    plt.legend(); plt.title(f"Distribution: {col}"); plt.show()

# Try some familiar numeric columns
for col in ["amount", "step"]:
    if col in df.columns:
        compare_hist(col)

# Correlation (numeric only)
real_corr = df.select_dtypes(include="number").corr()
syn_corr  = synthetic_df.select_dtypes(include="number").corr()
print("Real Corr shape:", real_corr.shape, "Synthetic Corr shape:", syn_corr.shape)

```

## 6. Store and Reuse Synthetic Data in S3
🌟 Goal:
Save synthetic data for sharing or for training other models.
```
synthetic_df.to_csv("synthetic_banksim.csv", index=False)
s3.upload_file("synthetic_banksim.csv", "synthetic-data-demo", "synthetic/synthetic_banksim.csv")

```

## 7. Cleanup (Optional)
```bash
# Remove all objects in the bucket
aws s3 rm s3://$BUCKET --recursive

# Delete bucket
aws s3 rb s3://$BUCKET

# Delete Glue Job
aws glue delete-job --job-name Glue-BankSim-Clean

# (Optional) Detach & delete Glue role
aws iam detach-role-policy --role-name AWSGlueServiceRole-SyntheticDemo --policy-arn arn:aws:iam::aws:policy/service-role/AWSGlueServiceRole
aws iam detach-role-policy --role-name AWSGlueServiceRole-SyntheticDemo --policy-arn arn:aws:iam::aws:policy/AmazonS3FullAccess
aws iam delete-role --role-name AWSGlueServiceRole-SyntheticDemo
```

{{% notice tip %}}
Important Notes:
- Keep Region consistent (e.g., ap-southeast-1) for S3/Glue/Macie/Studio to avoid permission/location errors.
- Bucket names must be globally unique; set the BUCKET variable at the start.
- Macie: Enable the service in the Region before creating a job. The trial version is usually free for 30 days.
- SDV/TVAE: First install may take time (will also pull torch/sklearn). Remember to restart the kernel after pip install.
- BankSim schema may vary (PaySim/BankSim variants). Schema enforcement is minimal here; if you need strict typing in Glue, adjust the mappings accordingly (or read schema from a crawler and export).
{{% /notice %}}







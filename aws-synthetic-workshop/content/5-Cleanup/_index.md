---
title : "Resource Cleanup"
date: 2025-07-16
weight : 5
chapter : false
pre : " <b> 5. </b> "
---

After completing the workshop, you need to clean up **AWS** resources to avoid incurring unwanted charges.

## 1. Delete Data on Amazon S3 (and the bucket)
### Method 1: Console
- Go to **AWS Console** → **S3**
- Find the bucket you created, for example:  
  - `synthetic-data-demo-`
- Steps:
  - Click **Empty** → Confirm
  - Then **Delete bucket** → Confirm

### Method 2: CLI
```bash
aws s3 rm s3://$BUCKET --recursive
aws s3 rb s3://$BUCKET
```

{{% notice warning %}}
The bucket must be completely empty before it can be deleted. Make sure all folders such as raw/, cleaned/, synthetic/, reports/, scripts/ have been removed.
{{% /notice %}}

## 2. Delete AWS Glue Resources
### Console 
-  **AWS Glue** → **ETL jobs** → select the job you created (e.g., Glue-BankSim-Clean) → Delete.

### CLI
```bash
# Delete Glue Job
aws glue delete-job --job-name Glue-BankSim-Clean

# (Optional) If you uploaded the script to S3, delete the script file
aws s3 rm s3://$BUCKET/scripts/glue_job_banksim.py

```
## 3. Macie — Clean Up Jobs & Scan Results
### Console
- Open **Amazon Macie** → **Jobs**.
- If **job** is **ONE_TIME** it will complete and stop; you can **Delete job** to tidy up resources.
- Go to **Findings** → **Select all** → **Archive** (to clear the dashboard).

### CLI
```python
import os, boto3
region = os.environ.get("AWS_REGION", "ap-southeast-1")
macie = boto3.client("macie2", region_name=region)

# List jobs and delete (example: delete job by name)
jobs = macie.list_classification_jobs()["items"]
for j in jobs:
    if j.get("name") == "macie-scan-cleaned":
        macie.delete_classification_job(jobId=j["jobId"])
        print("Deleted:", j["jobId"])

```
{{% notice tip %}}
You do not need to disable the entire Macie service for the account/region.
Simply deleting jobs and archiving findings is enough to avoid extra charges (especially during the trial period).
{{% /notice %}}

## 4. SageMaker — Stop/Delete Studio or Studio Lab Resources

### 4.1 If You Use SageMaker Studio
- **SageMaker** → **Studio** → **Domains** → **Users**:
- Select your  **User profile** → **Delete apps** (KernelGateway/JupyterServer…) if still running.
- **Delete Domain** and **User profile** if no longer needed. (Optional, think carefully) 

{{% notice warning %}}
Deleting a Domain will remove all apps/metadata related to Studio.
Only do this if you are certain you won’t use it again.
{{% /notice %}}

### 4.2 If You Use SageMaker Studio Lab
- Simply **Shutdown** the current **kernel/notebook**.
- **Studio Lab** does not create **training job/endpoints** on your AWS account, so nothing needs to be deleted except local workspace files (optional).


## 5. Delete Temporary IAM Role/Policy for Glue
### Console
- Go to **IAM** → **Roles**
- Find roles created for Glue, for example:
  - `AWSGlueServiceRole-SyntheticDemo.`
- Only delete if they are not used for other purposes.

{{% notice tip %}}
You can keep the IAM Role if you plan to reuse it in the future.
{{% /notice %}}

### CLI
```
aws iam detach-role-policy --role-name AWSGlueServiceRole-SyntheticDemo --policy-arn arn:aws:iam::aws:policy/service-role/AWSGlueServiceRole
aws iam detach-role-policy --role-name AWSGlueServiceRole-SyntheticDemo --policy-arn arn:aws:iam::aws:policy/AmazonS3FullAccess
aws iam delete-role --role-name AWSGlueServiceRole-SyntheticDemo
```

## 6. Check Billing

- Go to **Billing Dashboard** → **Cost Explorer**
- Filter by services: **S3**, **Glue**,**Macie**, **SageMaker** etc.
- Ensure no background resources are running (pending Glue jobs, active Studio apps…).
- Create a small AWS Budget (e.g., $5–10 USD) to receive alerts if costs are incurred.


## 7. Final Checklist  ✅

✅Deleted **S3 Bucket**  
✅Deleted **Glue Job/Script**  
✅Deleted **Macie jobs & findings**  
✅Deleted **Studio Apps** (if any)  
✅Deleted **IAM Roles** (if needed)  
✅Deleted **Billing** to ensure no extra costs

{{% notice info %}}
Some resources may take a few minutes to disappear from the AWS Console after deletion.
{{% /notice %}}

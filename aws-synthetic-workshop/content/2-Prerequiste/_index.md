---
title : "Preparation Steps"
date: 2025-06-18
weight : 2
chapter : false
pre : " <b> 2. </b> "
---

## Preparation Overview

To successfully complete this workshop, you need to prepare all the prerequisites related to your **AWS** account, working environment, and basic knowledge. This section provides step-by-step preparation guidelines.

## 1. AWS Account

### Account Requirements:
- An **AWS Account** that is fully activated and verified
- A **registered credit card** (used for verification only; no charges within the Free Tier)
- **S3 & SageMaker** must be available in a supported Region (e.g., `ap-southeast-1`)

### AWS Free Tier Coverage:
This workshop is designed to run entirely within the **AWS Free Tier**:

| Service               | AWS Free Tier                     | Usage in workshop           |
|-----------------------|------------------------------------|-----------------------------|
| **S3**                | 5GB, 20,000 GET/PUT requests       | < 500MB, a few hundred requests |
| **Training Job**      | Not included in Free Tier          | Use small instance + Spot   |
| **Glue**              | 1M ETL requests/month              | 1–2 small jobs, < 10 min/job |
| **Macie**             | 30-day free trial                  | Scan a few hundred MB of data |
| **SageMaker Studio**  | 250 hours t3.medium/month          | ~2–3 hours for model training |

{{% notice info %}}
**Estimated cost**: ~$0 if using Notebook only, < $1–2 USD for small-scale training or minimal data storage.
{{% /notice %}}

### IAM Setup and Access Permissions:

To run the workshop, your **AWS** account needs the following permissions:

#### **Minimum IAM Policy:**
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
}

```
#### **Recommendations:**
- Use AdministratorAccess in lab or learning environments.
- Avoid using the root account for operations.
- Attach IAM Role to SageMaker notebook instance if using Studio or Classic Notebook.

## 2. Working Environment
### Browser:
- **Modern browser**: Latest Chrome, Firefox, Safari
- **JavaScript enabled**
- **Cookies enabled for AWS Console**
- **Pop-up blocker disabled for AWS domains**

### Network:
- **Stable internet connection** ( ≥ 2 Mbps)
- **Access to AWS endpoints** (not blocked by firewall)
- **HTTPS support** (TLS 1.2+)

### Required Tools:
| Tool    | Minimum Requirement                              |
| ------- | ------------------------------------------------ |
| Python  | >= 3.10                                          |
| pip     | Install packages: `pandas`, `boto3`, `sagemaker` |
| Jupyter | Recommended: use SageMaker Studio                |
| AWS CLI | For interacting with S3, Glue, Macie via terminal|
```
pip install pandas boto3 sdv[all] matplotlib seaborn

```
AWS CLI:
```
aws configure
```

## 3. Basic Knowledge
### 3.1 AWS Overview for ML

#### **AWS Services Overview:**
| Service              | Main Role in Workshop                     |
| -------------------- | ----------------------------------------- |
| **Amazon S3**        | Store raw and synthetic datasets (.csv)   |
| **AWS Glue**         | Data preprocessing and standardization    |
| **Amazon Macie**     | Detect sensitive information (PII)        |
| **SageMaker Studio** | Train the synthetic data generation model |
| **IAM**              | Manage AWS resource access control        |


#### **AWS Regions và Availability Zones:**
- **Region**: Geographical area containing multiple data centers
- **AZ**:  Isolated data center within a Region
- **Workshop region**: **ap-southeast-1 (Singapore)**

### 3.2 Basic Machine Learning Knowledge
#### Key Terms:
| Term                        | Description                                                                       |
| --------------------------- | --------------------------------------------------------------------------------- |
| **Synthetic Data**          | Artificially generated data that mimics the statistical distribution of real data |
| **PII**                     | Personally Identifiable Information (emails, account numbers, etc.)               |
| **TVAE/CTGAN**              | Deep learning-based models for generating tabular data                            |
| **Train/Test Split**        | Dividing the dataset into training and testing sets                               |
| **Histogram & Correlation** | Tools for evaluating data similarity                                              |

## 4. Dataset & Folder Structure
### Required Dataset:
- BankSim Dataset from Kaggle (simulated bank transactions)
- Format: .csv
- Download and upload to S3

### Suggested Folder Structure:

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

## 5. Reference Materials

### **AWS Documentation:**
- [AWS Glue Documentation]()
- [Amazon Macie Documentation]()
- [SageMaker Python SDK](https://sagemaker.readthedocs.io/en/stable/)
- [AWS Boto3 S3 Docs](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/s3.html)
- [Amazon SageMaker Documentation]()
- [SDV Documentation]()
- [Pandas Documentation](https://pandas.pydata.org/docs/)


{{% notice tip %}}
If you encounter issues during preparation, check **AWS Support** or **AWS Community Forums** for help.
You don’t need to be an ML expert to join this workshop. As long as you understand the basics and follow the instructions, you’ll be able to build a safe, reusable synthetic data generation pipeline.
{{% /notice %}}






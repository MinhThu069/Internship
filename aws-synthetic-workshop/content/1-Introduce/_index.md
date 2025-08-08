---
title : "Introduce"
date: 2025-07-16
weight : 1 
chapter : false
pre : " <b> 1. </b> "
---

## Workshop Overview

This workshop guides you through building a **Synthetic Data Generation Platform with Privacy Preservation** on AWS.  
You will learn how to:
- Load sample financial data from Amazon S3
- Preprocess and standardize data using AWS Glue
- Detect and remove Personally Identifiable Information (PII) with Amazon Macie
- Train a data generation model (TVAE) using SageMaker Studio Lab
- Evaluate the quality and statistical distribution of the synthetic data

## Why Generate Synthetic Data?

In many organizations, especially **banking and finance**, real datasets often contain sensitive information and cannot be shared with technical teams or external collaborators. Manual anonymization:
- **Consumes significant time** and is error-prone
- **Risks PII leakage** if not done correctly
- **Hinders collaboration** between teams
- **Slows down AI research and development**

### Benefits of Synthetic Data:
- **Removes PII** while retaining the original statistical distribution
- **Safe for sharing** with Data Scientists, Developers, and Researchers
- **Accelerates model development** without violating compliance
- **Cost-efficient** (can be deployed using AWS Free Tier)

## System Architecture

### Data Processing Flow:
CSV → Amazon S3 → AWS Glue → Amazon Macie → SageMaker Studio Lab (TVAE) → Synthetic Data → Statistical Evaluation & Visualization

![Architecture](/images/draw.png)

### Key Services:

#### 1. **Amazon S3**
- Stores raw and synthetic datasets
- Integrates directly with Glue, Macie, and Studio Lab

#### 2. **AWS Glue**
- Data preprocessing: schema standardization, null value handling, column renaming
- Outputs data in CSV or Parquet format

#### 3. **Amazon Macie**
- Scans datasets to detect sensitive information (emails, account numbers, etc.)
- Exports PII detection reports

#### 4. **SageMaker Studio Lab + SDV (TVAE)**
- Trains the data generation model
- Generates synthetic datasets with a distribution similar to the original
- Visualizes and compares histograms and correlation matrices

## Final Objectives

By the end of the workshop, you will:
- Produce synthetic datasets **free of PII**
- Achieve ≥ 90% similarity in distribution between synthetic and original data
- Build a reusable pipeline applicable to multiple datasets
- Be ready to integrate into real-world AI/ML workflows

## Workshop Prerequisites

Before you start, ensure you have:
- An **AWS Account** with access to:
  - Amazon S3
  - AWS Glue
  - Amazon Macie
  - Amazon SageMaker
- For new accounts: **enable AWS Free Tier**
- **Minimum IAM permissions**:
  - `AmazonS3FullAccess` (or bucket-specific access)
  - `AWSGlueServiceRole`
  - `AmazonMacieFullAccess`
  - `AmazonSageMakerFullAccess`

{{% notice info %}}
This workshop is designed to run entirely within the **AWS Free Tier**. The estimated total cost is **< $1 USD** if the instructions are followed correctly.
{{% /notice %}}

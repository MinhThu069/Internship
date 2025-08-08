# AWS SageMaker Feature Store Workshop

[![AWS](https://img.shields.io/badge/AWS-SageMaker-orange.svg)](https://aws.amazon.com/sagemaker/)
[![Hugo](https://img.shields.io/badge/Hugo-Static%20Site-blue.svg)](https://gohugo.io/)
[![License](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

A hands-on workshop to help you **build a reusable Feature Store** using Amazon SageMaker.  
Learn how to create, store, and reuse features across machine learning pipelines — ensuring data consistency, traceability, and efficiency.

---

## 🧠 Problem Background

In most ML projects, **data and feature engineering** takes up more effort than model building. Repetitive processing, schema mismatches, and inconsistent features across training and serving lead to fragile systems.

This workshop focuses on **solving these issues** by using **Amazon SageMaker Feature Store** to build a consistent, auditable, and reproducible feature pipeline.

---

## 🏗️ Architecture Overview

```plaintext
Dataset CSV
    ↓
Amazon S3
    ↓
Pandas Processing (ETL)
    ↓
SageMaker Feature Store (Offline Store)
    ↓
XGBoost Training (SageMaker)
    ↓
Model Artifact Output
```
🎯 Learning Objectives
After completing this workshop, you will be able to:
✅ Upload datasets to Amazon S3
✅ Create a SageMaker Feature Group
✅ Register and store processed features
✅ Train an ML model (XGBoost) using offline features
✅ Track schema, ensure reproducibility, and improve accuracy

📁 Workshop Folder Structure
| Path                       | Description                                           |
| -------------------------- | ----------------------------------------------------- |
| `1_upload_to_s3/`          | Upload raw CSV data to Amazon S3                      |
| `2_feature_engineering/`   | Clean & transform data using Pandas                   |
| `3_register_featurestore/` | Define & ingest features into SageMaker Feature Store |
| `4_training_xgboost/`      | Train an ML model using features from the store       |
| `5_model_artifact/`        | Save and track trained model artifact                 |
| `6_test_feature_quality/`  | Validate and compare feature consistency              |
| `7_report/`                | Diagrams, screenshots, deployment evidence            |
| `8_conclusion/`            | Summary of results and key learnings                  |


🛠️ AWS Services Used
- Amazon S3: Store raw datasets
- Amazon SageMaker Feature Store: Manage offline features
- Amazon SageMaker Training: Model training (XGBoost)
- IAM & KMS: Secure access and encryption

💻 Prerequisites
✅ AWS account with SageMaker permissions
✅ Python 3.8+ with boto3, sagemaker, pandas
✅ Basic knowledge of Jupyter Notebook or script execution
✅ Familiarity with ML basics and XGBoost (optional)

📈 KPIs & Results
- Accuracy: >85%
- Feature pipeline reusable in ≥2 different training tasks
- Reduce processing time on future training runs by ~30%
- All schema and features are now traceable & versioned

🌟 Key Benefits
✅ Business Benefits
- Faster time to deployment
- Reduce duplication in feature pipelines
- Improve traceability and governance

🔧 Technical Improvements
- Feature lineage tracking
- Reproducible pipelines
- Better model stability & scalability

🔭 Long-Term Vision
- Foundation for real-time ML (online store extension)
- Easier collaboration between data & ML teams
- Building block for MLOps best practices

💰 Cost Estimation
This workshop was designed to stay within AWS Free Tier:

| Service                           | Estimated Cost | Notes                             |
| --------------------------------- | -------------- | --------------------------------- |
| Amazon S3                         | \~\$0          | Free Tier 5GB storage             |
| SageMaker Feature Store (Offline) | \~\$0–1        | Low cost for small datasets       |
| SageMaker Training                | \~\$0–2        | Basic instance types for training |
| Total                             | **< \$3 USD**  | Under normal usage limits         |


📚 Workshop Steps
1. Upload Data to S3
2. Feature Engineering with Pandas
3. Register Features to Feature Store
4. Train Model with XGBoost
5. Save and Inspect Model
6. Test and Compare Feature Quality
7. Generate Report and Architecture
8. Conclusion and Evaluation

🔐 Best Practices Implemented
🔒 Least Privilege IAM Roles

🔄 Feature reusability across pipelines

📦 Consistent feature schema validation

📊 Centralized feature store for model governance

🚀 Quick Start (Local)
```
git clone https://github.com/your-username/sagemaker-featurestore-workshop.git
cd sagemaker-featurestore-workshop
pip install -r requirements.txt
```
You can run the scripts in order or explore each notebook in the *.ipynb format.

🤝 Contributing
Pull requests are welcome. For major changes, please open an issue first.

📄 License
This project is licensed under the MIT License. See LICENSE for details.

📬 Contact & Feedback
📧 Email: danghuudung1607@gmail.com

🐛 Issues: GitHub Issues
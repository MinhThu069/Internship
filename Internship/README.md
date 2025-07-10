# 🧠 Nền tảng Tạo Dữ liệu Tổng hợp với Bảo toàn Quyền riêng tư

## 🎯 Mục tiêu dự án
Xây dựng một nền tảng tạo dữ liệu tổng hợp (synthetic data platform) trên AWS, phục vụ cho việc huấn luyện và kiểm thử các mô hình học máy mà **không cần truy cập vào dữ liệu thật**. Hệ thống giúp đảm bảo **quyền riêng tư** nhưng vẫn giữ lại **đặc trưng thống kê** quan trọng của tập dữ liệu ban đầu.

## 📌 Lý do thực hiện
Trong thực tế (đặc biệt tại ngân hàng, tài chính...), việc không được sử dụng dữ liệu thật do chứa PII khiến nhiều dự án AI bị trì hoãn. Dự án này giải quyết bài toán đó bằng cách:
- Tạo dữ liệu sinh ra từ mô hình TVAE.
- Kiểm duyệt PII tự động với Amazon Macie.
- Trực quan hóa và đánh giá dữ liệu đầu ra.

## 🏗️ Kiến trúc hệ thống (AWS-based)
- **Amazon S3**: Lưu trữ dữ liệu gốc và dữ liệu đầu ra.
- **AWS Glue**: Tiền xử lý và chuẩn hóa dữ liệu.
- **Amazon Macie**: Phát hiện thông tin định danh (PII).
- **SageMaker Studio Lab**: Huấn luyện mô hình (TVAE).
- **Jupyter Notebook**: Vẽ biểu đồ, kiểm thử kết quả.

## 🔄 Pipeline tổng quát
[Upload Data]
→ [AWS Glue ETL]
→ [Amazon Macie PII Scan]
→ [Local download]
→ [Train TVAE in Studio Lab]
→ [Synthetic Output]
→ [Evaluation & Visualization]
## 🧪 Mô hình sử dụng
- TVAE – Tabular Variational Autoencoder
- Huấn luyện với `sdv`, `pandas`, `matplotlib`, `seaborn`
- Kiểm thử bằng biểu đồ phân phối, tương quan và regex

## 📈 Kỳ vọng đầu ra
- Dữ liệu sinh có ≥ 90% tương đồng phân phối với dữ liệu thật
- Không chứa bất kỳ thông tin định danh (email, số tài khoản,...)
- Pipeline có thể tái sử dụng cho nhiều tập dữ liệu khác nhau

## ⏱️ Timeline
- Tổng thời gian triển khai: **8 tuần**
- Mỗi tuần xử lý một giai đoạn (đã mô tả trong proposal)

## 💰 Chi phí
Tối ưu để nằm trong AWS Free Tier:
- S3 + Glue + Macie: **~$0.45**
- Studio Lab: **Miễn phí**

## 📦 Cấu trúc thư mục gợi ý
synthetic-data-project/
├── data/ # Dữ liệu gốc
├── etl/ # Script tiền xử lý dữ liệu
├── pii_scan/ # Quét dữ liệu nhạy cảm
├── training/ # TVAE huấn luyện & sinh dữ liệu
├── evaluation/ # Biểu đồ & đánh giá đầu ra
├── output/ # Dữ liệu tổng hợp
├── docs/ # Proposal, sơ đồ, báo cáo
└── README.md
## 👩‍💻 Người thực hiện
- Nguyễn Thị Minh Thư  
- MSSV: 2180603043
- Đại học Công Nghệ TP.HCM  
- Email: nguyenthiminhthu92003@gmail.com  
- GVHD: Trương Thị Minh Châu  

---

## 📚 Tham khảo
- https://sdv.dev  
- https://aws.amazon.com/macie  
- AWS Free Tier Pricing  
- AWS Glue Docs  
- SageMaker Studio Lab
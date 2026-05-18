#  VN30 Market Structure Analysis via PCA

> **Ứng dụng Principal Component Analysis để khám phá cấu trúc ẩn của thị trường chứng khoán Việt Nam**

[![Python](https://img.shields.io/badge/Python-3.13-3776AB?style=flat&logo=python&logoColor=white)](https://www.python.org/)
[![Jupyter](https://img.shields.io/badge/Jupyter-Notebook-F37626?style=flat&logo=jupyter&logoColor=white)](https://jupyter.org/)
[![NumPy](https://img.shields.io/badge/NumPy-2.1-013243?style=flat&logo=numpy&logoColor=white)](https://numpy.org/)
[![Pandas](https://img.shields.io/badge/Pandas-2.2-150458?style=flat&logo=pandas&logoColor=white)](https://pandas.pydata.org/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

---

## Tổng quan dự án

Dự án phân tích **cấu trúc thị trường** của rổ **VN30** (30 cổ phiếu vốn hóa lớn nhất sàn HOSE) thông qua thuật toán **Principal Component Analysis (PCA) xây dựng từ đầu bằng NumPy** — không sử dụng `sklearn`.

### Câu hỏi nghiên cứu

> *"Điều gì thực sự đang dẫn dắt biến động giá cổ phiếu Việt Nam — yếu tố thị trường chung, hay đặc thù từng ngành?"*

### Kết quả nổi bật

| Phát hiện | Kết quả | Ý nghĩa |
|-----------|---------|---------|
| PC1 giải thích phương sai | **~45%** | Yếu tố thị trường chung chiếm ưu thế tuyệt đối |
| PC1 ↔ VN30-Index (Pearson r) | **~0.90** | PC1 ≡ Market Factor (Beta thị trường) |
| Số PC để đạt 80% phương sai | **~10–12** | Thị trường có nhiều chiều thông tin thực sự |
| Tracking với 10 cổ phiếu | **r ≈ 0.97** | Chỉ cần 1/3 rổ để theo dõi thị trường |

---

##  Cấu trúc notebook

```
1. Cài đặt & Import thư viện
2. Thu thập và tiền xử lý dữ liệu VN30 (yfinance)
3. Phân tích khám phá (EDA) — Returns & Correlation Heatmap
4. PCA From Scratch (NumPy)
   ├── 4.1  Zero-mean centering
   ├── 4.2  Ma trận hiệp phương sai
   ├── 4.3  Eigendecomposition (numpy.linalg.eigh)
   └── 4.4  Scree Plot & Biplot
5. Phân tích PC1 — Market Factor
   ├── 5.1  Trọng số cổ phiếu trong PC1
   └── 5.2  So sánh PC1 vs VN30-Index (time series + scatter)
6. Phân tích PC2 & PC3 — Sector Factors
7. Ứng dụng: Xây dựng danh mục PC1-Tracking
8. Kết luận & Hướng phát triển
```

---

##  Cài đặt

### Yêu cầu hệ thống

- Python ≥ 3.10
- Jupyter Notebook / JupyterLab / VS Code

### Cài đặt dependencies

```bash
git clone https://github.com/<your-username>/VN30_PCA_Market_Structure.git
cd VN30_PCA_Market_Structure
pip install -r requirements.txt
```

### Chạy notebook

```bash
jupyter notebook VN30_PCA_Market_Structure.ipynb
```

> **Lưu ý:** Cell đầu tiên tải dữ liệu trực tiếp từ Yahoo Finance (`yfinance`), cần kết nối internet khi chạy lần đầu.

---

##  Dữ liệu

| Thuộc tính | Chi tiết |
|-----------|---------|
| **Nguồn** | Yahoo Finance (`yfinance`) — đuôi `.VN` cho sàn HOSE |
| **Phạm vi** | 30 cổ phiếu rổ VN30 (danh sách cập nhật 04/2025) |
| **Giai đoạn** | 04/2024 – 04/2025 (~248 phiên giao dịch) |
| **Benchmark** | ETF `E1VFVN30.VN` (VinaCapital) làm proxy VN30-Index |
| **Biến phân tích** | Daily returns từ giá đóng cửa điều chỉnh (`auto_adjust=True`) |

### Danh mục VN30 (04/2025)

| Ngành | Mã cổ phiếu |
|-------|------------|
| Ngân hàng (14) | ACB, BID, CTG, HDB, LPB, MBB, SHB, SSB, STB, TCB, TPB, VCB, VIB, VPB |
| Bất động sản (3) | BCM, VHM, VIC |
| Hàng tiêu dùng (4) | MSN, MWG, SAB, VNM |
| Năng lượng (3) | GAS, PLX, POW |
| Nguyên liệu (2) | GVR, HPG |
| Chứng khoán (1) | SSI |
| Công nghệ (1) | FPT |
| Bảo hiểm (1) | BVH |
| Hàng không (1) | VJC |

---

##  Phương pháp

### PCA From Scratch

Thay vì dùng `sklearn.decomposition.PCA`, thuật toán được xây dựng thuần NumPy qua 4 bước:

```
Bước 1: Zero-mean centering       →  R̃ = R − R̄
Bước 2: Ma trận hiệp phương sai   →  Σ = R̃ᵀR̃ / (T-1)
Bước 3: Eigendecomposition        →  Σvₖ = λₖvₖ  (numpy.linalg.eigh)
Bước 4: PC scores                 →  zₖ = R̃ · vₖ
```

Sử dụng `numpy.linalg.eigh` (tối ưu cho ma trận đối xứng thực, PSD) thay vì `eig` tổng quát để đảm bảo độ chính xác số học.

---

##  Kết quả chính

### PC1 ≡ Market Factor
Tất cả 30 trọng số PC1 đều **dương** và PC1 đạt **Pearson r ≈ 0.90** với VN30-Index — thuật toán không giám sát tự hội tụ về yếu tố Beta thị trường.

### Cấu trúc One-Factor-Dominated
PC1 chiếm ~45% tổng phương sai, đặc trưng của **frontier market** theo phân loại MSCI: rủi ro hệ thống lấn át rủi ro đặc thù.

### Index Tracking hiệu quả
Chỉ 10 cổ phiếu (theo trọng số PC1) đủ để xây dựng danh mục tracking VN30 với **r > 0.97** — nền tảng cho ETF mini hoặc danh mục tham chiếu chi phí thấp.

---

##  Hướng phát triển

- **Rolling PCA** (window = 60 phiên) → theo dõi thay đổi cấu trúc thị trường theo thời gian
- **Factor Model**: `Returnᵢ = α + β₁·PC1 + β₂·PC2 + ε` → phân tách systematic vs idiosyncratic risk
- **Mở rộng sang VNINDEX** → so sánh cấu trúc VN30 vs thị trường rộng (~700 mã)
- **Tích hợp dữ liệu macro** (CPI, tỷ giá, lãi suất) → giải thích nguồn gốc kinh tế của từng PC
- **Backtesting momentum** trên PC1 score → đánh giá giá trị đầu tư thực tiễn

---

##  Tech Stack

| Thư viện | Phiên bản | Mục đích |
|---------|-----------|---------|
| `numpy` | 2.1.x | PCA from scratch, linear algebra |
| `pandas` | 2.2.x | Data manipulation |
| `scipy` | 1.14.x | Statistical tests (Pearson, Spearman) |
| `yfinance` | 1.3.x | Market data download |
| `matplotlib` | 3.9.x | Visualization |
| `seaborn` | 0.13.x | Statistical plots |
| `jinja2` | 3.1.x | DataFrame styled output |

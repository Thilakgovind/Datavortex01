# 🌀 DATA VORTEX Round 1 (Aaruush '26)
## Social Engine Telemetry Restoration & Exploratory Data Analysis (EDA)

[![Python 3.10+](https://img.shields.io/badge/Python-3.10%2B-blue.svg)](https://www.python.org/)
[![Status](https://img.shields.io/badge/Status-Restored%20%26%20Verified-success.svg)]()
[![License](https://img.shields.io/badge/License-MIT-green.svg)]()

> **Competition:** DATA VORTEX (Aaruush '26) — Round 1: Data Intake Restoration  
> **Team:** Data Vortex Forensic Analytics Team  
> **Repository:** [Thilakgovind/datavortex](https://github.com/Thilakgovind/datavortex)

---

## 📌 1. Project Overview

During upstream telemetry extraction from the distributed **Social Engine** platform, a catastrophic ingestion failure corrupted interaction events. The raw telemetry corpus contained duplicated records, impossible negative metrics, unstandardized timestamps across three distinct conventions, and character encoding debris (Mojibake and raw HTML injections).

This repository contains the complete, reproducible forensic restoration pipeline that transforms raw, defective telemetry into an auditable, analysis-ready dataset of **12,000 unique records × 14 columns**, accompanied by deep demographic Exploratory Data Analysis.

---

## 🛠️ 2. Summary of Defects & Restoration Pipeline

| Stage / Vector | Raw Defect | Verified Defect Count | Restoration Treatment | Final Verified State |
|---|---|---|---|---|
| **1. Deduplication** | Ingestion pipeline logged duplicate packets | **360 duplicate rows** (12,360 total) | Deduplication on primary key `post_id` | Exactly **12,000 unique records** |
| **2. Missing Platform** | Unrecorded source networks (NaN & `'NULL'`) | **1,784 records** | Standardized to `'Unknown'` to preserve volume without false attribution | 1,784 `'Unknown'`; 0 missing |
| **3. Negative Likes** | Telemetry sign-inversion faults (e.g., `-4,812`) | **525 records** | Enforced domain invariants via `abs()` | Minimum likes: **0**; 0 negative values |
| **4. Missing Likes** | Telemetry drops / unrecorded interactions | **1,858 records** | Coerced to `0` with explicit zero-interaction telemetry rationale | 1,815 zero-like entries |
| **5. Timestamp Normalization** | Mixed Unix epochs, ISO 8601, and DD-MM-YYYY | **12,000 records** | Cascading parser standardizing to UTC ISO string format | **100% (12,000/12,000)** UTC ISO datetimes |
| **6. Mojibake Cleaning** | Encoding corruption at string boundaries | **316 records** | Regex stripping targeted at string ends (`[\u00e9\u00a9\ufffd]+$`) | Terminal debris removed, internal text intact |
| **7. HTML Artifacts** | Trailing ampersands (`&amp;`, `&`) & tags | **341 &amp; / 663 tags** | Regex stripping + `html.unescape()` | Clean, human-readable text |
| **8. Demographic Join** | Link posts to user master table | **1,500 distinct users** | Left join on `user_id` against `users.csv` | **100% match (0 orphaned posts)** |
| **9. Feature Engineering** | Derive engagement metrics | — | `total_engagement = likes + shares + comments`<br>`engagement_rate = total_engagement / follower_count` | Computed for all 12,000 records |

---

## 📊 3. Key Analytical Insights (EDA)

1. **Strict Platform Parity:** Content volume is distributed virtually uniformly across networks: Facebook (2,074), YouTube (2,073), Twitter (2,049), Reddit (2,031), Instagram (1,989), and restored Unknown posts (1,784). Mean interactions per post are uniformly balanced between 5,468 and 5,532.
2. **Language Performance Disparity:** German (`de`) and Spanish (`es`) creator accounts produce the highest follower engagement rates (**61.6%** and **60.5%** respectively), outperforming English creators (**25.5%**) by more than **2.4×**.
3. **The "Scale Paradox":** Follower count has virtually no correlation with raw interaction volume ($r = -0.0109$), but has a statistically significant negative correlation with follower engagement rate (**$r = -0.3139$**). Accounts with large followings suffer acute engagement dilution, demonstrating that distributed micro-influencer activations deliver superior conversion efficiency.

---

## 📂 4. Repository Structure

```
├── Data_Vortex_Phase1_Pipeline.ipynb        # Complete, executed notebook with cleaning & EDA
├── cleaned_social_posts.csv                 # Restored dataset (12,000 rows × 14 columns)
├── cleaned_social_posts.json                # Restored dataset in JSON format
├── Round_1_Data_Intake_Restoration_EDA_Report.pdf  # Executive printable report with charts
├── Round_1_Data_Intake_Restoration_EDA_Report.md   # Complete report in Markdown
├── figures/                                 # High-resolution generated EDA charts
│   ├── fig1_platform_distribution.png
│   ├── fig2_avg_engagement_platform.png
│   ├── fig3_engagement_by_language.png
│   └── fig4_scale_paradox.png
├── Social_Engine_Posts_Corrupted.csv        # Original raw corrupted telemetry
├── users.csv                                # Master user demographic table (1,500 users)
└── README.md                                # Repository documentation
```

---

## 🚀 5. How to Reproduce

1. **Clone the repository:**
   ```bash
   git clone https://github.com/Thilakgovind/datavortex.git
   cd datavortex
   ```

2. **Install dependencies:**
   ```bash
   pip install pandas numpy matplotlib
   ```

3. **Run the restoration pipeline & EDA:**
   Open and run all cells in `Data_Vortex_Phase1_Pipeline.ipynb`, or launch with Jupyter / VS Code / Antigravity IDE.

# Data Vortex (Aaruush '26) - Round 1
## Data Intake Restoration & Exploratory Data Analysis

- **Event:** Data Vortex (Aaruush '26) - Round 1: Data Intake Restoration
- **Team:** Thilak Govind & Team
- **Repository:** [https://github.com/Thilakgovind/Datavortex01](https://github.com/Thilakgovind/Datavortex01)

---

## 1. Overview

In Round 1 of Data Vortex, we received a raw social media interaction dataset (`Social_Engine_Posts_Corrupted.csv`) containing 12,360 rows, alongside user demographic data (`users.csv`) containing 1,500 profiles. The raw dataset contained typical upstream logging anomalies: duplicated submissions, negative interaction numbers, unstandardized timestamp conventions, missing values, and corrupted character encodings.

Our objective was to restore the intake pipeline by:
1. Identifying and correcting all data anomalies systematically without dropping valid interaction records.
2. Standardizing timestamps into UTC ISO 8601 format and sanitizing text strings.
3. Joining posts with user demographic attributes.
4. Engineering engagement metrics and conducting exploratory analysis to evaluate user behavior.

---

## 2. Deliverables Checklist

| Required Component | Repository File | Status |
|---|---|---|
| **Cleaned Dataset (CSV)** | `cleaned_social_posts.csv` | Completed (12,000 rows, 14 columns) |
| **Cleaned Dataset (JSON)** | `cleaned_social_posts.json` | Completed (12,000 records) |
| **EDA Report (Markdown)** | `EDA_Report.md` | Completed (with embedded figures) |
| **EDA Report (PDF)** | `EDA_Report.pdf` | Completed |
| **Code Notebook** | `Data_Vortex_Phase1_Pipeline.ipynb` | Completed (runnable end-to-end) |
| **Raw Datasets** | `Social_Engine_Posts_Corrupted.csv`, `users.csv` | Included for complete reproducibility |

---

## 3. Data Cleaning and Restoration Pipeline

The table below outlines the defects identified in `Social_Engine_Posts_Corrupted.csv` and the restoration logic applied:

| Issue Found | Raw Count | Resolution Strategy | Resulting Clean State |
|---|---|---|---|
| **Duplicate Records** | 360 duplicate rows | Deduplicated on primary key `post_id` | Exactly 12,000 unique records |
| **Missing Platforms** | 1,784 missing / null values | Labeled as `'Unknown'` category | Kept all posts; prevents platform skew |
| **Negative Likes** | 525 negative values (< 0) | Converted with `abs()` | Min likes = 0; real magnitude preserved |
| **Missing Likes** | 1,858 null entries | Imputed with `0` | Treated as unrecorded interactions |
| **Mixed Timestamps** | 12,000 mixed formats | Parsed Unix Epoch, ISO 8601, and DD-MM to UTC | 100% unified `YYYY-MM-DD HH:MM:SS+00:00` |
| **Broken Text (Mojibake)** | 316 trailing byte artifacts | Stripped terminal garbage bytes using regex (`[^\x00-\x7F]+$`) | Clean text; legitimate accents preserved |
| **HTML Tags & Ampersands** | 341 `&amp;` / 663 HTML tags | Stripped tags and unescaped HTML entities | Clean plain text |
| **User Profile Join** | 1,500 user profiles | Left join on `user_id` against `users.csv` | 100% match rate across all 12,000 posts |
| **Feature Engineering** | Engagement metrics | Derived total interactions and engagement rate | `total_engagement = likes + shares + comments`<br>`engagement_rate = total_engagement / follower_count` |

---

## 4. Key Engineering Decisions

- **Missing platforms labeled as 'Unknown':** Dropping records with missing platforms would delete 1,784 posts (nearly 15% of the dataset). Imputing a platform would introduce false attribution. Using `'Unknown'` retains valid interaction counts while keeping platform comparisons unbiased.
- **Negative likes restored with absolute value:** Values like `-4,812` indicate a sign-bit flip during logging. Using `abs()` restores the genuine interaction count. Missing likes were filled with `0` to reflect no recorded likes without fabricating values.
- **Terminal regex text cleaning:** Encoding artifacts were located at the end of post strings. Anchoring regex replacement strictly to the end of the string (`$`) cleanly stripped corrupted byte sequences without removing valid accents or international characters within the text.

---

## 5. Summary of EDA Findings

1. **Platform Distribution:**  
   Post volume is balanced across major platforms: Facebook (2,074), YouTube (2,073), Twitter (2,049), Reddit (2,031), Instagram (1,989), alongside 1,784 Unknown posts. Average interactions per post are uniform across platforms (~5,468 to 5,532).
2. **Language Performance:**  
   German (61.6%) and Spanish (60.5%) creators generate over 2.4x higher engagement rates per follower compared to English accounts (25.5%), highlighting higher audience loyalty among regional creator communities.
3. **The Scale Paradox:**  
   - Follower count vs. raw interactions: $r = -0.0109$ (no correlation).
   - Follower count vs. engagement rate: $r = -0.3139$ (moderate inverse correlation).  
   As accounts grow, engagement rate drops due to audience dilution. Smaller accounts engage their audience more effectively per follower.

For in-depth analysis and charts, see [EDA_Report.md](EDA_Report.md).

---

## 6. Repository Structure

```
├── .gitignore                          # Git ignore rules for temporary files
├── README.md                           # Main repository documentation
├── EDA_Report.md                       # Comprehensive EDA report with inline charts
├── EDA_Report.pdf                      # Printable PDF report
├── Data_Vortex_Phase1_Pipeline.ipynb   # Complete cleaning pipeline, comments & EDA notebook
├── Social_Engine_Posts_Corrupted.csv   # Raw input dataset (12,360 rows)
├── users.csv                           # User demographic metadata (1,500 profiles)
├── cleaned_social_posts.csv            # Cleaned final dataset (12,000 rows x 14 columns)
├── cleaned_social_posts.json           # Cleaned final dataset in JSON format
└── figures/                            # Exported high-resolution visualization charts
    ├── fig1_platform_distribution.png
    ├── fig2_avg_engagement_platform.png
    ├── fig3_engagement_by_language.png
    └── fig4_scale_paradox.png
```

---

## 7. How to Run and Reproduce

### Prerequisites
Python 3.9+ with the following packages:
```bash
pip install pandas numpy matplotlib
```

### Execution Steps
1. Clone the repository:
   ```bash
   git clone https://github.com/Thilakgovind/Datavortex01.git
   cd Datavortex01
   ```

2. Run the notebook:
   Open `Data_Vortex_Phase1_Pipeline.ipynb` in VS Code or JupyterLab and execute all cells. The notebook will:
   - Load raw datasets
   - Apply cleaning and validation steps
   - Export `cleaned_social_posts.csv` and `cleaned_social_posts.json`
   - Generate and save all EDA figures into `figures/`

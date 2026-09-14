# Data Vortex (Aaruush '26) - Round 1: Data Intake Restoration

- **Team:** Thilak Govind & Team
- **Repository:** [https://github.com/Thilakgovind/Datavortex01](https://github.com/Thilakgovind/Datavortex01)

---

## Deliverables

This repository contains the three deliverables required for Phase 1:

1. **Cleaned Dataset:** [`cleaned_social_posts.csv`](cleaned_social_posts.csv)  
   Restored and validated social media interaction dataset (12,000 unique records, 14 columns).
2. **Exploratory Data Analysis (EDA) Report:** [`EDA_Report.pdf`](EDA_Report.pdf)  
   Comprehensive report covering data issues, engineering decisions, and behavioral findings with visualizations.
3. **Code Notebook:** [`Data_Vortex_Phase1_Pipeline.ipynb`](Data_Vortex_Phase1_Pipeline.ipynb)  
   Complete end-to-end Jupyter notebook containing:
   - **Cleaning Code:** Deduplication, missing value imputation, sign-bit correction on likes, UTC timestamp parsing, and Mojibake/HTML cleaning.
   - **Documentation:** Markdown cells detailing engineering trade-offs and data definitions.
   - **Reproducible Workflow:** Sequential, documented cells reproducing the cleaned outputs and visual analysis.

---

## Data Cleaning & Restoration Summary

| Defect Identified | Raw Occurrence | Resolution Strategy | Clean Result |
|---|---|---|---|
| Duplicate Records | 360 duplicate rows | Deduplicated on primary key `post_id` | Exactly 12,000 unique records |
| Missing Platforms | 1,784 missing / null values | Labeled as `'Unknown'` category | Zero rows lost; avoids platform bias |
| Negative Likes | 525 negative values (< 0) | Converted with `abs()` (sign-bit flip recovery) | Min likes = 0; preserved logged magnitude |
| Missing Likes | 1,858 null values | Imputed with `0` | Honest representation of unlogged interactions |
| Mixed Timestamps | 12,000 mixed records | Parsed Unix Epoch, ISO 8601, and DD-MM to UTC | Unified ISO UTC (`YYYY-MM-DD HH:MM:SS+00:00`) |
| Broken Text (Mojibake) | 316 trailing byte artifacts | Stripped terminal garbage bytes via regex (`[^\x00-\x7F]+$`) | Clean text; legitimate accents preserved |
| HTML Tags & Entities | 341 `&amp;` / 663 HTML tags | Stripped tags and unescaped HTML entities | Clean text content |
| User Profile Join | 1,500 user profiles | Left join on `user_id` against `users.csv` | 100% match rate across all 12,000 records |
| Feature Engineering | Interaction metrics | Calculated `total_engagement` and `engagement_rate` | Ready for downstream analytics |

---

## Key Engineering Decisions

- **Missing platforms labeled as 'Unknown':** Dropping records with missing platforms would discard 1,784 posts (14.9% of the dataset). Imputing the most frequent platform would create false associations. Using `'Unknown'` keeps all records intact without distorting platform comparisons.
- **Negative likes restored with absolute value:** Values like `-4,812` indicate a sign-bit flip during logging. Using `abs()` restores the genuine interaction count. Missing values are filled with `0` rather than fabricated averages.
- **Targeted terminal regex cleaning:** Character encoding errors occurred strictly at the end of post strings. Anchoring regex replacement to the end of the string (`$`) removes corrupted bytes without damaging valid accents or non-ASCII characters within the text.

---

## Reproducible Workflow

### Requirements
- Python 3.9+
- `pandas`, `numpy`, `matplotlib`

```bash
pip install pandas numpy matplotlib
```

### Steps to Run
1. Clone the repository:
   ```bash
   git clone https://github.com/Thilakgovind/Datavortex01.git
   cd Datavortex01
   ```
2. Place the raw input files (`Social_Engine_Posts_Corrupted.csv` and `users.csv`) in the directory.
3. Open and run all cells in `Data_Vortex_Phase1_Pipeline.ipynb`.
4. The notebook will automatically clean the data, validate all integrity rules, render EDA figures, and output `cleaned_social_posts.csv`.

# Data Vortex (Aaruush '26) - Round 1: Data Intake Restoration

- **Team:** Thilak Govind & Team
- **Repository:** [https://github.com/Thilakgovind/Datavortex01](https://github.com/Thilakgovind/Datavortex01)
- **Primary Notebook:** [Data_Vortex_Phase1_Pipeline.ipynb](Data_Vortex_Phase1_Pipeline.ipynb)

---

## 1. Overview

This repository contains the data cleaning code, technical documentation, and reproducible workflow for Round 1 of Data Vortex (Aaruush '26). The project focuses on restoring a corrupted intake pipeline processing social media interactions (`Social_Engine_Posts_Corrupted.csv`, 12,360 rows) and integrating user profile data (`users.csv`, 1,500 profiles).

---

## 2. Data Cleaning Code & Pipeline

All cleaning, validation, and transformation logic is implemented in [`Data_Vortex_Phase1_Pipeline.ipynb`](Data_Vortex_Phase1_Pipeline.ipynb).

### Summary of Issues Handled

| Defect Identified | Raw Occurrence | Resolution Strategy | Clean State |
|---|---|---|---|
| Duplicate Records | 360 duplicate rows | Deduplicated on `post_id` | Exactly 12,000 unique records |
| Missing Platforms | 1,784 null/blank rows | Imputed as `'Unknown'` category | Zero rows dropped; avoids platform bias |
| Negative Likes | 525 negative values (< 0) | Converted via `abs()` (sign-bit flip recovery) | Min likes = 0; preserved interaction scale |
| Missing Likes | 1,858 null values | Imputed with `0` | Honest representation of unlogged activity |
| Mixed Timestamps | 12,000 mixed records | Parsed Unix epoch, ISO 8601, and DD-MM to UTC | Unified ISO UTC (`YYYY-MM-DD HH:MM:SS+00:00`) |
| Broken Text (Mojibake) | 316 trailing byte artifacts | Removed terminal garbage bytes via regex (`[^\x00-\x7F]+$`) | Clean text; legitimate accents preserved |
| HTML Tags & Entities | 341 `&amp;` / 663 HTML tags | Stripped tags and unescaped HTML entities | Clean text content |
| User Profile Join | 1,500 user profiles | Left join on `user_id` against `users.csv` | 100% match rate across all 12,000 records |
| Feature Engineering | Interaction metrics | Calculated `total_engagement` and `engagement_rate` | Ready for downstream analytics |

### Key Engineering Decisions

1. **Retaining Missing Platforms as `'Unknown'`:** Dropping records with missing platforms would discard 1,784 posts (14.9% of the dataset). Imputing the mode would artificially distort platform statistics. Keeping them as `'Unknown'` preserves complete user activity records while keeping platform analytics unbiased.
2. **Recovering Negative Likes with `abs()`:** Likes cannot be negative in real-world logging systems. Values like `-4,812` typically indicate a sign-bit flip. Taking the absolute value preserves the actual interaction magnitude. Missing values are filled with `0` rather than imputed averages.
3. **Targeted End-of-String Regex:** Character encoding corruptions occurred strictly at the end of post strings. Anchoring regex replacements to the end of the string (`$`) cleanly removes corrupted bytes without damaging valid non-ASCII characters or accents within the text body.

---

## 3. Documentation of Outputs

The pipeline produces a standardized dataset containing 12,000 rows across 14 columns:
- `post_id`: Unique integer ID (primary key).
- `user_id`: Foreign key referencing user demographics.
- `platform`: Categorical (`Facebook`, `YouTube`, `Twitter`, `Reddit`, `Instagram`, `Unknown`).
- `post_text`: Cleaned text body free of HTML entities and encoding artifacts.
- `timestamp`: Standardized UTC timestamp string.
- `likes`, `shares`, `comments`: Non-negative interaction counts.
- `age`, `country`, `language`, `follower_count`: Joined user attributes.
- `total_engagement`: Computed interaction sum (`likes + shares + comments`).
- `engagement_rate`: Derived ratio (`total_engagement / follower_count`).

---

## 4. Reproducible Workflow

### Environment Requirements
- Python 3.9+
- `pandas`
- `numpy`
- `matplotlib`

Install dependencies:
```bash
pip install pandas numpy matplotlib
```

### Execution
1. Clone the repository:
   ```bash
   git clone https://github.com/Thilakgovind/Datavortex01.git
   cd Datavortex01
   ```
2. Place `Social_Engine_Posts_Corrupted.csv` and `users.csv` in the root folder.
3. Open `Data_Vortex_Phase1_Pipeline.ipynb` in VS Code or JupyterLab.
4. Execute all cells from top to bottom. The notebook will process the data, perform validation checks, generate EDA plots, and export `cleaned_social_posts.csv` and `cleaned_social_posts.json`.

---

## 5. Repository Structure

```
├── .gitignore                          # Git ignore configuration
├── README.md                           # Documentation & reproducible workflow
└── Data_Vortex_Phase1_Pipeline.ipynb   # Complete cleaning pipeline, code & EDA
```

# DATA VORTEX Round 1 (Aaruush '26) — Data Intake Restoration & EDA Report

**Project Title:** Social Engine Telemetry Restoration & Audience Behavioral Analysis  
**Repository:** [vs03-oss/Data-Vortex-Round-1](https://github.com/vs03-oss/Data-Vortex-Round-1)  
**Deliverables:** Cleaned Dataset (`.csv`, `.json`), Runnable Jupyter Notebook (`Data_Vortex_Phase1_Pipeline.ipynb`), Final Report  
**Date:** September 14, 2026  

---

## 1. Executive Summary

In Round 1 of the **DATA VORTEX** competition, an upstream telemetry extraction failure simulated across the **"Social Engine"** distributed platform generated severe corruption across raw post records. The incoming dataset (`Social_Engine_Posts_Corrupted.csv`) contained **12,360 raw records**, marred by duplication, invalid negative numbers, missing attributes, heterogeneous timestamp formats, and character encoding anomalies (Mojibake and raw HTML artifacts).

Our intake restoration pipeline implemented an auditable, six-stage forensic cleanup:
1. **Deduplication:** Resolved 360 redundant events to isolate exactly **12,000 unique posts**.
2. **Missing Platform Normalization:** Preserved event integrity by designating 1,784 missing platforms as `'Unknown'` rather than fabricating platform origins.
3. **Engagement Telemetry Rectification:** Inverted 525 negative interaction counts using absolute value magnitudes and initialized unrecorded metrics to 0 with explicit telemetry rationale.
4. **Text Content Sanitization:** Stripped terminal Mojibake artifacts, decoded HTML entities, eliminated injected markup tags, and normalized irregular whitespace.
5. **Timestamp Standardization:** Harmonized Unix epoch offsets, ISO 8601 strings, and day-first date stamps into a unified UTC ISO representation (`YYYY-MM-DD HH:MM:SS+00:00`).
6. **Master Demographic Integration:** Seamlessly merged the restored corpus with the **1,500 user profiles** from `users.csv` (100% key match), deriving `total_engagement` and `engagement_rate`.

Downstream Exploratory Data Analysis (EDA) revealed remarkable cross-platform volume parity, localized language-driven performance peaks (German and Spanish creators achieving ~60% average engagement rates), and uncovered the **"Scale Paradox"** ($r = -0.314$), where follower scale exhibits an inverse relationship with audience engagement rates.

---

## 2. Ingestion & Corruption Inventory

Comprehensive audit of `Social_Engine_Posts_Corrupted.csv` vs. verified restoration outcomes:

| Defect / Feature | Raw State Description | Verified Defect Count | Restoration Strategy | Cleaned State Verification |
|---|---|---|---|---|
| **Duplicate Records** | Redundant telemetry packets with identical `post_id` | **360 duplicate rows** (12,360 total rows) | Deduplication on primary key `post_id` (`drop_duplicates`) | Exactly **12,000 unique records** |
| **Missing Platforms** | Unrecorded source network attributes (NaN & literal `'NULL'`) | **1,784 missing records** | Imputed as `'Unknown'` to retain event footprint without bias | 1,784 tagged as `'Unknown'`; 0 missing |
| **Negative Likes** | Impossible negative integers (e.g., `-4,812`) | **525 records** | Domain invariant enforcement via `abs()` (sign rectification) | Minimum like count: **0**; 0 negative values |
| **Unrecorded Likes** | Missing / null interaction counts | **1,858 records** | Initialized to `0` with documented zero-interaction telemetry rationale | 1,815 zero-like entries |
| **Timestamp Diversity** | Mixed Unix epochs, ISO 8601, DD-MM-YYYY strings | **12,000 records** | Multi-branch parser coercing to UTC standard format | **100% (12,000/12,000)** formatted as UTC ISO |
| **Mojibake Artifacts** | Broken UTF-8 byte sequences at string terminations (`\u00e9`, `\u00a9`, `\ufffd`) | **316 records** | Targeted boundary regex stripping termination noise | Clean strings with preserved internal characters |
| **Trailing Ampersands** | Trailing HTML entity fragments (`&amp;`, `&`) | **341 records** | Terminal ampersand stripping regex | Zero trailing ampersands |
| **HTML Tag Injections** | Embedded markup elements (`<div>`, `<br>`, etc.) | **663 records** | Tag stripping regex `<[^>]+>` + `html.unescape` | Pure text content |
| **User Profile Join** | Post corpus linked to `users.csv` on `user_id` | **1,500 distinct users** | Left join preserving all posts | **100% match (0 orphaned posts)**; 14 total columns |

---

## 3. Data Restoration Methodology & Defensibility

### 3.1 Platform Handling: Preserving Signal via `'Unknown'`
Rather than dropping records with missing platforms (which would discard over 14.8% of our data corpus) or arbitrarily imputing the modal platform, missing values were codified as `'Unknown'`. This explicitly preserves the post volume and downstream engagement interactions without introducing artificial platform distribution distortion.

### 3.2 Engagement Rectification & Defensible Assumptions
- **Negative Likes:** In standard social media telemetry architectures, interaction counters are strictly non-negative. Negative values occur during telemetry sign-inversion faults. Taking the absolute magnitude (`abs()`) restores the underlying interaction count while strictly adhering to domain invariants.
- **Null Interactions:** Missing interaction values were coerced to `0`. We document this as an explicit engineering operational assumption: *unrecorded telemetry pings are assumed to reflect zero logged audience interactions at the intake timestamp*.

### 3.3 Text Normalization & Encoding Defense
Mojibake artifacts in this dataset consistently localized at string boundaries as trailing debris (`#SummerÃ©`, `#Tech&amp;`). By applying targeted regular expressions constrained to terminal positions (`$`), we successfully eliminated encoding debris while safeguarding legitimate internal characters (such as European accentuation or math symbols). Standard `html.unescape()` restored valid punctuation.

### 3.4 Multi-Format Timestamp Normalization
A robust cascading parser evaluated each timestamp:
1. Digits-only values converted via Unix epoch seconds (`unit='s', utc=True`).
2. Standard ISO strings parsed directly with UTC enforcement.
3. European day-first strings parsed with `dayfirst=True` to prevent day/month transposition.

---

## 4. Exploratory Data Analysis (EDA) & Key Findings

### 4.1 Platform Parity Analysis
The restored corpus reveals a nearly uniform volume distribution across major social channels:
- **Facebook:** 2,074 posts (17.28%)
- **YouTube:** 2,073 posts (17.28%)
- **Twitter:** 2,049 posts (17.08%)
- **Reddit:** 2,031 posts (16.92%)
- **Instagram:** 1,989 posts (16.58%)
- **Unknown (Restored):** 1,784 posts (14.87%)

*Insight:* The strict volume balance across recognized networks points to synthetic or load-balanced intake telemetry simulation.

### 4.2 Cross-Platform Engagement Performance
Average total engagement per post across platforms:
- **Instagram:** 5,532.4 interactions
- **YouTube:** 5,516.1 interactions
- **Facebook:** 5,498.8 interactions
- **Twitter:** 5,482.3 interactions
- **Reddit:** 5,468.9 interactions
- **Unknown:** 5,502.1 interactions

*Insight:* Aggregate engagement across all platforms hovers within a narrow band (5,468 – 5,532), indicating uniform baseline content exposure algorithms across the simulated platforms.

### 4.3 Demographic Disparity: Language Engagement Conversion
When computing the engagement rate per follower ($\text{Engagement Rate} = \frac{\text{Total Engagement}}{\text{Follower Count}}$), notable geographic and linguistic variances emerge:
- **German (`de`):** **61.6%** average engagement rate
- **Spanish (`es`):** **60.5%** average engagement rate
- **French (`fr`):** 43.8% average engagement rate
- **English (`en`):** **25.5%** average engagement rate

*Strategic Takeaway:* While English accounts for substantial post volume, German- and Spanish-speaking creator accounts drive over **2.4× greater follower engagement conversion**. Marketing spend and influencer partnerships yield vastly superior ROI when allocated to regional European creators.

### 4.4 The Scale Paradox (Follower Count vs. Engagement)
Statistical correlation analysis demonstrates the **"Scale Paradox"**:
- **Correlation (Followers vs. Raw Volume):** $r = -0.0109$ (effectively zero correlation).
- **Correlation (Followers vs. Engagement Rate):** $r = -0.3139$ (statistically significant negative correlation).

*Mathematical Insight:* Accounts with vast follower bases experience acute audience dilution, resulting in diminishing returns on engagement rates. Micro- and mid-tier accounts achieve substantially higher interaction density, affirming the strategic superiority of distributed micro-creator campaigns over single mega-influencer activations.

---

## 5. Verification & Submission Package

The final submission workspace contains all required artifacts:
- **`cleaned_social_posts.csv`:** 12,000 rows × 14 columns, verified UTC timestamps, no negative metrics, fully merged.
- **`cleaned_social_posts.json`:** Valid JSON record array of the cleaned corpus.
- **`Data_Vortex_Phase1_Pipeline.ipynb`:** Fully executable, self-contained Jupyter notebook with executed cells, detailed documentation, and inline visualizations.
- **`Round_1_Data_Intake_Restoration_EDA_Report.pdf`:** Printable executive report with embedded figures and tables.
- **`README.md`:** Complete repository manual with reproduction instructions.

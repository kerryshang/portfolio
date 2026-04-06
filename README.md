# Google Ads Campaign Performance Analysis
### Brave ISA — Data Analytics & Data Engineering Program

> End-to-end campaign analysis: from raw Google Ads data extraction → SQL-based transformation → BI reporting & strategic recommendations.

---

## 📋 Project Overview

**Client:** Brave ISA (Education/Training Provider)  
**Product:** Data Analytics (DA) & Data Engineering (DE) Bootcamp Programs  
**Campaigns:** 3 national generic search campaigns (FY23–FY24)  
**Currency:** CAD  
**Role:** Business Analyst — Planned keywords & ad content, extracted and transformed campaign data, built analytics-ready datasets, and delivered performance insights.

This project is part of a broader initiative where I **extracted, cleaned, and transformed campaign data across multiple sources using SQL**, building unified analytics-ready datasets to support BI reporting and business analysis across 5+ initiatives.

---

## 🔧 Tools & Skills Used

| Category | Tools |
|----------|-------|
| **Data Extraction** | Google Ads API, Google Ads Editor |
| **Data Transformation** | SQL (CTEs, window functions, aggregations), Excel Power Query |
| **Analysis** | Pivot Tables, calculated fields, segmentation analysis |
| **Visualization** | Excel Charts, HTML/JS Interactive Dashboard |
| **Reporting** | Excel (formatted reports), Markdown documentation |

---

## 📊 Key Metrics at a Glance

| Metric | Value |
|--------|-------|
| Total Impressions | 49,593 |
| Total Clicks | 967 |
| Click-Through Rate (CTR) | 1.95% |
| Total Spend | $3,207.14 CAD |
| Total Conversions | 73 |
| Conversion Rate | 7.55% |
| Average CPC | $3.32 |
| Average CPA | $43.93 |

---

## 🏗️ Data Pipeline & Methodology

### 1. Data Extraction
- Pulled raw campaign data from Google Ads across multiple dimensions: campaign, ad group, keyword, demographic (age, gender, household income), device, click type, and network
- Exported datasets covering FY23 DA Program, FY23 DA Trial 836, and FY24 DE Program

### 2. Data Cleaning & Transformation
- Unified inconsistent column headers across multiple export formats
- Handled edge cases in calculated fields (division by zero for CVR/CPA on zero-conversion keywords)
- Created custom calculated fields:
  ```
  CPA  = IF(Conversions=0, IF(Cost>0, "WASTE", 0), Cost/Conversions)
  CVR  = IF(Conversions=0, IF(Clicks=0, "WASTE", 0), Conversions/Clicks)
  ```
- Built pivot table structures for multi-dimensional analysis (keyword × match type × campaign)

### 3. Analysis Dimensions
- **Campaign Level:** Cross-campaign performance comparison (FY23 vs FY24, DA vs DE)
- **Ad Group Level:** 8 ad groups evaluated on CVR, CPA, and budget efficiency
- **Keyword Level:** 100+ keywords analyzed across 3 match types (Broad, Phrase, Exact)
- **Demographic Level:** Age, gender, and household income segmentation
- **Network & Device:** Google Search vs Search Partners, Mobile vs Desktop vs Tablet

---

## 📈 Key Findings

### Campaign-Level Insights
**FY24 DE Program outperformed FY23 DA across all efficiency metrics:**

| Metric | FY23 DA | FY24 DE | Change |
|--------|---------|---------|--------|
| CPA | $54.47 | $33.66 | ↓ 38% improvement |
| CPC | $4.32 | $2.58 | ↓ 40% reduction |
| Conv. Rate | 7.93% | 7.66% | ~ Comparable |
| Impressions | 13,282 | 34,131 | ↑ 157% growth |

> **Interpretation:** The DE program attracted higher search volume at lower cost, suggesting stronger market demand and better keyword-market fit for data engineering training.

### Ad Group Performance

| Tier | Ad Group | CVR | CPA | Recommendation |
|------|----------|-----|-----|----------------|
| 🟢 Top | Mentorship | 12.12% | $20.01 | Scale budget aggressively |
| 🟢 Top | Job Guarantee | 11.11% | $32.01 | Scale with monitoring |
| 🟢 Top | Career | 8.44% | $32.63 | Scale budget |
| 🟡 Mid | Training | 7.86% | $41.49 | Optimize keywords |
| 🟡 Mid | BA Bootcamp | 5.26% | $81.74 | Restructure ads |
| 🔴 Low | DA Bootcamp | 2.08% | $196.57 | Pause — CPA 4.5× avg |
| 🔴 Low | Near me | 0% | ∞ | Pause — $156 wasted |

### Keyword Insights
**High-value keywords identified:**
- `[data engineering training courses]` — 42.9% CVR, $5.82 CPA (exact match)
- `data engineering coaching` — 33.3% CVR, $5.50 CPA
- `get into data engineering` — 13.2% CVR, $19.98 CPA
- `data analyst training and placement` — 14.7% CVR, $30.22 CPA

**Anomaly flagged:** `data analyst online training` showed 72.7% CVR (8 conversions from 11 clicks) — recommended conversion tracking audit before scaling.

**Match type finding:** Exact match keywords in FY24 DE achieved 40% CVR with $5.97 CPA vs Broad match at 6.8% CVR with $37.81 CPA — a strong case for shifting budget toward exact match.

### Demographic Insights
- **Age:** 25–34 cohort drives 53% of conversions (39/73) with the best CPA ($32.89)
- **Gender:** Males convert at a slightly higher rate (9.4% vs 7.7%) with lower CPA ($37.75 vs $44.25)
- **Income:** Lower 50% household income delivers 42% of conversions with the best CPA ($32.75) — aligns with the ISA (Income Share Agreement) model targeting career changers

---

## 💡 Strategic Recommendations

### Immediate Actions (Budget Impact: ~$350 savings/month)
1. **Pause** "Near me (location)" ad group → $156/month wasted on 0 conversions
2. **Pause** "DA Bootcamp" ad group → $196.57 CPA is unsustainable
3. **Add negative keywords:** "security analyst", "comptia", "power bi service" to eliminate irrelevant traffic

### Optimization Actions (Expected CPA reduction: 15–20%)
4. **Shift budget from Broad → Exact match** for proven converting keywords
5. **Increase bid adjustments** for 25–34 age demographic (+20% recommended)
6. **Scale "Mentorship" ad group** — only $80 spend on best-performing group

### Growth Actions
7. **Expand exact match variants** for top DE keywords ("get into data engineering", "data engineering coaching")
8. **Continue DE program investment** — better market fit evidenced by 38% lower CPA than DA
9. **Test new ad copy** emphasizing "placement" and "mentorship" — these themes drive highest CVR

---

## 📂 Repository Structure

```
├── README.md                              # This file — full case study
├── Campaign_Performance_Analysis.xlsx     # Raw data (multi-sheet workbook)
├── Campaign_Performance_Report.xlsx       # Polished analysis report
├── Campaign_Dashboard.html               # Interactive dashboard (open in browser)
```

---

## 🔍 How to View

1. **Dashboard:** Download `Campaign_Dashboard.html` and open in any modern browser
2. **Excel Report:** Open `Campaign_Performance_Report.xlsx` in Excel or Google Sheets
3. **Raw Data:** `Campaign_Performance_Analysis.xlsx` contains all source data across 7 sheets

---

## 📝 About This Project

This analysis demonstrates my ability to:
- **Extract** raw data from Google Ads platform across multiple campaign dimensions
- **Clean & transform** messy multi-source data into unified, analytics-ready datasets
- **Analyze** campaign performance using segmentation, calculated metrics, and comparative analysis
- **Visualize** findings through both static reports and interactive dashboards
- **Recommend** actionable, data-driven strategies with quantified expected impact

*Part of a portfolio showcasing Business Analysis skills across 5+ real-world projects.*

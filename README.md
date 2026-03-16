# AI & The Future of Work: 2024–2030
### A Tableau Data Story exploring AI's impact on the global job market

📊 **[View the interactive Tableau Story on Tableau Public](https://public.tableau.com/views/ai_jobmarket/AITheFutureofWorkADataStory?:language=de-DE&:sid=&:redirect=auth&:display_count=n&:origin=viz_share_link)**

---

## Project Overview

This project explores how Artificial Intelligence is reshaping the global job market between 2024 and 2030. Rather than presenting a single static dashboard, it is structured as a Tableau Story with three interactive dashboards — each designed for a different stakeholder audience: career changers, HR directors, and policymakers.

The goal is to demonstrate that the same dataset can tell meaningfully different stories depending on who is asking the question — and that translating data into actionable decisions for non-technical audiences is as important as the analysis itself.

---

## Business Questions Addressed

- **Career Changers:** Which roles offer strong salaries, low automation risk, and are accessible without years of prior experience?
- **HR Directors:** Which roles in our industry are at highest automation risk, and where should we adjust hiring plans before 2030?
- **Policymakers (Germany):** Which industries face the greatest structural disruption, and where should national reskilling programmes be directed?

---

## Dashboard Structure

| Slide | Title | Audience |
|-------|-------|----------|
| 1 | Cover | — |
| 2 | Intro & Navigation | All |
| 3 | Career Explorer: Find Your Future-Proof Role | Career Changers |
| 4 | HR Director Dashboard: Germany Workforce Planning | HR Directors |
| 5 | Germany Labour Market Outlook 2024–2030 | Policymakers |

---

## Dataset

- **Title:** AI Impact on Job Market: Increasing vs Decreasing Jobs (2024–2030)
- **Source:** [Kaggle](https://www.kaggle.com) — originally featured in Makeover Monday 2025, Week 37
- **Size:** 30,000 rows, 13 columns, covering 8 countries
- **Important:** This is a synthetic dataset modelled on patterns from U.S. BLS, OECD, McKinsey, and WEF reports. It is designed to reflect plausible scenarios and is suitable for educational and portfolio purposes. Findings should not be treated as ground truth.

---

## Data Exploration & Quality Findings

### 1. Establishing the Data Grain

Before building any visualisations, a duplicate check was run to establish the grain of the dataset — the level at which each row is unique.

The grain is: **Job Title + Industry + Location + Experience Required (Years)**

This means the same job title appears multiple times in the dataset — once per industry, country, and experience level combination. This has important implications for aggregation: filtering by industry and country alone is not sufficient to obtain a unique value per role. Experience level must also be considered.

The distinction between grain columns and attribute columns is key here:
- **Grain columns** (Job Title, Industry, Location, Experience Required) categorise what a row represents
- **Attribute columns** (Median Salary, Automation Risk %, Job Openings) describe or measure that row

### 2. Job Status Field Inconsistency

A data quality check revealed that **7,500 out of 30,000 rows (25%)** contained a contradiction: the `Job Status` field (labelled "Increasing" or "Decreasing") did not match the actual direction of change between `Job Openings (2024)` and `Projected Openings (2030)`.

Since the openings figures are the more objective measure, the original `Job Status` field was replaced throughout with a derived calculated field — `Actual Job Status` — based directly on the numeric comparison:

```
IF [Projected Openings (2030)] > [Job Openings (2024)]
THEN "Increasing"
ELSE "Decreasing"
END
```

This appears to be a result of the synthetic data generation process assigning `Job Status` independently rather than deriving it from the openings data.

### 3. AI Impact Level Inconsistency

The same Job Title + Industry + Location combination can appear with different AI Impact Levels across experience levels. This reflects a plausible real-world pattern — AI centrality of a role may differ by seniority. Since explicit seniority labels are absent from the dataset, `Experience Required (Years)` serves as the seniority proxy. Values are aggregated across experience levels in the HR Director dashboard unless the Experience filter is applied.

---

## Key Calculated Fields

### Net Job Change (2024–2030)
```
SUM([Projected Openings (2030)]) - SUM([Job Openings (2024)])
```

### Actual Job Status
```
IF [Projected Openings (2030)] > [Job Openings (2024)]
THEN "Increasing"
ELSE "Decreasing"
END
```

### Total Structural Change
```
SUM(ABS([Projected Openings (2030)] - [Job Openings (2024)]))
```
Sums the absolute value of each role's change row by row before aggregating. Captures total gross job displacement — both gains and losses — regardless of direction. This reveals the true scale of workforce disruption that net figures hide.

### Career Sweet Spot
```
IF AVG([Median Salary (USD)]) >= WINDOW_AVG(AVG([Median Salary (USD)]))
AND AVG([Automation Risk (%)]) <= WINDOW_AVG(AVG([Automation Risk (%)]))
THEN "Sweet Spot"
ELSE "Other"
END
```
A table calculation scoped to Job Title + Industry. The average is computed dynamically across all currently visible marks — meaning the quadrant classification updates automatically when filters are applied.

### Normalised Metrics (HR Director Heatmap)
Each of the six heatmap metrics is normalised to a 0–1 scale using WINDOW_MIN/MAX to enable a single unified color scale across metrics of different magnitudes. Example:

```
(AVG([Automation Risk (%)]) - WINDOW_MIN(AVG([Automation Risk (%)])))
/ (WINDOW_MAX(AVG([Automation Risk (%)])) - WINDOW_MIN(AVG([Automation Risk (%)])))
```

Metrics where lower is better (Automation Risk, AI Impact Level) are inverted: `1 - (normalised value)`.

---

## Key Findings

### Germany Labour Market (Policymaker Dashboard)
- Net job change: approx **-40,000 / -0.21%** of total market
- Total structural change: approx **12,000,000 / 65%** of total market
- Average automation risk: approx **50%**
- The finance sector shows the largest net job losses — primary retraining priority
- Structural change is roughly equal across all industries — confirming this is a systemic challenge, not sector-specific
- Automation risk varies by only ~2% across industries — broad national reskilling initiatives are needed, not sector-targeted responses

*Note: Germany is used as the reference case. The headline net change figure (-0.21%) masks a 65% structural disruption rate — meaning millions of workers will need to transition between roles even though overall job numbers appear stable.*

---

## Dashboard Design Decisions

- **Automation Risk % over AI Impact Level:** Automation Risk is a precise continuous metric vs a vague categorical label. Note: AI Impact Level indicates how central AI is to a role — not how at risk it is. These are distinct concepts.
- **Experience filter pre-set to 0–2 years (Career Changer):** Career changers typically cannot compete for roles requiring extensive prior experience. This filter runs in the background.
- **Dynamic quadrant coloring:** The Career Changer sweet spot recalculates based on the current filtered view — "above average salary" always means above average for the current selection.
- **Job Title + Industry as the unit of analysis:** Job Title alone is not a unique identifier — the same role exists across multiple industries at different salary levels.
- **Three separate dashboards for three audiences:** Demonstrates stakeholder communication thinking — the same data tells different stories for different decision-makers.
- **Heatmap with normalisation (HR Director):** Six metrics of different scales are normalised to 0–1 so a single color scale applies meaningfully. Actual values shown in tooltips.
- **Static recommendations:** Dynamic role-level text recommendations (e.g. listing specific job titles meeting a threshold) are not feasible in Tableau — string aggregation across marks is not natively supported. In a production environment, Python would generate these programmatically.

---

## Tool Considerations

Tableau excels at visual analytics and interactive storytelling. For complex operations — global sorting across multiple dimensions, independent color scales per column, dynamic string generation — workarounds are required due to Tableau's architecture.

In a production environment, data transformation and complex logic would be handled upstream in SQL or Python, with Tableau serving as the presentation layer. Python handles normalisation, string aggregation, and programmatic recommendations trivially. Power BI offers more flexibility for table-heavy reporting.

---

## Tools Used

- **Tableau Public** (Desktop App) — data connection, calculated fields, dashboard and story assembly
- **CSV source data** — cleaned and configured within Tableau (no external preprocessing required)
- **Canva** — cover slide design

---

## About This Project

This is Project 1 of a data analyst portfolio built during a career transition from marketing and sales into data analytics. The portfolio focuses on e-commerce and marketing data, with each project demonstrating a distinct technical skillset. The core differentiator is business acumen — the ability to translate data into decisions for non-technical stakeholders, grounded in prior experience on the business side of organisations.

*Robert W. — March 2026*

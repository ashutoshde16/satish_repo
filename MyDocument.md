# MyDocument – Learnings, Challenges, Assumptions

## Assumptions

- **Dataset scope**: The NYC jobs CSV is the single source of truth. “Highest-paid skills in the US market” is interpreted as **highest-paid skills in this NYC job postings dataset** (no external US salary data).
- **Last 2 years**: Defined using the maximum `Posting Date` in the data minus 730 days (e.g. if latest posting is 2019-12-17, “last 2 years” is 2017-12-17 to 2019-12-17).
- **Salary comparison**: Salaries are normalized to **annual** for KPIs: Hourly × 2080, Daily × 260; otherwise the range midpoint is used. Missing or invalid salary values are excluded from salary-based KPIs.
- **Degree vs salary (KPI 3)**: “Higher degree” is inferred from text in **Minimum Qual Requirements** using keywords (e.g. baccalaureate, master's, PhD). Correlation is observational (postings that mention higher degrees have higher average salary in this sample).
- **Skills (KPI 6)**: Skills are parsed from **Preferred Skills** by splitting on delimiters (e.g. commas, bullets). Only phrases with 4–80 characters and appearing in at least 5 postings are used to avoid noise.
- **Target path**: Processed output is written to `/notebook/processed_nyc_jobs` (Parquet). In Docker this is under the mounted notebook directory; adjust if running elsewhere.

## Learnings

- **CSV types**: All columns are read as string. Profiling and casting (salary, dates, integers) are necessary before aggregation and feature engineering.
- **Duplicate rows**: The same job can appear as both Internal and External; counts are “posting records” not unique jobs unless deduplicated by `Job ID` (or similar).
- **Long text columns**: Columns like Job Description and Minimum Qual Requirements are useful for NLP or keyword extraction but were dropped in the processed dataset to keep a compact schema; they can be reattached from the raw data if needed.
- **Visualization**: Matplotlib is used for bar charts; results are collected with `.toPandas()` for plotting. For very large data, sampling or pre-aggregation in Spark is preferable to collecting full result sets.

## Challenges

- **Degree extraction**: Minimum Qual Requirements are free text; the degree-level logic is heuristic (regex). Some postings mention multiple degrees (e.g. “bachelor or equivalent”); we map to a single level and may misclassify edge cases.
- **Skills extraction**: Preferred Skills format varies (bullets, commas, line breaks). Splitting and trimming yields mixed-quality “skills”; filtering by length and minimum count improves robustness but may drop valid rare skills.
- **Date handling**: Some date fields are null or malformed; we filter null posting dates where needed (e.g. “last 2 years” KPI) and use Spark’s `to_date` with a single format.

## KPIs Related calculation and logic declaration 

## Salary Normalization Rationale (Hourly × 2080, Daily × 260)

### Why normalization is required
The NYC Jobs dataset contains salary values expressed using different **frequencies**:
- Hourly
- Daily
- Annual

For analytical KPIs (average salary, salary distribution, ranking of jobs or agencies, correlations), salaries must be **comparable**. Directly aggregating mixed frequencies would produce misleading results. Therefore, all salaries are normalized to a **common annual equivalent**.

Annual salary is chosen because it is the most commonly understood and reported unit in labor market analytics.

---

### Why Hourly × 2080
This conversion is based on standard full-time employment assumptions in the United States:
- 40 working hours per week
- 52 working weeks per year

```
40 × 52 = 2080 hours per year
```

**Example**:
- Hourly rate: $50/hour
- Annual equivalent: `50 × 2080 = $104,000`

The factor **2080** is widely used in:
- US HR and payroll systems
- Government labor statistics
- Compensation benchmarking

This approach avoids making assumptions about overtime and ensures conservative, standardized comparisons.

---

### Why Daily × 260
This conversion assumes a standard 5-day work week:
- 5 working days per week
- 52 weeks per year

```
5 × 52 = 260 working days per year
```

**Example**:
- Daily rate: $400/day
- Annual equivalent: `400 × 260 = $104,000`

---

### Why holidays and leave are not subtracted
Paid leave, holidays, and benefits are not consistently defined in job postings. Subtracting them would introduce assumptions that vary by agency and role.

For analytics, **consistency and reproducibility** are prioritized over precision when precise benefit data is unavailable.

---

### Impact on KPIs
Without normalization, hourly or daily roles would appear significantly underpaid compared to annual roles, leading to incorrect conclusions in:
- Salary distributions
- Agency-level averages
- Highest-paid job rankings
- Skill-to-salary analysis

By normalizing all salaries to annual equivalents, KPI results accurately reflect market compensation.

---

### Documented Assumption
All salary-based KPIs in this project assume **full-time employment equivalents** when normalizing hourly and daily compensation.

This assumption is explicitly documented to ensure transparency, auditability, and interpretability of results.


## KPI Calculations

This section describes how each KPI is calculated, including the logic, assumptions, and PySpark-based approach used in the analysis.

---

### 1. Number of Job Postings per Category (Top 10)
**Objective:** Identify which job categories have the highest number of open positions.

**Logic:**
- Group records by `Job Category`
- Count number of postings per category
- Sort in descending order
- Select top 10 categories

**Why it matters:**
This KPI highlights demand trends across functional areas and helps understand which domains are most actively hiring.

---

### 2. Salary Distribution per Job Category
**Objective:** Understand how compensation varies across job categories.

**Logic:**
- Use normalized annual salary (`salary_mid_annual`)
- Group by `Job Category`
- Compute average (and optionally min/max) salary per category
- Visualize distribution using bar plots or box plots

**Assumptions:**
- Salary midpoint represents a fair approximation of expected compensation
- Only valid, positive annualized salaries are included

---

### 3. Correlation Between Higher Degree and Salary
**Objective:** Measure whether higher education requirements are associated with higher salaries.

**Feature Engineering:**
- Extract degree indicators from `Minimum Qual Requirements`
- Encode degree level numerically (e.g., Doctorate = 4, Master = 3, Bachelor = 2, Other = 1)

**Logic:**
- Compute correlation between `degree_level` and `salary_mid_annual`

**Interpretation:**
- Positive correlation suggests higher education requirements generally lead to higher compensation
- Correlation does not imply causation

---

### 4. Highest Salary Job Posting per Agency
**Objective:** Identify the highest-paying job within each NYC agency.

**Logic:**
- Partition data by `Agency`
- Order postings by `salary_annual_to` in descending order
- Select the top-ranked posting per agency using window functions

**Why it matters:**
Provides insight into compensation ceilings and senior leadership or specialized roles within agencies.

---

### 5. Average Salary per Agency for the Last 2 Years
**Objective:** Compare agency-level compensation trends over recent postings.

**Logic:**
- Filter postings where `posting_date` falls within the last two calendar years
- Use normalized annual midpoint salary
- Group by `Agency`
- Calculate average salary

**Assumptions:**
- Posting date reflects market-relevant compensation timing
- Older postings are excluded to avoid outdated salary benchmarks

---

### 6. Highest Paid Skills in the US Market
**Objective:** Identify which skills are associated with the highest compensation.

**Logic:**
- Tokenize `Preferred Skills` free-text field into individual skills
- Normalize text (lowercase, trim whitespace)
- Explode skills so each skill maps to a job posting
- Aggregate average annualized midpoint salary (`salary_mid_annual`) per skill
- Rank skills by average salary

**Considerations:**
- Skills are derived from unstructured text and may contain noise or synonyms
- Results reflect market trends, not guaranteed compensation

---

### Assumptions & Limitations
- Degree extraction relies on keyword matching and may miss uncommon phrasing
- Some postings do not specify degree requirements and are classified as `Unspecified`
- Analysis reflects posted salary ranges, not negotiated compensation

---

### General Notes
- All salary-based KPIs use **annualized salary equivalents** as documented in the Salary Normalization section
- Invalid or missing salary records are excluded from salary-driven calculations
- KPIs are computed using PySpark DataFrame and SQL APIs to ensure scalability


## Considerations

- **Feature removal**: Columns dropped in the processed dataset were chosen to reduce size and focus on structured fields (agency, category, salary, dates, derived degree, posting year). Long text and redundant location/contact fields were removed; they can be kept if downstream use requires them.
- **Deployment**: The solution runs in the provided Docker setup (Jupyter + Spark master/workers). No additional deployment steps are required for the take-home; see below for optional deployment and triggering.

## Deployment (proposal)

- **Containers**: Use `docker compose -f ./docker-compose.yml --project-name my_assesment up -d` to start the cluster and Jupyter.
- **Paths**: Ensure the dataset is mounted (e.g. `./dataset:/dataset`) and the notebook directory is mounted (e.g. `./jupyter/notebook:/notebook`). Processed output path (`/notebook/processed_nyc_jobs`) must be writable by the Jupyter container.
- **Resources**: Per INSTALL.md, at least 8GB RAM is recommended for Spark.

## Triggering the code (suggested approach)

1. **Interactive (notebook)**: Open `assesment_notebook.ipynb` in Jupyter and run all cells (Kernel → Restart & Run All). This runs exploration, KPIs, processing, and tests.
2. **Batch (conversion to script)**: Export the notebook to a `.py` script (e.g. `jupyter nbconvert --to script assesment_notebook.ipynb`) and run with `spark-submit` from the Jupyter/Spark container, e.g.  
   `spark-submit --master spark://master:7077 /notebook/assesment_notebook.py`
3. **Scheduling**: For recurring runs, use a scheduler (e.g. cron or Airflow) to run the script or a wrapper that starts the cluster (if needed), runs the job, and optionally shuts down the cluster.

# NYC Job Postings Data Analysis

## 1. Project Overview
This project analyzes NYC job posting data to extract insights related to:
* Job demand trends
* Salary distribution
* Education requirements
* Skills associated with higher-paying roles
* The dataset was processed using PySpark, and several data cleaning, transformation, and feature engineering steps were applied before performing KPI analysis.


# 2. Learnings

* Performed data profiling and exploratory analysis on large datasets using PySpark to identify data quality issues and patterns.
* Cleaned and standardized messy, inconsistent real-world datasets, improving reliability for downstream analytics.
* Implemented feature engineering techniques, including deriving average salary metrics and extracting structured insights from unstructured text fields.
* Utilized window functions, aggregations, and advanced filtering to support complex analytical queries.
* Built reusable data preprocessing pipelines using modular PySpark functions to improve efficiency and maintainability.
* Developed business KPI visualizations using Matplotlib to support data-driven decision-making.


# 3. Challenges Faced During the Project
# 3.1. Inconsistent Data Formats

* The Posting_Date column contained mixed data formats, including timestamps, placeholder values (e.g., 1900-01-01), and non-date text entries.
* To enable accurate time-series analysis:
* Applied regular expression-based pattern extraction
* Converted valid values using PySpark date functions
* Filtered out placeholder and invalid records
* This preprocessing ensured reliable temporal trend analysis.

# 3.2. Handling Unstructured Text Fields

* Columns such as Preferred_Skills and Min_Qual_Requirements consisted of long, free-text descriptions.

* Challenges included:
* Extracting structured skill keywords
* Identifying education levels from inconsistent phrasing
* Removing noise and redundant text
* Solutions implemented:
* Text normalization (lowercasing, trimming, removing special characters)
* Pattern matching using regex
* Keyword-based classification logic

This enabled transformation of unstructured content into analyzable features.

# 3.3. Missing and Placeholder Values

* Several columns contained placeholder entries such as:
* Not Specified
* Error Name
* Default or system-generated dates
* These values could distort aggregation and correlation analysis.

# 3.4. Mitigation steps:

* Replaced placeholders with null
* Applied conditional filtering
* Used imputation logic where appropriate
* This improved data quality and analytical accuracy.

# 3.5. Weak Correlation Between Education and Salary

* Initial correlation analysis showed a relatively weak relationship between education level and salary.

* This occurred because salary is influenced by multiple factors including:
* Job category
* Agency/department
* Years of experience
* Location and demand
* This highlighted the importance of:
* Multi-variable analysis
* Feature engineering
* Considering confounding variables in business analytics

# 4. KPI Calculations & Logic

# 4.1 Top 10 Job Categories by Number of Postings

* Objective: Identify highest demand categories.
* Logic:
* Group by Job Category
* Count postings
* Sort descending
* Select top 10

# 4.2 Salary Distribution per Job Category

* Objective: Compare compensation across domains.
* Logic:
* Use normalized salary_mid_annual
* Group by Job Category
* Compute average salary
* Visualize using bar plots

# 4.3 Correlation Between Degree Level and Salary

* Feature Engineering:
* Encode degree level numerically
* PhD = 5
* Master = 4
* Bachelor = 3
* Associate = 2
* Unspecified =1

* Logic:
* Compute correlation between degree_level and salary_mid_annual
* Note: Correlation does not imply causation.

# 4.4 Highest Salary Posting per Agency

* Logic:
* Partition by Agency
* Order by salary_annual_to (descending)
* Select top-ranked record using window functions
* This identifies compensation ceilings per agency.

# 4.5 Average Salary per Agency (Last 2 Years)

* Logic:
* Filter postings within last 2 calendar years
* Group by Agency
* Compute average annual midpoint salary
* This avoids outdated compensation data.

# 4.6 Highest Paid Skills in the Market

* Logic:
* Tokenize Preferred_Skills
* Normalize text
* Explode skill arrays
* Aggregate average salary_mid_annual
* Rank skills

* Note: Results reflect market trends and may include textual noise.

# 5. Considerations

*  Dataset contains government job postings, so pay scales are standardized.
* Skills were extracted using keyword-based methods (not advanced NLP).
*  High-cardinality columns were removed to improve aggregation efficiency.
*  Salary ranges were simplified using midpoint approximation.

# 6. Assumptions

* Salary midpoint represents expected compensation.
* Education levels were inferred using keyword extraction.
* Only relevant analytical columns were retained.
* Extracted skills represent major competencies for roles.

# 7. Final Output

* Data cleaning
* Feature engineering
* Salary normalization
* Feature selection

The processed dataset was stored as a structured analytical file for downstream reporting and visualization.
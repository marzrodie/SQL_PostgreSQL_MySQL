# Data Analyst Job Market — SQL Analysis

Which skills should a data analyst learn to be both hireable and well paid? This project answers that with five SQL queries against ~785K job postings from 2023, focused on remote data analyst roles.

Built following Luke Barousse's *SQL for Data Analytics* course. The dataset and the five questions come from the course; the query write-ups, charts, and analysis below are mine.

**Key findings**
- SQL is the most requested skill for remote data analysts and appears in 8 of the 10 highest-paying roles.
- Python and Tableau pair high demand (236 and 230 salaried postings) with ~$100K average pay — the best balance of the two.
- The best-paid "skills" in the data (SVN at $400K, Solidity at $179K) each come from a handful of postings and aren't a realistic target.

📁 [SQL queries](/project_sql/)

# Background

This project examines the Data analyst job market to identify which skills pay the most and appear most in postings, making it easier for job seekers to focus their efforts

### The questions I wanted to answer through my SQL queries were:

1. What are the top-paying data analyst jobs?
2. What skills are required for these top-paying jobs?
3. What skills are the most in demand for data analysts?
4. Which skills are associated with higher salaries?
5. What are the most optimal skills to learn?

# Tools I Used

- **SQL:** Every query and insight below.
- **PostgreSQL:** Database for the job postings data.
- **Visual Studio Code:** Query editing and execution.
- **Excel:** Charts, built from exported query results.
- **Git & GitHub:** Version control and hosting.

# The Analysis

Each query in this project investigates a specific aspect of the data analyst job market.
Here is how i approached each question:

### 1. Top Paying Data Analyst jobs

To find the highest paying roles, i filtered data analyst positions by average yearly salary and location, focusing on remote work. This query surfaces the most lucrative opportuities in the field.

```sql
SELECT
    job_id,
    job_title,
    job_location,
    job_schedule_type,
    salary_year_avg,
    job_posted_date,
    name AS company_name
FROM
    job_postings_fact
LEFT JOIN company_dim ON job_postings_fact.company_id = company_dim.company_id
WHERE
    job_title_short = 'Data Analyst' AND
    job_location = 'Anywhere' AND
    salary_year_avg IS NOT NULL
ORDER BY
    salary_year_avg DESC
LIMIT 10
```

Here is the breakdown of the top remote data analyst jobs in 2023:

1. **The $650K posting is an outlier, not a benchmark.** It's a single listing from Mantys; the next nine roles run from $184K to $336K. I kept it in the table but wouldn't build a salary expectation on it.
2. **Titles skew senior.** 6 of the ten are Director, Principal, or Associate Director roles — "analyst" in the title doesn't mean analyst-level pay.
3. **Employers span industries.** Meta, AT&T, Pinterest, UCLA Health — no single sector dominates.

![Top 10 highest paying remote jobs in Data Analysis](Assests/Top_10_highest_paying_remote_Data_Analyst_2.png)

*Bar graph of the top ten salaries for Data analyst roles. Chart built in Excel from the query output*

### 2. Skills for Top Paying Jobs

To see which skills the highest-paying roles require, i joined the job postings table witht eh skills data table. The result shows what employers value most in the top-compensation positions.

```sql

WITH top_paying_jobs AS (

        SELECT
            job_id,
            job_title,
            salary_year_avg,
            name AS company_name
        FROM
            job_postings_fact
        LEFT JOIN company_dim ON job_postings_fact.company_id = company_dim.company_id
        WHERE
            job_title_short = 'Data Analyst' AND
            job_location = 'Anywhere' AND
            salary_year_avg IS NOT NULL
        ORDER BY
            salary_year_avg DESC
        LIMIT 10
)

SELECT 
    top_paying_jobs.*,
    skills
FROM top_paying_jobs
INNER JOIN skills_job_dim ON top_paying_jobs.job_id = skills_job_dim.job_id
INNER JOIN skills_dim ON skills_job_dim.skill_id = skills_dim.skill_id
ORDER BY
    salary_year_avg DESC

```
Here is the breakdownof the most in-demand skilss across the ten highest-paying Data analyst jobs in 2023:

1. **SQL** leads, appearing in 8 of the 10 postings.
2. **Python** follows close behind at 7
3. **Tableau** is also in string demend at 6.

Other skills like R, Snowflake, Pandas, and Excel appear less frequently but still show up across the list.

![Skills in the top paying Data Analyst jobs](Assests/Skills_in_demand.png)

*Bar graph of skill counts across the ten highest-paying data analyst roles.*

### 3. In-Demand Skills for Data Analysts

This query counts the skills requested most often across remote data analyst postings, regardless of whether a salary was listed.

```sql
SELECT 
    skills,
    COUNT(skills_job_dim.job_id) AS demand_count
FROM
    job_postings_fact
INNER JOIN 
    skills_job_dim ON job_postings_fact.job_id = skills_job_dim.job_id
INNER JOIN 
    skills_dim ON skills_job_dim.skill_id = skills_dim.skill_id
WHERE
    job_title_short = 'Data Analyst' AND
    job_work_from_home = TRUE
GROUP BY
    skills

ORDER BY
    demand_count DESC
LIMIT 5
```
Here is the breakdown of the most in-demand skills for Data Analyst in 2023

1. **SQL** and **Excel** remain fundamental, underlining the need for a strong foundation in data processing and spreadsheet work.
2. **Python**, **Tableau**, and **Power BI** are close behind, reflecting the growing weight placed on technical skills in data storytelling and decision support.

![Top 5 skills in demand](Assests/Top_5_skills_in_demand_Table.png)

*Demand for the top five skills across data analyst job postings*

### 4. Skills Based on Salary

Averaging salaries by skills reveals which skills command the highest pay.

```sql
SELECT 
    skills,
    ROUND(AVG(salary_year_avg), 0) AS avg_salary
FROM
    job_postings_fact
INNER JOIN 
    skills_job_dim ON job_postings_fact.job_id = skills_job_dim.job_id
INNER JOIN 
    skills_dim ON skills_job_dim.skill_id = skills_dim.skill_id
WHERE
    job_title_short = 'Data Analyst' 
    AND salary_year_avg IS NOT NULL
    AND job_work_from_home = TRUE
GROUP BY
    skills

ORDER BY
    avg_salary DESC
LIMIT 25
```

Here is the breakdown of the top-paying skillls for Data Analyst:

The top of this list is driven by tiny sample sizes — SVN ($400K) and Solidity ($179K) each appear in only 2 postings. The groupings below focus on skills with enough postings to mean something:

1. **Big Data and Machine Learning:** The highest salaries go to the analysts working with Big Data technologies (PySpark, Couchbase), machine learning tools (DataRobot, Jupyter), and Python libraries (Pandas, NumPy), this is a sign of how much the industry values data processing and predictive modeling.
2. **Software Development and Deployment Proficiency:** Familiarity with development and deployment tools (GitLab, Kubernetes, Airflow) marks a lucrative crossover between analysis and engineering, where automation and pipeline management carry a premium.
3. **Cloud Computing:** Cloud and data engineering platforms (Elasticsearch, Databricks, GCP) point to the rise of cloud-based analytics and cloud proficiency lifts earning potential noticeably.

![Highest paying skills](Assests/Top_25_skills_in_demand_Table.png)

*Table of the average salary for the top 10 paying skills for data analysts*

### 5. Most Optimal Skills to Learn

Combining insights from demand and salary data, this query aimed to pinpoint skills that are both in high demand and have high salaries, offering a strategic focus for skills development.


```sql
WITH skills_demand AS (
    SELECT 
        skills_dim.skill_id,
        skills_dim.skills,
        COUNT(skills_job_dim.job_id) AS demand_count
    FROM
        job_postings_fact
    INNER JOIN 
        skills_job_dim ON job_postings_fact.job_id = skills_job_dim.job_id
    INNER JOIN 
        skills_dim ON skills_job_dim.skill_id = skills_dim.skill_id
    WHERE
        job_title_short = 'Data Analyst' 
        AND salary_year_avg IS NOT NULL
        AND job_work_from_home = TRUE
    GROUP BY
        skills_dim.skill_id
),
 average_salary AS (
    SELECT 
        skills_job_dim.skill_id,
        ROUND(AVG(salary_year_avg), 0) AS avg_salary
    FROM
        job_postings_fact
    INNER JOIN 
        skills_job_dim ON job_postings_fact.job_id = skills_job_dim.job_id
    INNER JOIN 
        skills_dim ON skills_job_dim.skill_id = skills_dim.skill_id
    WHERE
        job_title_short = 'Data Analyst' 
        AND salary_year_avg IS NOT NULL
        AND job_work_from_home = TRUE
    GROUP BY
        skills_job_dim.skill_id
)

SELECT 
    skills_demand.skill_id,
    skills_demand.skills,
    demand_count,
    avg_salary
FROM
    skills_demand
INNER JOIN
    average_salary ON skills_demand.skill_id = average_salary.skill_id
WHERE
    demand_count > 10
ORDER BY
    avg_salary DESC,
    demand_count DESC
LIMIT 25;
```

![The top 25 skills to learn](Assests/Optimal_skills_demand_vs_pay.png)

*Average salary for the top 25 paying skills for remote data analysts*

Here’s a breakdown of the most optimal skills for Data Analysts in 2023:

1. **High-Demand Programming Languages:** Python and R stand out for their high demand, with demand counts of 236 and 148 respectively. Despite their high demand, their average salaries are around $101,397 for Python and $100,499 for R, indicating that proficiency in these languages is highly valued but also widely available.
2. **Cloud Tools and Technologies:** Skills in specialized technologies such as Snowflake, Azure, AWS, and BigQuery show significant demand with relatively high average salaries, pointing towards the growing importance of cloud platforms and big data technologies in Data Analysis.
3. **Business Intelligence and Visualization Tools:** Tableau and Looker, with demand counts of 230 and 49 respectively, and average salaries around $99,288 and $103,795 highlight the critical role of data visualization and business intelligence in deriving actionable insights from data.
4. **Database Technologies:** the demand for skills in traditional and NoSQL databases (Oracle, SQL Server, NoSQL) with average salaries ranging from $97,786 to $104,534 reflects the enduring need for data storage, retrieval and management expertise.


# What I Learned

- **CTEs made query 5 possible.** Joining demand and salary meant two aggregations over the same tables. Two CTEs joined on `skill_id` was cleaner than a nested subquery and let me debug each half on its own.
- **`LEFT JOIN` vs `INNER JOIN` changed the row count.** `LEFT JOIN` on `company_dim` in query 1 kept postings with no company name; `INNER JOIN` on the skills tables in query 2 dropped postings with no listed skills. Each choice was deliberate.
- **A filter is a claim.** Every query is limited to remote roles, and most to roles with a listed salary. That's a small slice of 785K postings, so each section says so instead of presenting results as market-wide.
- **Averages need a floor.** Query 4's top skills each came from one or two postings. Adding `demand_count > 10` in query 5 is what made the "optimal skills" list trustworthy.

# Conclusion

For remote data analyst roles in 2023: SQL and Excel get you shortlisted, Python and Tableau move you up the salary range, and cloud skills (Snowflake, Azure, AWS) are where pay is high and demand is still growing. The very highest salaries in the data are single-posting outliers and not worth planning around.

**Next step:** rerun the same five queries filtered to health, fitness, and insurance industry postings.

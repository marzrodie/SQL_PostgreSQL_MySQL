# Introduction

Get an overview of the Data job market! Mainly focusing on the Data Analyst roles, this project highlights the top-paying jobs, skills that are in demand, and the raltionship between demand and salary in Data analytics.

You can find the SQL queriesi used here: [project_sql_folder](/project_sql/).

# Background

This project examines the Data analyst job market to identify which skills pay the most and appear most in postings, making it easier for job seekers to focus their efforts

### The questions I wanted to answer through my SQL queries were:

1. What are the top-paying data analyst jobs?
2. What skills are required for these top-paying jobs?
3. What skills are the most in demand for data analysts?
4. Which skills are associated with higher salaries?
5. What are the most optimal skills to learn?

# Tools i used

This analysis of the Data analyst job market was built witht he following tools:

1. **SQL:** Queried the dataset and produced every insight below.
2. **PostgresSQL:** Stored and served the job posting data.
3. **Visual Studio Code:** The editor of choice for database management and query execution.
4. **Git and GitHub:** For version control and public hosting for scripts.

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

Here is the breakdown of the top data analyst jobs in 2023:

1. **Salaries vary widely:** The ten highest paying Data analyst roles span $184,000 to $650,000, showing the real earning potential in the field.
2. **Employers are spread across industries:** Companies including Meta and AT&T offer top salaries, pointing to demand across several industries.
3. **Titles vary just as much:** Roles range fromData analyst to Director of analytics, reflecting the range of specialisations within the Data.

![Top 10 highest paying remote jobs in Data Analysis](Assests/Top_10_highest_paying_remote_Data_Analyst_2.png)

*Bar graph of the top ten salaries for Data analyst roles. Claude generated this chart from my SQL query results*

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
Here is the breakdown of the most demanded skills for the top 10 highest paying data analyst jobs in 2023:

1. **SQL** is leading with a bold count of 8.
2. **Python** follows closely with a bold count of 7
3. **Tableau** is also highly sought after, with a bold count of 6. Other skills like R, Snowflake, Pandas, and Excel show varying degrees of demand

![Skills in the top paying Data Analyst jobs](Assests/Skills_in_demand.png)

*The bar graph visualizing the count of skills for the top 10 paying jobs for Data Analyst; Claude generated this graph*

### 3. In-Demand Skills for Data Analysts

This query helped identify the skills most frequently requested in job postings, directing focus to areas with high demand.

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
Here is the breakdown of the most demanded skills for Data Analyst in

1. **SQL** and **Excel** remain fundamental, emphasizing the need for strong foundational skills in data processing and spreadsheet manipulation.
2. **Programming** and **Visualization Tools** like **Python**, **Tableau**, and **Power BI** are essential, pointing towards the increasing importance of the technical skills in data storytelling and decision support.

![Top 5 skills in demand](Assests/Top_5_skills_in_demand_Table.png)

*Table of the demand for the top 5 skills in Data Analysis job postings*

### 4. Skills Based on Salary

Exploring the average salaries associated with different skills revealed which skills are the highest paying.

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

Here is the breakdown of the results for the top paying skills for Data Analyst:

1. ** High Demand for Big Data and ML Skills:** Top salaries are commanded by analysts skilled in big data technologies (PySpark, Couchbase), machine learning tools (DataRobot, Jupyter), and Python libraries (Pandas, Numpy), reflecting the industry’s high valuation of data processing and predictive modeling capabilities.
2. **Software Development and Deployment Proficiency:** Knowledge in development and deployment tools (GitLab, Kubernetes, Airflow) indicates a lucrative crossover between data analysis and engineering, with a premium on skills that facilitate automation and efficient data pipeline management.
3. **Cloud Computing Expertise:** Familiarity with cloud and data engineering tools (Elasticsearch, Databricks, GCP) underscore the growing importance of cloud-based analytics environments, suggesting that cloud proficiency significantly boosts earning potential in data analytics.

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

*Table of the most optimal skills for data analyst sorted by salary*

Here’s a breakdown of the most optimal skills for Data Analysts in 2023:

1. **High-Demand Programming Languages:** Python and R stand out for their high demand, with demand counts of 236 and 148 respectively. Despite their high demand, their average salaries are around $101,397 for Python and $100,499 for R, including that proficiency in these languages is highly valued but also widely available.
2. **Cloud Tools and Technologies:** Skills in specialized technologies such and Snowflake, Azure, AWS, and BigQuery show significant demand with relatively high average salaries, pointing towards the growing importance of cloud platforms and big data technologies in Data Analysis.
3. ** Business Intelligence and Visualization Tools:** Tableau and Looker, with demand counts of 230 and 49 respectively, and average salaries around $99,288 and $103,795 highlight the critical role of data visualization and business intelligence in deriving actionable insights from data.
4. **  Database Technologies:** the demand for skills in traditional and NoSQL databases (Oracle, SQL Server, NoSQL) with average salaries ranging from $97,786 to $104,534 reflects the enduring need for data storage, retrieval and management expertise.


# What i learned

Through this adventure, I have turbocharged my SQL toolkit with some serious firepower:

1.  **Complex Query Crafting:** Mastered the art of advanced SQL, merging tables like a pro and wielding WITH clauses for ninja-level temp table maneuvers.
2.  **Data Aggregation:** Got cozy with GROUP BY and turned aggregate functions like COUNT() and AVG() into my data-summarizing sidekicks.
3.  **Analytical Wizardry:** Level up my real-world puzzle-solving skills, turning questions into actionable, insightful SQL queries.

# Conclusions

### Insights

1. **Top-Paying Data Analyst Jobs:** the highest-paying jobs for data analyst that allow remote work offer a wide range of salaries, the highest at $650,000
2. **Skills for Top-Paying Jobs** High-paying Data Analyst jobs require advanced proficiency in SQL, suggesting it’s a critical skill for earning a top salary.
3. **Most In-Demand Skills:** SQL is also the most demanded skill in the Data Analyst job market, thus making it essential for job seekers.
4. **Skills with Higher Salaries** Specialized skills, such as SVN and Solidity, are associated with the highest average salaries, indicating a premium on niche expertise.
5. **Optimal Skills for Job Market Value:** SQL leads in demand and offers for a high average salary, positioning it as one of the most optimal skills for data analysts to learn to maximize their market value

### Closing Thoughts

This project enhanced my SQL skills and provided valuable insights into the Data Analyst job market. The findings from the analysis serve as a guide to prioritizing skill development and job search efforts. Aspiring Data Analyst can better position themselves in a competitive job market b y focusing on high-demand, high-salary skills. This exploration highlights the importance of continuous learning and adaptation to emerging trends in the field of Data Analytics

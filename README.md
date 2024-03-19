# Data Professional Survey Data cleaning SQL + Power BI Project

## Introduction:

In today's data-driven world, understanding the landscape of data professionals is crucial for organizations seeking to harness the power of data effectively. This project focuses on cleaning and analyzing a dataset obtained from a survey conducted among data professionals. Through meticulous data cleaning using Microsoft SQL Server Management Studio (MSSMS) and visualization with Power BI, I aimed to distill actionable insights into various aspects of data professionals' roles, backgrounds, challenges, and aspirations.

<br/><br/>

## Exploratory Data Analysis:
<br/><br/>
EDA involved exploring the Data Professional survey data to answer key questions, such as:
<br/><br/>

1. What is the distribution of job titles among data professionals?

2. Which programming language is the most favored among data professionals?

3. What is the average yearly salary of data professionals across different industries?

4. How do salary levels vary based on gender?

5. What are the age demographics of data professionals?

6. Is there a relationship between educational attainment and salary levels?

7. What is the ethnic diversity among data professionals?

8. What other trends can be seen in the data?

<br/><br/>

## Data sources:

The dataset used for this analysis is the "Power BI-Professionals Survey.xlsx" file.

<br/><br/>

## Tools and Technologies:

 - Microsoft SQL Server Management Studio for data manipulation and analysis.
 - PowerBI for data visualization and statistical analysis.
 - Obsidian for documentation purposes.

<br/><br/>

## Data cleaning and preparation:
<br/><br/>



### Removing Columns with No Data:
In this step, we performed data cleaning to remove columns that contain no data. These columns were identified as having no meaningful information for our analysis. The SQL code executed the ALTER TABLE command to drop multiple columns from the dbo.JobSurveys table. The columns removed include Browser, OS, City, Referrer, and Country. By dropping these columns, we streamlined the dataset and focused on relevant information for our analysis.

```sql
ALTER TABLE dbo.JobSurveys
DROP COLUMN Browser,
DROP COLUMN OS,
DROP COLUMN City,
DROP COLUMN Referrer,
DROP COLUMN Country;
```

<br/><br/>



### Updating "Other" Responses:

Next, we observed that several columns contained numerous roles categorized under "Other" due to free-form responses. To standardize and improve the clarity of this data, we replaced occurrences of "Other (Please Specify):" with an empty string, effectively removing this prefix while retaining the rest of the entries. This adjustment ensures consistency and  facilitates more straightforward analysis of each column


```sql 
UPDATE dbo.JobSurveys
SET [Q1 - Which Title Best Fits your Current Role?] = REPLACE([Q1 - Which Title Best Fits your Current Role?], 'Other (Please Specify):', '')
WHERE [Q1 - Which Title Best Fits your Current Role?] LIKE 'Other (Please Specify):%';
```

```sql
UPDATE dbo.JobSurveys
SET [Q4 - What Industry do you work in?] = REPLACE([Q4 - What Industry do you work in?], 'Other (Please Specify):', '')
WHERE [Q4 - What Industry do you work in?] LIKE 'Other (Please Specify):%';
```

```sql
UPDATE dbo.JobSurveys
SET [Q5 - Favorite Programming Language] = REPLACE([Q5 - Favorite Programming Language], 'Other:', '')
WHERE [Q5 - Favorite Programming Language] LIKE 'Other:%';
```

```sql
UPDATE dbo.JobSurveys
SET [Q11 - Which Country do you live in?] = REPLACE([Q11 - Which Country do you live in?], 'Other (Please Specify):', '')
WHERE [Q11 - Which Country do you live in?] LIKE 'Other (Please Specify):%';
```

```sql
UPDATE dbo.JobSurveys
SET [Q13 - Ethnicity] = REPLACE([Q13 - Ethnicity], 'Other (Please Specify):', '')
WHERE [Q13 - Ethnicity] LIKE 'Other (Please Specify):%';
```

<br/><br/>

### Removing Specific Responses:

```sql
DELETE FROM dbo.JobSurveys
WHERE [Q1 - Which Title Best Fits your Current Role?] LIKE 'Does a social media analyst count?';
```


<br/><br/>
### Standardizing Job Titles:

```sql
UPDATE dbo.JobSurveys
SET [Q1 - Which Title Best Fits your Current Role?] = REPLACE([Q1 - Which Title Best Fits your Current Role?], 'Finance Analyst ', 'Financial Analyst')
WHERE [Q1 - Which Title Best Fits your Current Role?] LIKE 'Finance Analyst ';
```


<br/><br/>



### Deleting Unique Entries:
Entries with unique job titles that occurred only once in the dataset were removed to streamline the data and avoid skewing analysis results.

```sql
DELETE FROM dbo.JobSurveys
WHERE [Q1 - Which Title Best Fits your Current Role?] IN (
    SELECT [Q1 - Which Title Best Fits your Current Role?]
    FROM dbo.JobSurveys
    GROUP BY [Q1 - Which Title Best Fits your Current Role?]
    HAVING COUNT(*) < 2
);
```

<br/><br/>



Similar to the process applied to the job title column, the columns related to industry and favorite programming language underwent refinement to ensure consistency and accuracy. Here's a breakdown of the steps taken:

<br/><br/>

### Standardizing "None" Entries:

```sql
UPDATE dbo.JobSurveys
SET [Q4 - What Industry do you work in?] = 'None'
WHERE [Q4 - What Industry do you work in?] LIKE '%none%'
OR [Q4 - What Industry do you work in?] LIKE '%non%'
OR [Q4 - What Industry do you work in?] LIKE '%not%';
```

<br/><br/>


### Standardizing Student Entries:

```sql
UPDATE dbo.JobSurveys
SET [Q4 - What Industry do you work in?] = 'Student'
WHERE [Q4 - What Industry do you work in?] LIKE '%student%'
```

<br/><br/>
### Refinement by Removing Entries with One Unique Count:

To ensure the dataset's integrity and clarity for subsequent analysis, entries with only one unique count in the Q5 - Favorite Programming Language column were removed. This process was carefully executed within a transaction to maintain data consistency and accurately reflect the intended changes. Here's how this refinement was implemented:

```sql
BEGIN TRANSACTION
```

A transaction was initiated using the BEGIN TRANSACTION statement. This step ensures that all subsequent changes made to the dataset are performed as a single unit of work, allowing for rollback in case of errors or discrepancies.

```sql
DELETE FROM dbo.JobSurveys
WHERE [Q5 - Favorite Programming Language] IN (
    SELECT [Q5 - Favorite Programming Language]
    FROM dbo.JobSurveys
    GROUP BY [Q5 - Favorite Programming Language]
    HAVING COUNT(*) < 2
);
```

Once the deletion process was completed and verified, the transaction was committed using the COMMIT TRANSACTION statement. This action finalizes the changes made to the dataset, ensuring that they are permanently applied.

```sql
COMMIT TRANSACTION
```

<br/><br/>
### Adjusting "225k+" Entries:
Entries displaying "225k+" in the Q3 - Current Yearly Salary (in USD) column were adjusted to "225k-300k" to ensure consistency and facilitate the calculation of average yearly salary.

```sql
UPDATE dbo.JobSurveys
SET [Q3 - Current Yearly Salary (in USD)] = '225k-300k'
WHERE [Q3 - Current Yearly Salary (in USD)] LIKE '225k+'
```

<br/><br/>

### Calculating Average Yearly Salary:
A new column named AverageYearlySalary was created to store the calculated average yearly salary based on the provided salary ranges. The calculation involved converting the salary ranges to numerical values and computing the average. Special consideration was given to entries with "0-" prefixes, which were treated as "20k" to maintain consistency.

```sql
UPDATE dbo.JobSurveys
SET AverageYearlySalary = 
    CASE 
        WHEN [Q3 - Current Yearly Salary (in USD)] LIKE '0%' THEN 
            CONVERT(DECIMAL(10, 2), 20)
        ELSE 
            (CONVERT(DECIMAL(10, 2), SUBSTRING([Q3 - Current Yearly Salary (in USD)], 1, CHARINDEX('k', [Q3 - Current Yearly Salary (in USD)]) - 1))
            + CONVERT(DECIMAL(10, 2), SUBSTRING([Q3 - Current Yearly Salary (in USD)], CHARINDEX('-', [Q3 - Current Yearly Salary (in USD)]) + 1, LEN([Q3 - Current Yearly Salary (in USD)]) - CHARINDEX('-', [Q3 - Current Yearly Salary (in USD)]) - 1))) / 2
    END;
```

By executing these SQL statements, the dataset was refined to provide a more accurate representation of yearly salary data. The calculated AverageYearlySalary column now enables further analysis and visualization of salary trends among data professionals.

<br/><br/>

### Standardizing Country Names:

```sql
UPDATE dbo.JobSurveys
SET [Q11 - Which Country do you live in?] = 'Nigeria'
WHERE [Q11 - Which Country do you live in?] LIKE 'Africa (Nigeria)'
```

```sql
UPDATE dbo.JobSurveys
SET [Q11 - Which Country do you live in?] = 'Argentina'
WHERE [Q11 - Which Country do you live in?] LIKE 'Argentine'
```

```sql
UPDATE dbo.JobSurveys
SET [Q11 - Which Country do you live in?] = 'Brazil'
WHERE [Q11 - Which Country do you live in?] LIKE 'Brazik'
```


```sql
UPDATE dbo.JobSurveys
SET [Q11 - Which Country do you live in?] = 'Portugal'
WHERE [Q11 - Which Country do you live in?] LIKE 'Portugsl'
```


<br/><br/>

### Removing Entries with Less than Two Unique Counts:

To enhance the dataset's suitability for analysis and ensure the reliability of the data, entries with less than two unique counts were removed. This process involved identifying and eliminating entries with limited representation. Here's how this refinement was implemented:

```sql
DELETE FROM dbo.JobSurveys
WHERE [Q13 - Ethnicity] IN (
    SELECT [Q13 - Ethnicity]
    FROM dbo.JobSurveys
    GROUP BY [Q13 - Ethnicity]
    HAVING COUNT(*) < 2
```

```sql
DELETE FROM dbo.JobSurveys
WHERE [Q8 - If you were to look for a new job today, what would be the ] IN (
    SELECT [Q8 - If you were to look for a new job today, what would be the ]
    FROM dbo.JobSurveys
    GROUP BY [Q8 - If you were to look for a new job today, what would be the ]
    HAVING COUNT(*) < 2
);
```

<br/><br/>

### Standardizing Responses for Difficulty in Breaking into Data:

To facilitate more meaningful analysis, responses indicating "Neither easy nor difficult" in the [Q7 - How difficult was it for you to break into Data?] column were standardized to "Medium". This adjustment helps streamline the analysis by categorizing responses into clearer distinctions. Here's how this refinement was implemented:

```sql
UPDATE dbo.JobSurveys
SET [Q7 - How difficult was it for you to break into Data?] = REPLACE([Q7 - How difficult was it for you to break into Data?], 'Neither easy nor difficult', 'Medium')
WHERE [Q7 - How difficult was it for you to break into Data?] LIKE 'Neither easy nor difficult';
```

<br/><br/>

### Removal of Rows with Null Values:

To ensure the dataset's completeness and accuracy, rows containing null values in the [Q6 - How Happy are you in your Current Position with the follow...] column were removed. This process helps maintain data integrity by eliminating incomplete or missing data points. Here's how this refinement was implemented:

```Sql
DELETE FROM dbo.JobSurveys
WHERE [Q6 - How Happy are you in your Current Position with the followi] IS NULL OR 
[Q6 - How Happy are you in your Current Position with the follow1] IS NULL OR 
[Q6 - How Happy are you in your Current Position with the follow2] IS NULL OR 
[Q6 - How Happy are you in your Current Position with the follow3] IS NULL OR 
[Q6 - How Happy are you in your Current Position with the follow4] IS NULL OR 
[Q6 - How Happy are you in your Current Position with the follow5] IS NULL
```


<br/><br/>

### Deletion of Entries with Missing Education Information:

To ensure the dataset's completeness and reliability, entries where respondents have not provided their highest level of education were removed. This process helps maintain data integrity by eliminating incomplete or missing data points, particularly for an important measure like education level. Here's how this refinement was implemented:

```sql
DELETE FROM dbo.JobSurveys
WHERE [Q12 - Highest Level of Education] LIKE ''
```

<br/><br/>

### Renaming Columns for Clarity:

```sql
EXEC sp_rename 'dbo.JobSurveys.[Q6 - How Happy are you in your Current Position with the followi]', 'Happiness with Salary', 'COLUMN';

EXEC sp_rename 'dbo.JobSurveys.[Q6 - How Happy are you in your Current Position with the follow1]', 'Happiness with Work/Life Balance', 'COLUMN';

EXEC sp_rename 'dbo.JobSurveys.[Q6 - How Happy are you in your Current Position with the follow2]', 'Happiness with Coworkers', 'COLUMN';

EXEC sp_rename 'dbo.JobSurveys.[Q6 - How Happy are you in your Current Position with the follow3]', 'Happiness with Management', 'COLUMN';

EXEC sp_rename 'dbo.JobSurveys.[Q6 - How Happy are you in your Current Position with the follow4]', 'Happiness with Upward Mobility', 'COLUMN';

EXEC sp_rename 'dbo.JobSurveys.[Q6 - How Happy are you in your Current Position with the follow5]', 'Happiness with Learning New Things', 'COLUMN';

EXEC sp_rename 'dbo.JobSurveys.[Q8 - If you were to look for a new job today, what would be the ]', 'Most important thing on new job', 'COLUMN';
```

<br/><br/>

## Data visualization:

### Summary of Dashboard Insights

The dashboard presents a comprehensive overview of the Survey data, offering valuable insights derived from diverse visualizations.

 - Key Statistics:
The dashboard provides key insights into various aspects of respondents' work satisfaction, including Work/Life Balance Happiness, Salary Happiness, Management Happiness, and Coworker Happiness. Additionally, it offers insights into average salary distribution across different job titles and the demographic composition of survey respondents, such as gender distribution, average age, and total survey takers.

 - Interactive Features:
The dashboard provides key insights into various aspects of respondents' work satisfaction, including Work/Life Balance Happiness, Salary Happiness, Management Happiness, and Coworker Happiness. Additionally, it offers insights into average salary distribution across different job titles and the demographic composition of survey respondents, such as gender distribution, average age, and total survey takers.

 - Total Satisfaction Trends:
The dashboard provides key insights into various aspects of respondents' work satisfaction, including Work/Life Balance Happiness, Salary Happiness, Management Happiness, and Coworker Happiness. Additionally, it offers insights into average salary distribution across different job titles and the demographic composition of survey respondents, such as gender distribution, average age, and total survey takers.

 - Salary Analysis by Job Title:
The clustered bar chart presents the average salary distribution across different job titles, allowing stakeholders to compare salary levels among various roles within the surveyed population. This chart helps identify disparities and trends in compensation across different job categories.

 - Demographic Insights:
The pie chart depicting gender distribution offers insights into the demographic composition of survey respondents, highlighting the gender diversity within the surveyed population. Additionally, the cards displaying average age and total survey takers provide demographic statistics that contribute to a better understanding of the surveyed cohort.

 - Programming Language Preferences:
The clustered column chart showcases the distribution of favorite programming languages based on the number of votes received. This chart offers insights into the popularity of different programming languages among survey respondents, aiding in understanding technological preferences within the surveyed population.

 - Geographical Representation:
The map feature allows users to visualize the geographical distribution of survey respondents, enabling them to identify concentrations of respondents across different regions. This geographical representation provides insights into the global reach and representation of the survey data.


<br/><br/>

## Dashboard image:

![dashboard image](https://github.com/DiogoGravanita/Data-Professional-Survey-Data-Cleaning-SQL-Power-BI-Project/assets/163042130/9d2be06d-df48-48df-ba14-0d18642807ad)


<br/><br/>
<br/><br/>
## Results/findings
<br/><br/>






### What is the distribution of job titles among data professionals?
The survey reveals a significant predominance of Data Analysts, comprising over 250 respondents, followed by students or those actively seeking jobs in the data domain, totaling 50+. Following closely are Data Engineers, Data Scientists, Business Analysts, and Database Developers.

![image](https://github.com/DiogoGravanita/Data-Professional-Survey-Data-Cleaning-SQL-Power-BI-Project/assets/163042130/e391612a-e4e5-4841-bb46-e9764428d6c6)


<br/><br/>

### Which programming language is the most favored among data professionals?
Python emerges as the most favored programming language among data professionals, followed by R and SQL.

![image](https://github.com/DiogoGravanita/Data-Professional-Survey-Data-Cleaning-SQL-Power-BI-Project/assets/163042130/8375baa8-1678-49be-b084-ae37b4372bae)


<br/><br/>

### What is the average yearly salary of data professionals across different industries?
The average yearly salary among data professionals stands at $53,000. Notably, Analytics Managers command the highest salaries, followed by Data Scientists, Analytics Engineers, and Data Engineers.


![image](https://github.com/DiogoGravanita/Data-Professional-Survey-Data-Cleaning-SQL-Power-BI-Project/assets/163042130/032671f2-becd-45a4-9a5c-a357cdad040d)


<br/><br/>

### How do salary levels vary based on gender?
The analysis indicates that both males and females earn similar average salaries.

![image](https://github.com/DiogoGravanita/Data-Professional-Survey-Data-Cleaning-SQL-Power-BI-Project/assets/163042130/0e14a43b-ebf1-4ccd-a2f1-66f4f309b697)


<br/><br/>

### What are the age demographics of data professionals?
The majority of survey respondents fall within the age bracket of 20 to 40 years, with an average age of 29.7 years.

![image](https://github.com/DiogoGravanita/Data-Professional-Survey-Data-Cleaning-SQL-Power-BI-Project/assets/163042130/ff7f9245-7be1-40cd-9e9b-a01a7aa9d250)
![image](https://github.com/DiogoGravanita/Data-Professional-Survey-Data-Cleaning-SQL-Power-BI-Project/assets/163042130/2605a506-83c7-4d51-8b24-48072d149796)


<br/><br/>

### Is there a relationship between educational attainment and salary levels?
On average, individuals with PHD doctorates command salaries exceeding $100,000, followed by those with master's degrees at $60,000. High school graduates and bachelor's degree holders earn approximately $50,000, with associate degree holders trailing closely.

![image](https://github.com/DiogoGravanita/Data-Professional-Survey-Data-Cleaning-SQL-Power-BI-Project/assets/163042130/2c5f9f69-0312-499d-a3e4-3d6a29b3982d)


<br/><br/>

### What is the ethnic diversity among data professionals?
Approximately 39% of survey respondents identified as White/Caucasian, followed by 25% Asian, 17% African American/Black, and subsequent percentages for Hispanics and Indians.


![image](https://github.com/DiogoGravanita/Data-Professional-Survey-Data-Cleaning-SQL-Power-BI-Project/assets/163042130/8e2bd64a-2b4b-466c-b568-00643df94044)

<br/><br/>

### Other Trends in the Data:
Upon analyzing the visualizations, it's evident that there's a relatively lower satisfaction level concerning salary (4.28 out of 10) compared to other aspects. Coworker happiness ranks highest at 5.87, closely followed by Work/Life Balance, while Management satisfaction lags slightly behind at 5.38.

Additionally, Python emerges as the overwhelmingly favored programming language, followed by R, while SQL, despite its significance in the field, registers comparatively lower favoritism.

<br/><br/>

## Conclusion:

In conclusion, this data cleaning and analysis project sheds light on various aspects of the data professional landscape, offering valuable insights into job titles, programming language preferences, salary trends, demographic characteristics, and educational attainment within the industry.

Our analysis reveals that Data Analysts constitute the majority of respondents, underlining the significance of this role in the data ecosystem. Python emerges as the preferred programming language, indicative of its versatility and widespread adoption across various data domains.

Salary trends showcase disparities based on job roles and educational backgrounds, with Analytics Managers commanding the highest salaries and individuals with PHD doctorates earning significantly more on average. However, gender-based salary discrepancies were not discernible in our analysis.

Demographic insights underscore the industry's diversity, with significant representation from various ethnic backgrounds. Moreover, the age distribution reveals a youthful workforce, with a substantial proportion falling within the 20 to 40 age bracket.

This project encapsulates not only the present state but also hints at future trajectories within the data professional landscape. By highlighting areas of strength and potential improvement, our findings serve as a foundational resource for stakeholders, policymakers, and aspiring data professionals alike, facilitating informed decision-making and strategic planning in this dynamic field.

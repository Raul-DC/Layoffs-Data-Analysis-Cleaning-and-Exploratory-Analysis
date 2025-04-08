# 👞😭 Global Layoffs Data Analysis Project 🔍

*Where the decay of IT jobs began...*

---

## Table of Contents

- [Introduction](#introduction)
- [Data Cleaning Process](#data-cleaning-process)
- [Exploratory Data Analysis (EDA)](#exploratory-data-analysis-eda)
- [Conclusion & Recommendations](#conclusion--recommendations)
- [Screenshot Guidelines](#screenshot-guidelines)

---

## Introduction

![image](https://github.com/user-attachments/assets/85fecb70-58de-495c-abb0-ed8cda3c9123)

In this project, raw data regarding global layoffs is first processed through a robust data cleaning workflow. After cleaning the data, an exploratory data analysis (EDA) is performed to reveal key insights about trends, distribution of layoffs by company, industry, country, and time intervals. The project simulates a real-world data pipeline—from raw data ingestion and preparation to a refined analytical reporting stage.

![image](https://github.com/user-attachments/assets/9138e9d0-c38f-4d73-be7b-1e5dd0fc9076)

---

## Data Cleaning Process 🧹

This section summarizes the steps followed to clean and standardize the dataset:

1. **📦 Database & Table Setup:**  
   - Initialize MySQL Workbench and create a new schema (named `world_layoffs`).
   - Use the Table Data Import Wizard to load the dataset into a working table.
   - Create a duplicate staging table to work on data without altering the original.

   ![image](https://github.com/user-attachments/assets/b7bab74b-988b-4533-ad88-47876c1f9027)

2. **🚫 Removal of Duplicates:**  
   - Execute queries using `ROW_NUMBER() OVER (...)` partitioning by key columns (company, location, etc.) to identify duplicate records.

   ![image](https://github.com/user-attachments/assets/0dd87dc0-efcd-45e9-a856-32c35c7a06af)
   *(Detection of duplicates)*

   - Apply a Common Table Expression (CTE) for detecting duplicates and then remove rows with `row_num > 1`.

  *We need to create a second staging table first*
   ```sql
   CREATE TABLE   `layoffs_staging2` (
	`company` text,
	`location` text,
	`industry` text,
	`total_laid_off` int DEFAULT NULL,
	`percentage_laid_off` text,
	`date` text,
	`stage` text,
	`country` text,
	`funds_raised_millions` int DEFAULT NULL,
	`row_num` int
	) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;
   ```
*Now we can proceed to copy the data*
 ```sql
INSERT INTO layoffs_staging2
	SELECT *,
	ROW_NUMBER() OVER(
	PARTITION BY company, location, industry, total_laid_off, percentage_laid_off, `date`, stage, country, funds_raised_millions
	) AS row_num
	FROM layoffs_staging;
```

*And now at last, we can delete the duplicates*
```sql
DELETE FROM layoffs_staging2
WHERE row_num >1;
```

3. **💾 Data Standardization:**
   
   - Trim strings and correct inconsistencies in text fields (e.g., standardizing multiple versions of "Crypto" and handling punctuation in country names).
     
   ```sql
   UPDATE layoffs_staging2
	  SET company = TRIM(company);
   ```

   ```sql
   UPDATE layoffs_staging2
	  SET industry = 'Crypto'
  	WHERE industry LIKE 'Crypto%';
   ```

   ```sql
   UPDATE layoffs_staging2
	  SET country = TRIM(TRAILING '.' FROM country)
  	WHERE country LIKE 'United States%';
   ```

   - Convert date strings into a proper date format using `STR_TO_DATE` and update the schema to switch the column type accordingly. ❗❗❗❗

   ```sql
   SELECT `date`,
	  STR_TO_DATE(`date`, '%m/%d/%Y')
	  FROM layoffs_staging2;
   ```

   ```sql
   UPDATE layoffs_staging2
	  SET `date` = STR_TO_DATE(`date`, '%m/%d/%Y')
   ```

   

5. **💬 Handling NULLs and Blank Values:**
   
   - Identify and inspect NULL or blank values in key columns.

   *There we saw the nulls*
   ```sql
   SELECT *
	  FROM layoffs_staging2
  	WHERE total_laid_off IS NULL;
   ```

   *There we saw which industries are blank or nulls*
   ```sql
   SELECT DISTINCT industry
	  FROM layoffs_staging2;
   ```

   *We found correlations among companies with null values*
   ```sql
   SELECT *
	  FROM layoffs_staging2
	  WHERE industry IS NULL
	  OR industry = '';
   ```
   *We gave a closer look at Airbnb's values*
   ```sql
   SELECT *
  	FROM layoffs_staging2
	  WHERE company = 'Airbnb';
   ```
   
   - Convert empty values to Null values for manipulation

   ```sql
   UPDATE layoffs_staging2
	  SET industry = NULL
	  WHERE industry = '';
   ```

   - Update records by joining duplicate entries, ensuring that missing information is populated with valid values wherever possible.
  
   ```sql
   UPDATE layoffs_staging2 t1
	  JOIN layoffs_staging2 t2
		 ON t1.company = t2.company
  	SET t1.industry = t2.industry
	  WHERE (t1.industry IS NULL OR t1.industry = '')
	  AND t2.industry IS NOT NULL;
   ```

   - Remove uninformative rows where critical numeric fields (e.g., total layoffs) are missing.

   ```sql
   DELETE
  	FROM layoffs_staging2
	  WHERE total_laid_off IS NULL
	  AND percentage_laid_off IS NULL;
   ```
   
  *How the table ended after the cleaning*
  ![image](https://github.com/user-attachments/assets/4a344406-0188-4022-a303-6bc79ef4f8c6)


7. **🗑️ Removal of Unnecessary Columns:**  
   - Drop helper columns (e.g., the temporary `row_num`) to streamline the final dataset.

   ```sql
   ALTER TABLE layoffs_staging2
  	DROP COLUMN row_num;
   ```

   - Verify the cleaned dataset with a final SELECT statement.

   ```sql
   SELECT * FROM layoffs_staging2
   ```

   ![image](https://github.com/user-attachments/assets/e86d0e3e-0aca-4505-93f5-1008ba830441)


---

## Exploratory Data Analysis (EDA) 🔎

With the cleaned dataset in place, the next phase is to perform an exploratory analysis to extract actionable insights:

1. **👓 Initial Data Examination:**

   - Run queries to display the overall dataset.

   ```sql
   SELECT * FROM layoffs_staging2
   ```

   - Use aggregate functions like `MAX()`, `MIN()` to understand the range of values in key fields such as `total_laid_off` and `percentage_laid_off`.

   > **[Screenshot Tag]**: Capture the output of summary queries (e.g., maximum and minimum layoffs).

3. **🏢 Analysis by Company:**  
   - Group data by company to compute the sum of layoffs.
   - Rank companies based on the number of layoffs, highlighting leaders like Amazon, Google, etc.
   - Identify companies that have experienced 100% workforce reduction.

   > **[Screenshot Tag]**: Show the result of the grouping query by company with rankings.

4. **🏭 Industry Analysis:**  
   - Aggregate layoffs by industry to pinpoint which sectors are most affected.
   - Present visual cues (if using visualization tools later) to compare industries such as Technology, Retail, Finance, etc.

   > **[Screenshot Tag]**: Display the grouped results by industry.

5. **🌎 Country Analysis:**  
   - Compute the total layoffs per country to see the global distribution.
   - Highlight specific trends, such as the notable figures for the United States and other key nations.

   > **[Screenshot Tag]**: Display output of country-wise aggregation queries.

6. **📆 Temporal Analysis:**  
   - Analyze the timeline of layoffs by extracting the year and month from date fields.
   - Use rolling totals to observe trends over months and identify the peak periods.
   - Implement window functions like `SUM() OVER (ORDER BY month)` to calculate cumulative totals.

   > **[Screenshot Tag]**: Capture a timeline chart (if available) or query result showing monthly totals and rolling sums.

7. **🛠️ Advanced Analysis:**  
   - Leverage ranking functions like `DENSE_RANK()` to determine yearly leaders.
   - Filter top companies per year to assess shifting trends over time.
   - Compare trends between different time periods (e.g., 2020 vs. 2021 vs. 2022).

   > **[Screenshot Tag]**: Screenshot of advanced ranking query results with top five companies per year.

---

## Conclusion & Recommendations 📝

After completing the EDA, the project reveals vital insights such as:
- The fluctuation of layoffs over time and the identification of peak periods.
- Which companies and industries bore the brunt of workforce reductions.
- Geographic trends that may inform broader economic research.

Based on these insights, future steps could include:
- Integrating visualization tools for a more intuitive understanding of trends.
- Further segmentation (e.g., by company size or market sector).
- In-depth statistical analysis or predictive modeling for proactive decision-making.

> **[Screenshot Tag]**: Optionally, include a final summary dashboard or key insights overview captured from your analysis tool.

---

## Screenshot Guidelines

For clarity and visual support, here se sugieren etiquetas para las capturas de pantalla en cada sección:

- **Introduction:**  
  - *Screenshot of the raw dataset overview in MySQL Workbench.*  

- **Data Cleaning Process:**  
  - *Schema creation & data import screen.*  
  - *Query output showing duplicate detection and the deletion process.*  
  - *Before-and-after snapshots of data standardization (dates, text fields).*  
  - *Final cleaned table display in the MySQL interface.*

- **Exploratory Data Analysis (EDA):**  
  - *Query results of descriptive statistics (max, min values).*  
  - *Grouping and ranking results by company, industry, and country.*  
  - *Temporal analysis outputs with monthly breakdowns and rolling totals.*  
  - *Advanced ranking outputs (top five companies per year).*

- **Conclusion & Recommendations:**  
  - *Screenshot of an overview dashboard or key insights summary (si aplicable) que resuma las recomendaciones finales.*

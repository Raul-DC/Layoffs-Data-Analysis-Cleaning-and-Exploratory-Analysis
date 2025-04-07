# 👞 Layoffs Data Analysis 🔍

![image](https://github.com/user-attachments/assets/85fecb70-58de-495c-abb0-ed8cda3c9123)

 This project combines data cleaning and exploratory data analysis (EDA) on a global layoffs dataset from 2020-2023. Using MySQL, I standardized the dataset by removing duplicates, handling null values, and correcting inconsistencies. The EDA phase reveals trends by company, industry, country, and time, including rolling totals and ranked insights. Key findings highlight the most impacted sectors (e.g., Crypto, Retail) and temporal spikes in layoffs, with advanced techniques like window functions and CTEs. Ideal for showcasing SQL proficiency and analytical storytelling.

This project analyzes global layoffs data through a two-phase approach: 

1. **Data Cleaning**: Prepared raw data by removing duplicates, standardizing formats (e.g., dates, industries), and addressing null values.  
2. **Exploratory Analysis**: Investigated trends across companies, industries, and timelines, using aggregations, rolling totals, and rankings.  


---

### 🔑 **Key Sections**  

#### 1. 🧹 **Data Cleaning**  
- **Objective**: Transform raw data into a reliable dataset.  
- **Steps**:  
  - Removed duplicates via `ROW_NUMBER()` and staging tables.  
  - Standardized industries (e.g., merged "Crypto Currency" variants).  
  - Fixed date formats and handled nulls with joins.  
- 📌 **Screenshot Suggestions**:  
  - Before/after deduplication query results.  
  - Example of industry standardization (e.g., "Crypto" variants).  

#### 2. 🔎 **Exploratory Analysis**  
- **Insights**:  
  - **Top Companies**: Amazon, Google, and Meta led layoffs.  
  - **Industries**: Consumer, Retail, and Crypto were most impacted.  
  - **Temporal Trends**: 2022 saw the highest layoffs (160K+), with spikes in March 2023.  
- 📌 **Screenshot Suggestions**:  
  - Rolling total layoffs by month (CTE output).  
  - Top 5 companies by year (DENSE_RANK results).  

#### 3. 🛠️ **Advanced Techniques**  
- **Rolling Totals**: Tracked cumulative layoffs monthly.  
- **Rankings**: Used `DENSE_RANK()` to compare companies annually.  
- 📌 **Screenshot Suggestions**:  
  - SQL query for rolling totals.  
  - Ranked table snippet (e.g., Uber #1 in 2020).  

#### 4. 📝 **Conclusion**  
- **Summary**: Layoffs peaked post-2021, driven by tech and consumer sectors.  
- **Recommendations**: Further analysis could link layoffs to funding stages or economic events.  

---

### 🧬 **How to Reproduce** 
1. Clone the repository.  
2. Import the dataset into MySQL using the `Table Data Import Wizard`.  
3. Run queries sequentially from the scripts provided.  

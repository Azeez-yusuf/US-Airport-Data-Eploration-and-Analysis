# US-Airport-Data-Eploration-and-Analysis
This project is undertaken for academic and exploratory purposes, utilizing datasets across US airports, airlines, flights, and cancellation codes. The primary objective is to analyze historical flight data to uncover key trends, identify seasonal patterns, and isolate the primary factors influencing US air transportation efficiency and disruptions.

## The Raw Data
The dataset for this project was sourced from data.gov as a ZIP archive containing four CSV files: airports, airlines, flights, and cancellation codes. To ensure clarity, consistency, and analytical readiness, the raw data underwent a structured preprocessing phase. This involved redefining key variables, removing irrelevant columns, and engineering new fields to enhance data quality and depth.

## Tools and Steps
•	EXCEL; Used for:
1. Data loading and inspection
2. Data manipulations
3. Handling missing values,
4. Cleaning and
5. Formatting
#### Formulae used in Excel
1. =TRIM(CLEAN(CONCAT(IF(CODE(MID(B2,ROW(INDIRECT("1:"&LEN(B2))),1))>=127,"",CHAR(CODE(MID(B2,ROW(INDIRECT("1:"&LEN(B2))),1))))))) : To remove all speacial characters
2. =TIME(INT(A2/100), MOD(A2,100), 0) : To convert 3 or 4 digits integer time, to standard time
3. =TIME(LEFT(H2,2), RIGHT(H2,2), 0) : To convert 4 digit time ( such as '0005 to 12:05 ) to standard time

   
•  SQL; Steps took involved:
1. Loading the refined datasets into the Database
2. Feature Engineering
3. Analysis

• Power BI; Steps took involved:
1. Transformation
2. Feature Engineering
3. Analysis
4. Visualizations

## Challenges
•	Challenge  1: Several improper casing of alphabets, a lot of misspellings, duplicates columns and rows, values errors, nulls and missing values were observed in the raw dataset.
      o	Solution: I use MS EXCEL to perform data manipulations, transformations, cleaning and formatting

•	Challenge 2: Another hurdle was loading the dataset to SQL server base, this was as a result of incorrect formatting of the CSV file to correspond with the SQL data type format
      o	Solution: This was also handled using Excel and SQL

• Challenge 3: After successive analysis in Sql, Creating successful relationships between exported files was herculian task in Power BI
      o	Solution: This was done by noting the type of relationship between the export files and applying them
      
## Feature Engineering
Feature engineering in SQL for data analysis involves strategically crafting new, informative features from existing data to enhance model performance and extract deeper insights.

## Feature Engineering CODES

#### DataBase Creation AND Importation Process

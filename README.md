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


-- ------------------------------------------------------------------------------------------------------------------------------------------------

```sql

CREATE DATABASE IF NOT EXISTS US_Flights_2015;
USE US_Flights_2015;

-- -----------------------------------------------------
-- Table 1: airlines (Dimension)
-- -----------------------------------------------------
DROP TABLE IF EXISTS airlines;
CREATE TABLE airlines (
    IATA_CODE CHAR(2) PRIMARY KEY,
    AIRLINE VARCHAR(100) NOT NULL
);

-- -----------------------------------------------------
-- Table 2: airports (Dimension)
-- -----------------------------------------------------
DROP TABLE IF EXISTS airports;
CREATE TABLE airports (
    IATA_CODE CHAR(3) PRIMARY KEY,
    AIRPORT VARCHAR(150) NOT NULL,
    CITY VARCHAR(100) NOT NULL,
    STATE CHAR(2) NOT NULL,
    COUNTRY VARCHAR(50) NOT NULL,
    LATITUDE DECIMAL(8, 5) NULL,    -- Accommodates missing entries
    LONGITUDE DECIMAL(8, 5) NULL    -- Accommodates missing entries
);

-- -----------------------------------------------------
-- Table 3: cancellation_codes (Dimension)
-- -----------------------------------------------------
DROP TABLE IF EXISTS cancellation_codes;
CREATE TABLE cancellation_codes (
    CANCELLATION_REASON CHAR(1) PRIMARY KEY,
    CANCELLATION_DESCRIPTION VARCHAR(100) NOT NULL
);

-- -----------------------------------------------------
-- Table 4: flights (Fact Table)
-- -----------------------------------------------------
DROP TABLE IF EXISTS flights;
CREATE TABLE flights (
    DATE DATE NOT NULL,              -- Native YYYY-MM-DD format matches perfectly
    DAY_OF_WEEK TINYINT NOT NULL,
    AIRLINE CHAR(2) NOT NULL,
    FLIGHT_NUMBER INT NOT NULL,
    TAIL_NUMBER VARCHAR(10) NULL,
    ORIGIN_AIRPORT CHAR(3) NOT NULL,
    DESTINATION_AIRPORT CHAR(3) NOT NULL,
    SCHEDULED_DEPARTURE TIME NOT NULL,
    DEPARTURE_TIME VARCHAR(10) NULL,  -- Staged as VARCHAR to prevent import crashes on blank cells
    DEPARTURE_DELAY INT NULL,
    TAXI_OUT INT NULL,
    WHEELS_OFF VARCHAR(10) NULL,     -- Staged as VARCHAR
    SCHEDULED_TIME TIME NULL,
    ELAPSED_TIME TIME NULL,
    AIR_TIME TIME NULL,
    DISTANCE INT NOT NULL,
    WHEELS_ON VARCHAR(10) NULL,      -- Staged as VARCHAR
    TAXI_IN INT NULL,
    SCHEDULED_ARRIVAL TIME NOT NULL,
    ARRIVAL_TIME VARCHAR(10) NULL,    -- Staged as VARCHAR
    ARRIVAL_DELAY INT NULL,
    DIVERTED TINYINT NOT NULL,        -- 0 or 1 representation
    CANCELLED TINYINT NOT NULL,       -- 0 or 1 representation
    CANCELLATION_REASON CHAR(1) NULL,
    AIR_SYSTEM_DELAY INT NULL,
    SECURITY_DELAY INT NULL,
    AIRLINE_DELAY INT NULL,
    LATE_AIRCRAFT_DELAY INT NULL,
    WEATHER_DELAY INT NULL,
    
    -- Enforce Star Schema Relational Integrity
    CONSTRAINT fk_flights_airline FOREIGN KEY (AIRLINE) REFERENCES airlines(IATA_CODE),
    CONSTRAINT fk_flights_origin FOREIGN KEY (ORIGIN_AIRPORT) REFERENCES airports(IATA_CODE),
    CONSTRAINT fk_flights_dest FOREIGN KEY (DESTINATION_AIRPORT) REFERENCES airports(IATA_CODE),
    CONSTRAINT fk_flights_cancellation FOREIGN KEY (CANCELLATION_REASON) REFERENCES cancellation_codes(CANCELLATION_REASON)
);

```

### FEATURE ENGINEERING  

-- ----------------------------------Post-Load Optimization Script------------------------------------------------------------------------

```sql

-- Turn off the safety guardrail ---------------------------------
SET SQL_SAFE_UPDATES = 0;

-- 1. Remove the trailing spaces found in the airlines dataset
UPDATE airlines 
SET AIRLINE = TRIM(AIRLINE);

-- 2. Clean empty strings to genuine SQL NULLs in the flights table
UPDATE flights 
SET 
    DEPARTURE_TIME = NULLIF(DEPARTURE_TIME, ''),
    WHEELS_OFF = NULLIF(WHEELS_OFF, ''),
    WHEELS_ON = NULLIF(WHEELS_ON, ''),
    ARRIVAL_TIME = NULLIF(ARRIVAL_TIME, ''),
    CANCELLATION_REASON = NULLIF(CANCELLATION_REASON, '');

-- 3. Safely optimize columns from VARCHAR storage to native structural TIME storage
ALTER TABLE flights 
    MODIFY COLUMN DEPARTURE_TIME TIME NULL,
    MODIFY COLUMN WHEELS_OFF TIME NULL,
    MODIFY COLUMN WHEELS_ON TIME NULL,
    MODIFY COLUMN ARRIVAL_TIME TIME NULL;


 --  Turn on the safety guardrail ---------------------------------------------
SET SQL_SAFE_UPDATES = 1;


```


-- ----------------------------------------------------------------------------------------------------------------------------

-- CORE ANALYSIS ---------------------------------------------------------------------------------------------------------------




## Results/Findings
### A. Carrier Operational Resilience (Monthly Trended Degradation)  
Overall flight volume significantly dropped from Month 1 to Month 2 across the entire aviation network. Despite lower flight numbers, operational resilience was highly volatile: while traditional heavyweights like American Airlines drastically reduced their average delays, several carriers—most notably Virgin America and JetBlue—suffered severe operational bottlenecks, seeing their average delays spike by over 100%.

#### Key Insights
1. The Spike Vulnerability (Virgin America & JetBlue):

-  Virgin America saw its average departure delays skyrocket from 7.7 minutes in Month 1 to 25.7 minutes in Month 2.

-  JetBlue Airways followed a similar negative trend, with departure delays worsening from 12.1 minutes to 19.9 minutes.

**Business Translation:** These carriers failed to stabilize operations even when handling fewer flights, pointing to deep underlying systemic or staffing constraints.

2. The Efficiency Champions (American Airlines & Southwest):

- American Airlines managed to cut its average departure delays nearly in half (13.9 minutes down to 7.1 minutes).

- Southwest Airlines Co, despite remaining the highest-volume carrier by a wide margin, successfully reduced its average arrival delays from 6.6 minutes down to a negligible 1.5 minutes.

**Business Translation:** These organizations effectively optimized scheduling and ground-handling efficiencies as capacity tightened.

3. Persistent Bottom-Performers (Frontier & American Eagle):

- Frontier Airlines and American Eagle consistently registered severe delays in both months, routinely averaging 20+ minutes per departure.

**Business Translation:** Their issues are chronic and structural rather than seasonal flukes.

## Recommendations
A. Carrier Operational Resilience (Monthly Trended Degradation)
1. Conduct a Root-Cause Deep Dive on Outliers: Investigate what drove the massive operational degradation for Virgin America and JetBlue in Month 2. Was it localized weather at major hubs, tech outages, or labor shortages?

2. Review Service Level Agreements (SLAs): If these carriers are core corporate partners or vendors, use these performance numbers to audit contract compliance and renegotiate terms based on actual reliability data.


### B. Q1 Carrier Reliability & Flight Cancellation Analysis  
During the first quarter (Q1), overall carrier reliability was heavily split. American Eagle Airlines Inc emerged as a massive operational risk outlier, cancelling more than 1 in every 10 scheduled flights—a rate more than double that of any other carrier. On the flip side, industry heavyweights like Delta Air Lines and regional operators like Hawaiian Airlines demonstrated outstanding stability, maintaining cancellation rates well below 1%.  

#### Key Insights
1. The Severe Operational Outlier (American Eagle):  
- American Eagle Airlines Inc scheduled 65,459 flights but cancelled a staggering 6,923 of them, resulting in a 10.58% cancellation rate.

**Business Translation:** This is a critical operational failure. A double-digit cancellation rate means business travelers face extreme uncertainty, creating major disruptions to downstream operations and schedules.

2. The High-Risk Tier ( ~ 4% Cancellation Rate):
- JetBlue Airways (4.24\%) and Atlantic Southeast Airlines (4.14\%) represent the next highest tier of disruption. While much lower than American Eagle, canceling over 4% of flights still represents a significant risk compared to industry averages.  

3.  The Reliability Superstars (Under 1%):
- Hawaiian Airlines Inc (0.30%), Alaska Airlines Inc (0.54%), and Delta Air Lines Inc (0.89\%) proved to be the most resilient carriers in the network.

**Business Translation:** Delta's performance is especially impressive because they achieved a sub-1% cancellation rate while managing a massive volume of 140,355 scheduled flights.  

4. The High-Volume Anchor (Southwest):
- Southwest Airlines Co operated the largest network by far with 218,098 scheduled flights. They maintained a stable cancellation rate of 1.99% (4,337 cancellations). Given their massive footprint, keeping cancellations around 2% indicates strong baseline operational resilience.

  
## Recommendations
A. Implement a Carrier Hierarchy/Policy Rule: For corporate travel and high-priority logistics, implement a routing policy that prioritizes "Tier 1" resilient carriers (Delta, Alaska, Southwest) and actively avoids or flags "High-Risk" carriers like American Eagle unless absolutely necessary.  

B. Supplier Audit Engagement: If American Eagle or JetBlue are under contract as primary vendors, share this $Q1$ data with their account management teams to demand a formal corrective action plan regarding their capacity constraints.


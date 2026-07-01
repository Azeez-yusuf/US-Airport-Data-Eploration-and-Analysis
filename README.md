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
**Executive Overview:** Overall flight volume significantly dropped from Month 1 to Month 2 across the entire aviation network. Despite lower flight numbers, operational resilience was highly volatile: while traditional heavyweights like American Airlines drastically reduced their average delays, several carriers—most notably Virgin America and JetBlue—suffered severe operational bottlenecks, seeing their average delays spike by over 100%.

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

### Recommendations
A. Carrier Operational Resilience (Monthly Trended Degradation)
1. Conduct a Root-Cause Deep Dive on Outliers: Investigate what drove the massive operational degradation for Virgin America and JetBlue in Month 2. Was it localized weather at major hubs, tech outages, or labor shortages?

2. Review Service Level Agreements (SLAs): If these carriers are core corporate partners or vendors, use these performance numbers to audit contract compliance and renegotiate terms based on actual reliability data.


### B. Q1 Carrier Reliability & Flight Cancellation Analysis  
**Executive Overview:** During the first quarter (Q1), overall carrier reliability was heavily split. American Eagle Airlines Inc emerged as a massive operational risk outlier, cancelling more than 1 in every 10 scheduled flights—a rate more than double that of any other carrier. On the flip side, industry heavyweights like Delta Air Lines and regional operators like Hawaiian Airlines demonstrated outstanding stability, maintaining cancellation rates well below 1%.  

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

  
### Recommendations
1. Implement a Carrier Hierarchy/Policy Rule: For corporate travel and high-priority logistics, implement a routing policy that prioritizes "Tier 1" resilient carriers (Delta, Alaska, Southwest) and actively avoids or flags "High-Risk" carriers like American Eagle unless absolutely necessary.  

2. Supplier Audit Engagement: If American Eagle or JetBlue are under contract as primary vendors, share this $Q1$ data with their account management teams to demand a formal corrective action plan regarding their capacity constraints.



### C. Root-Cause Attrition Mix Changes Over Time  
**Executive Overview:** The core driver behind flight cancellations underwent a dramatic shift between the two months. In Month 2, overall flight cancellations fell, but severe Weather became the absolute dominant cause of disruptions, growing to trigger nearly 3 out of every 4 cancellations. Conversely, internal, self-inflicted Airline or Carrier operational failures dropped sharply.

#### Key Insights  
1. The Weather Surge:
- In Month 1, weather was already the leading cause of disruptions, accounting for 49.26% (10,669 flights) of cancellations.

- By Month 2, weather surged to account for a staggering 71.52% (5,750 flights) of all canceled operations.

**Business Translation:** While the absolute number of flights canceled went down, external environmental factors completely dominated Month 2. When disruptions occurred, they were overwhelmingly caused by mother nature rather than internal errors.  

2. Dramatic Internal Recovery (Airline or Carrier Operations):
- Internal carrier issues (such as maintenance failures, missing crews, or aircraft availability problems) dropped significantly from 30.46\% in Month 1 to just 11.37% in Month 2.

**Business Translation:** This is a massive positive sign for airline operations. It indicates that carriers successfully ironed out their own internal logistical problems, crew shortages, and technical bottlenecks between the two periods.

3. Stable Infrastructure (National Air System):
- Disruption contributions from the National Air System (which includes air traffic control limits, airport capacity constraints, and airport security bottlenecks) remained relatively steady, shifting slightly from 20.27% down to 17.10%.

**Business Translation:** Heavy airport congestion and air traffic control constraints acted as a consistent, baseline pressure across both months but did not cause major new disruptions.  

3. Security Disruptions are Negligible:
- Security breaches or airport evacuations remained entirely statistically irrelevant at 0.01% for both months.


### Recommendations
1. Cross-Reference Hub Locations: Map out the primary hub airports for Virgin America and JetBlue to confirm if regional weather events match this 71.52% surge. This will help determine if their high delays were due to poor performance or simply bad luck with geographic weather patterns.

2. Optimize Bad-Weather Contingency Plans: Because weather is an uncontrollable risk, focus supplier conversations on recovery speed rather than prevention. Audit how quickly carriers get their schedules back on track once weather conditions clear up.


### D. Macro Delays—Controllable vs. Uncontrollable Cost Drivers  
**Executive Overview:** The single biggest operational driver of flight delays is Cascading Impact (the network effect of an early-morning delayed flight disrupting a separate, late-afternoon flight), accounting for a staggering 24 minutes of delay per impacted plane across both months. While internal carrier efficiencies stayed flat, severe bottlenecking in national infrastructure (air traffic control) and worsening weather pushed overall system delays higher in Month 2.

#### Key Insights 

1. The Dominant Threat: Cascading Delays (Uncontrollable / Reactive):

- Average cascading delays held firm at an incredibly high 24.02 minutes in Month 1 and 23.93 minutes in Month 2.  

**Business Translation:** This is the "domino effect." When an aircraft gets delayed early in the day due to weather or maintenance, that exact same plane remains late for its next 4 or 5 scheduled flights. It represents a massive system-wide vulnerability.

2. The Internal Operational Floor: Carrier Delays (Controllable):

- Internal airline-controlled delays (plane cleaning, baggage handling, mechanical fixes) remained virtually unchanged, moving slightly from 17.94 minutes to 18.31 minutes.  

**Business Translation:** Airlines are operating at a consistent operational baseline. Their internal processes are stable, but they aren't improving, meaning they remain a fixed, predictable expense/risk factor.

3. The Rising External Pressures: Infrastructure & Weather (Uncontrollable):

- National Air System (NAS) Delays jumped from 13.49 minutes to 15.99 minutes.

- Direct Extreme Weather Delays shot up by over 70%, from 3.01 minutes to 5.18 minutes.

**Business Translation:** The system is getting tighter. Increased air traffic control holds (NAS) combined with worsening weather mean the broader aviation environment became more hostile in Month 2.


### Recommendations
1. Prioritize Hub Isolation Strategies: Partner with carriers that employ "point-to-point" networks or build substantial ground-time buffers at their primary hubs. This keeps a localized delay from bleeding out across the rest of the operational network. 

2. Financial Contingency Adjustments: Since infrastructural (NAS) and weather delays are actively trending upward, increase Q2 operational risk/cost buffers by 8–10% to account for higher anticipated crew timeouts and passenger rebooking expenses.

### E. Persistent Hub Gridlocks (High-Volume Origin Analysis)  
**Executive Overview:** The worst departure gridlocks are concentrated in regional and feeder airports rather than massive international hubs. Wilmington Airport (ILG) ranks as the most disrupted origin point in the network, keeping flights waiting for an average of nearly 40 minutes before takeoff. Crucially, while these airports suffer extreme departure delays, their cancellation rates remain at zero, indicating that regional gridlocks cause massive schedule friction but do not completely break flight operations.

#### Key Insights   
1. The Top 3 Operational Bottlenecks:
- Wilmington Airport (ILG, DE): Highest average delay at 39.20 minutes across 54 total departures.

- University Park Airport (SCE, PA): Averaged 35.21 minutes of delay across 138 departures.

- Del Norte County Airport (CEC, CA): Averaged 33.33 minutes of delay across 125 departures.

**Business Translation:** Flights departing from these locations are systematically falling behind schedule by over half an hour, severely compressing tight business windows.  

2. The Extreme "Feeder" Exposure:
- Airports like Cherry Capital Airport (TVC, MI) manage noticeably higher regional traffic ($383$ departures) while still saddled with a painful 28.91-minute average delay.
- 
**Business Translation:** TVC represents the largest absolute breakdown point in this list. Because it handles more flights, its 29-minute delay is injecting significant downstream disruption back into major airline networks.
  
3. Delays vs. Cancellations (The Silver Lining):
- Every single airport on this top 10 list recorded 0 total cancellations.
  
**Business Translation:** The planes will fly, but they will almost certainly be late. This reinforces our macro delay analysis: regional delays are likely feeding the massive "Cascading Late Aircraft" domino effect we observed globally.


### Recommendations
1. Re-Route Time-Sensitive Travel: Avoid booking tight, same-day connections if the origin flight is departing from any of these top 10 bottleneck locations (especially TVC, SCE, or ILG). Build a mandatory 90-minute layover buffer for connections involving these regional nodes.

2. Regional Fleet Strategy Audit: If our business operations rely on regional cargo or charter pipelines through these areas, audit whether the delay is carrier-specific or an infrastructure limitation at the airport itself (e.g., lack of de-icing equipment or limited runway capacity).


### F. Ground Control Congestion & Tarmac Efficiencies  
**Executive Overview:** Ground bottlenecks are driven by two completely different phenomena: massive, high-volume international hubs operating at capacity, and tiny regional transit stations prone to local constraints. John F. Kennedy International Airport (JFK) and LaGuardia Airport (LGA) top the charts, forcing planes to sit on the tarmac for an average of 26 minutes per flight before takeoff. Alarmingly, the data reveals worst-case extreme scenarios where a single aircraft was trapped on a taxiway for nearly 3 hours before getting airborne.

  
#### Key Insights 
1. The High-Volume Gateways (The Expected Bottlenecks):

- JFK Airport (17,767 departures) and LaGuardia Airport (18,742 departures) lead the network in ground delays, with average tarmac wait times of 26.04 minutes and 25.86 minutes respectively.

**Business Translation:** Because these airports process an enormous volume of heavy jets, physical runway congestion is a chronic, predictable issue.

2. The Small Regional Traps (The Surprise Bottlenecks):

- Small airports like Pellston Regional (PLN) and Falls International (INL) have tiny flight volumes (around 100 departures total) yet their average tarmac times match JFK at over 25 minutes.

**Business Translation:** These smaller airfields aren't suffering from traffic congestion. Instead, their tarmac delays are typically caused by limited infrastructure—such as having only a single active runway, needing manual aircraft de-icing, or experiencing localized regional ground staffing shortages.

3. The Extreme Risks (Worst-Case Taxi Outages):

- The worst_single_taxi_out_mins column uncovers massive operational breakdowns. At LaGuardia (LGA), a single flight sat on the tarmac for 177 minutes (nearly 3 hours). Similarly, Detroit (DTW) and JFK recorded nightmare peak taxi times of 159 minutes and 155 minutes respectively.

**Business Translation:** These extreme tail risks are massive pain points for passenger satisfaction and can trigger severe crew-timeout violations.

### REcommendations
1. Re-evaluate Turnaround Buffers at Major Hubs: Ensure that flights operating through the New York metro area (JFK/LGA) or Detroit (DTW) have an extra 30-minute padding built directly into their arrival-to-departure windows to handle ground congestion without delaying the next leg.

2. Tarmac Delay Policy Monitoring: Monitor carriers flying out of LGA, DTW, and JFK closely. Since their peak taxi-out times hover close to the 3-hour mark (155–177 minutes), they run a high risk of crossing federal passenger protection limits, which can result in steep regulatory fines.


### G. High-Risk Route Volatility Corridor Analysis
**Executive Overview:** The most volatile flight corridors in our network are low-frequency regional connections, often servicing resort towns or smaller industrial cities. The single worst performing corridor is Jackson, WY to New York, NY (JAC → JFK), which has a 100% delay rate and keeps arriving passengers waiting an average of nearly 2.5 hours late. Multiple routes on this top 10 list present a total certainty of delay (100% probability), making them severe bottlenecks for time-sensitive travel.

#### Key Insights 
1. The Guaranteed Delay Routes (100% Probability): Four specific routes operate with a 100% probability of delay. Every single flight analyzed on these routes arrived late;  
- Jackson (JAC) → New York (JFK): Averaging a massive 149.00 minutes of arrival delay.
- Hayden (HDN) → Denver (DEN): Averaging 121.00 minutes of arrival delay.Trenton (TTN) → Detroit (DTW): Averaging 114.33 minutes of arrival delay.
- Detroit (DTW) → Trenton (TTN): Averaging 109.67 minutes of arrival delay.
**Business Translation:** These corridors are fundamentally broken from a scheduling standpoint. A 100% delay rate indicates a systemic flaw—likely due to severe regional winter weather patterns (e.g., ski destinations like Jackson Hole and Hayden) combined with air traffic flow restrictions at major hubs.
2. The Trenton-Detroit Double Bottleneck:
- The connection between Trenton, NJ (TTN) and Detroit, MI (DTW) is a severe problem in both directions. Flying to Detroit averages a 114-minute delay, while the return flight back to Trenton averages a 109-minute delay. Both carry a 100% delay guarantee.
**Business Translation:** The operational teams servicing this specific route are trapped in a vicious loop where the morning delay in one direction automatically dooms the afternoon leg in the other direction.
3. High-Volume Risk Exposure:
- While routes like Wilmington to Tampa (ILG → TPA) have slightly lower average delays (86.10 minutes), they have higher flight frequencies (20 Q1 flights).
**Business Translation:** Because this route runs more frequently, it accumulates a larger absolute volume of delayed hours, creating a bigger overall drag on corporate schedules.

### REcommendations
1. Blacklist or Embargo the 100% Risk Routes: Issue an immediate policy directive advising corporate travel planners to avoid booking direct flights on the top four volatile corridors (JAC → JFK, HDN → DEN, TTN ↔ DTW). For example, travelers going to New York from Jackson should route through a larger hub with higher frequency buffers.

2. Enforce Alternate Transit for Short Corridors: For short-haul regional corridors experiencing massive delays—such as Trenton (TTN) to Detroit (DTW) or vice versa—evaluate whether alternative transit options or shifting to virtual meetings is a much more reliable business decision.



### H. Diurnal (Time-of-Day) Inefficiency S-Curves  
**Executive Overview:** Flight performance degrades systematically as the operating day progresses. While early flights (Block 0) enjoy minimal departure delays averaging just 5.93 minutes, late-day operations (Block 2) experience a massive spike, with delays stretching to 16.10 minutes—an increase of nearly 172%. Late-day flights also carry the highest overall risk of outright cancellation at 3.18%.

#### Key Insights 
1. The Early Window Advantage (Block 0):  

- Volume: 293,593 flights.

- Performance: Boasts the lowest departure delays (5.93 minutes) and minimal arrival friction (2.78 minutes).

**Business Translation:** This is the clean-slate window. Early in the operating cycle, airports are un-congested and planes are generally positioned where they need to be, resulting in maximum reliability.

2. The Mid-Day Volume Crunch (Block 1): 

- Volume: 642,297 flights—by far the heaviest traffic load in the dataset.

- Performance: Average departure delays more than double compared to the morning, jumping to 14.52 minutes. However, cancellations drop to their lowest point at 2.78%.

**Business Translation:** As volume peaks, infrastructure pushes to its absolute limits, driving up delays. However, airlines fight aggressively to keep planes moving rather than canceling them during peak hours to preserve network flow.

3. The Late-Day Backlog Wall (Block 2):  

- Volume: 92,592 flights.

- Performance: Suffers the worst operational metrics across the board, with departure delays hitting 16.10 minutes and cancellations peaking at 3.18%.

**Business Translation:** This block inherits the compounding exhaustion of the entire day. The delay curve peaks here because all the minor morning hiccups have ballooned into massive late-day bottlenecks, frequently resulting in safety or crew-timeout cancellations.


### Recommendations
1. Shift Mission-Critical Travel to the Morning: Mandate an internal policy prioritizing early-block flights (Block 0) for executive travel, tight connection routings, or high-priority, time-sensitive cargo shipments.

2. Re-Optimize Late-Day Flight Buffers: For any critical operations scheduled during the final block (Block 2), mandate a minimum 2-hour downstream scheduling pad to safely absorb the baseline 16-minute delay reality.



### I. Long-Haul Fleet Protection vs. Short-Haul Commuter Sacrifice  
**Executive Overview:** Airlines actively protect long-haul routes at the direct expense of short-haul commuter flights. While departure delays are relatively similar across all flight types, Short-Haul routes (<500 miles) suffer a cancellation rate of 3.79%—more than double the 1.64% cancellation rate of Long-Haul routes (>1500 miles). Long-haul flights also successfully make up time in the air, boasting the lowest arrival delays.

#### Key Insights 
1. The Short-Haul "Sacrifice" Tier (<500 miles):

- Volume: 383,322 flights.

- Performance: Faces a steep 3.79% cancellation rate and the highest average arrival delay at 9.77 minutes.

**Business Translation:** When operations melt down, airlines prefer to cancel short commuter flights. Passengers can easily be rebooked onto another flight a few hours later, or moved to ground transit, making these routes the primary operational "sacrificial lambs."

2. The Long-Haul "Protected" Tier (>1500 miles):

- Volume: 128,118 flights.

- Performance: Enjoys a highly stable 1.64% cancellation rate and a significantly lower arrival delay of just 5.23 minutes.

**Business Translation:** Long-haul flights utilize large, expensive aircraft and generate massive revenue. Canceling a long-haul flight leaves hundreds of passengers stranded across continents and messes up international aircraft positioning, so airlines protect them at all costs.

3. The Mid-Air Correction Phenomenon:

- Interestingly, while Long-Haul departure delays average 12.48 minutes, their arrival delays drop to 5.23 minutes.

**Business Translation:** On longer routes, pilots have the cruise time necessary to fly slightly faster and burn extra fuel to make up for ground delays, ensuring they arrive nearly on time. Short-haul flights do not have this luxury; a 11.33-minute ground delay translates directly into a 9.77-minute late arrival because the flight time is too short to optimize.


#### Recommendation
1. Mandate Alternative Regional Travel Policies: For critical business travel under 300 miles, evaluate rail or vehicle alternatives. Shifting away from short-haul regional flights completely avoids the high 3.79% cancellation risk sector.

2. Buffer Hub-and-Spoke Connections: If a traveler must take a short-haul feeder flight to connect to a long-haul flight, ensure the short-haul leg is scheduled with an aggressive buffer window, recognizing that it is the most likely link in the chain to be delayed or cancelled.


### J. Chronic Asset Degradation & Capital Risk (The Tail-Number Audit)
**Executive Overview:** Our operational disruptions aren't just caused by external weather or broad logistics; specific physical aircraft are chronically failing. The absolute worst-performing asset in the network, plane N79011 (operated by United Air Lines), averages a staggering 233 minutes of delay on every single flight it takes. United Air Lines operates 8 out of the top 10 most degraded aircraft on this high-risk list, exposing a concentrated hardware maintenance or fleet aging bottleneck.  

#### Key Insights 
1. The Severe Capital Liabilities (The Worst Outliers):

- Plane N79011 (United): Floped 5 legs but cost passengers 1,166 lost minutes, averaging 233.20 delay minutes per flight.

- Plane N228UA (United): Flew 6 legs, causing 1,323 lost minutes, averaging 220.50 delay minutes per flight.

**Business Translation:** These two airframes are severe operational bottlenecks. Averaging over 3.5 hours late per takeoff means these individual planes are likely experiencing chronic mechanical faults, part procurement delays, or severe technical issues.

2. The High-Exposure Revenue Drain (Plane N5FSAA):

- While other aircraft on the list flew very few legs, N5FSAA (operated by American Airlines) logged a high volume of 41 flights. Despite this high utilization, it cost the network a massive 4,546 lost passenger minutes, averaging 110.88 minutes late per flight.

**Business Translation:** This is the most financially damaging single asset on the list. Because it is actively being flown heavily while averaging a nearly 2-hour delay per leg, it is actively injecting massive, predictable friction into American Airlines' broader daily schedule.

3. United Air Lines' Fleet Vulnerability:

- Out of the 10 worst planes in the entire network, 8 are operated by United Air Lines (all with the "UA" or specific numerical tail identifiers).

**Business Translation:** This indicates that United's operational delays are heavily tied to specific, underperforming physical assets in their fleet, rather than just broad systemic issues.


#### Recommendation
1. Incorporate Tail-Number Alerts into Logistics Procurement: Work with your software engineering or data teams to flag these specific 10 tail numbers in your logistics tracking pipelines. If high-priority corporate cargo or VIP travel is scheduled on any of these aircraft, trigger an automatic re-routing.

2. Issue an Asset Performance Inquiry to United: Use this specific tail-audit report to engage United Air Lines' account management team. Request clarity on their fleet modernization and maintenance turnaround times for these specific airframes, using the data to negotiate service level credits or discounts.


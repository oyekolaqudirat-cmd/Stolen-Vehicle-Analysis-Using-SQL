# Stolen Vehicle Analysis Using SQL
An SQL-based analysis of stolen vehicle data to identify theft patterns across time, vehicle types, regions, vehicle age, and vehicle makes.

**Table Of Content**
- Project Overview
- Tools
- Project Objectives
- Data Preparation and Cleanings
- Key Findings
- Techniques Used
- Project Outcome
- Data Source
- Conclusion

## Project Overview

An SQL-based analysis of stolen vehicle data to identify theft patterns across time, vehicle types, regions, vehicle age, and vehicle makes.

**Tools**: SQL

**Project Objectives**
- Identify when vehicle theft occurs most frequently.
- Analyze theft patterns by vehicle type and region.
- Examine the average age of stolen vehicles.
- Compare regional theft volumes with population density.
- Analyze stolen vehicle makes and make types.
- Calculate population-adjusted theft rates.
  
**Data Preparation & Cleaning**
- Created foreign key relationships between the three tables.
- Checked for duplicate/distinct records and missing values.
- Removed empty records.
- Replaced selected NULL values with N/A.
- Extracted year, month, month number, and day of the week from the theft date.
- Created a vehicle age column using the model year and theft date.
- Validated vehicle age for negative values and potential outliers.

  ## Key Findings
  
**Theft Patterns**
- Monday recorded the highest number of stolen vehicles, with 749 thefts.
- March recorded the highest proportion of thefts, accounting for approximately 18% of total thefts.
  
**Vehicle Type & Region**
- Auckland recorded the highest number of stolen vehicles, with 1,630 thefts.
- Station wagons were the most frequently stolen vehicle type.
- Articulated trucks and special-purpose vehicles recorded the lowest theft counts.
  
**Vehicle Age**
- Calculated the average age of stolen vehicles by vehicle type.
- The maximum recorded vehicle age was 81 years, with no negative or apparent outlier values identified.

**Regional Analysis**
- Auckland accounted for approximately 35% of recorded vehicle thefts.
- Southland accounted for approximately 0.5%.
- Regional theft volumes were compared with population density.
- Per-capita and per-1,000-resident measures were calculated to account for differences in regional population size.

**Make & Vehicle Type**
- Standard vehicles accounted for more than 95% of stolen vehicles.
- Luxury vehicles accounted for less than 5%.

**SQL Techniques Used**

FOREIGN KEY • JOIN • CTE • PIVOT • SUBQUERY • GROUP BY • CASE • WINDOW FUNCTIONS • RANK() • DATENAME() • DATEPART() • YEAR() • ALTER TABLE • UPDATE • DELETE • Aggregate Functions

**Project Outcome**

The analysis demonstrates the use of SQL for data cleaning, relational data preparation, feature engineering, exploratory analysis, aggregation, population-adjusted analysis, and extracting actionable insights from multiple related datasets.

**Data Source and coverage**
The dataset covers vehicles stolen in New Zealand from 2021-2022

[Click here to download the dataset](https://mavenanalytics.io/data-playground/motor-vehicle-thefts)


**Conclusion**

The stolen vehicle analysis provided insights into the patterns and characteristics of vehicle theft across different time periods, vehicle types, makes, and regions. The analysis showed that vehicle theft was concentrated around specific days and months, with Monday recording the highest number of thefts and March accounting for a notable share of reported cases.
The analysis also revealed that station wagons were the most frequently stolen vehicle type, while standard vehicle makes accounted for the vast majority of reported thefts compared with luxury makes. Geographically, Auckland recorded the highest number of thefts, highlighting significant differences in theft concentration across regions. When population size was considered, the regional comparison provided a more meaningful view of theft levels relative to the population.
Overall, the analysis demonstrates how SQL can be used to transform raw vehicle-theft data into meaningful insights by identifying temporal patterns, vehicle characteristics, and geographic trends. These findings can support further investigation into factors associated with vehicle theft and demonstrate the use of data analysis to identify patterns within real-world datasets.

```SQL
--Creating raletionship between the tables
--FOREIGN KEY
ALTER TABLE stolen_vehicles
ADD CONSTRAINT FK_make_id
FOREIGN KEY (make_id)
REFERENCES make_details (make_id)

ALTER TABLE  stolen_vehicles
ADD CONSTRAINT FK_location_id
FOREIGN KEY (location_id)
REFERENCES locations (location_id)

SELECT *
FROM locations

SELECT *
FROM make_details

SELECT *
FROM stolen_vehicles

--Checking for distinct values
SELECT DISTINCT *
FROM Locations

SELECT DISTINCT *
FROM make_details

SELECT DISTINCT *
FROM stolen_vehicles

--Checking for null values
SELECT
  SUM(CASE WHEN vehicle_type IS NULL THEN 1 ELSE 0 END) AS Missing_type,
  SUM(CASE WHEN make_id IS NULL THEN 1 ELSE 0 END) AS Missing_id,
  SUM(CASE WHEN model_year IS NULL THEN 1 ELSE 0 END) AS Missing_year,
  SUM(CASE WHEN vehicle_desc IS NULL THEN 1 ELSE 0 END) AS Missing_desc,
  SUM(CASE WHEN color IS NULL THEN 1 ELSE 0 END) AS Missing_color,
  SUM(CASE WHEN date_stolen IS NULL THEN 1 ELSE 0 END) AS Missing_date,
  SUM(CASE WHEN location_id IS NULL THEN 1 ELSE 0 END) AS Missing_location
FROM stolen_vehicles

--Deleting empty rows
DELETE FROM stolen_vehicles
WHERE vehicle_id >= 4539

--Replacing Null Values
UPDATE stolen_vehicles
SET vehicle_type = 'N/A'
WHERE vehicle_type IS NULL

UPDATE stolen_vehicles
SET vehicle_desc = 'N/A'
WHERE vehicle_desc IS NULL

--Extracting week, day and year from the stolen date
ALTER TABLE stolen_vehicles
ADD year_stolen INT

ALTER TABLE stolen_vehicles
ADD month_stolen varchar(30)

ALTER TABLE stolen_vehicles
ADD day_stolen varchar(30)

ALTER TABLE stolen_vehicles
ADD month_number INT

UPDATE stolen_vehicles
SET month_number = DATEPART(Month, date_stolen)

UPDATE stolen_vehicles
SET year_stolen = YEAR(date_stolen)

UPDATE stolen_vehicles
SET month_stolen = DATENAME(month, date_stolen)

UPDATE stolen_vehicles
SET day_stolen = DATENAME(weekday, date_stolen)

--Analysis
--What day of the week are vehicles most often and least often stolen
--The day of the week with the number of stolen vehicles is monday with 749 stolen vehicles.
SELECT day_stolen, COUNT(*) AS Numbers_of_stolen_vehicles
FROM stolen_vehicles
GROUP BY day_stolen
ORDER BY Numbers_of_stolen_vehicles DESC

--What month are vehicles stolen the most
--March rounds up to 18 percent of the stolen vehicles amking it the highest in all the months.
SELECT month_stolen,  COUNT(*) AS Numbers_of_stolen_vehicles,
COUNT(*) * 100 / (SELECT COUNT(*) 
                  FROM stolen_vehicles) AS Percents
FROM stolen_vehicles 
GROUP BY month_stolen, month_number
ORDER BY month_number, Numbers_of_stolen_vehicles DESC


--What types of vehicles are most often and least often stolen? Does this vary by region
--Auckland accounts as the region with the most theft, with a total of 1630 stolen vehicle from the region.
-- station wagon the most stolen vehicle and the least stolen vehicle is articulated truck and special purpose vehicle.
SELECT  region, vehicle_type, COUNT(*) AS Numbers_of_stolen_vehicles
FROM stolen_vehicles sv
JOIN locations ls
   ON sv.location_id = ls.location_id
GROUP BY  region, vehicle_type
ORDER BY  region, Numbers_of_stolen_vehicles DESC

--What is the average age of the vehicles that are stolen? Does this vary based on the vehicle type
--Create Vehicle age column
ALTER TABLE stolen_vehicles
ADD vehicle_age INT

UPDATE stolen_vehicles
SET vehicle_age = Year(date_stolen) - model_year

--Vehicle age ranged from the minimum recorded age to 81 years, 
--with no negative or apparent outlier values identified.

--Checking for negative outlier in vehicle age
SELECT vehicle_age
FROM stolen_vehicles
WHERE vehicle_age < 0

--Maximum Vehicle age 
SELECT MAX(vehicle_age)
FROM stolen_vehicles

--Average age of vehicle
SELECT vehicle_type, AVG(vehicle_age) AS Avg_age_before_stolen
FROM stolen_vehicles
GROUP BY vehicle_type
ORDER BY Avg_age_before_stolen DESC

--Which regions have the most and least number of stolen vehicles vs the density of the city
--The region with that ranks first in vehicle theft is Auckland which account for 35% of stolen vehicle in all region 
--while Southland has the least of 0.5%.
SELECT region, 
ROUND(MAX(density), 2) AS Density,
COUNT(*) AS number_of_stolen_vehicles,
COUNT(*) * 100.0 / SUM(COUNT(*)) OVER() AS Percent_per_region,
RANK() OVER(ORDER BY MAX(density) DESC) AS Ranks
FROM stolen_vehicles sv
    JOIN locations ls
   ON sv.location_id = ls.location_id
GROUP BY region,  density
ORDER BY Density DESC

--Make name/ make type concentration
--Using CTE
--Standard vehicles are the most stolen with over 95% and luxury less than 5% 
WITH MNMT_mix AS (
    SELECT  make_name, make_type, COUNT(*) AS vehicles_stolen
    FROM stolen_vehicles sv
    JOIN make_details md
      ON sv.make_id = md.make_id
    GROUP BY  make_name, make_type)
SELECT make_name, [Luxury], [Standard]
FROM MNMT_mix 
PIVOT (
    SUM (vehicles_stolen)
    FOR make_type IN ([Luxury], [Standard])
) AS Products;

--Using Subquery
SELECT make_name, [Luxury], [Standard] 
FROM(   SELECT make_name, 
           make_type, 
           COUNT(*) AS vehicles_stolen
        FROM stolen_vehicles sv
        JOIN make_details md
           ON sv.make_id = md.make_id
        GROUP BY make_name, make_type) AS MNMT_mix
PIVOT(
     SUM(vehicles_stolen) 
     FOR make_type IN ([Luxury], [Standard])
) AS Products;

--Group by make type
SELECT make_type, COUNT(*) AS Stolen_Vehicles,
COUNT(*) * 100.0 / SUM(COUNT(*)) OVER()  AS Percent_by_type
FROM make_details md
JOIN stolen_vehicles sv
   ON md.make_id = sv.make_id
GROUP BY make_type


--Theft rate per capita by region
SELECT region, 
    CAST(COUNT(*) AS DECIMAL(10, 4)) / 
      MAX(population) AS Per_capita
FROM stolen_vehicles sv
JOIN Locations ls
  ON sv.location_id = ls.location_id
GROUP BY region, population
ORDER BY Per_capita DESC

--Theft rate per 1000 residents
SELECT region, 
    CAST(COUNT(*) * 1000.0 / 
       (population) AS DECIMAL(4, 2)) AS Theft_rate_per_1000
FROM stolen_vehicles sv
JOIN Locations ls
  ON sv.location_id = ls.location_id
GROUP BY region, population
ORDER BY Per_capita DESC
```
![](https://github.com/oyekolaqudirat-cmd/Stolen-Vehicle-Analysis-Using-SQL/blob/main/Vehicle%20theft%20data%20modelling.png)

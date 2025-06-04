# :earth_africa: Maji Ndogo water access

## Table of Contents 
- [Project Aim](#project-aim)
- [Objectives](#objectives)
- [Key stakeholders](#key-stakeholders)
- [Data Exploration](#data-exploration)
- [Data Cleaning](#data-cleaning)
- [Key Findings](#key-findings)
- [Data Integrity](#data-integrity)
- [Recommendations](#recommendations)
- [Conclusions](#conclusions)
  
## Project Aim 
- The project aims to evaluate the water access and quality in Maji Ndogo. Improve the water sources and infrastructure based on data collected from a survey of over 60,000 records.

## Objectives
- To understand the current state of water access and quality in Maji Ndogo
- To clean up the records and ensure it is viable for analysis
- Assess the quality of water and 
- Draw insights from our data 
- Reduce the time spent waiting in queues to access water 

## Key stakeholders
- President Naledi 
- Chike
- Auditors
- Field Surveyors
  
## Data Exploration

````sql
--Showing the list of all tables in my database
SHOW TABLES;
````

**Answer**:

![
show_tables_maji_ndogo](https://github.com/user-attachments/assets/089492b3-9192-46a7-bc8d-485df8b13a3f)

````sql
--Show columns from the location, visits and water_source tables to understand the table structure
SELECT * FROM location LIMIT 5;
SELECT * FROM visits LIMIT 5;
SELECT * FROM water_source LIMIT 5;
````
### Water_Sources
- Rivers = High risk of contaminants as they are open water sources
- Wells = Prone to contamination due to ageing infrastructure
- Tap_in_home = Taps in each household serving at least 6 people
- Shared taps = Taps shared by multiple people in a public area
- Broken taps = Faulty taps in households due to broken infrastructure

````sql
--To find all unique types of water sources
SELECT DISTINCT
type_of_water_source
FROM
water_source;
````

**Answer**:

![water_source_unique](https://github.com/user-attachments/assets/2d428ecf-7251-4a92-a163-a10f66d1793b)

## Data Cleaning 
### Well Pollution Errors
- In the pollution table, our sources are classed by description of contaminant, contaminant level and visit results 
- If the contaminant level, i.e. (biological column) > 0.01, then the results should not report Clean
- Unfortunately, there are obvious errors when I draw a query for biological > 0.01 and results = 'Clean'

![biological_results](https://github.com/user-attachments/assets/a0fdda55-bac8-49d2-adbc-ce680e2b1e1a)

- First, we create a backup table in case there are errors in our code, we can copy a new one and try again
- From the table, we also notice some descriptions begin with 'Clean' even when the biological contaminant is above 0.01
- The description column should only have 'Clean' when the biological contaminant is below 0.01.
- Therefore, 'Clean' has to be removed from all descriptions where the biological > 0.01

````sql
CREATE TABLE well_pollution_copy AS
SELECT * FROM well_pollution;
--Removes 'Clean' from the description where biological > 0.01
UPDATE
well_pollution_copy
SET
description = 'Bacteria: E. coli'
WHERE
description = 'Clean Bacteria: E. coli';
UPDATE
well_pollution_copy
SET
description = 'Bacteria: Giardia Lamblia'
WHERE
description = 'Clean Bacteria: Giardia Lamblia';
--Updates the results to 'Contaminated: Biological' where biological> 0.01
UPDATE
well_pollution_copy
SET
results = 'Contaminated: Biological'
WHERE
biological > 0.01 AND results = 'Clean';
````

### Employee Contact Errors
- The employee emails are missing from the email column in the employee table 
- The email for each employee consists of their first name and last name with a fullstop('.') in between, with the domain name 'ndogowater.gov' at the end
- After creating an email for each employee, it is updated in the email column

````sql
SELECT
CONCAT( 
LOWER(REPLACE(employee_name, ' ', '.')), 'ndogowater.gov') AS new_email
FROM
employee;
UPDATE
employee
SET 
email = CONCAT(LOWER(REPLACE(employee_name, ' ', '.')), 'ndogowater.gov');
```` 
- The phone numbers of the employees are stored as strings and have 13 characters because of a trailing space at the end
- They have to be converted to 12 numerical characters starting with '+99' as area code
- The trim function is used to remove the trailing space and the new phone numbers are updated to the phone_number column

````sql
SELECT
LENGTH(phone_number)
FROM
employee
;
SELECT
TRIM(phone_number) AS Trimmed_phone_number
FROM employee;
UPDATE 
employee
SET phone_number = RTRIM(LTRIM(phone_number));
````
Where do the employees live?

````sql
- Number of employees grouped by town name
SELECT
town_name, COUNT(town_name)
FROM
employee
GROUP BY
town_name;
````
![no _of_employees_by_town_name](https://github.com/user-attachments/assets/02be47f7-65a5-42b8-ad8c-bea9f07b92bd)

- Pres. Naledi has directed that emails be sent out congratulating the top 3 field surveyors
- So, from the visits table, the assigned ID of the top 3 field surveyors can be found by the number of their visits to water sources

````sql
- Number of visits made by the top 3 employees
SELECT COUNT(visit_count), assigned_employee_id 
FROM
visits
GROUP BY
assigned_employee_id 
ORDER BY COUNT(visit_count) DESC
LIMIT 3;
````

![no _of_visits_by_employees](https://github.com/user-attachments/assets/aeb337f3-5e36-4907-a9ee-856cab4064b3)

- The assigned_id is then used to fetch the employee names and emails from the employee_table due to the linked key(assigned_id)

````sql
SELECT 
*
FROM 
employee
WHERE 
(assigned_employee_id = 1)
OR
(assigned_employee_id = 30)
OR 
(assigned_employee_id = 34);
````

## Key Findings
### Water Source analysis
- How many records per town?
  
````sql
SELECT province_name, town_name, COUNT(*) 
FROM
location
GROUP BY
province_name, town_name;
````
How many sources are in the Urban and Rural areas?

````sql
SELECT location_type, COUNT(*) AS num_sources
FROM
location
GROUP BY
location_type;
````

![image](https://github.com/user-attachments/assets/5da8c774-0bff-4ba3-98a7-9131e988496e)

- It is obvious there are many more sources in Rural areas than Urban, but to engage more with these values, let's present them in percentages 

````sql
SELECT 23740/(15910+23740)*100;
````

![image](https://github.com/user-attachments/assets/9228ec24-76b3-4b21-ae3d-5c64674483d6)

- About 60% of water sources in Maji Ndogo are from Rural areas
- How many wells, taps and rivers are there? 

````sql
SELECT type_of_water_source, COUNT(*) as total_no_per_type
FROM
water_source
GROUP BY
type_of_water_source;
````

![image](https://github.com/user-attachments/assets/b0bfdfa6-0dc8-4957-b431-52a2de2994fe)

- These present the sum of each type of water source 
- How many people did we survey in total?

````sql
SELECT 
	SUM(number_of_people_served) AS Total_survey_number
FROM
water_source;
````

- What is the percentage of people served by each water source?

````sql
SELECT type_of_water_source,
round((sum(number_of_people_served)/27628140)*100,0) AS percentage_per_source
FROM 
water_source
GROUP BY
type_of_water_source
ORDER BY 
round(sum(number_of_people_served)) DESC;
````

![image](https://github.com/user-attachments/assets/c2bac246-080a-4b33-8d01-f068f52661bc)

- 43% of citizens of Maji Ndogo use public taps
- 31% of Maji Ndogo has taps installed at home, but (14/31), i.e. 45% are broken due to damaged infrastructure
- 18% of Maji Ndogo uses wells, but only 26% of them are clean
- A systemic approach to providing solutions to the water problems in Maji Ndogo is to start with the sources affecting most people
  
````sql
SELECT 
type_of_water_source, 
round(sum(number_of_people_served)) AS population_served, 
RANK() OVER (ORDER BY round(sum(number_of_people_served)) DESC) AS RANK_
FROM  water_source
GROUP BY type_of_water_source
ORDER BY RANK_ ASC;
````

![image](https://github.com/user-attachments/assets/a268e493-e80a-4b3b-9489-06087bd7e011)

- From the table, over 11 million people are using shared taps, so we average about 2000 people per shared tap
  
How long did the survey take?

````sql
SELECT
DATEDIFF("2023-07-14", "2021-01-01");
````

- I get 924 days

What is the average queue time for water?

````sql
SELECT
avg(nullif(time_in_queue, 0)) as time_waiting
FROM
visits;
````

- The average person waits 123 minutes to fetch water from a public source
  
- What is the average queue time on different days of the week?

````sql
SELECT
dayname(time_of_record) as day_of_week,
round(avg(nullif(time_in_queue, 0))) as avg_queue_time
FROM
visits
GROUP BY
dayname(time_of_record);
````

![image](https://github.com/user-attachments/assets/c0caa27a-500b-4b83-ab8b-bff173ce4834)

At what time of the day do people collect water?

````sql
SELECT
time_format(time_of_record, '%H:00') AS hour_of_day , 
round(avg(nullif(time_in_queue, 0))) as time_waiting 
FROM
visits
GROUP BY
time_format(time_of_record, '%H:00');
````

- To break this down further, I would like to view the queue times for each hour in a day for all days of the week
  
````sql
SELECT
time_format(time_of_record, '%H:00') AS hour_of_day , 
round(avg(nullif(time_in_queue, 0))) as time_waiting 
FROM
visits
GROUP BY
time_format(time_of_record, '%H:00') ;

SELECT
TIME_FORMAT(TIME(time_of_record), '%H:00') AS hour_of_day,
-- Sunday
ROUND(AVG(
CASE 
WHEN DAYNAME(time_of_record) = 'Sunday' THEN time_in_queue
ELSE NULL 
END
    ),0) AS Sunday,
-- Monday
ROUND(AVG(
CASE
WHEN DAYNAME(time_of_record) = 'Monday' THEN time_in_queue
ELSE NULL 
END
),0) AS Monday,
-- Tuesday
ROUND(AVG(
CASE
WHEN DAYNAME(time_of_record) = 'Tuesday' THEN time_in_queue
ELSE NULL 
END
),0) AS Tuesday,
-- Wednesday
ROUND(AVG(
CASE
WHEN DAYNAME(time_of_record) = 'Wednesday' THEN time_in_queue
ELSE NULL 
END
),0) AS Wednesday,
-- Thursday
ROUND(AVG(
CASE
WHEN DAYNAME(time_of_record) = 'Thursday' THEN time_in_queue
ELSE NULL 
END
),0) AS Thursday,

-- Friday
ROUND(AVG(
CASE
WHEN DAYNAME(time_of_record) = 'Friday' THEN time_in_queue
ELSE NULL 
END
),0) AS Friday,

-- Saturday
ROUND(AVG(CASE
WHEN DAYNAME(time_of_record) = 'Saturday' THEN time_in_queue
ELSE NULL 
END
),0) AS Saturday
FROM
visits
WHERE 
time_in_queue != 0
GROUP BY
hour_of_day
ORDER BY 
hour_of_day;
````

![image](https://github.com/user-attachments/assets/3eb7cafb-ead4-4f03-b107-322aa6e95370)

- From this table, we can understand that Saturday has the longest queue times, as that is when people have the most free time 
- Also, on working days, the averages are higher before 9 AM and after 5 PM because most of the population should be working between those hours
- Sunday has the shortest queue times, one could speculate that this may be due to religious or cultural reasons
  
## Data Integrity
The data was audited, and it revealed some discrepancies between the auditor's score and our original scores 



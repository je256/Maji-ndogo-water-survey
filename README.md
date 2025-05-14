# :earth_africa: Maji Ndogo water access

## Table of Contents 
- Project aim
- Objectives
- Key Stakeholders
- Data Cleaning 
- Data 


### Project Aim 
- The project aims to evaluate the water access and quality in Maji Ndogo. Improve the water sources and infrastructure based on data collected from a survey of over 60,000 records.

### Objectives
- To understand the current state of water access and quality in Maji Ndogo
- To clean up the records and ensure it is viable for analysis
- Assess the quality of water and 
- Draw insights from our data 
- Reduce the time spent waiting in queues to access water 

### Key stakeholders
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

![show_tables_maji_ndogo](https://github.com/user-attachments/assets/089492b3-9192-46a7-bc8d-485df8b13a3f)



````sql
--Show columns from the location, visits and water_source tables to understand the table structure
SELECT * FROM location LIMIT 5;
SELECT * FROM visits LIMIT 5;
SELECT * FROM water_source LIMIT 5;
````
### Water_Sources
- Rivers = High risk of contaminants as they an open water sources
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

- From the table, we also notice some descriptions begin with 'Clean' even when the biological contaminant is above 0.01
- The description column should only have 'Clean' when the biological contaminant is below 0.01.
- Therefore, 'Clean' has to be removed from all descriptions where the biological > 0.01

````sql
--Removes 'Clean' from the description where biological > 0.01
UPDATE
well_pollution
SET
description = 'Bacteria: E. coli'
WHERE
description = 'Clean Bacteria: E. coli';
UPDATE
well_pollution
SET
description = 'Bacteria: Giardia Lamblia'
WHERE
description = 'Clean Bacteria: Giardia Lamblia';
--Updates the results to 'Contaminated: Biological' where biological> 0.01
UPDATE
well_pollution
SET
results = 'Contaminated: Biological'
WHERE
biological > 0.01 AND results = 'Clean';


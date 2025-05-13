# :earth_africa: Maji Ndogo water access

## Table of Content 
- Project aim
- Objectives
- Key Stakeholders
- Data Cleaning 
- Data 


### Project Aim 
- The aim of the project is to evaluate the water access and quality in Maji Ndogo. Improve the water sources and infrasturcture based on data realized from a survey of over 60,000 records.

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
  
## Data Cleaning

- Showing the list of the all tables in my database
SHOW TABLES;

**Answer**:

![show_tables_maji_ndogo](https://github.com/user-attachments/assets/089492b3-9192-46a7-bc8d-485df8b13a3f)

- Show columns from the location, visits and water_source table to understand table structure

````sql
SELECT * FROM location LIMIT 5;
SELECT * FROM visits LIMIT 5;
SELECT * FROM water_source LIMIT 5;
````
### Water_Sources
- Rivers = High risk of contaminants as it an open water source
- Wells = Prone to contamination due to aging infrastructure
- Tap_in_home = taps in each household serving atleast 6 people
- Shared taps = Taps shared by multiple people in a public area
- Broken taps = Faulty taps in households due to broken infrastructure


# 💧 Maji Ndogo: Data Analytics Case Study  
*A Structured SQL Analysis of Rural Water Access*

---

## 📌 Project Overview

This project analyzes water accessibility in the fictional but data-rich village of **Maji Ndogo**, using SQL to uncover water supply issues, infrastructure challenges, and service inequalities. Over 60,000 records were examined, from water sources to employee records, and the findings are aimed at improving infrastructure planning and emergency response.

---

## 🎯 Objectives

- Clean and prepare raw survey data
- Identify broken or misclassified water sources
- Quantify population served by each type of source
- Prioritize water infrastructure for repairs using ranking
- Analyze queue times to improve citizen experience

---

## 🗃️ Data Schema (Key Tables)

- `employee`: Staff who collected survey data
- `location`: Geographic details of water source locations
- `visits`: Logs of survey visits and queue times
- `water_source`: Types and conditions of water points
- `well_pollution`: Contamination and biological test results
- `water_quality`: Labels like "Clean" or "Polluted"

---

## 🧼 Part 1: Data Cleaning & Fixes

### ✅ Example: Standardize Emails
```sql
UPDATE employee
SET email = CONCAT(
  LOWER(REPLACE(employee_name, ' ', '.')),
  '@ndogowater.gov'
);


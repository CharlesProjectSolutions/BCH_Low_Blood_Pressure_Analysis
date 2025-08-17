# 📌 Project Overview
During pediatric surgeries, every moment is critical. Monitoring patient signs, especially blood pressure, can mean the difference between life and death. This project analyzes blood pressure data to identifies cases where **systolic blood pressure dropped below a safe threshold** for at least **14 consecutive minutes**, helping Boston Children's Hospistal **(BCH)** assess risks and quickly spot potentially dangerous trends in real-time.

## ⚠️ Problem Statement
BCH face a complex challenge: identifying prolonged periods of low blood pressure across different pediatric age groups, using data from multiple sources.  Without clear tracking, they may miss critical cases, which can lead to potential health risks for children undergoing pediatric surgery. 

## 🎯 Objective & Requirements
Our project aimed to:
- Develop an automated system to detect low blood pressure events
- Create age-specific thresholds for identifying critical BP drops
- Generate comprehensive, actionable reports for BCH medical team

  ## 🛠️ Methodology
We developed a Python-powered ETL (Extract, Transform, Load) solution designed to:
1. Import necessary libraries (pandas) needed for our analysis
2. Read both CSV files into a Pandas DataFrames
3. Convert date/time fields to proper datetime format
4. Integrate and preprocess patient demographic and blood pressure data
   <p align="center">
       ## BCH Data Model
    <img width="1194" height="897" alt="Blood Presure Data Model" src="https://github.com/user-attachments/assets/cd2f670c-b261-48f4-b3d9-8553e3edb744" />
</p>
5. Implement dynamic, age-based blood pressure thresholds

6. Analyze and identify continuous periods of low blood pressure

7.  Generate detailed, clear reports for medical review





# 🧹 Data Cleaning & 📊 Exploratory Data Analysis (EDA) Project using SQL

This repository contains my work on data cleaning and exploratory data analysis (EDA), completed entirely using **SQL**. The project focuses on transforming raw data into a clean, structured format and extracting insights through SQL queries.

---

## 📌 Project Overview

The objective of this project is to:
- Perform **data cleaning** operations using SQL
- Conduct **exploratory data analysis (EDA)** directly within the database
- Summarize and interpret key trends, anomalies, and relationships in the data

---

## 🧾 Dataset

> **Name:** layoff.csv 
> **Description:** [country, location, layoff count, industry, Layoff Percentage, Date, Fund Raised]
> **Final Output Name:** output.csv

---

## 🧽 Data Cleaning (in SQL)

I performed several data cleaning tasks using SQL, including:

- Identifying and handling **NULL/missing values**
- Removing **duplicate rows**
- Fixing **data type mismatches**
- Standardizing **categorical values**
- Formatting columns (e.g., dates, text case)
- Removing or flagging **outliers** (if applicable)

## 📊 Exploratory Data Analysis (EDA)

EDA is a crucial step in understanding the underlying structure and behavior of the dataset before moving on to modeling or visualization. In this project, all exploratory steps were performed using **SQL queries**, allowing insights to be extracted directly from the database without external tools.

The analysis focused on:
- Identifying key metrics such as average values, totals, and frequencies
- Segmenting data by time, category, region, or other dimensions
- Highlighting trends, spikes, and patterns in customer behavior or sales
- Discovering anomalies and unusual data points for further investigation

By leveraging SQL's power to filter, aggregate, and join data efficiently, I was able to uncover actionable insights that inform decision-making and guide future analysis.


**Example SQL commands used:**

select *from layoffs;
create table layoff_staging like layoffs;
select *from layoff_staging1;
insert layoff_staging select *from layoffs; 
with uni as (select *,row_number() over (partition by company,location,industry,total_laid_off,percentage_laid_off,'date',stage,country,funds_raised_millions) as row_num from layoff_staging)
delete from uni where row_num>1;


CREATE TABLE `layoff_staging1` (
  `company` text,
  `location` text,
  `industry` text,
  `total_laid_off` int DEFAULT NULL,
  `percentage_laid_off` text,
  `date` text,
  `stage` text,
  `country` text,
  `funds_raised_millions` int DEFAULT NULL,
  `row_nu` INT
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

insert layoff_staging1 
select *,row_number() over (partition by company,location,industry,total_laid_off,percentage_laid_off,`date`,stage,country,funds_raised_millions) as row_num from layoff_staging;
select * from layoff_staging1 where location='DÃ¼sseldorf';
 
 delete from layoff_staging1 where row_nu>1;
update layoff_staging1 set company=trim(company);
select distinct(industry) from layoff_staging1 order by 1;
update layoff_staging1 set industry='Crypto' where industry like 'Crypto%';
select distinct country from layoff_staging1;
update layoff_staging1 set country=trim(trailing '.' from country);
update layoff_staging1 set `date`=str_to_date(`date`, '%m/%d/%Y');
alter table layoff_staging1 modify column `date` date;
update layoff_staging1 set location='Dusseldorf' where location='DÃ¼sseldorf';
select *from layoff_staging1 where total_laid_off is null and percentage_laid_off is null;
select t1.industry,t2.industry from layoff_staging1 t1 join layoff_staging1 t2 on t1.company=t2.company where(t1.industry is null or t1.industry='') and (t2.industry is not null);
update layoff_staging1 t1 join layoff_staging1 t2 on t1.company=t2.company set t1.industry=t2.industry where(t1.industry is null) and (t2.industry is not null);
select *from layoff_staging1 where industry is null or industry='';
select *from layoff_staging1 where company='Airbnb';
update layoff_staging1 set industry=NULL where industry='';
select *from layoff_staging1 where industry is null or industry='';
delete from layoff_staging1 where total_laid_off is null and percentage_laid_off is null;
select count(*) from layoff_staging1;
alter table layoff_staging1 drop column row_nu;
select substring(`date`,1,7) as `month`, sum(total_laid_off) from layoff_staging1 where substring(`date`,1,7) is not null group by `month` order by 1 asc ;
with rolling as(select substring(`date`,6,2) as `month`, substring(`date`,1,4) as `year` ,sum(total_laid_off) as total_off from layoff_staging1 where `date`  is not null group by `month`,`year` order by 1 asc)
select `year`,`month`,total_off,sum(total_off)  over(partition by `year` order by `month`) as rolling_total from rolling;
with roll as(select substring(`date`,6,2) as `month`,substring(`date`,1,4) as `year`,sum(total_laid_off) as total from layoff_staging1 where `date` is not null group by `month`,`year` order by 1 asc)
select `month`,`year`, sum(total) over(partition by `year` order by `month`) as total,total from roll;
select company,year(`date`),sum(total_laid_off) from layoff_staging1 group by company, year(`date`) order by 2 asc;
with comm (company,years,total) as(select company,year(`date`),sum(total_laid_off) from layoff_staging1 group by company,year(`date`))
,company_rank as (select *,dense_rank() over(partition by years order by total desc) as ranking from comm where years is not null)
select *from company_rank where ranking<=5;


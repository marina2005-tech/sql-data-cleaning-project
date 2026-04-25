# sql-data-cleaning-project
#Data Cleaning

USE world_layoff;
Select *
from layoffs;

#1.Remove Duplicates
#2.Standardize the data
#3.Remove NULLS
#4.Remove any Columns


#Making the Backup copy

CREATE TABLE layoffs_staging
LIKE layoffs;

INSERT layoffs_staging
SELECT *
FROM layoffs;

WITH duplicate_cte AS(
SELECT *,
ROW_NUMBER() OVER(
PARTITION BY stage, percentage_laid_off, industry, country, location, funds_raised_millions, company, industry, total_laid_off,' date'
) 
AS row_num 
FROM layoffs_staging
)

#We checked for the rows that are duplicated (row num is > 1

SELECT *
FROM duplicate_cte
WHERE row_num > 1;

SELECT *
FROM layoffs_staging
WHERE company = 'Casper';



CREATE TABLE `layoffs2` (
  `company` text,
  `location` text,
  `industry` text,
  `total_laid_off` int DEFAULT NULL,
  `percentage_laid_off` text,
  `date` text,
  `stage` text,
  `country` text,
  `funds_raised_millions` int DEFAULT NULL,
  `row_num` int
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;


INSERT INTO layoffs2
SELECT *
FROM (
    SELECT *,
           ROW_NUMBER() OVER (
               PARTITION BY stage, percentage_laid_off, industry, country,
                            location, funds_raised_millions, company,
                            industry, total_laid_off, `date`
           ) AS row_num
    FROM layoffs_staging
) 
t;



SELECT *
FROM layoffs2
WHERE row_num>1;

SET SQL_SAFE_UPDATES = 0;

DELETE 
FROM layoffs2
WHERE row_num>1;


SELECT *
FROM layoffs2
WHERE row_num=1;

TRUNCATE TABLE layoffs2;




INSERT INTO layoffs2
SELECT *
FROM (
    SELECT *,
           ROW_NUMBER() OVER (
               PARTITION BY stage, percentage_laid_off, industry, country,
                            location, funds_raised_millions, company,
                            industry, total_laid_off, `date`
           ) AS row_num
    FROM layoffs_staging
) 
t;

SELECT *
FROM layoffs2
WHERE row_num>1;


SELECT *
FROM layoffs2
WHERE row_num=1;


DELETE 
FROM layoffs2
WHERE row_num>1;


SELECT *
FROM layoffs2;   #We removed all duplicated rows

#The cleaned data will be layoffs2

#STANDRADIZING DATA
SELECT company, Trim(company)
FROM layoffs2;

UPDATE layoffs2
SET company = TRIM(company);

SELECT DISTINCT(industry)
FROM layoffs2
ORDER BY 1;


SELECT DISTINCT(industry)
FROM layoffs2
WHERE industry LIKE 'Crypto%';


UPDATE layoffs2
SET industry = 'Crypto'
WHERE industry LIKE 'Crypto%';

SELECT DISTINCT (location)
FROM layoffs2
ORDER BY 1;

SELECT DISTINCT (country), TRIM(TRAILING '.' FROM country)
FROM layoffs2
ORDER BY 1;

UPDATE layoffs2
SET country = TRIM(TRAILING '.' FROM country)
WHERE country LIKE 'United States%';

SELECT DISTINCT (country) 
FROM layoffs2
ORDER BY 1;

SELECT `date`
FROM layoffs2;

SELECT `date`,
STR_TO_DATE(`date`,'%m/%d/%Y')
FROM layoffs2;

UPDATE layoffs2
SET `date`= STR_TO_DATE(`date`,'%m/%d/%Y');

ALTER TABLE layoffs2
MODIFY COLUMN `date`  DATE;

# NULLS

SELECT * 
FROM layoffs2
WHERE total_laid_off IS NULL
AND percentage_laid_off IS NULL;

SELECT *
FROM layoffs2
WHERE industry IS NULL 
OR industry  = '';

SELECT *
FROM layoffs2
WHERE company = 'Airbnb';

/*UPDATE layoffs2 t
JOIN layoffs s
ON t.company = s.company
SET t.industry = s.industry
WHERE t.industry IS NULL OR t.industry = '';
*/

/*SELECT t.industry,s.industry
FROM layoffs2 t
JOIN layoffs s
ON t.company = s.company;
*/

UPDATE  layoffs2 t1
JOIN layoffs2 t2
	ON t1.company=t2.company
SET t1.industry = t2.industry
WHERE (t1.industry IS NULL OR t1.industry='')
AND t2.industry IS NOT NULL;

UPDATE layoffs2 t1
JOIN layoffs t2
ON t1.company=t2.company
SET  t1.industry = NULL
WHERE (t1.industry ='');


SELECT t1.industry,t2.industry
FROM layoffs2 t1
JOIN layoffs t2
	ON t1.company=t2.company
WHERE (t1.industry IS NULL OR t1.industry='')
AND t2.industry IS NOT NULL;


UPDATE layoffs2 t1
JOIN layoffs t2
	ON t1.company=t2.company
SET t1.industry=t2.industry
WHERE (t1.industry IS NULL)
AND t2.industry IS NOT NULL;


SELECT * 
FROM layoffs2
WHERE company= 'Airbnb'; #it worked



SELECT * 
FROM layoffs2
WHERE company LIKE 'Bally%'; 

SELECT * 
FROM layoffs2
WHERE total_laid_off IS NULL
AND percentage_laid_off IS NULL;


DELETE  #TO REMOVE THE NULLS THAT HAS NO KNOWLEDGABLE DATA
FROM layoffs2
WHERE total_laid_off IS NULL
AND percentage_laid_off IS NULL;


ALTER TABLE layoffs2
DROP row_num;



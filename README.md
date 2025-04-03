#     Netflix Movies and TV Shows Data Analysis using SQL

![Netflix_Logo_RGB](https://github.com/user-attachments/assets/c86dfc7a-9fd2-4628-9035-053d72cf32e2)

## Overview
This project involves a comprehensive analysis of Netflix's movies and TV shows data using SQL. The goal is to extract valuable insights and answer various business questions based on the dataset. 

## Objectives

- Analyze the distribution of content types (movies vs TV shows).
- Identify the most common ratings for movies and TV shows.
- List and analyze content based on release years, countries, and durations.
- Explore and categorize content based on specific criteria and keywords.

## Schema

```sql
DROP TABLE IF EXISTS netflix;
CREATE TABLE netflix
(
    show_id      VARCHAR(5),
    type         VARCHAR(10),
    title        VARCHAR(250),
    director     VARCHAR(550),
    casts        VARCHAR(1050),
    country      VARCHAR(550),
    date_added   VARCHAR(55),
    release_year INT,
    rating       VARCHAR(15),
    duration     VARCHAR(15),
    listed_in    VARCHAR(250),
    description  VARCHAR(550)
);
```

## Business Problems and Solutions

### 1. Count the Number of Movies vs TV Shows
```sql
select type, count(*) 
from netflix
group by type;
```
###  2. Find the most common rating for movies and TV shows
```sql
SELECT type , rating as most_common_rating from 
(
select type,  rating , count(*),
rank() over(partition by type order by count(*) desc) as rank
from netflix
group by type , rating
)
where rank = 1;
```
### 3. List all movies released in a specific year (e.g., 2020)
```sql
select * from netflix
where type = 'Movie' and release_year = '2020';
```
###  4. Find the top 5 countries with the most content on Netflix
```sql
select country, count(show_id) 
from netflix
group by country
order by 2 DESC
LIMIT 5;
```
###  5. Find all the movies/TV shows by director 'Rajiv Chilaka'!
```sql
SELECT *
FROM netflix
WHERE 
	director = 'Rajiv Chilaka';
```
###  6. Find each year and the average numbers of content release by India on netflix. 
###  return top 5 year with highest avg content release !
```sql
SELECT 
	country,
	release_year,
	COUNT(show_id) as total_release,
	ROUND(
		COUNT(show_id)::numeric/
								(SELECT COUNT(show_id) FROM netflix WHERE country = 'India')::numeric * 100 
		,2
		)
		as avg_release
FROM netflix
WHERE country = 'India' 
GROUP BY country, 2
ORDER BY avg_release DESC 
LIMIT 5;
```
### 7. List all movies that are documentaries
```sql
SELECT * FROM netflix
WHERE listed_in LIKE '%Documentaries'
```
### 8. Find all content without a director
```sql
SELECT * FROM netflix
WHERE director IS NULL
```
### 9. Find how many movies actor 'Salman Khan' appeared in last 10 years!
```sql
SELECT * FROM netflix
WHERE 
	casts LIKE '%Salman Khan%'
	AND 
	release_year > EXTRACT(YEAR FROM CURRENT_DATE) - 10
```

### 10. Find the top 10 actors who have appeared in the highest number of movies produced in India.
```sql
SELECT 
	UNNEST(STRING_TO_ARRAY(casts, ',')) as actor,
	COUNT(*)
FROM netflix
WHERE country = 'India'
GROUP BY 1
ORDER BY 2 DESC
LIMIT 10
```
### 15. Categorize Content Based on the Presence of 'Kill' and 'Violence' Keywords and count 
```sql
SELECT 
    category,
	TYPE,
    COUNT(*) AS content_count
FROM (
    SELECT 
		*,
        CASE 
            WHEN description ILIKE '%kill%' OR description ILIKE '%violence%' THEN 'Bad'
            ELSE 'Good'
        END AS category
    FROM netflix
) AS categorized_content
GROUP BY 1,2
ORDER BY 2;
```

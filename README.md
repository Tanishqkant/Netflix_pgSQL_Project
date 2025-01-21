# Netflix_pgSQL_Project
![Netflix Logo](https://github.com/Tanishqkant/Netflix_pgSQL_Project/blob/main/Netflix_logo.jpg)

# Objective
#schema
```sql
drop table if exists netflix;

create table  netflix(
show_id varchar(6),
type varchar(10),
title varchar(150),
director varchar(210),
casts varchar(1000),
country	varchar(150),
date_added varchar(50),
release_year int,
rating varchar(10),
duration varchar(15),	
listed_in varchar(100),
description varchar(250)

);
```

## 15 Business Problems and Solutions
select * from netflix;
-- it will return the total number of rows in the data.
select 
count(*) as total_content
from netflix

-- It will show types of content we have in the data set.
select 
distinct type
from netflix 


## 1.Count number of movies and TV shows 
```sql
select 
type,
count(*)as  total_content
from netflix
group by type
```

## 2.Find the most common rating for movies and TV shows
```sql
select 
type,
rating 
from 
(select 
type,
rating,
count(*),
rank() over(partition by type order by count(*) desc) as ranking 
from netflix
group by 1,2
) as t1
where 
ranking = 1
```
-- order by 1, 3 desc

## 3.List all movies released in specific year(e.g 2020)
```sql
select  * from netflix
where 
  type = 'Movie'
  and
  release_year = 2020
```  


## 4. Find the top 5 countries wuth the most content on Netflix
```sql
select
 unnest(string_to_array(country,',')) as new_country,
count(show_id) as total_content
from netflix
group by 1 
order by 2 desc
limit 5
```


-- select 
-- -- This string_to_array will conver the string to the array. 
--    -- string_to_array(country,',') as new_country
-- -- The unnest will edit the country one after one.
--    unnest(string_to_array(country,',')) as new_country
--  from Netflix




##5. Identify the longest movie?
```sql
select * from netflix
where
   type ='Movie'
   and 
   duration = (select MAX(duration) from netflix)
```

## 6. Find content added in last 5 years
```sql
SELECT
*
FROM netflix
WHERE TO_DATE(date_added, 'Month DD, YYYY') >= CURRENT_DATE - INTERVAL '5 years'
```

## 7.Find all the movies/TV shows by director 'Rajiv Chilaka'
```sql
select * from netflix
-- the LIKE operator is used for pattern matching in text (string) searches. It allows you to find rows where a column's value matches a specified pattern.
-- % (Percentage Sign) → Matches zero or more characters
-- _ (Underscore) → Matches exactly one character
-- ILIKE will return the result even if the name of the director is in small letters
where director ILIKE '%Rajiv Chilaka%'
```
## 8.List all the TV shows with more than 5 seasons
```sql
SELECT *
FROM netflix
WHERE 
	TYPE = 'TV Show'
	AND
	SPLIT_PART(duration, ' ', 1)::INT > 5
```
## 9.Couunt the number of coontent item in each genre
```sql
select
unnest(string_to_array(listed_in,',')) as genre,
count(show_id) as total_content
from netflix
group by 1
```


## 10. Find each year and the average number of content release by India on netflix,
-- return top 5 year with highest avg content release
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
LIMIT 5
```

## 11. List all movies that are documentaries.
```sql
SELECT * FROM netflix
WHERE listed_in LIKE '%Documentaries'
```

##12. Find all content without a director
```sql
select * from netflix 
where director is null
```

## 13. Find how many movies actor "Salman Khan" appeared in last 10 years;
```sql
SELECT * FROM netflix
WHERE 
	casts LIKE '%Salman Khan%'
	AND 
	release_year > EXTRACT(YEAR FROM CURRENT_DATE) - 10

```
## 14. Find top 10 actors who have appeared in the highest number of movies produced in India.
```sql	
 select 
	UNNEST(STRING_TO_ARRAY(casts,',')) as actors,
	COUNT(*) as toatal_content
	from netflix
	where country ilike '%India%'
	group by 1 
	order by 2 desc
	limit 10
```

## 15. Categorize the content based on the presence of the keyword 'Kill' and 'Voilence' in the description field.Label content containing these keywords as 'Bad' and all other 
 content as 'Good'. Count how many items fall Into each category.
 ```sql
with new_table
as 
(
	select *,
	case 
	when 
	description  ILIKE '%kill%'
	or 
	description  ILIKE '%voilence%' then 'Bad_Content'
	else 'Good_Content'
	end category
	from netflix
)
select 
category,
count(*) as total_content
from new_table
group by 1
```

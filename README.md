# **NETFLIX_Data_Analysis**

![NETFLIX_LOGO](https://github.com/vj220803/NETFLIX_Data_Analysis/blob/main/NETFLIX_LOGO.jpg)

## Overview and Goal Of Project:
This project demonstrates exploratory data analysis on the **Netflix Movies and TV Shows** dataset using **PostgreSQL**. The dataset consists of various metadata attributes such as show ID, type, title, director, cast, country, date added, release year, rating, duration, genre, and description. Generating an insightful Analysis of the Movies and TV Shows entries avilable in the dataset, Making business more efficient by focusing on the weak and strong points allowing them to make acurate Data Driven Dicision.

This Project deals with mainly 17 Business Problems which can be productive for targeting specific age group users, Foreign contents importance and future enhancement for more profit in which country to invest more, advertising most famous content to people more and attracting their attention by best homepage recommendation, Collaborating with future young genration having better ideas which will result in future profit investment,etc.

---------------------------------------------------------------------------------------------------------------
![netflix_flow](https://github.com/vj220803/NETFLIX_Data_Analysis/blob/main/NETFLIX_flowchart.PNG)


----------------------------------------------------------------------------------------------------------------

## Dataset Overview

- **Source**: Netflix Movies and TV Shows 
- **Format**: CSV
- **Total Records**: 6235
- **Columns**:
  - `show_id`: Unique identifier for each title.
  - `type`: Movie or TV Show.
  - `title`: Name of the show.
  - `director`: Name of the director(s).
  - `cast`: List of actors.
  - `country`: Country of origin.
  - `date_added`: When it was added to Netflix.
  - `release_year`: Year of release.
  - `rating`: Age classification.
  - `duration`: Runtime or number of seasons.
  - `listed_in`: Genre(s).
  - `description`: Short summary.

---

![NETFLIX_poster](https://github.com/vj220803/NETFLIX_Data_Analysis/blob/main/netflix_poster.png.png)

## Tools Used

- **PostgreSQL 17.5**
- **pgAdmin 4**
- **SQL** 

---

## SQL Queries Performed

### Basic Queries

**View all records from the dataset**
   ```sql
   SELECT * FROM netflix;
   ```

1. **Displaying distinct entries in type column**
```sql
SELECT DISTINCT type from netflix;
```

2. **Count Number of Movies vs TV Shows**
```sql
SELECT type, COUNT(*) as total_content 
FROM netflix
GROUP BY type;
```

3. **Find the most common rating for movies and TV shows**
```sql
SELECT type, rating from 
(
	SELECT type, rating,
	COUNT(*),
	RANK() OVER(PARTITION BY type ORDER BY COUNT(*) DESC) as ranking
	FROM netflix
	GROUP BY 1,2
) as t1
WHERE ranking = 1;
```

4. **List all movies released in a specific year(eg. 2020)**

```sql
SELECT * FROM netflix
WHERE type = 'Movie' AND release_year = 2020;
```

5. **if you want top three year where maximum movies released**
```sql
SELECT release_year, COUNT(*) as movie_released
FROM netflix
WHERE type = 'Movie' 
GROUP BY 1
ORDER BY 2 DESC
LIMIT 3;
```

6. **Find the top 5 countries with the most content on Netflix**
```sql
SELECT UNNEST(STRING_TO_ARRAY( country, ',')) as new_country,
COUNT(show_id) as total_content
FROM netflix 
GROUP BY 1
ORDER BY 2 DESC
LIMIT 5;
```

7. **Identify the longest movie**
```sql
SELECT * FROM netflix
WHERE type = 'Movie' AND duration = (SELECT MAX(duration) FROM netflix)
```

8. **Find content added in the last five years**
```sql
SELECT * FROM netflix
WHERE TO_DATE(date_added, 'Month DD, YYYY') >= CURRENT_DATE - INTERVAL '5 years';
-- No movie added in the last five years( till current date)
```

9. **Find all the movies/TV shows by director 'Rajiv Chilaka'**
```sql
SELECT * FROM netflix
WHERE director ILIKE '%rajiv Chilaka%'
```

10. **List all TV shows with more than 5 seasons**
```sql
SELECT * FROM netflix
WHERE type = 'TV Show' AND SPLIT_PART(duration,' ',1)::numeric > 5
```

11. **Count the number of content items in each genre.**
```sql
SELECT UNNEST(STRING_TO_ARRAY(listed_in,',')) as genre,
COUNT(show_id) as genre_count
FROM netflix 
GROUP BY 1
ORDER BY 2 DESC;
```

12.**Find each year and the average number of content release by India on Netflix. Return top 5 year with highest avg content**
```sql
SELECT 
EXTRACT(YEAR FROM TO_DATE(date_added, 'Month DD,YYYY')) as year, 
COUNT(*) as yearly_content,
ROUND(COUNT(*)::numeric/(SELECT COUNT(*) FROM netflix WHERE country='India')::numeric*100,2) as avg_content_per_year
FROM netflix
WHERE country = 'India'
GROUP BY 1
ORDER BY 3 DESC
LIMIT 5;
```

13. **list all movies that are documentries**
```sql
SELECT * FROM netflix
WHERE type = 'Movie' and listed_in ILIKE '%Documentaries%'
```

14. **Find all content without director**
```sql
SELECT * FROM netflix
WHERE director IS NULL
```

15. **Find how many movies actor 'Salman Khan' appeared in last 20 years**
```sql
SELECT * FROM netflix
WHERE casts ILIKE '%Salman Khan%' AND
release_year > EXTRACT(YEAR FROM CURRENT_DATE) - 20
```

16. **Find top 10 actors who have appeared in the highest number of movies produced in India**
```sql
SELECT 
UNNEST (STRING_TO_ARRAY(casts,',')) as actors,
COUNT(*) as total_count
FROM netflix
WHERE country ILIKE '%India%' 
GROUP BY 1
ORDER BY 2 DESC
LIMIT 10;
```

17. **Categorize the content based on the presence of the keywords 'kill' and 'violence' in the descriptive field, lable content containing these keywords as 'bad_content' and all other content as 'good'. count how many items fall into each category**
```sql
WITH new_table 
AS
(
SELECT * , 
	CASE
	WHEN 
		description ILIKE '%kill%' OR
		description ILIKE '%violence' THEN 'bad_content' ELSE 'good_content' 
	END category 
FROM netflix)

SELECT category , COUNT(*) as total_cat_count
FROM new_table
GROUP BY category
ORDER BY 2 DESC;


SELECT category, COUNT(*) as cat_count
FROM (
SELECT *,
	CASE
		WHEN description ILIKE '%kill%' OR
		description ILIKE '%violence%' THEN 'bad_content' ELSE 'good_content'
	END category
FROM netflix
GROUP BY 1
ORDER BY 2 DESC
) as t1
```
   



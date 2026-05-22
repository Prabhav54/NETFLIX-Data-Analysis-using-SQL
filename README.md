# Netflix Media Optimization & Content Strategy Engine (SQL)

![Netflix Portfolio Project Logo](https://github.com/najirh/netflix_sql_project/blob/main/logo.png)

## 📌 Project Overview
This project delivers an end-to-end relational data analytics solution designed to ingest, clean, and analyze Netflix’s global media catalog. By developing an optimized relational schema in PostgreSQL and executing 15 comprehensive business-intelligence queries, I extracted actionable insights regarding content distribution patterns, regional production outputs, and viewer-demographic targeting. The underlying engineering addresses complex data quality issues—such as multi-valued string arrays—using advanced SQL mechanics.

## 🎯 Strategic Objectives
* **Inventory Distribution Analysis:** Segment and evaluate the catalog's structural balance between feature films and multi-season TV shows.
* **Demographic & Rating Mapping:** Identify audience target concentrations across content formats via statistical mode calculation.
* **Geospatial & Production Profiling:** Isolate top-performing production nations and calculate localized growth metrics, with an emphasis on high-growth regions like India.
* **Deep Content Categorization:** Extract data tracking specific content themes, genres, and personnel arrays using optimized pattern matching.

## 🗄️ Relational Database Schema

```sql
DROP TABLE IF EXISTS netflix;
CREATE TABLE netflix
(
    show_id      VARCHAR(5) PRIMARY KEY,
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

🚀 Complex Business Problems & SQL Solutions
1. Global Content Distribution Analysis
Objective: Calculate the macro-level composition of the Netflix catalog.

SQL
SELECT 
    type,
    COUNT(*) AS total_content
FROM netflix
GROUP BY type

🚀 Complex Business Problems & SQL Solutions
1. Global Content Distribution Analysis
Objective: Calculate the macro-level composition of the Netflix catalog.

SQL
SELECT 
    type,
    COUNT(*) AS total_content
FROM netflix
GROUP BY type;
2. Audience Target Mode Identification
Objective: Calculate the most frequent content rating for Movies and TV Shows utilizing window ranking partitions.

SQL
WITH RatingCounts AS (
    SELECT 
        type,
        rating,
        COUNT(*) AS rating_count
    FROM netflix
    WHERE rating IS NOT NULL
    GROUP BY type, rating
),
RankedRatings AS (
    SELECT 
        type,
        rating,
        rating_count,
        RANK() OVER (PARTITION BY type ORDER BY rating_count DESC) AS rank
    FROM RatingCounts
)
SELECT 
    type,
    rating AS most_frequent_rating,
    rating_count
FROM RankedRatings
WHERE rank = 1;
3. Temporal Content Traversal
Objective: Isolate all features published in a targeted operational year (e.g., 2020).

SQL
SELECT 
    show_id, 
    title, 
    director, 
    release_year 
FROM netflix
WHERE type = 'Movie' 
  AND release_year = 2020;
4. Top 5 High-Volume Production Nations
Objective: Parse comma-delimited country arrays, normalize whitespace anomalies, and rank the top 5 sovereign content producers.

SQL
SELECT country, total_content
FROM (
    SELECT 
        TRIM(UNNEST(STRING_TO_ARRAY(country, ','))) AS country,
        COUNT(*) AS total_content
    FROM netflix
    GROUP BY 1
) AS normalized_countries
WHERE country IS NOT NULL 
  AND country != ''
ORDER BY total_content DESC
LIMIT 5;
5. Runtime Outlier Identification
Objective: Extract the longest standalone movie runtime by casting text components to numerical integers.

SQL
SELECT 
    title, 
    duration
FROM netflix
WHERE type = 'Movie'
  AND duration IS NOT NULL
ORDER BY SPLIT_PART(duration, ' ', 1)::INT DESC
LIMIT 1;
6. Dynamic Rolling Ingestion Analysis
Objective: Extract all operational entries added to the ecosystem within a 5-year rolling interval relative to the current timestamp.

SQL
SELECT 
    show_id, 
    title, 
    date_added
FROM netflix
WHERE TO_DATE(date_added, 'Month DD, YYYY') >= CURRENT_DATE - INTERVAL '5 years';
7. Filmography Profile: Targeted Personnel
Objective: Locate and isolate all directory entries managed by 'Rajiv Chilaka'.

SQL
SELECT 
    show_id, 
    title, 
    type
FROM (
    SELECT 
        *,
        TRIM(UNNEST(STRING_TO_ARRAY(director, ','))) AS director_name
    FROM netflix
) AS expanded_directors
WHERE director_name = 'Rajiv Chilaka';
8. Long-Form Media Isolation
Objective: Track high-investment TV Shows extending beyond 5 standard seasons.

SQL
SELECT 
    show_id, 
    title, 
    duration
FROM netflix
WHERE type = 'TV Show'
  AND SPLIT_PART(duration, ' ', 1)::INT > 5;
9. Normalized Genre Cluster Counting
Objective: Dissolve compound tags to count aggregate catalog distribution across separate, distinct genres.

SQL
SELECT 
    TRIM(UNNEST(STRING_TO_ARRAY(listed_in, ','))) AS genre,
    COUNT(*) AS total_content
FROM netflix
GROUP BY 1
ORDER BY total_content DESC;
10. Regional Market Penetration & Trend Analytics (India Focus)
Objective: Calculate the exact annual release density scaling in India, demonstrating year-over-year proportional impact against total regional releases.

SQL
SELECT 
    release_year,
    COUNT(show_id) AS annual_releases,
    ROUND(
        COUNT(show_id)::numeric /
        (SELECT COUNT(show_id) FROM netflix WHERE country ILIKE '%India%')::numeric * 100, 2
    ) AS regional_percentage_contribution
FROM netflix
WHERE country ILIKE '%India%'
GROUP BY release_year
ORDER BY annual_releases DESC
LIMIT 5;
11. Specific Semantic Document Categorization
Objective: Pull all film features explicitly grouped under the 'Documentaries' tag.

SQL
SELECT 
    show_id, 
    title, 
    listed_in
FROM netflix
WHERE type = 'Movie' 
  AND listed_in ILIKE '%Documentaries%';
12. Catalog Lineage & Metadata Completeness Audit
Objective: Audit structural data gaps by filtering for rows missing director credentials.

SQL
SELECT 
    show_id, 
    title, 
    type 
FROM netflix
WHERE director IS NULL;
13. Dynamic Talent Cast Traversal
Objective: Measure actor-specific content longevity by counting titles featuring 'Salman Khan' within the last 10 years of current active records.

SQL
SELECT 
    show_id, 
    title, 
    release_year
FROM netflix
WHERE casts LIKE '%Salman Khan%'
  AND release_year > EXTRACT(YEAR FROM CURRENT_DATE) - 10;
14. High-Frequency Talent Identification (Regional)
Objective: Unnest cast matrices to isolate and rank the top 10 most heavily featured actors inside Indian production circles.

SQL
SELECT 
    TRIM(UNNEST(STRING_TO_ARRAY(casts, ','))) AS actor,
    COUNT(*) AS total_appearances
FROM netflix
WHERE country ILIKE '%India%'
GROUP BY actor
HAVING TRIM(UNNEST(STRING_TO_ARRAY(casts, ','))) IS NOT NULL
ORDER BY total_appearances DESC
LIMIT 10;
15. Semantic Keyword Sentiment Grouping
Objective: Execute custom string flags using CASE conditionals to flag descriptive content as 'Mature/Violent' vs. 'General Audience' based on target string arrays.

SQL
SELECT 
    content_safety_category,
    COUNT(*) AS content_count
FROM (
    SELECT 
        CASE 
            WHEN description ILIKE '%kill%' OR description ILIKE '%violence%' THEN 'Mature/Violent'
            ELSE 'General Audience'
        END AS content_safety_category
    FROM netflix
) AS structural_sentiment
GROUP BY content_safety_category;
📈 Key Engineering Insights & Conclusions
Data Quality Optimization: Multi-value text grouping strings (like country and casts) initially caused significant structural skew. By utilizing advanced arrays, parsing strings with STRING_TO_ARRAY, mapping with UNNEST, and removing trailing white spaces via TRIM, I built a highly robust, clean, and sound relational layout model.

Performance Ingestion Check: Replacing hardcoded historical string values with dynamic system calls (such as CURRENT_DATE and EXTRACT) guarantees that the database pipeline updates automatically without manual updates.

Strategic Indexing Potential: Because queries rely heavily on text matching (ILIKE, LIKE) across extensive text fields, adding a GIN index to the casts, director, and description tables will drastically speed up database searches in high-concurrency settings.

👨‍💻 Developed By
Prabhav Khare

GitHub: github.com/Prabhav54

LinkedIn: linkedin.com/in/prabhav-khare

Email: prabhavkhare54@gmail.com
    COUNT(*) AS total_content
FROM netflix
GROUP BY type;



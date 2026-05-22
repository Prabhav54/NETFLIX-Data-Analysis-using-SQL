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
GROUP BY type;



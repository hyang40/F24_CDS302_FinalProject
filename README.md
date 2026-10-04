# Movie Database from Scratch: Relational Design & SQL Analysis

A normalized SQLite movie database built from six independent Kaggle datasets, designed from an E-R model through a 7-table relational schema and queried with SQL to analyze ratings, genres, directors and box office.

> CDS 302 · Scientific Data and Databases · George Mason University · Fall 2024
> Team project (4 members) · SQLite3 / SQL

**📦 Database file:** [Download the final `.db` (Google Drive)](https://drive.google.com/file/d/1Hy-41lgQeHoT8DTi6KUzMGY-hIju1-bG/view?usp=drive_link)

![Relational schema of the movie database: 5 entity tables and 2 junction tables](images/ERD.png)

---

## Project Goal

> Build a single relational database that connects movies to their directors, cast, genres, critic reviews and box office, so users can find films that match their preferences (for example, by a favorite actor and director together).

Movie data is scattered across sources that each cover one piece (cast, critics, revenue). The goal was to integrate them into one consistent, queryable structure.

## Data Sources

Six public Kaggle datasets, integrated into one database:

| Dataset | Content | Used for |
|---|---|---|
| [IMDb Actors and Movies](https://www.kaggle.com/datasets/rishabjadhav/imdb-actors-and-movies/data) | Actor names, birth/death years, profession | `Actor` |
| [Movie Industry](https://www.kaggle.com/datasets/danielgrijalvas/movies) | 6,820 films (1986–2016): director, genre, gross, runtime, IMDb rating | `Director`, `Movie` |
| [IMDB 5000+ Movies & Multiple Genres](https://www.kaggle.com/datasets/rakkesharv/imdb-5000-movies-multiple-genres-dataset) | Multi-genre labels | `Genre`, `Movie_Genre` |
| [Box Office Collections](https://www.kaggle.com/datasets/anotherbadcode/boxofficecollections) | Box office, Metascore, votes, runtime | `Movie` |
| [Top 100 Rotten Tomatoes Movies by Genre](https://www.kaggle.com/datasets/prasertk/top-100-rotten-tomatoes-movies-by-genres) | Tomatometer score, review count | `Review` |
| [IMDb Movie Reviews, Genres & Emotions](https://www.kaggle.com/datasets/fahadrehman07/movie-reviews-and-emotion-dataset) | Critic reviews and descriptions | Reference |

## Database Design

**1. E-R model**: 5 entities (Movie, Director, Actor, Genre, Review) and their relationships

| Relationship | Cardinality | Implementation |
|---|---|---|
| Director → Movie | 1 : N | `Director_ID` foreign key in `Movie` |
| Movie ↔ Actor | M : N | Junction table `Movie_Actor` (composite PK) |
| Movie ↔ Genre | M : N | Junction table `Movie_Genre` (composite PK) |
| Movie → Review | 1 : N | `Movie_ID` foreign key in `Review` |

**2. Relational schema**: E-R model reduced to 7 tables (full DDL in [`schema.sql`](schema.sql))

```
Movie       (Movie_ID PK, Title, Year, IMDB_Rating, Metascore, Box_Office, Time_Minute, Votes, Director_ID FK)
Director    (Director_ID PK, Name, Country, Birth_Year, Death_Year)
Actor       (Actor_ID PK, Name, Birth_Year, Death_Year, Primary_Profession)
Genre       (Genre_ID PK, Main_Genre)
Review      (Review_ID PK, Movie_ID FK, RatingTomatometer, Number_of_Reviews)
Movie_Actor (Movie_ID FK, Actor_ID FK)   -- composite PK
Movie_Genre (Movie_ID FK, Genre_ID FK)   -- composite PK
```

**3. Data integration**: removed duplicates, standardized formats across sources, handled missing values (for example, budgets recorded as 0) and validated referential integrity before loading.

## SQL Analysis

Five research questions, each exercising a different SQL technique:

| # | Question | Technique |
|---|---|---|
| 1 | Which movies have IMDb ≥ 8 **or** Metascore ≥ 80? | Set operations (`UNION`) |
| 2 | How many movies and what average IMDb rating per genre? | Aggregates + multi-table `JOIN` |
| 3 | Which genres earn the most at the box office? | `GROUP BY` + `ORDER BY` |
| 4 | Which directors average above 7.5 on IMDb? | `HAVING` |
| 5 | What is the highest-rated movie in each genre? | CTE (`WITH`) |

Example: highest-rated movie per genre

```sql
WITH GenreMaxRatings AS (
    SELECT G.Genre_ID, MAX(M.IMDB_Rating) AS Max_Rating
    FROM Genre G
    JOIN Movie_Genre MG ON G.Genre_ID = MG.Genre_ID
    JOIN Movie M        ON MG.Movie_ID = M.Movie_ID
    GROUP BY G.Genre_ID
)
SELECT DISTINCT G.Main_Genre, M.Title, GM.Max_Rating
FROM Genre G
JOIN Movie_Genre MG     ON G.Genre_ID = MG.Genre_ID
JOIN Movie M            ON MG.Movie_ID = M.Movie_ID
JOIN GenreMaxRatings GM ON G.Genre_ID = GM.Genre_ID
WHERE M.IMDB_Rating = GM.Max_Rating
ORDER BY GM.Max_Rating DESC;
```

## Key Findings

| Finding | Result |
|---|---|
| Highest-rated genres (avg. IMDb) | Drama **7.35**, Science Fiction **7.29** |
| Highest-rated directors (avg. IMDb) | Andy Wachowski **8.7**, Jonathan Demme **8.6**, Christopher Nolan **8.5** |
| Top movie by genre | *The Dark Knight* (Drama) **9.0**, *Inception* (Sci-Fi) **8.8** |

## Limitations

- **Entity matching across sources.** The six datasets have no shared IDs, so movies and people were linked by title and name, which can miss or mismatch records (remakes, same-name people).
- **Curated, not representative.** Some sources are "top" lists (e.g., Top 100 Rotten Tomatoes per genre), so the rankings describe well-known films, not all movies.
- **Small-sample director averages.** Directors with only a few films in the database can top the average-rating list.
- **Multi-genre double counting.** In the box-office-by-genre query, a film with several genres counts toward each of them, so genre totals overlap.

## Repository Contents

| File | Description |
|---|---|
| `movie.db` | Final SQLite database (also on [Google Drive](https://drive.google.com/file/d/1Hy-41lgQeHoT8DTi6KUzMGY-hIju1-bG/view?usp=drive_link)) |
| `schema.sql` | `CREATE TABLE` statements for all 7 tables, with primary and foreign keys |
| `queries.sql` | The five analysis queries |
| `images/erd.png` | Relational schema diagram (dbdiagram.io) |
| `data/` | Source CSVs (BoxOfficeCollections, Director, Genre, names, OldActor, rating_rottentomatto, titles) |
| `CDS_302_Final_Research_Report.pdf` | Final research report |
| `CDS_302_Project_movies.pdf` | Presentation slides |

## My Role

- **E-R modeling (lead):** designed the entity-relationship model, defined the five entities and set the cardinality of each relationship
- **Schema design:** refined the schema and reduced the E-R model to the 7-table relational design, including the two junction tables for the many-to-many relationships
- **Database build:** created the SQLite database, defined primary and foreign keys and loaded the integrated data

## Tech Stack

SQLite3 · SQL (JOIN, UNION, GROUP BY, HAVING, CTE) · dbdiagram.io · Kaggle datasets

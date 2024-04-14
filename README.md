# IMDb-Movies-Analysis-using-SQL-Project

use imdb;
------ SEGMENT 1: Database - Tables, Columns, Relationships


----- Q1. What are the different tables in the database and how are they connected to each other in the database?


/* Ans. The database contains the following tables:

1. movie: This table stores information about movies. It has a primary key `id` and columns such as `title`, `year`, `date_published`, `duration`, `country`, `worlwide_gross_income`, `languages`, and `production_company`.
2. genre: This table represents the genres of movies. It has a composite primary key `(movie_id, genre)` and contains the movie ID and genre for each movie.
3. director_mapping: This table maps movies to directors. It has a composite primary key `(movie_id, name_id)` and contains the movie ID and director name for each movie.
4. role_mapping: This table maps movies to actors/actresses and their roles. It has a composite primary key `(movie_id, name_id)` and contains the movie ID, actor/actress name, and role category for each movie.
5. names: This table stores information about people involved in movies, such as actors, actresses, and directors. It has a primary key `id` and columns like `name`, `height`, `date_of_birth`, and `known_for_movies`.
6. ratings: This table contains ratings information for movies. It has a primary key `movie_id` and columns such as `avg_rating`, `total_votes`, and `median_rating`.
These tables are connected to each other using foreign keys. 
The `movie_id` column in the genre, director_mapping, and role_mapping tables references the `id` column in the movie table. 
The `name_id` column in the director_mapping and role_mapping tables references the `id` column in the names table. 
These relationships allow for the association of movies with genres, directors, and actors/actresses.
*/
----- Q2. Find the total number of rows in each table of the schema.
SELECT COUNT(*) AS total_rows FROM movie;   --- 7997 rows
SELECT COUNT(*) AS total_rows FROM genre;   --- 14662 rows
SELECT COUNT(*) AS total_rows FROM director_mapping; --- 3867 rows 
SELECT COUNT(*) AS total_rows FROM role_mapping; --- 15615 rows
SELECT COUNT(*) AS total_rows FROM names;  --- 25735 rows
SELECT COUNT(*) AS total_rows FROM ratings; --- 7997 rows

----- Q3. Identify which columns in the movie table have null values.
SELECT 
    column_name
FROM information_schema.columns
WHERE table_name = 'movie'
    AND is_nullable = 'YES';
    
--- Ans:-  Column Names: languages, production_company, title, worldwide_gross_income, year
----------------------------------------------------------------------------------------------------------------------------------------------------
------- SEGMENT 2: Movie Release Trends
----- Q1. Determine the total number of movies released each year and analyse the month-wise trend.
SELECT 
    YEAR(date_published) AS release_year,
    MONTH(date_published) AS release_month,
    COUNT(*) AS movie_count
FROM
    movie
GROUP BY
    release_year, release_month
ORDER BY
    release_year, release_month;
--- Ans: Majority of the movies are released in the months of Sept, Oct, Nov in 2018 & 2017. In 2017 there were 3052 movies released which was highest in 3 years. 
----- Q2. Calculate the number of movies produced in the USA or India in the year 2019.
SELECT 
    COUNT(*) AS movie_count
FROM
    movie
WHERE
    (country = 'USA' OR country = 'India')
    AND year = 2019;
--- Ans: movie_count: 887
----------------------------------------------------------------------------------------------------------------------------------------------------
------- SEGMENT 3: Production Statistics and Genre Analysis
----- Q1.Retrieve the unique list of genres present in the dataset.
SELECT DISTINCT genre
FROM genre;
--- Ans: Genre; Drama, Fantasy, Triller, Comedy, Horror, Family, Romance, Adventure, Action, Sci-fi, Crime, Mystery, Others
----- Q2. Identify the genre with the highest number of movies produced overall.
SELECT genre, COUNT(*) AS movie_count
FROM genre
GROUP BY genre
ORDER BY movie_count DESC
LIMIT 1;
--- Ans: Genre - Drama || Movie Count - 4285
----- Q3. Determine the count of movies that belong to only one genre.
SELECT COUNT(*) AS movie_count
FROM (
    SELECT movie_id
    FROM genre
    GROUP BY movie_id
    HAVING COUNT(*) = 1
) AS single_genre_movies;
--- Ans: Movie Count - 3289
----- Q4. Calculate the average duration of movies in each genre.
SELECT genre, AVG(duration) AS average_duration
FROM movie
JOIN genre ON movie.id = genre.movie_id
GROUP BY genre;
/* Ans: genre	 	average_duration
		Drama	 	106.7746
		Fantasy	 	105.1404
		Thriller 	101.5761
		Comedy	 	102.6227
		Horror	 	92.7243
		Family	 	100.9669
		Romance	 	109.5342
		Adventure 	101.8714
		Action	  	112.8829
		Sci-Fi		97.9413
		Crime		107.0517
		Mystery		101.8
		Others		100.16
*/
----- Q5. Find the rank of the 'thriller' genre among all genres in terms of the number of movies produced.
SELECT genre, movie_count, genre_rank
FROM (
    SELECT genre, COUNT(*) AS movie_count,
           RANK() OVER (ORDER BY COUNT(*) DESC) AS genre_rank
    FROM genre
    GROUP BY genre
) AS genre_counts
WHERE genre = 'thriller';
--- Ans: Triller || Movie Count - 1484 || Rank - 3
----------------------------------------------------------------------------------------------------------------------------------------------------
------- SEGMENT 4: Ratings Analysis and Crew Members
----- Q1. Retrieve the minimum and maximum values in each column of the ratings table (except movie_id).
SELECT MIN(avg_rating) AS min_avg_rating, MAX(avg_rating) AS max_avg_rating,
       MIN(total_votes) AS min_total_votes, MAX(total_votes) AS max_total_votes,
       MIN(median_rating) AS min_median_rating, MAX(median_rating) AS max_median_rating
FROM ratings;
--- Ans. Avg_rating : Min - 1.0 || Max - 10.0
		 total_votes: Min - 100 || Max - 725138
         median_rating: Min - 1 || Max - 10

----- Q2. Identify the top 10 movies based on average rating.
SELECT id, title, avg_rating
FROM movie
INNER JOIN ratings ON movie.id = ratings.movie_id
ORDER BY avg_rating DESC
LIMIT 10;
/* Ans: 
id			title								avg_rating
tt10914342	Kirket								10
tt6735740	Love in Kilnerry					10
tt9537008	Gini Helida Kathe					9.8
tt10370434	Runam								9.7
tt10867504	Fan									9.6
tt9526826	Android Kunjappan Version 5.25		9.6
tt10869474	Safe								9.5
tt10901588	The Brighton Miracle				9.5
tt9680166	Yeh Suhaagraat Impossible			9.5
tt10405902	Shibu								9.4
*/
----- Q3 Summarise the ratings table based on movie counts by median ratings.
SELECT median_rating, COUNT(movie_id) AS movie_count
FROM ratings
GROUP BY median_rating;
/* Ans:
median_rating	movie_count
8				1030
7				2257
3				283
6				1975
9				429
2				119
4				479
5				985
10				346
1				94
*/
----- Q4. Identify the production house that has produced the most number of hit movies (average rating > 8).
SELECT COUNT(movie.id) AS hit_movie_count, movie.production_company, AVG(ratings.avg_rating) AS average_rating
FROM movie
INNER JOIN ratings ON movie.id = ratings.movie_id
WHERE ratings.avg_rating > 8 AND movie.production_company IS NOT NULL
GROUP BY movie.production_company
ORDER BY hit_movie_count DESC
Limit 1;
--- Ans: production_company:- Dream Warrior Picture || Hit_movie_count:- 3 || average_rating:- 8.63333

----- Q5. Determine the number of movies released in each genre during March 2017 in the USA with more than 1,000 votes.

SELECT genre.genre, COUNT(movie.id) AS movie_count
FROM movie
JOIN genre ON movie.id = genre.movie_id
JOIN ratings ON movie.id = ratings.movie_id
WHERE movie.country = 'USA'
  AND YEAR(movie.date_published) = 2017
  AND MONTH(movie.date_published) = 3
  AND ratings.total_votes > 1000
GROUP BY genre.genre
ORDER BY movie_count DESC;
/* Ans:
genre		movie_count
Drama		16
Comedy		8
Crime		5
Horror		5
Action		4
Sci-Fi		4
Thriller	4
Romance		3
Fantasy		2
Mystery		2
Family		1
*/
----- Q6. Retrieve movies of each genre starting with the word 'The' and having an average rating > 8.
SELECT COUNT(g.movie_id) AS movie_count, g.genre
FROM genre g
INNER JOIN (
    SELECT m.id
    FROM movie m
    INNER JOIN ratings r ON m.id = r.movie_id
    WHERE r.avg_rating > 8 AND m.title LIKE 'The%'
) AS sub ON g.movie_id = sub.id
GROUP BY g.genre;
/* Ans: 
movie_count	genre
7			Drama
1			Horror
1			Mystery
3			Crime
1			Action
1			Thriller
1			Romance
*/
----------------------------------------------------------------------------------------------------------------------------------------------------
------- SEGMENT 5: Crew Analysis
----- Q1. Identify the columns in the names table that have null values.
SELECT COLUMN_NAME
FROM INFORMATION_SCHEMA.COLUMNS
WHERE TABLE_NAME = 'names' 
AND IS_NULLABLE = 'YES';
--- Ans: Columns with Null Values are:- date_of_birth, height, known_for_movies, name

----- Q2. Determine the top three directors in the top three genres with movies having an average rating > 8.
SELECT genre.genre AS top_genre, AVG(ratings.avg_rating) AS highest_rated, names.name AS director_name
FROM movie
INNER JOIN ratings ON movie.id = ratings.movie_id
INNER JOIN genre ON genre.movie_id = movie.id
INNER JOIN director_mapping ON movie.id = director_mapping.movie_id
INNER JOIN names ON names.id = director_mapping.name_id
WHERE ratings.avg_rating > 8
GROUP BY top_genre, director_name
ORDER BY highest_rated DESC
LIMIT 3;
/* Ans: 
top_genre	highest_rated	director_name
Romance		9.7				Srinivas Gundareddy
Drama		9.6				Balavalli Darshith Bhat
Action		9.5				Pradeep Kalipurayath
*/

----- Q3. Find the top two actors whose movies have a median rating >= 8.
SELECT names.name AS actor_name, AVG(ratings.median_rating) AS average_median_rating
FROM names
INNER JOIN role_mapping ON names.id = role_mapping.name_id
INNER JOIN ratings ON role_mapping.movie_id = ratings.movie_id
GROUP BY actor_name
HAVING AVG(ratings.median_rating) >= 8
ORDER BY average_median_rating DESC
LIMIT 2;
/* Ans:
rating	name
10		Aamir Qureshi
10		Aarav Mavi
*/

----- Q4. Identify the top three production houses based on the number of votes received by their movies.
SELECT movie.production_company, SUM(ratings.total_votes) AS total_votes
FROM movie
INNER JOIN ratings ON movie.id = ratings.movie_id
GROUP BY movie.production_company
ORDER BY total_votes DESC
LIMIT 3;
/* Ans:
production_company			total_votes
Marvel Studios				2656967
Twentieth Century Fox		411163
Warner Bros.				2396057
*/

----- Q5. Rank actors based on their average ratings in Indian movies released in India.
SELECT names.name AS actor_name, AVG(ratings.avg_rating) AS average_rating
FROM movie
INNER JOIN role_mapping ON movie.id = role_mapping.movie_id
INNER JOIN names ON role_mapping.name_id = names.id
INNER JOIN ratings ON movie.id = ratings.movie_id
WHERE movie.country = 'India' AND role_mapping.category = 'actor'
GROUP BY names.name
ORDER BY average_rating DESC;

----- Q6. Identify the top five actresses in Hindi movies released in India based on their average ratings.
SELECT names.name AS actress_name, AVG(ratings.avg_rating) AS average_rating
FROM movie
INNER JOIN role_mapping ON movie.id = role_mapping.movie_id
INNER JOIN names ON role_mapping.name_id = names.id
INNER JOIN ratings ON movie.id = ratings.movie_id
WHERE movie.country = 'India' AND movie.languages LIKE '%Hindi%' AND role_mapping.category = 'actress'
GROUP BY names.name
ORDER BY average_rating DESC
LIMIT 5;
/* Ans:
actress_name		average_rating
Pranati Rai Prakash	9.4
Leera Kaljai		9.2
Puneet Sikka		8.7
Bhairavi Athavle	8.4
Radhika Apte		8.4
*/

----------------------------------------------------------------------------------------------------------------------------------------------------
------- SEGMENT 6: Broader Understanding of Data
----- Q1. Classify thriller movies based on average ratings into different categories.
SELECT
    movie.title AS movie_title,
    ratings.avg_rating AS average_rating,
    CASE
        WHEN ratings.avg_rating >= 8.5 THEN 'Excellent'
        WHEN ratings.avg_rating >= 7.5 THEN 'Very Good'
        WHEN ratings.avg_rating >= 6.5 THEN 'Good'
        ELSE 'Average or Below'
    END AS rating_category
FROM
    movie
    INNER JOIN genre ON movie.id = genre.movie_id
    INNER JOIN ratings ON movie.id = ratings.movie_id
WHERE
    genre.genre = 'Thriller'
ORDER BY
    ratings.avg_rating DESC;

----- Q2. analyse the genre-wise running total and moving average of the average movie duration.
SELECT
    genre.genre AS movie_genre,
    movie.duration AS movie_duration,
    SUM(movie.duration) OVER (PARTITION BY genre.genre ORDER BY movie.year) AS running_total,
    AVG(movie.duration) OVER (PARTITION BY genre.genre ORDER BY movie.year ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS moving_average
FROM
    movie
    INNER JOIN genre ON movie.id = genre.movie_id
GROUP BY
    genre.genre, movie.duration, movie.year
ORDER BY
    genre.genre, movie.year;

----- Q3. Identify the five highest-grossing movies of each year that belong to the top three genres.
WITH top_three_genres AS (
    SELECT genre, COUNT(*) AS movie_count
    FROM genre
    GROUP BY genre
    ORDER BY movie_count DESC
    LIMIT 3
),
highest_grossing_movies AS (
    SELECT m.year, m.title, m.worlwide_gross_income, g.genre,
           ROW_NUMBER() OVER (PARTITION BY m.year, g.genre ORDER BY m.worlwide_gross_income DESC) AS `rank`
    FROM movie m
    INNER JOIN genre g ON m.id = g.movie_id
    INNER JOIN top_three_genres t ON g.genre = t.genre
)
SELECT year, genre, title, worlwide_gross_income
FROM highest_grossing_movies
WHERE `rank` <= 5
ORDER BY year, genre, `rank`;

----- Q4. Determine the top two production houses that have produced the highest number of hits among multilingual movies.
WITH hit_movies AS (
    SELECT m.production_company, COUNT(*) AS hit_count
    FROM movie m
    INNER JOIN ratings r ON m.id = r.movie_id
    WHERE r.avg_rating >= 7.0
    AND m.production_company IS NOT NULL
    GROUP BY m.production_company
),
top_production_houses AS (
    SELECT production_company, hit_count
    FROM hit_movies
    ORDER BY hit_count DESC
    LIMIT 2
)
SELECT production_company, hit_count
FROM top_production_houses;

--- Ans: A24:- 7 || Warner Bros. :- 6
----- Q5. Identify the top three actresses based on the number of Super Hit movies (average rating > 8) in the drama genre.

WITH super_hit_drama_movies AS (
    SELECT m.id AS movie_id, m.title, r.avg_rating
    FROM movie m
    INNER JOIN ratings r ON m.id = r.movie_id
    INNER JOIN genre g ON m.id = g.movie_id
    WHERE r.avg_rating > 8.0
    AND g.genre = 'drama'
),
actresses AS (
    SELECT m.id AS movie_id, nm.name
    FROM movie m
    INNER JOIN role_mapping rm ON m.id = rm.movie_id
    INNER JOIN names nm ON rm.name_id = nm.id
    WHERE rm.category = 'actress'
),
actress_movie_count AS (
    SELECT a.name, COUNT(*) AS movie_count
    FROM super_hit_drama_movies s
    INNER JOIN actresses a ON s.movie_id = a.movie_id
    GROUP BY a.name
),
ranked_actresses AS (
    SELECT name, movie_count, ROW_NUMBER() OVER (ORDER BY movie_count DESC) AS `rank`
    FROM actress_movie_count
)
SELECT name, movie_count
FROM ranked_actresses
WHERE `rank` <= 3;

---- Option 2
-- Query to identify the top three actresses based on the number of Super Hit movies (average rating > 8) in the drama genre

SELECT a.name, COUNT(*) AS movie_count
FROM movie m
INNER JOIN ratings r ON m.id = r.movie_id
INNER JOIN genre g ON m.id = g.movie_id
INNER JOIN role_mapping rm ON m.id = rm.movie_id
INNER JOIN names a ON rm.name_id = a.id
WHERE r.avg_rating > 8.0
AND g.genre = 'drama'
AND rm.category = 'actress'
GROUP BY a.name
ORDER BY movie_count DESC
LIMIT 3;

/* Ans:
name				movie_count
Parvathy Thiruvothu	2
Susan Brown			2
Amanda Lawrence		2
*/
----- Q6. Retrieve details for the top nine directors based on the number of movies, including average inter-movie duration, ratings, and more.
WITH director_movie_count AS (
    SELECT dm.name_id, nm.name, COUNT(*) AS movie_count
    FROM director_mapping dm
    INNER JOIN names nm ON dm.name_id = nm.id
    GROUP BY dm.name_id, nm.name
),
director_average_duration AS (
    SELECT dm.name_id, AVG(m.duration) AS average_duration
    FROM director_mapping dm
    INNER JOIN movie m ON dm.movie_id = m.id
    GROUP BY dm.name_id
),
director_total_ratings AS (
    SELECT dm.name_id, SUM(r.total_votes) AS total_votes
    FROM director_mapping dm
    INNER JOIN ratings r ON dm.movie_id = r.movie_id
    GROUP BY dm.name_id
),
ranked_directors AS (
    SELECT dmc.name_id, dmc.name, dmc.movie_count, ad.average_duration, tr.total_votes,
           ROW_NUMBER() OVER (ORDER BY dmc.movie_count DESC) AS `rank`
    FROM director_movie_count dmc
    LEFT JOIN director_average_duration ad ON dmc.name_id = ad.name_id
    LEFT JOIN director_total_ratings tr ON dmc.name_id = tr.name_id
)
SELECT name, movie_count, average_duration, total_votes
FROM ranked_directors
WHERE `rank` <= 9;

-- 2nd option

SELECT nm.name, COUNT(*) AS movie_count, AVG(m.duration) AS average_duration, SUM(r.total_votes) AS total_votes
FROM director_mapping dm
INNER JOIN names nm ON dm.name_id = nm.id
INNER JOIN movie m ON dm.movie_id = m.id
INNER JOIN ratings r ON dm.movie_id = r.movie_id
GROUP BY dm.name_id, nm.name
ORDER BY movie_count DESC
LIMIT 9;

/* Ans:
name				movie_count	average_duration	total_votes
A.L. Vijay			5			122.6				1754
Andrew Jones		5			86.4				1989
Chris Stokes		4			88					3664
Justin Price		4			86.5				5343
Jesse V. Johnson	4			95.75				14778
Steven Soderbergh	4			100.25				171684
Sion Sono			4			125.5				2972
Özgür Bakar			4			93.5				1092
Sam Liu				4			78					28557
*/
----------------------------------------------------------------------------------------------------------------------------------------------------
/*  Segment 7: Recommendations
Based on the analysis, provide recommendations for the types of content Bolly Movies should focus on producing.
*/
/* Ans: Based on the Analysis of the IMBd Movies, the recommendations for the types of content Bolly Movies should focus on producing is:-

          1. The 'Triller' genre has caught the highest attention and interest amongst the audience as the amount of 'Thriller' movies watched is good,
	         so the Bollywood movie production houses should keep their interest towards producing more 'Thriller' genre movies. 
       
          2. The 'Drama' genre has gained the overall average highest IMDb rating by the audience, so the Bollywood movies production houses 
             should focus more on producing quality content movies in the 'Drama' genre as they have been doing.
       
          3. The Bollywood movie production houses should also focus on producing good quality movies in other genres as well for the 
             growth of the bollywood movie industry.

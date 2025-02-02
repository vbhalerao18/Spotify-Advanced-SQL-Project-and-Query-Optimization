# Spotify-Advanced-SQL-Project-and-Query-Optimization

This project involves analyzing a Spotify dataset by normalizing a denormalized structure and applying advanced SQL techniques. Key tasks include transforming data into a relational schema, writing complex queries (e.g., joins, subqueries, window functions), and optimizing query performance through indexing and query refactoring. The goal is to gain expertise in advanced SQL while extracting meaningful insights and ensuring scalable, high-performance queries on large datasets.

-- create table
DROP TABLE IF EXISTS spotify;
CREATE TABLE spotify (
    artist VARCHAR(255),
    track VARCHAR(255),
    album VARCHAR(255),
    album_type VARCHAR(50),
    danceability FLOAT,
    energy FLOAT,
    loudness FLOAT,
    speechiness FLOAT,
    acousticness FLOAT,
    instrumentalness FLOAT,
    liveness FLOAT,
    valence FLOAT,
    tempo FLOAT,
    duration_min FLOAT,
    title VARCHAR(255),
    channel VARCHAR(255),
    views FLOAT,
    likes BIGINT,
    comments BIGINT,
    licensed BOOLEAN,
    official_video BOOLEAN,
    stream BIGINT,
    energy_liveness FLOAT,
    most_played_on VARCHAR(50)
);

SELECT * FROM public.spotify

--Retrieve the names of all tracks that have more than 1 billion streams.
Select * from spotify  where spotify.stream > 1000000000


--List all albums along with their respective artists.
Select DISTINCT album, artist from spotify


--Get the total number of comments for tracks where licensed = TRUE.
Select SUM(spotify."comments")From spotify where spotify.licensed='true'


--Find all tracks that belong to the album type single.
Select * From spotify where spotify.album_type='single'


--Count the total number of tracks by each artist
SELECT spotify.artist, COUNT(spotify.track) AS track_count
FROM spotify
GROUP BY spotify.artist;


--Calculate the average danceability of tracks in each album.
SELECT album, AVG(danceability) AS avg_danceability
FROM spotify
GROUP BY album order by 2 desc


--Find the top 5 tracks with the highest energy values.

SELECT track, MAX(energy)
FROM public.spotify Group by 1
Order by 2 desc
limit 5


--List all tracks along with their views and likes where official_video = TRUE.

Select 
track , 
Sum(spotify."views") as Views,
Sum(spotify.likes) as Likes
From spotify
group by 1
order by 2 desc


--For each album, calculate the total views of all associated tracks.

Select  track, album , Sum(spotify."views") from spotify group by 1, 2 order by 3 desc


---Retrieve the track names that have been streamed on Spotify more than YouTube

SELECT *
FROM (
    SELECT
        track,
        --most_played_on,
        COALESCE(SUM(CASE WHEN most_played_on = 'YouTube' THEN stream END), 0) AS streamed_on_youtube,
        COALESCE(SUM(CASE WHEN most_played_on = 'Spotify' THEN stream END), 0) AS streamed_on_spotify
    FROM spotify
    GROUP BY track
) AS tl
WHERE streamed_on_spotify > streamed_on_youtube
AND streamed_on_youtube <> 0;

---Find the top 3 most-viewed tracks for each artist using window functions.
With Rank_art AS
(Select
artist,
track,
Sum(Views) As total_view,
DENSE_RANK() over(Partition by artist order by Sum(Views) desc ) as rank
From spotify
group by 1,2
order by 1,3 desc)

Select * from Rank_art where rank <=3

--Write a query to find tracks where the liveness score is above the average.
WITH liveness_score AS (
    SELECT 
        track,
        spotify.liveness,
        AVG(spotify.liveness) OVER() AS AverageLiveness
    FROM spotify
)
SELECT *
FROM liveness_score
WHERE liveness > AverageLiveness
ORDER BY liveness DESC;

---Use a WITH clause to calculate the difference between the highest and lowest energy values for tracks in each album

WITH cte
AS
(SELECT 
	album,
	MAX(energy) as highest_energy,
	MIN(energy) as lowest_energery
FROM spotify
GROUP BY 1
)
SELECT 
	album,
	highest_energy - lowest_energery as energy_diff
FROM cte
ORDER BY 2 DESC






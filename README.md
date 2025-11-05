🎵 Spotify Data Analysis SQL Queries

Welcome to my Spotify Data Analysis repository! Here, I explore a dataset of Spotify tracks using SQL Server (SSMS). 
The queries demonstrate a variety of SQL techniques including aggregation, window functions, filtering, and ranking.


-- Create the database and use it
CREATE DATABASE spotify_;
USE spotify_;

-- Explore the table
SELECT * FROM Spotify_table;

-- 1️⃣ Tracks with more than 1 billion streams
SELECT * 
FROM Spotify_table 
WHERE views > 1000000000;

-- 2️⃣ List all albums with their respective artists
SELECT Album, Artist 
FROM Spotify_table;

-- 3️⃣ Total comments for licensed tracks
SELECT Track, SUM(comments) AS Total_comments 
FROM Spotify_table 
WHERE licensed = 1
GROUP BY Track;

-- 4️⃣ Tracks of album type `single`
SELECT Track, Album, Album_type 
FROM Spotify_table 
WHERE album_type='single';

-- 5️⃣ Count of tracks by each artist
SELECT Artist, COUNT(track) AS No_of_Tracks 
FROM Spotify_table 
GROUP BY Artist 
ORDER BY No_of_Tracks DESC;

-- 6️⃣ Average danceability per album
SELECT Album, AVG(danceability) AS Avg_danceability 
FROM Spotify_table 
GROUP BY Album 
ORDER BY Avg_danceability DESC;

-- 7️⃣ Top 5 tracks with highest energy
SELECT TOP 5 Track, MAX(Energy) AS Max_energy 
FROM Spotify_table 
GROUP BY Track 
ORDER BY Max_energy DESC;

-- 8️⃣ Tracks with official video
SELECT Track, Views, Likes 
FROM Spotify_table 
WHERE official_video = 1;

-- 9️⃣ Total views per album
SELECT Album, SUM(views) AS Total_views 
FROM Spotify_table 
GROUP BY Album 
ORDER BY Total_views DESC;

-- 🔟 Tracks streamed more on Spotify than YouTube
SELECT * 
FROM (
    SELECT Track,
           COALESCE(SUM(CASE WHEN most_playedon='Spotify' THEN Stream END),0) AS Spotify_count,
           COALESCE(SUM(CASE WHEN most_playedon='YouTube' THEN Stream END),0) AS Youtube_count
    FROM Spotify_table
    GROUP BY Track
) AS R1
WHERE Spotify_count > Youtube_count
  AND Youtube_count <> 0;

-- 1️⃣1️⃣ Top 3 most-viewed tracks per artist (Window Function)
WITH artist_view AS (
    SELECT Artist, Track, SUM(views) AS total_v,
           DENSE_RANK() OVER (PARTITION BY Artist ORDER BY SUM(views) DESC) AS rank_
    FROM Spotify_table 
    GROUP BY Artist, Track
)
SELECT * 
FROM artist_view 
WHERE rank_ <= 3;

-- 1️⃣2️⃣ Tracks with liveness above average
SELECT Track, Liveness
FROM Spotify_table
WHERE Liveness > (SELECT AVG(Liveness) FROM Spotify_table);

-- 1️⃣3️⃣ Energy difference per album (WITH clause)
WITH diff AS (
    SELECT Album,
           MAX(Energy) AS Highest_,
           MIN(Energy) AS Lowest_
    FROM Spotify_table
    GROUP BY Album
)
SELECT Album, Highest_ - Lowest_ AS Total_diff
FROM diff
ORDER BY Album DESC;

-- 1️⃣4️⃣ Tracks with high energy-to-liveness ratio
SELECT Album, Track, Energy, Liveness, (Energy/Liveness) AS Ratio
FROM Spotify_table
WHERE (Energy/Liveness) > 1.2;

-- 1️⃣5️⃣ Cumulative sum of likes by views
SELECT Track, Views, Likes,
       SUM(Likes) OVER (ORDER BY Views) AS Cumulative_likes
FROM Spotify_table
ORDER BY Views DESC;

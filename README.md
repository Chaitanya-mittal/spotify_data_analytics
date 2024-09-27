# Spotify Advanced SQL Project and Query Optimization P-6
Project Category: Advanced
[Click Here to get Dataset](https://www.kaggle.com/datasets/sanjanchaudhari/spotify-dataset)

![Spotify Logo](https://github.com/najirh/najirh-Spotify-Data-Analysis-using-SQL/blob/main/spotify_logo.jpg)

## DEFINE SCHEMA

```sql
Drop table if exists spotify;
create table spotify (
	artist varchar(255),
	track varchar(255),
	album varchar(255),
	album_type varchar(50),
	danceability FLOAT,
	energy FLOAT,
	loudness FLOAT,
	speechiness FLOAT,
	acousticness FLOAT,
	instrumentalness FLOAT,
	liveness FLOAT,
	valence FLOAT,
	tempo FLOAT ,
	duration_min FLOAT , 
	title varchar(255),
	channel varchar(255),
	views BIGINT,
	likes BIGINT,
	comments BIGINT,
	licensed BOOLEAN , 
	official_video BOOLEAN , 
	stream BIGINT,
	energy_liveness FLOAT,
	most_played_on varchar(50)
);

```
## Project Steps

### 1. Data Exploration

####  1.1 count number of entries 

```sql
select count(*)
from spotify;
```

#### 1.2 Find distinct artists

```sql
select distinct artist
from spotify;

select count(distinct(artist))
from spotify;
```

#### 1.3 find distinct album

```sql
select count(distinct(album))
from spotify;
```

#### 1.4 find all album_type

```sql
select count(distinct(album_type))
from spotify;

select distinct album_type
from spotify;
```

#### 1.5 Analyze duration

```sql
select max(duration_min) , min(duration_min)
from spotify; 
```

#### 1.6 Finding songs with duration 0

```sql
select * 
from spotify
where duration_min = 0;
``` 

#### 1.7 Delete the songs with duration 0

```sql
Delete  
from spotify
where duration_min = 0;
```

#### 1.8 Find all the artists for each track

```sql
select album , track , STRING_AGG(artist,', ') as artists 
from spotify
group by 1 , 2 
```

## 15 Business Problems


### 1. Retrieve the names of all tracks that have more than 1 billion streams.

```sql
select track , stream
from spotify
where stream >= 100000000;
```

### 2. List all albums along with their respective artists.

```sql
select album ,STRING_AGG(artist ,  ', ')
from spotify
group by 1
order by 1;
```

### 3. Get the total number of comments for tracks where licensed = TRUE.

```sql
-- Total comments for each track
select album , track , SUM(DISTINCT(comments)) as total_comments
from spotify
where licensed = true
group by 1,2
order by 4 desc;

-- Overall Total_comments
select SUM(Distinct(comments)) as total_comments
from spotify
where licensed='true'
```

### 4. Find all tracks that belong to the album type single.

```sql
select track
from spotify
where album_type='single';
```

### 5. Count the total number of tracks by each artist.

```sql
select artist , count(track) as number_of_tracks
from spotify
group by 1;
```

### 6. Calculate the average danceability of tracks in each album.

```sql
select album, AVG(danceability)
from spotify
group by 1
order by 1;
```

### 7. Find the top 5 tracks with the highest energy values.

```sql
select track , energy 
from spotify
group by 1,2
order by  2 desc
LIMIT 5;
```

### 8. List all tracks along with their views and likes where `official_video = TRUE`.

```sql
select track, album  , AVG(views) , AVG(likes)
from spotify
where official_video = 'true'
group by 1 , 2
order by 1, 2;
```

### 9. For each album, calculate the total views of all associated tracks.

```sql
select album , track  , SUM(DISTINCT(views)) as total_views  
from spotify
group by 1 , 2 
order by 4 desc;
```

### 10. Retrieve the track names that have been streamed on Spotify more than YouTube.

```sql
-- WAY:1
select track , album 
	SUM(CASE WHEN most_played_on = 'Spotify' THEN stream ELSE -stream END) as resultant_streams  , 
	CASE WHEN SUM(CASE WHEN most_played_on = 'Spotify' THEN stream ELSE -stream END) > 0 then 'spotify'
	ELSE 'youtube' END as Platform 
	from spotify
	group by 1,2,
	Order by 1,2;

-- WAY:2
select track , album
from
(
select track , album 
	SUM(CASE WHEN most_played_on = 'Spotify' THEN stream ELSE 0  END) as resultant_streams_spotify,
	SUM(CASE WHEN most_played_on = 'Youtube' THEN stream ELSE 0  END) as resultant_streams_youtube
	from spotify
	group by 1,2
	Order by 1,2
) as t
where resultant_streams_spotify > resultant_streams_youtube ;
```

### 11. Find the top 3 most-viewed tracks for each artist using window functions.

```sql
WITH artist_hits 
AS 
(SELECT ARTIST ,  TRACK , ROUND(AVG(views),2) as views ,DENSE_RANK() OVER(PARTITION BY artist Order by AVG(views) desc) as priority
FROM SPOTIFY
GROUP BY 1 , 2 
)
select * 
from artist_hits
where priority <=3;
```

### 12. Write a query to find tracks where the liveness score is above the average.

```sql
select * 
from Spotify 
where liveness > (Select avg(liveness) as avg_liveness
from spotify);
```

### 13. **Use a `WITH` clause to calculate the difference between the highest and lowest energy values for tracks in each album.**

```sql
WITH Energy_Level
AS
	(select album ,  min(energy) as min_energy , max(energy) as max_energy
	from spotify
	group by 1)
select album , max_energy - min_energy as diff_energy 
from Energy_Lev
Order by 2 desc;
```

### 14. Find tracks where the energy-to-liveness ratio is greater than 1.2.

```sql
select distinct(track) , energy_liveness
from spotify
where energy_liveness > 1.2
	order by 2;
```

### 15. Calculate the cumulative sum of likes for tracks ordered by the number of views, using window functions.

```sql
SELECT
    track,
    views,
    likes,
    SUM(likes) OVER (Partition by track ) AS cumulative_likes
FROM
    spotify
	order by views desc;
```

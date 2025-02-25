#  Introduction to Database Systems



##  1 Single-Table SQL

**编写SQL查询语句来完成以下任务。你不需要连接任何表。假设你可以访问以下模式的表格，其中每个主键都用大写字母表示：**

- **Songs(SONG_ID, song_name, album_id, weeks_in_top_40)**
- **Artists(ARTIST_ID, artist_name, first_yr_active)**
- **Albums(ALBUM_ID, album_name, artist_id, yr_released, genre)**

**(a) 查找在Top 40中待的周数最少的5首歌曲，按从少到多的顺序排序。如果有并列情况，按歌曲名称的字母顺序打破并列。**

```sql
SELECT song_name
FROM Songs
ORDER BY weeks_in_top_40 ASC, song_name ASC
LIMIT 5;
```

**(b) 查找每位名字以字母'B'开头的艺术家的名字和首次活跃年份。**

```sql
SELECT artist_name, first_yr_active
FROM Artists
WHERE artist_name ~ '^B.*';
```

**(c) 查找每种类型的专辑发布总数。**

```sql
SELECT genre, COUNT(album_id)
FROM Albums
GROUP BY genre;
```

**(d) 查找每种类型的专辑发布总数。排除发布数量少于10的类型。**

```sql
SELECT genre, COUNT(*)
FROM Albums
GROUP BY genre
HAVING COUNT(*) >= 10;
```

**(e) 查找在2000年发布专辑最多的类型。假设没有并列情况。**

```sql
SELECT genre
FROM Albums
WHERE yr_released = 2020
GROUP BY genre
ORDER BY COUNT(*) DESC
LIMIT 1;
```



## 2 Multi-Table SQL

**编写SQL查询语句来完成以下任务。使用前一个问题中的相同表格（从首页复制）。你需要使用连接操作：**

- **Songs(SONG_ID, song_name, album_id, weeks_in_top_40)**
- **Artists(ARTIST_ID, artist_name, first_yr_active)**
- **Albums(ALBUM_ID, album_name, artist_id, yr_released, genre)**

**(a) 查找在2020年发布了“乡村”类型专辑的所有艺术家的名字。**

```sql
SELECT artist_name
FROM Artists AS ar INNER JOIN Albums AS al ON ar.ARTIST_ID = al.artist_id
WHERE yr_released = 2020 AND genre = ‘country’
GROUP BY Artists.artist_id, artist_name;
```

**(b) 查找那首在Top 40中待的周数最多的歌曲所属专辑的名字。假设只有一首这样的歌曲。**

```sql
SELECT album_name
FROM Songs INNER JOIN Albums ON Songs.album_id = Albums.ALBUM_ID
ORDER BY Songs.weeks_in_top_40 DESC
LIMIT 1;
```

**(c) 查找每位艺术家的名字以及他们的歌曲在Top 40中待的最长周数。包括那些没有发布专辑的艺术家。**

```sql
WITH SongOfArtists(SONG_ID, weeks_in_top_40, artist_id) AS
(SELECT SONG_ID, weeks_in_top_40, artist_id
FROM Songs INNER JOIN Albums ON Songs.album_id = Albums.ALBUM_ID)

SELECT artist_name, MAX(weeks_in_top_40)
FROM Artists LEFT JOIN SongOfArtists ON Artists.ARTIST_ID = SongOfArtists.artist_id
GROUP BY Artists.artist_id, artist_name;
```


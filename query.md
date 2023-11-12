# 以下のデータを抽出するクエリの記述

## 1.よく見られているエピソードを知りたいです。エピソード視聴数トップ3のエピソードタイトルと視聴数を取得してください

```sql
SELECT title, views
FROM episodes
ORDER BY views DESC
LIMIT 3;
```

## 2.よく見られているエピソードの番組情報やシーズン情報も合わせて知りたいです。エピソード視聴数トップ3の番組タイトル、シーズン数、エピソード数、エピソードタイトル、視聴数を取得してください

```sql
SELECT 
    p.title AS 'Program Title',
    s.season_number AS 'Season Number',
    e.episode_number AS 'Episode Number',
    e.title AS 'Episode Title',
    e.views AS 'Views'
FROM episodes e
JOIN seasons s ON e.season_id = s.id
JOIN programs p ON s.program_id = p.id
ORDER BY e.views DESC
LIMIT 3;
```

## 2.本日の番組表を表示するために、本日、どのチャンネルの、何時から、何の番組が放送されるのかを知りたいです。本日放送される全ての番組に対して、チャンネル名、放送開始時刻(日付+時間)、放送終了時刻、シーズン数、エピソード数、エピソードタイトル、エピソード詳細を取得してください。なお、番組の開始時刻が本日のものを本日方法される番組とみなすものとします

```sql
SELECT 
    ch.name AS 'Channel Name',
    ps.start_time AS 'Start Time',
    ps.end_time AS 'End Time',
    s.season_number AS 'Season Number',
    e.episode_number AS 'Episode Number',
    e.title AS 'Episode Title',
    e.description AS 'Episode Description'
FROM 
    program_schedules ps
INNER JOIN 
    channels ch ON ps.channel_id = ch.id
INNER JOIN 
    episodes e ON ps.episode_id = e.id
INNER JOIN 
    seasons s ON e.season_id = s.id
WHERE 
    DATE(ps.start_time) = CURDATE()
ORDER BY 
    ps.start_time ASC;
```

## 4.ドラマというチャンネルがあったとして、ドラマのチャンネルの番組表を表示するために、本日から一週間分、何日の何時から何の番組が放送されるのかを知りたいです。ドラマのチャンネルに対して、放送開始時刻、放送終了時刻、シーズン数、エピソード数、エピソードタイトル、エピソード詳細を本日から一週間分取得してください

```sql
SELECT 
    ch.name AS 'Channel Name',
    ps.start_time AS 'Start Time',
    ps.end_time AS 'End Time',
    s.season_number AS 'Season Number',
    e.episode_number AS 'Episode Number',
    e.title AS 'Episode Title',
    e.description AS 'Episode Description'
FROM 
    program_schedules ps
INNER JOIN 
    channels ch ON ps.channel_id = ch.id
INNER JOIN 
    episodes e ON ps.episode_id = e.id
INNER JOIN 
    seasons s ON e.season_id = s.id
WHERE 
    ch.name = 'ドラマ' AND
    ps.start_time BETWEEN CURDATE() AND DATE_ADD(CURDATE(), INTERVAL 7 DAY)
ORDER BY 
    ps.start_time ASC;
```

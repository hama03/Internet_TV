# インターネットTV

好きな時間に好きな場所で話題の動画を無料で楽しめる「インターネットTVサービス」を新規に作成することになりました。データベース設計をした上で、データ取得する SQL を作成してください。

仕様は次の通りです。サービスのイメージとしては ABEMA を頭に思い浮かべてください。

ドラマ1、ドラマ2、アニメ1、アニメ2、スポーツ、ペットなど、複数のチャンネルがある
各チャンネルの下では時間帯ごとに番組枠が1つ設定されており、番組が放映される
番組はシリーズになっているものと単発ものがある。シリーズになっているものはシーズンが1つものと、シーズン1、シーズン2のように複数シーズンのものがある。各シーズンの下では各エピソードが設定されている
再放送もあるため、ある番組が複数チャンネルの異なる番組枠で放映されることはある
番組の情報として、タイトル、番組詳細、ジャンルが画面上に表示される
各エピソードの情報として、シーズン数、エピソード数、タイトル、エピソード詳細、動画時間、公開日、視聴数が画面上に表示される。単発のエピソードの場合はシーズン数、エピソード数は表示されない
ジャンルとしてアニメ、映画、ドラマ、ニュースなどがある。各番組は1つ以上のジャンルに属する
KPIとして、チャンネルの番組枠のエピソードごとに視聴数を記録する。なお、一つのエピソードは複数の異なるチャンネル及び番組枠で放送されることがあるので、属するチャンネルの番組枠ごとに視聴数がどうだったかも追えるようにする
番組、シーズン、エピソードの関係について、以下のようなイメージです(シリーズになっているものの例)。

番組：鬼滅の刃
シーズン：1
エピソード：1話、2話、...、26話

## sqlコマンド

`sudo apt update`
`sudo apt install mysql-server`
`sudo systemctl status mysql.service`
`sudo mysql -u root`
`SHOW DATABASES;`
`USE internetTV;`
`SHOW TABLES;`
`SELECT * FROM table_name;`
`DELETE FROM table_name;`
`ALTER TABLE channels AUTO_INCREMENT = 1;`
`DELETE FROM channels WHERE channels_id = 5;`
`TRUNCATE TABLE channels;`

## Step1テーブル作成

### チャンネル情報を管理:channels

```sql
CREATE TABLE IF NOT EXISTS channels (
    id BIGINT(20) NOT NULL AUTO_INCREMENT,
    name VARCHAR(100) NOT NULL,
    description TEXT,
    PRIMARY KEY (id)
);
```

### ジャンル情報を管理:genres

```sql
CREATE TABLE IF NOT EXISTS genres (
    id BIGINT(20) NOT NULL AUTO_INCREMENT,
    name VARCHAR(100) NOT NULL,
    PRIMARY KEY (id)
);
```

### 番組情報を管理:programs

```sql
CREATE TABLE IF NOT EXISTS programs (
    id BIGINT(20) NOT NULL AUTO_INCREMENT,
    title VARCHAR(200) NOT NULL,
    description TEXT,
    genre_id BIGINT(20),
    PRIMARY KEY (id),
    FOREIGN KEY (genre_id) REFERENCES genres(id)
        ON DELETE RESTRICT ON UPDATE CASCADE
);
```

### シリーズ番組のシーズン情報を管理:seasons

```sql
CREATE TABLE IF NOT EXISTS seasons (
    id BIGINT(20) NOT NULL AUTO_INCREMENT,
    program_id BIGINT(20) NOT NULL,
    season_number INT(11) NOT NULL,
    PRIMARY KEY (id),
    FOREIGN KEY (program_id) REFERENCES programs(id)
        ON DELETE CASCADE ON UPDATE CASCADE
);
```

### 各エピソード情報を管理:episodes

```sql
CREATE TABLE IF NOT EXISTS episodes (
    id BIGINT(20) NOT NULL AUTO_INCREMENT,
    season_id BIGINT(20),
    episode_number INT(11),
    title VARCHAR(200) NOT NULL,
    description TEXT,
    duration INT(11) NOT NULL,
    views BIGINT(20) NOT NULL DEFAULT 0,
    PRIMARY KEY (id),
    FOREIGN KEY (season_id) REFERENCES seasons(id)
        ON DELETE SET NULL ON UPDATE CASCADE
);

```

### 番組スケジュール情報を管理: program_schedules

```sql
CREATE TABLE IF NOT EXISTS program_schedules (
    id BIGINT(20) NOT NULL AUTO_INCREMENT,
    channel_id BIGINT(20) NOT NULL,
    episode_id BIGINT(20) NOT NULL,
    start_time DATETIME NOT NULL,
    end_time DATETIME NOT NULL,
    PRIMARY KEY (id),
    FOREIGN KEY (channel_id) REFERENCES channels(id)
        ON DELETE RESTRICT ON UPDATE CASCADE,
    FOREIGN KEY (episode_id) REFERENCES episodes(id)
        ON DELETE RESTRICT ON UPDATE CASCADE
);
```

## Step2サンプルコード格納

```sql
INSERT INTO channels (name, description) VALUES (
'アニメ1', '最新のアニメシリーズを放送するチャンネル'),
('アニメ2', '様々なアニメジャンルをカバーするチャンネル'),
('映画', '国内外の様々な映画を放送するチャンネル'),
('ドラマ1', '国内ドラマを中心に放送するチャンネル'),
('ドラマ2', '海外ドラマを中心に放送するチャンネル'),
('ニュース', '24時間最新ニュースを提供するチャンネル'
);
```

```sql
INSERT INTO genres (name) VALUES (
'アニメ'),
('映画'),
('ドラマ'),
('ニュース'
);
```

```sql
INSERT INTO programs (title, description, genre_id) VALUES (
('鬼滅の刃', '若き剣士の成長と冒険を描いたアニメシリーズ。', (SELECT id FROM genres WHERE name = 'アニメ')),
('リコリス・リコイル', 'カフェを舞台にした日常と、秘密組織に所属する少女たちの活躍を描いたアニメ。', (SELECT id FROM genres WHERE name = 'アニメ')),
('宇宙よりも遠い場所', '南極を目指す少女たちの冒険と友情を描いたアニメーション作品。', (SELECT id FROM genres WHERE name = 'アニメ')),
('君の名は。', '運命に導かれた二人の物語を描いたアニメ映画。', (SELECT id FROM genres WHERE name = '映画')),
('君の膵臓をたべたい', '余命宣告を受けた少女とクラスメイトの交流を描いた青春ドラマ映画。', (SELECT id FROM genres WHERE name = '映画')),
('半沢直樹', '銀行内での権力闘争と主人公の復讐劇を描いたサスペンスドラマ。', (SELECT id FROM genres WHERE name = 'ドラマ')),
('プリズン・ブレイク', '冤罪で死刑判決を受けた兄を救うため、弟が刑務所からの脱獄を計画するドラマシリーズ。', (SELECT id FROM genres WHERE name = 'ドラマ')),
('世界の今を伝えるニュース', '国内外の最新ニュースを提供する番組。', (SELECT id FROM genres WHERE name = 'ニュース'))
);

```

```sql
INSERT INTO seasons (program_id, season_number) VALUES (
(SELECT id FROM programs WHERE title = '鬼滅の刃'), 1),
((SELECT id FROM programs WHERE title = '鬼滅の刃'), 2),
((SELECT id FROM programs WHERE title = 'リコリス・リコイル'), 1),
((SELECT id FROM programs WHERE title = '宇宙よりも遠い場所'), 1),
((SELECT id FROM programs WHERE title = '半沢直樹'), 1),
((SELECT id FROM programs WHERE title = 'プリズン・ブレイク'), 1),
((SELECT id FROM programs WHERE title = 'プリズン・ブレイク'), 2),

```

```sql
INSERT INTO episodes (season_id, episode_number, title, description, duration, views) VALUES (
(SELECT id FROM seasons WHERE program_id = (SELECT id FROM programs WHERE title = '鬼滅の刃') AND season_number = 1), 1, '鬼滅の刃 第1話', '第1話の説明', 30, 100),
((SELECT id FROM seasons WHERE program_id = (SELECT id FROM programs WHERE title = '鬼滅の刃') AND season_number = 1), 2, '鬼滅の刃 第2話', '第2話の説明', 30, 150),
((SELECT id FROM seasons WHERE program_id = (SELECT id FROM programs WHERE title = '鬼滅の刃') AND season_number = 2), 1, '鬼滅の刃 第1話(シーズン2)', '第1話の説明', 30, 150),
((SELECT id FROM seasons WHERE program_id = (SELECT id FROM programs WHERE title = '鬼滅の刃') AND season_number = 2), 2, '鬼滅の刃 第2話(シーズン2)', '第2話の説明', 30, 150),
((SELECT id FROM seasons WHERE program_id = (SELECT id FROM programs WHERE title = 'リコリス・リコイル') AND season_number = 1), 1, 'リコリス・リコイル 第1話', '第1話の説明', 30, 250),
((SELECT id FROM seasons WHERE program_id = (SELECT id FROM programs WHERE title = 'リコリス・リコイル') AND season_number = 1), 2, 'リコリス・リコイル 第2話', '第2話の説明', 30, 200),
((SELECT id FROM seasons WHERE program_id = (SELECT id FROM programs WHERE title = '半沢直樹') AND season_number = 1), 1, '半沢直樹 第1話', '第1話の説明', 60, 300),
((SELECT id FROM seasons WHERE program_id = (SELECT id FROM programs WHERE title = '半沢直樹') AND season_number = 1), 2, '半沢直樹 第2話', '第2話の説明', 60, 300),
((SELECT id FROM seasons WHERE program_id = (SELECT id FROM programs WHERE title = 'プリズン・ブレイク') AND season_number = 1), 1, 'プリズン・ブレイク 第1話', '第1話の説明', 60, 100),
((SELECT id FROM seasons WHERE program_id = (SELECT id FROM programs WHERE title = 'プリズン・ブレイク') AND season_number = 1), 2, 'プリズン・ブレイク 第2話', '第2話の説明', 60, 150),
((SELECT id FROM seasons WHERE program_id = (SELECT id FROM programs WHERE title = 'プリズン・ブレイク') AND season_number = 2), 1, 'プリズン・ブレイク 第1話(シーズン2)', '第1話の説明', 60, 150),
((SELECT id FROM seasons WHERE program_id = (SELECT id FROM programs WHERE title = 'プリズン・ブレイク') AND season_number = 2), 1, 'プリズン・ブレイク 第2話(シーズン2)', '第2話の説明', 60, 200
);
```

鬼滅の刃 第1話
鬼滅の刃 第2話
鬼滅の刃 第1話(シーズン2)
鬼滅の刃 第2話(シーズン2)
リコリス・リコイル 第1話
リコリス・リコイル 第2話
半沢直樹 第1話
半沢直樹 第2話
プリズン・ブレイク 第1話
プリズン・ブレイク 第2話
プリズン・ブレイク 第1話(シーズン2)
プリズン・ブレイク 第2話(シーズン2)

```sql
INSERT INTO program_schedules (channel_id, episode_id, start_time, end_time) VALUES (
(SELECT id FROM channels WHERE name = 'アニメ1'), (SELECT id FROM episodes WHERE title = '鬼滅の刃 第1話'), '2023-11-13 20:00:00', '2023-11-13 20:30:00'),
((SELECT id FROM channels WHERE name = 'アニメ1'), (SELECT id FROM episodes WHERE title = '鬼滅の刃 第2話'), '2023-11-14 20:00:00', '2023-11-14 20:30:00'),
((SELECT id FROM channels WHERE name = 'アニメ1'), (SELECT id FROM episodes WHERE title = '鬼滅の刃 第1話(シーズン2)'), '2023-11-15 20:00:00', '2023-11-15 20:30:00'),
((SELECT id FROM channels WHERE name = 'アニメ1'), (SELECT id FROM episodes WHERE title = '鬼滅の刃 第2話(シーズン2)'), '2023-11-16 20:00:00', '2023-11-16 20:30:00'),
((SELECT id FROM channels WHERE name = 'アニメ2'), (SELECT id FROM episodes WHERE title = 'リコリス・リコイル 第1話'), '2023-11-13 21:00:00', '2023-11-13 21:30:00'),
((SELECT id FROM channels WHERE name = 'アニメ2'), (SELECT id FROM episodes WHERE title = 'リコリス・リコイル 第2話'), '2023-11-14 21:00:00', '2023-11-14 21:30:00'),
((SELECT id FROM channels WHERE name = 'ドラマ1'), (SELECT id FROM episodes WHERE title = '半沢直樹 第1話'), '2023-11-15 22:00:00', '2023-11-15 23:00:00'),
((SELECT id FROM channels WHERE name = 'ドラマ1'), (SELECT id FROM episodes WHERE title = '半沢直樹 第2話'), '2023-11-16 22:00:00', '2023-11-16 23:00:00'),
((SELECT id FROM channels WHERE name = 'ドラマ2'), (SELECT id FROM episodes WHERE title = 'プリズン・ブレイク 第1話'), '2023-11-17 20:00:00', '2023-11-17 21:00:00'),
((SELECT id FROM channels WHERE name = 'ドラマ2'), (SELECT id FROM episodes WHERE title = 'プリズン・ブレイク 第2話'), '2023-11-18 20:00:00', '2023-11-18 21:00:00'),
((SELECT id FROM channels WHERE name = 'ドラマ2'), (SELECT id FROM episodes WHERE title = 'プリズン・ブレイク 第1話(シーズン2)'), '2023-11-19 20:00:00', '2023-11-19 21:00:00'),
((SELECT id FROM channels WHERE name = 'ドラマ2'), (SELECT id FROM episodes WHERE title = 'プリズン・ブレイク 第2話(シーズン2)'), '2023-11-19 21:00:00', '2023-11-19 22:00:00'
);
```

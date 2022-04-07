## Problem

Write a query that will return the list of posts, where each post can have:

  * only `database` keyword
  * only `sql` keyword
  * exactly `database` and `sql` keywords


## Input Data

`posts`

| id | title   | content | category | created_at          | updated_at          |
| -- | ------- | ------- | -------- | ------------------- | ------------------- |
| 1  | Title A | ---     | backend  | 2021-03-17 00:00:00 | 2021-03-17 00:00:00 |
| 2  | Title B | ---     | devops   | 2021-04-12 00:00:00 | 2021-04-12 00:00:00 |
| 3  | Title C | ---     | devops   | 2021-04-26 00:00:00 | 2021-04-26 00:00:00 |
| 4  | Title D | ---     | dba      | 2021-05-13 00:00:00 | 2021-05-13 00:00:00 |
| 5  | Title E | ---     | backend  | 2021-05-14 00:00:00 | 2021-05-14 00:00:00 |
| 6  | Title F | ---     | dba      | 2021-06-13 00:00:00 | 2021-06-13 00:00:00 |
| 7  | Title G | ---     | backend  | 2021-07-22 00:00:00 | 2021-07-22 00:00:00 |


`keywords`

| id  | content  | created_at                 | updated_at                 |
| --- | -------- | -------------------------- | -------------------------- |
| 1   | database | 2022-03-14 14:25:47.778108 | 2022-03-14 14:25:47.778108 |
| 2   | sql      | 2022-03-14 14:25:47.778108 | 2022-03-14 14:25:47.778108 |
| 3   | rails    | 2022-03-14 14:25:47.778108 | 2022-03-14 14:25:47.778108 |
| 4   | ruby     | 2022-03-14 14:25:47.778108 | 2022-03-14 14:25:47.778108 |
| 5   | aws      | 2022-03-14 14:25:47.778108 | 2022-03-14 14:25:47.778108 |
| 6   | linux    | 2022-03-14 14:25:47.778108 | 2022-03-14 14:25:47.778108 |
| 7   | .net     | 2022-03-14 14:25:47.778108 | 2022-03-14 14:25:47.778108 |
| 8   | python   | 2022-03-14 14:25:47.778108 | 2022-03-14 14:25:47.778108 |
| 9   | csv      | 2022-03-14 14:25:47.778108 | 2022-03-14 14:25:47.778108 |


`post_keywords`

| id  | post_id | keyword_id | created_at                 | updated_at                 |
| --- | ------- | ---------- | -------------------------- | -------------------------- |
| 1   | 1       | 1          | 2022-03-14 14:25:47.803197 | 2022-03-14 14:25:47.803197 |
| 2   | 1       | 2          | 2022-03-14 14:25:47.803197 | 2022-03-14 14:25:47.803197 |
| 3   | 1       | 3          | 2022-03-14 14:25:47.803197 | 2022-03-14 14:25:47.803197 |
| 4   | 2       | 5          | 2022-03-14 14:25:47.803197 | 2022-03-14 14:25:47.803197 |
| 5   | 2       | 8          | 2022-03-14 14:25:47.803197 | 2022-03-14 14:25:47.803197 |
| 6   | 3       | 5          | 2022-03-14 14:25:47.803197 | 2022-03-14 14:25:47.803197 |
| 7   | 3       | 6          | 2022-03-14 14:25:47.803197 | 2022-03-14 14:25:47.803197 |
| 8   | 4       | 1          | 2022-03-14 14:25:47.803197 | 2022-03-14 14:25:47.803197 |
| 9   | 4       | 2          | 2022-03-14 14:25:47.803197 | 2022-03-14 14:25:47.803197 |
| 10  | 4       | 5          | 2022-03-14 14:25:47.803197 | 2022-03-14 14:25:47.803197 |
| 11  | 4       | 6          | 2022-03-14 14:25:47.803197 | 2022-03-14 14:25:47.803197 |
| 12  | 5       | 1          | 2022-03-14 14:25:47.803197 | 2022-03-14 14:25:47.803197 |
| 13  | 5       | 2          | 2022-03-14 14:25:47.803197 | 2022-03-14 14:25:47.803197 |
| 14  | 5       | 3          | 2022-03-14 14:25:47.803197 | 2022-03-14 14:25:47.803197 |
| 15  | 5       | 4          | 2022-03-14 14:25:47.803197 | 2022-03-14 14:25:47.803197 |
| 16  | 5       | 9          | 2022-03-14 14:25:47.803197 | 2022-03-14 14:25:47.803197 |
| 17  | 6       | 1          | 2022-03-14 14:25:47.803197 | 2022-03-14 14:25:47.803197 |
| 18  | 6       | 2          | 2022-03-14 14:25:47.803197 | 2022-03-14 14:25:47.803197 |
| 19  | 7       | 1          | 2022-03-14 14:25:47.803197 | 2022-03-14 14:25:47.803197 |
| 20  | 7       | 5          | 2022-03-14 14:25:47.803197 | 2022-03-14 14:25:47.803197 |
| 21  | 7       | 6          | 2022-03-14 14:25:47.803197 | 2022-03-14 14:25:47.803197 |
| 22  | 7       | 7          | 2022-03-14 14:25:47.803197 | 2022-03-14 14:25:47.803197 |
| 23  | 8       | 1          | 2022-03-14 14:25:47.803197 | 2022-03-14 14:25:47.803197 |


## Expected Result

| id | title   | content | category | created_at          | updated_at          |
| -- | ------- | ------- | -------- | ------------------- | ------------------- |
| 6  | Title F | ---     | dba      | 2021-06-13 00:00:00 | 2021-06-13 00:00:00 |


## Setup Script

```sql
DROP TABLE IF EXISTS posts CASCADE;
DROP SEQUENCE IF EXISTS posts_id_seq;

CREATE SEQUENCE posts_id_seq;

CREATE TABLE posts (
  id BIGINT NOT NULL DEFAULT NEXTVAL('posts_id_seq'::regclass),
  title VARCHAR,
  content VARCHAR,
  category VARCHAR,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  PRIMARY KEY (id)
);

INSERT INTO posts (title, content, category, created_at, updated_at) VALUES
('Title A', '---', 'backend', '2021-03-17', '2021-03-17'),
('Title B', '---', 'devops',  '2021-04-12', '2021-04-12'),
('Title C', '---', 'devops',  '2021-04-26', '2021-04-26'),
('Title D', '---', 'dba',     '2021-05-13', '2021-05-13'),
('Title E', '---', 'backend', '2021-05-14', '2021-05-14'),
('Title F', '---', 'dba',     '2021-06-13', '2021-06-13'),
('Title G', '---', 'backend', '2021-07-22', '2021-07-22'),
('Title H', '---', 'design',  '2021-07-22', '2021-07-22');

DROP TABLE IF EXISTS keywords CASCADE;
DROP SEQUENCE IF EXISTS keywords_id_seq;

CREATE SEQUENCE keywords_id_seq;

CREATE TABLE keywords (
  id BIGINT NOT NULL DEFAULT NEXTVAL('keywords_id_seq'::regclass),
  content VARCHAR,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  PRIMARY KEY (id)
);

INSERT INTO keywords (content) VALUES
('database'),
('sql'),
('rails'),
('ruby'),
('aws'),
('linux'),
('.net'),
('python'),
('csv');

DROP TABLE IF EXISTS post_keywords CASCADE;
DROP SEQUENCE IF EXISTS post_keywords_id_seq;

CREATE SEQUENCE post_keywords_id_seq;

CREATE TABLE post_keywords (
  id BIGINT NOT NULL DEFAULT NEXTVAL('post_keywords_id_seq'::regclass),
  post_id BIGINT NOT NULL,
  keyword_id BIGINT NOT NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  PRIMARY KEY (id),
  FOREIGN KEY (post_id) REFERENCES posts (id),
  FOREIGN KEY (keyword_id) REFERENCES keywords (id)
);

INSERT INTO post_keywords (post_id, keyword_id) VALUES
(1, 1),
(1, 2),
(1, 3),
(2, 5),
(2, 8),
(3, 5),
(3, 6),
(4, 1),
(4, 2),
(4, 5),
(4, 6),
(5, 1),
(5, 2),
(5, 3),
(5, 4),
(5, 9),
(6, 1),
(6, 2),
(7, 1),
(7, 5),
(7, 6),
(7, 7),
(8, 1);
```


## Solution

```sql
SELECT
  posts.*

FROM
  posts

WHERE
  posts.id IN (
    SELECT
      post_id

    FROM
      post_keywords
      JOIN keywords ON keywords.id = post_keywords.keyword_id

    GROUP BY
      post_id

    HAVING
      COUNT(keywords.id) = SUM(CASE WHEN keywords.content IN ('sql', 'database') THEN 1 ELSE 0 END)
  )
```


## Keywords

* [subqueries](https://www.postgresql.org/docs/current/queries-table-expressions.html#QUERIES-SUBQUERIES)
* [`SWITCH CASE`](https://www.postgresql.org/docs/current/functions-conditional.html#FUNCTIONS-CASE)
* [aggregate functions](https://www.postgresql.org/docs/current/functions-aggregate.html)

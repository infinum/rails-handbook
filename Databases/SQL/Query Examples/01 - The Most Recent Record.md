## Problem

Write a query that will return the most recent post for each post category.


## Input Data


| id | title   | content | category | created_at          | updated_at                 |
| -- | ------- | ------- | -------- | ------------------- | -------------------------- |
| 1  | Title A | ---     | backend  | 2021-05-12 00:00:00 | 2021-05-12 15:32:27.423371 |
| 2  | Title B | ---     | devops   | 2021-04-12 00:00:00 | 2021-05-12 15:32:27.423371 |
| 3  | Title C | ---     | devops   | 2021-04-26 00:00:00 | 2021-05-12 15:32:27.423371 |
| 4  | Title D | ---     | dba      | 2021-05-13 00:00:00 | 2021-05-12 15:32:27.423371 |
| 5  | Title E | ---     | backend  | 2021-05-13 00:00:00 | 2021-05-12 15:32:27.423371 |


## Expected Result

| id | title   | content | category | created_at          | updated_at                 |
| -- | ------- | ------- | -------- | ------------------- | -------------------------- |
| 5  | Title E | ---     | backend  | 2021-05-13 00:00:00 | 2021-05-12 15:32:27.423371 |
| 4  | Title D | ---     | dba      | 2021-05-13 00:00:00 | 2021-05-12 15:32:27.423371 |
| 3  | Title C | ---     | devops   | 2021-04-26 00:00:00 | 2021-05-12 15:32:27.423371 |


## Setup Script

```sql
DROP TABLE IF EXISTS posts;
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

INSERT INTO posts (title, content, category, created_at) VALUES
('Title A', '---', 'backend', '2021-05-12'),
('Title B', '---', 'devops',  '2021-04-12'),
('Title C', '---', 'devops',  '2021-04-26'),
('Title D', '---', 'dba',     '2021-05-13'),
('Title E', '---', 'backend', '2021-05-13');
```


## Solution

```sql
SELECT DISTINCT ON (category) *
FROM posts
ORDER BY category, created_at DESC;


-- alternative solution, using WINDOW function

SELECT *
FROM (
  SELECT posts.*,
          ROW_NUMBER() OVER (
            PARTITION BY category ORDER BY created_at DESC
          ) AS rn
  FROM posts
) posts
WHERE rn = 1;
```


## Keywords

* [`DISTINCT ON` (PostgreSQL specific)](https://www.postgresql.org/docs/current/sql-select.html#SQL-DISTINCT)
* basic usage of [`WINDOW` functions](https://www.postgresql.org/docs/current/functions-window.html)
  * [`ROW_NUMBER()`](https://www.postgresql.org/docs/current/functions-window.html#FUNCTIONS-WINDOW-TABLE)

## Problem

Write a query that will return five posts published/created after a specific post (let's use a post with ID = 5 as a reference).
If there are less than five posts published after that post, populate the list with the posts from the same category (`backend`) first (ordered by `created_at` ascending). Then use all other posts ordered by the `created_at` ascending.


## Input Data

| id | title   | content | category | created_at          | updated_at          |
| -- | ------- | ------- | -------- | ------------------- | ------------------- |
| 1  | Title A | ---     | backend  | 2021-03-17 00:00:00 | 2021-03-17 00:00:00 |
| 2  | Title B | ---     | devops   | 2021-04-12 00:00:00 | 2021-04-12 00:00:00 |
| 3  | Title C | ---     | devops   | 2021-04-26 00:00:00 | 2021-04-26 00:00:00 |
| 4  | Title D | ---     | dba      | 2021-05-13 00:00:00 | 2021-05-13 00:00:00 |
| 5  | Title E | ---     | backend  | 2021-05-14 00:00:00 | 2021-05-14 00:00:00 |
| 6  | Title F | ---     | dba      | 2021-06-13 00:00:00 | 2021-06-13 00:00:00 |
| 7  | Title G | ---     | backend  | 2021-07-22 00:00:00 | 2021-07-22 00:00:00 |


## Expected Result

| id | title   | content | category | created_at          | updated_at          |
|----|---------|---------|----------|---------------------|---------------------|
| 6  | Title F | ---     | backend  | 2021-06-13 00:00:00 | 2021-06-13 00:00:00 |
| 7  | Title G | ---     | backend  | 2021-07-22 00:00:00 | 2021-07-22 00:00:00 |
| 1  | Title A | ---     | backend  | 2021-03-17 00:00:00 | 2021-03-17 00:00:00 |
| 2  | Title B | ---     | devops   | 2021-04-12 00:00:00 | 2021-04-12 00:00:00 |
| 3  | Title C | ---     | devops   | 2021-04-26 00:00:00 | 2021-04-26 00:00:00 |


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

INSERT INTO posts (title, content, category, created_at, updated_at) VALUES
('Title A', '---', 'backend', '2021-03-17', '2021-03-17'),
('Title B', '---', 'devops',  '2021-04-12', '2021-04-12'),
('Title C', '---', 'devops',  '2021-04-26', '2021-04-26'),
('Title D', '---', 'dba',     '2021-05-13', '2021-05-13'),
('Title E', '---', 'backend', '2021-05-14', '2021-05-14'),
('Title F', '---', 'backend', '2021-06-13', '2021-06-13'),
('Title G', '---', 'backend', '2021-07-22', '2021-07-22');
```


## Solution

```sql
SELECT
  *

FROM
  posts

WHERE
  id != 5

ORDER BY
  CASE WHEN id > 5 THEN 0 ELSE 1 END,
  CASE WHEN category = 'backend' THEN 0 ELSE 1 END,
  created_at

LIMIT
  5
```


## Keywords

* circular [order](https://www.postgresql.org/docs/current/queries-order.html)
* custom order with [`CASE`](https://www.postgresql.org/docs/current/functions-conditional.html#FUNCTIONS-CASE)

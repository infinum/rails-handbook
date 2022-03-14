## Problem

For the posts table, we want to separate category to its own table and then reference category by the foreign key.
Write a query that will update current posts table data to the suggested structure.


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


`categories`

| id | name
| -- | -------
| 1  | backend
| 2  | devops
| 3  | dba


## Expected Result

| id | title   | content | category | created_at          | updated_at          |
| -- | ------- | ------- | -------- | ------------------- | ------------------- |
| 1  | Title A | ---     | 1        | 2021-03-17 00:00:00 | 2021-03-17 00:00:00 |
| 2  | Title B | ---     | 2        | 2021-04-12 00:00:00 | 2021-04-12 00:00:00 |
| 3  | Title C | ---     | 2        | 2021-04-26 00:00:00 | 2021-04-26 00:00:00 |
| 4  | Title D | ---     | 3        | 2021-05-13 00:00:00 | 2021-05-13 00:00:00 |
| 5  | Title E | ---     | 1        | 2021-05-14 00:00:00 | 2021-05-14 00:00:00 |
| 6  | Title F | ---     | 3        | 2021-06-13 00:00:00 | 2021-06-13 00:00:00 |
| 7  | Title G | ---     | 1        | 2021-07-22 00:00:00 | 2021-07-22 00:00:00 |


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

DROP TABLE IF EXISTS categories;
DROP SEQUENCE IF EXISTS categories_id_seq;

CREATE SEQUENCE categories_id_seq;

CREATE TABLE categories (
  id BIGINT NOT NULL DEFAULT NEXTVAL('categories_id_seq'::regclass),
  name VARCHAR,
  PRIMARY KEY (id)
);

INSERT INTO categories (name) VALUES
('backend'),
('devops'),
('dba');
```


## Solution

```sql
UPDATE
  posts

SET
  category = categories.id

FROM
  categories

WHERE
  posts.category = categories.name
```


## Keywords

* `UPDATE` with `FROM`
  * using `FROM` as `JOIN` within `UPDATE` statement

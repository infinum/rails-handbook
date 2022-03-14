## Problem

Write a query that will return 3 most recent images for each post.


## Input Data

| id | post_id | image_url    | created_at          | updated_at                 |
|----|---------|--------------|---------------------|----------------------------|
| 1  | 1       | images.com/1 | 2021-03-12 00:00:00 | 2022-02-11 10:26:06.634393 |
| 2  | 2       | images.com/2 | 2021-03-15 00:00:00 | 2022-02-11 10:26:06.634393 |
| 3  | 2       | images.com/3 | 2021-03-20 00:00:00 | 2022-02-11 10:26:06.634393 |
| 4  | 3       | images.com/4 | 2021-04-13 00:00:00 | 2022-02-11 10:26:06.634393 |
| 5  | 1       | images.com/5 | 2021-04-15 00:00:00 | 2022-02-11 10:26:06.634393 |
| 6  | 3       | images.com/6 | 2021-04-17 00:00:00 | 2022-02-11 10:26:06.634393 |
| 7  | 3       | images.com/7 | 2021-04-18 00:00:00 | 2022-02-11 10:26:06.634393 |
| 8  | 1       | images.com/8 | 2021-05-01 00:00:00 | 2022-02-11 10:26:06.634393 |
| 9  | 1       | images.com/9 | 2021-05-02 00:00:00 | 2022-02-11 10:26:06.634393 |
| 10 | 3       | images.com/0 | 2021-05-13 00:00:00 | 2022-02-11 10:26:06.634393 |


## Expected Result

| id | post_id | image_url    | created_at          | updated_at                 |
|----|---------|--------------|---------------------|----------------------------|
| 9  | 1       | images.com/9 | 2021-05-02 00:00:00 | 2022-02-11 10:26:06.634393 |
| 8  | 1       | images.com/8 | 2021-05-01 00:00:00 | 2022-02-11 10:26:06.634393 |
| 5  | 1       | images.com/5 | 2021-04-15 00:00:00 | 2022-02-11 10:26:06.634393 |
| 3  | 2       | images.com/3 | 2021-03-20 00:00:00 | 2022-02-11 10:26:06.634393 |
| 2  | 2       | images.com/2 | 2021-03-15 00:00:00 | 2022-02-11 10:26:06.634393 |
| 10 | 3       | images.com/0 | 2021-05-13 00:00:00 | 2022-02-11 10:26:06.634393 |
| 7  | 3       | images.com/7 | 2021-04-18 00:00:00 | 2022-02-11 10:26:06.634393 |
| 6  | 3       | images.com/6 | 2021-04-17 00:00:00 | 2022-02-11 10:26:06.634393 |


## Setup Script

```sql
DROP TABLE IF EXISTS post_images;
DROP SEQUENCE IF EXISTS post_images_id_seq;

CREATE SEQUENCE post_images_id_seq;

CREATE TABLE post_images (
  id BIGINT NOT NULL DEFAULT NEXTVAL('post_images_id_seq'::regclass),
  post_id BIGINT,
  image_url VARCHAR,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  PRIMARY KEY (id)
);

INSERT INTO post_images (post_id, image_url, created_at) VALUES
(1, 'images.com/1', '2021-03-12'),
(2, 'images.com/2', '2021-03-15'),
(2, 'images.com/3', '2021-03-20'),
(3, 'images.com/4', '2021-04-13'),
(1, 'images.com/5', '2021-04-15'),
(3, 'images.com/6', '2021-04-17'),
(3, 'images.com/7', '2021-04-18'),
(1, 'images.com/8', '2021-05-01'),
(1, 'images.com/9', '2021-05-02'),
(3, 'images.com/0', '2021-05-13');
```


## Solution

```sql
SELECT
  id, post_id, image_url, created_at, updated_at

FROM (
  SELECT
    post_images.*,
    ROW_NUMBER() OVER (
      PARTITION BY post_id ORDER BY created_at DESC
    ) AS rn

  FROM
    post_images
) post_images

WHERE rn <= 3;
```


## Keywords

* basic usage of `WINDOW` functions
  * partitions and frames
  * `ROW_NUMBER()`
  * subqueries

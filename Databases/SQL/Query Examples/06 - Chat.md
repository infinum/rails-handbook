## Problem

You have to find a chat between two users with IDs 1 and 4. Users can be provided in no particular order.


## Input Data

| id | user1_id | user2_id | created_at          |
|----|----------|----------|---------------------|
| 1  | 1        | 2        | 2022-02-11 08:30:00 |
| 2  | 3        | 2        | 2022-02-12 09:30:00 |
| 3  | 4        | 1        | 2022-02-13 10:30:00 |


## Expected Result

| id | user1_id | user2_id | created_at          |
|----|----------|----------|---------------------|
| 3  | 4        | 1        | 2022-02-13 10:30:00 |


## Setup

```sql
DROP TABLE IF EXISTS chats;
DROP SEQUENCE IF EXISTS chats_id_seq;

CREATE SEQUENCE chats_id_seq;

CREATE TABLE chats (
  id BIGINT NOT NULL DEFAULT NEXTVAL('chats_id_seq'::regclass),
  user1_id VARCHAR,
  user2_id VARCHAR,
  created_at TIMESTAMP,
  PRIMARY KEY (id)
);

INSERT INTO chats (user1_id, user2_id, created_at) VALUES
(1, 2, '2022-02-11 08:30:00'),
(3, 2, '2022-02-12 09:30:00'),
(4, 1, '2022-02-13 10:30:00');
```


## Solution (1)

```sql
SELECT
  *

FROM
  chats

WHERE
  (1, 4) IN ((user1_id, user2_id), (user2_id, user1_id))
```


## Solution (2)

```sql
SELECT
  *

FROM
  chats

WHERE
  (user1_id, user2_id) IN ((1, 4), (4, 1))
```


## Keywords

* tuples

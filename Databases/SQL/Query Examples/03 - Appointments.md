## Problem

Write a query that will generate appointment slots of 30 minutes (consultations with content editor), between 08:00 and 16:00 for current date and for a **content editor with ID = 1**. For each slot, you need to put status *BUSY* if there is scheduled appointment for the slot, otherwise *FREE*.


## Input Data

| id | content_editor_id | start_time          | end_time            |
|----|-------------------|---------------------|---------------------|
| 1  | 1                 | 2022-02-11 08:30:00 | 2022-02-11 09:00:00 |
| 2  | 1                 | 2022-02-11 09:30:00 | 2022-02-11 10:00:00 |
| 3  | 1                 | 2022-02-11 09:30:00 | 2022-02-11 10:00:00 |
| 4  | 1                 | 2022-02-11 13:30:00 | 2022-02-11 14:00:00 |


## Expected Result

| start_time          | end_time            | status |
|---------------------|---------------------|--------|
| 2022-02-11 08:00:00 | 2022-02-11 08:30:00 | FREE   |
| 2022-02-11 08:30:00 | 2022-02-11 09:00:00 | BUSY   |
| 2022-02-11 09:00:00 | 2022-02-11 09:30:00 | FREE   |
| 2022-02-11 09:30:00 | 2022-02-11 10:00:00 | BUSY   |
| 2022-02-11 10:00:00 | 2022-02-11 10:30:00 | FREE   |
| 2022-02-11 10:30:00 | 2022-02-11 11:00:00 | FREE   |
| 2022-02-11 11:00:00 | 2022-02-11 11:30:00 | FREE   |
| 2022-02-11 11:30:00 | 2022-02-11 12:00:00 | FREE   |
| 2022-02-11 12:00:00 | 2022-02-11 12:30:00 | FREE   |
| 2022-02-11 12:30:00 | 2022-02-11 13:00:00 | FREE   |
| 2022-02-11 13:00:00 | 2022-02-11 13:30:00 | FREE   |
| 2022-02-11 13:30:00 | 2022-02-11 14:00:00 | BUSY   |
| 2022-02-11 14:00:00 | 2022-02-11 14:30:00 | FREE   |
| 2022-02-11 14:30:00 | 2022-02-11 15:00:00 | FREE   |
| 2022-02-11 15:00:00 | 2022-02-11 15:30:00 | FREE   |
| 2022-02-11 15:30:00 | 2022-02-11 16:00:00 | FREE   |


## Setup Script

```sql
DROP TABLE IF EXISTS appointments;
DROP SEQUENCE IF EXISTS appointments_id_seq;

CREATE SEQUENCE appointments_id_seq;

CREATE TABLE appointments (
  id BIGINT NOT NULL DEFAULT NEXTVAL('appointments_id_seq'::regclass),
  content_editor_id BIGINT NOT NULL,
  start_time TIMESTAMP,
  end_time TIMESTAMP,
  PRIMARY KEY (id)
);

INSERT INTO appointments (content_editor_id, start_time, end_time) VALUES
(1, CURRENT_DATE + INTERVAL '08:30', CURRENT_DATE + INTERVAL '09:00'),
(1, CURRENT_DATE + INTERVAL '09:30', CURRENT_DATE + INTERVAL '10:00'),
(2, CURRENT_DATE + INTERVAL '09:30', CURRENT_DATE + INTERVAL '10:00'),
(1, CURRENT_DATE + INTERVAL '13:30', CURRENT_DATE + INTERVAL '14:00');
```


## Solution

```sql
WITH generated_appointments AS (
  SELECT
    start_time,
    (start_time + INTERVAL '30' MINUTE) AS end_time

  FROM GENERATE_SERIES(
    CURRENT_DATE + INTERVAL '08:00',
    CURRENT_DATE + INTERVAL '15:30',
    INTERVAL '30' MINUTE
  ) AS t(start_time)
)

SELECT
  g.start_time,
  g.end_time,
  CASE WHEN a.id IS NULL
  THEN 'FREE'
  ELSE 'BUSY'
  END AS status

FROM generated_appointments AS g
  LEFT JOIN appointments AS a ON
    (g.start_time, g.end_time) = (a.start_time, a.end_time) AND
    a.content_editor_id = 1;
```


## Keywords

*	`LEFT JOIN`
*	`NULL` checking
*	`SWITCH CASE`
*	`DATETIME` manipulation
*	`INTERVAL`
*	`CTE` (or `WITH` clause)

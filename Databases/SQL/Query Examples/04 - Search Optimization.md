## Problem

You have a mission to optimize querying users by their name and surname by using this query:

```sql
SELECT *
FROM users
WHERE first_name || ' ' || last_name ILIKE '%joe%'
```

How would you speed up this query?


## Input Data

| id | first_name | last_name | role   |
|----|------------|-----------|--------|
| 1  | John       | Doe       | author |
| 2  | Joe        | Smith     | author |
| 3  | Mark       | Cohen     | admin  |


## Setup

```sql
DROP TABLE IF EXISTS users;
DROP SEQUENCE IF EXISTS users_id_seq;

CREATE SEQUENCE users_id_seq;

CREATE TABLE users (
  id BIGINT NOT NULL DEFAULT NEXTVAL('users_id_seq'::regclass),
  first_name VARCHAR,
  last_name VARCHAR,
  role VARCHAR,
  PRIMARY KEY (id)
);

INSERT INTO users (first_name, last_name, role) VALUES
('John', 'Doe', 'author'),
('Joe', 'Smith', 'author'),
('Mark', 'Cohen', 'admin');
```


## Solution

```sql
CREATE EXTENSION pg_trgm;

CREATE INDEX idx_users
  ON users USING GIN ((first_name || ' ' || last_name) gin_trgm_ops);
```

> Go ahead and check the performance benefits (you can use `EXPLAIN` to check execution plan)


## Keywords

*	defining index on expression (concatenation of first name and last name)
*	extension `pg_trgm`
* `GIN` index

## Problem (1)

You have to export users with the surname “Smith” to CSV.


## Problem (2)

You have to import new users from a large CSV file.


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


## Solution (1)

```sql
COPY (SELECT * FROM users WHERE last_name = 'Smith')
TO '/tmp/users_smith.csv'
WITH DELIMITER ';' CSV HEADER;
```


## Solution (2)

```sql
COPY users
FROM '/tmp/new_users.csv'
CSV HEADER;
```

> You can read more about PostgreSQL's `COPY` [command on our blog](https://infinum.com/the-capsized-eight/superfast-csv-imports-using-postgresqls-copy).

## Keywords

* [`COPY`](https://www.postgresql.org/docs/current/sql-copy.html)

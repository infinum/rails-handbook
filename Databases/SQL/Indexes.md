> An index makes the query fast.

## Data Structure

The data structure that is used for indexes is a **B-tree** (balanced tree). That's because the tree depth is equal for all nodes (the distance between root and leaf nodes).

## Why is a query not using an index?

The execution plan of a query can show that the database does not use the index, and uses **sequential scan**.
Possible situations:

  * majority of rows are getting fetched as part of the SQL query
    * although the database reads more data, it might need to execute fewer read operations
  * there is no index available for specified columns

> NOTE: Using an index does not mean a query is executed in the best way!


## Composite Index

It is possible to define an index on two or more columns - that kind of an index is called **composite** (or **concatenated**, **combined**) index.

**Order matters** - the most important thing is how to choose the column order so the index can be used as often as possible! We as developers should have a feeling for the data (business domain) and properly choose columns for an index.

Example:

```sql
CREATE INDEX idx_employee_department_1 ON employees(employee_id, department_id);

-- different approach

CREATE INDEX idx_employee_department_2 ON employees(department_id, employee_id);
```

What index would you create, depends on data usage from the domain perspective. If queries will mostly use the `department_id` column in the `WHERE` part of a query, then it's recommended to use the `idx_employee_department_2` index. Otherwise, if `employee_id` will be used repeatedly, it would probably be more performant to use the `idx_employee_department_1` index.

When both of the columns would be used on a similar frequency, use the column with more scattered data, on the most left position of an index (in our example it would be `emplyee_id`).


## Function-Based Index

When you have an index whose definition contains a function or an expression, you have a **function-based index**.
Instead of copying the column data directly into the index, the database applies a function on the column and stores the result into the index.

Example:

```sql
SELECT *
FROM users
WHERE LOWER(email) = 'user@mail.com'

-- we store lower-cased email as an index value
CREATE INDEX idx_users_email ON users(LOWER(email))
```


## Partial Index

In some situations, where we query only a part of the table, it makes sense to index only a specific partition of the table.
For example, we don't want to index posts that are deleted (`deleted_at IS NOT NULL`). So, we will create a partial index:

```sql
-- index not deleted posts by published_at
CREATE INDEX idx_posts_undeleted ON posts(published_at) WHERE deleted_at IS NULL
```

The main benefit of partial indexes is the smaller size of the index.

The index size is reduced vertically and horizontally:

  * reduced number of rows - only posts that are not deleted (`deleted_at IS NULL`)
  * reduced number of columns - you can skip `deleted_at` column from the index definition

Resources:

  * [PostgreSQL Documentation](https://www.postgresql.org/docs/current/indexes-partial.html)

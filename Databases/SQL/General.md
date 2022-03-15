## Intro

When you work on backend systems, you will always have some kind of data storage where you will persist the application data.
The majority of our applications, use a relational database as a data storage solution.

To be able to communicate with your database, you need to use SQL.
We usually leave that communication to our ORM (`ActiveRecord`). But sometimes, ORM is not powerful enough for a specific problem and you must write raw SQL.

To be able to **solve complex problems** and to get **better understanding** of how the ORM works, here are some query examples and advice to help you with that!


## Convention

All query examples will be compatible with **PostgreSQL standard**.

SQL keywords are formatted in all-capitals to make them stand out from the text, as in **SELECT**.

SQL tables and columns, are spelled in lowercase, and words are separated by underscores, as in `users`, `first_name`.

It would make your life easier if you align and format a query you currently working on. It would be easier to debug and identify parts of the query.

SQL is correctly pronounced "ess-cue-ell", not "see-quell".

In the context of database-related usage, the word index refers to an ordered collection of information. The preferred plural of this word is indexes.


## SQL Playground Database

We setup a PostgreSQL database instance that you can use to test you queries.
The credentials are stored in our **1Password**.


## DB GUI Clients

There are plenty of GUI tools for interacting with PostgreSQL. Here is the list of the most popular GUI clients - feel free to choose the one you like:

  * [Postico](https://eggerapps.at/postico/) (only for Mac - we have a full version license)
  * [TablePlus](https://tableplus.com/)
  * [Azure DataStudio](https://docs.microsoft.com/en-us/sql/azure-data-studio)
  * [ArcType](https://arctype.com/)
  * [pgAdmin](https://www.pgadmin.org/)
  * [JetBrains DataGrip](https://www.jetbrains.com/datagrip/)
  * [SQLPro for Postgres](http://macpostgresclient.com/)


Additional Resources:

  * [Best PostgreSQL GUIs in 2021](https://retool.com/blog/best-postgresql-guis-in-2020/)
  * [Awesome Postgres / GUIs](https://dhamaniasad.github.io/awesome-postgres/#gui)


## Tools & Resources

  * [How to Interpret PostgreSQL Explain Analyze Output](https://www.cybertec-postgresql.com/en/how-to-interpret-postgresql-explain-analyze-output/)
  * [PostgreSQL execution plan visualizer #1](https://explain.dalibo.com/)
  * [PostgreSQL execution plan visualizer #2](http://tatiyants.com/pev/#/plans/new)
  * [PostgreSQL execution plan visualizer #3](https://explain.depesz.com/)
  * [PostgreSQL Cheat Sheet](https://postgrescheatsheet.com)
  * [Most Useful PostgreSQL Commands with Examples](https://technobytz.com/most-useful-postgresql-commands.html)
  * [DB Fiddle](https://www.db-fiddle.com/)
  * [PostgreSQL Ecosystem](https://github.com/EfficiencyGeek/postgresql-ecosystem)
  * [PGTune](https://pgtune.leopard.in.ua/)
  * [PG Wiki - Don't Do This](https://wiki.postgresql.org/wiki/Don't_Do_This)
  * [How the PostgreSQL Query Optimizer Works](https://www.cybertec-postgresql.com/en/how-the-postgresql-query-optimizer-works/)

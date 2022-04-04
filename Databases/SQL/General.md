## Intro

Working on backend systems almost always necessitates the use of a database where application data can be stored.
The majority of our applications use a relational database as a data storage solution.

SQL (structured query language) is the tool for communicating with the database.
We usually communicate with the database through the ORM (`ActiveRecord`). But sometimes, the ORM is not powerful enough for a specific problem and you must write raw SQL.

To be able to **solve complex problems** and to get a **better understanding** of how the ORM works, here are some query examples and advice to help you with that!


## Convention

All query examples will be compatible with the **PostgreSQL standard**.

SQL keywords are formatted in all capitals to make them stand out from the text, as in **SELECT**.

SQL tables and columns are spelled in lowercase, and words are separated by underscores, as in `users`, `first_name`.

It would make your life easier if you align and format a query you are currently working on. It makes it easier to debug and identify parts of the query.

SQL is correctly pronounced "ess-cue-ell", not "see-quell".

In the context of database-related usage, the word index refers to an ordered collection of information. The preferred plural of this word is *indexes*.


## SQL Playground Database

We have set up a PostgreSQL database instance that you can use to test your queries.
The credentials are stored in our **1Password**.

![db-er-diagram](https://user-images.githubusercontent.com/10612221/161582654-63ce291d-c6e7-4b67-b37d-8d858a591cd5.svg)

## DB GUI Clients

There are plenty of GUI tools for interacting with PostgreSQL. Here is the list of the most popular GUI clients - feel free to choose the one you like:

  * [Postico](https://eggerapps.at/postico/) (only for Mac - we have a full version license that you can acquire from your team lead)
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


## Ruby & Rails Database Related Gems

  * [active_record_extended](https://github.com/georgekaraszi/ActiveRecordExtended)
  * [rollup](https://github.com/ankane/rollup)
  * [groupdate](https://github.com/ankane/groupdate)
  * [hightop](https://github.com/ankane/hightop)
  * [active_median](https://github.com/ankane/active_median)
  * [jsonb_accessor](https://github.com/madeintandem/jsonb_accessor)
  * [fx](https://github.com/teoljungberg/fx)
  * [activerecord-postgres_enum](https://github.com/bibendi/activerecord-postgres_enum)
  * [active_record_doctor](https://github.com/gregnavis/active_record_doctor)
  * [hairtrigger](https://github.com/jenseng/hair_trigger)
  * [strong_migrations](https://github.com/ankane/strong_migrations)
  * [database_consistency](https://github.com/djezzzl/database_consistency)

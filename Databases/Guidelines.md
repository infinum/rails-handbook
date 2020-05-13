- foreign key constraints
- mirror model validations to database contraints (not nulls mostly)
- unique compound and partial indexes where needed
- use jsonb fields where appropriate (like for freezing receipt details for user/address)
- try to conform 3NF, but it's okay to deviate for some stuff

- use schema.rb or structure.sql
- when you switch branches, be careful what you commit in schema.rb/structure.sql

- use views
- use materialized views
- write raw sql when needed

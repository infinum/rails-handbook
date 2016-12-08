# Writing migrations

## Schema migrations

The [rails migrations guide](http://edgeguides.rubyonrails.org/active_record_migrations.html) does a very good job of
explaining how schema migrations work in rails.

I would just point out that you can [pass modifiers](http://edgeguides.rubyonrails.org/active_record_migrations.html#passing-modifiers) when you are generating migrations from command line. Not many people know of this.

## Data migrations

Here is where things get a bit complicated. As your project grows and evolves so does your data. At some point you might realize you forgot to add a default to a field. Or that you want to change an enumeration. Or do any kind of manipulation on existing data in your database.

There are two way of dealing with those problems:  

1. Write a rake task
2. Write a data migration

Your first instinct would be to write a simple rake task. You have access to all your models, and it is easily testable and runnable. You can even delete the file afterwards. The problem with rake task is that you have to remember to run it. It does not run automatically. Also if you are writing a big feature and you need that change in the middle of your schema migrations then you have a problem.

Writing a data migration is a bit trickier. You start of by writing a schema migration but instead of doing `add_column` or `rename_column` in your `change` method you do a `Model.update_all()`. The problems start to arise when after a month of developing a new feature, you introduce some validations that break that migration. Or you remove a method you are using in that migration. Or you remove the model entirely. And you notice the problem when you try to deploy your application to the server.

## Make your data migrations foolproof

__Write raw sql queries inside of your data migrations.__

Example:  

``` ruby
def change
  execute(<<-SQL
    UPDATE users
    SET role =
      CASE role
      WHEN '2' THEN 'developer'
      WHEN '3' THEN 'client'
      END
  SQL
  )
end
```

To make your life a bit easier I suggest you write the query with ActiveRecord, and just use `to_sql` method on the query.

**Reversible data migration**

By default, the example above will throw `ActiveRecord::IrreversibleMigration` if you try to rollback the migration. If you need to be able to do a rollback you will need to write your own `down` method:

``` ruby
def up
  execute(<<-SQL
    UPDATE users
    SET role =
      CASE role
      WHEN '2' THEN 'developer'
      WHEN '3' THEN 'client'
      END
  SQL
  )
end

def down
  execute(<<-SQL
    UPDATE users
    SET role =
      CASE role
      WHEN 'developer' THEN '2'
      WHEN 'client' THEN '3'
      END
  SQL
  )
end
```

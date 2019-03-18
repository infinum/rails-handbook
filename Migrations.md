## Schema migrations

The [Rails Migrations Guide](http://edgeguides.rubyonrails.org/active_record_migrations.html) does a very good job at
explaining how schema migrations work in rails.

I would just like to point out that you can [pass modifiers](http://edgeguides.rubyonrails.org/active_record_migrations.html#passing-modifiers) when you are generating migrations from the command line. Not many people know of this.

## Data migrations

Here is where things get a bit complicated. As your project grows and evolves, so does your data. At some point, you might realize you forgot to add a default to a field. Or that you want to change an enumeration. Or manipulate the existing data in your database in any way.

There are two way of dealing with these problems:  

1. Write a rake task
2. Write a data migration

Your first instinct would be to write a simple rake task. You have access to all your models, and it is easily testable and runnable. You can even delete the file afterwards. The problem with the rake task is that you have to remember to run it. It does not run automatically. Also, if you are writing a big feature and need that change in the middle of your schema migrations, then you have a problem.

Writing a data migration is a bit trickier. You start of by writing a schema migration, but instead of doing `add_column` or `rename_column` in your `change` method, you do a `Model.update_all()`. Problems start to arise when, after a month of developing a new feature, you introduce some validations that break that migration. Or you remove a method you are using in that migration. Or you remove the model entirely. And you notice the problem only when you try to deploy your application to the server.

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

To make your life a bit easier, I suggest you write the query with ActiveRecord, and just use the `to_sql` method on the query.

**Reversible data migration**

If you try to roll back the migration, the example above will, by default, throw `ActiveRecord::IrreversibleMigration`. If you need to be able to do a rollback, you will need to write your own `down` method:

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

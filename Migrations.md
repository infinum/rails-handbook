# Writing migrations

## Schema migrations

The [rails migrations guide](http://edgeguides.rubyonrails.org/active_record_migrations.html) does a very good job of
explaining how schema migrations work in rails.

I would just point out that you can [pass modifiers](http://edgeguides.rubyonrails.org/active_record_migrations.html#passing-modifiers) when you are generating migrations from command line. Not many people know of this.

## Data migrations

Here is where things get more complicated. As your project grows and evolves so does your data. At some point you might realize you forgot to add a default to a field. Or that you want to change an enumeration. Or any kind of manipulation of existing data in your database.

There are two way of dealing with those problems:
1. Write a rake task
2. Write a data migration

Your first instinct would be to write a simple rake task. You have access to all your models, and it is easily testable and runnable. You can even delete the file afterwards. The problem with rake task is that you have to remember to run it. It does not run automatically. Also if you are writing a big feature and you need that change in the middle of your schema migrations then you have a problem.

Writing a data migration is a bit trickier. You start of by writing a schema migration but instead of doing `add_column` or `rename_column` in your `change` method you do a `Model.update_all()`. The problems start to arise when after a month of developing a new feature you introduce some validations that break that migration. Or you remote a method you are using in the migration. Or you remote the model entirely. And you figure out the problem when you try to do the migration on deploy.

## Make your data migrations foolproof

1. Add [migration_data](https://github.com/ka8725/migration_data) gem.
2. Generate migrations as usual but instead of `change` use `data` method.
3. Write raw sql queries inside of your data migrations.

Example:
``` ruby
def data
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

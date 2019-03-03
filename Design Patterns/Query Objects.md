Query Objects store complex SQL queries, data aggregation, and filtering methods.

The goal of this pattern is to remove code for querying sets of objects from
models/controllers and provide a simple yet powerful interface for complex
data aggregation.

## In practice

Query objects live in the `app/queries` folder.

Their naming convention is similar to that of controllers.
Each object should bear the plural of the name of the model it queries
suffixed by the word 'Query'. For example, an object that queries articles should be
called ArticlesQuery.

Each object should be passed a relation as an optional argument from which it queries
the data. If no relation has been passed, it queries all
objects by default.

```Ruby
  # queries all articles
  ArticlesQuery.new

  # this is the same as the example above
  ArticlesQuery.new(Article.all)

  # queries only published articles
  ArticlesQuery.new(Article.where(published: true))
```

Query Objects can be used in model scopes and relation conditions.
They should be accessible and useable from any location in your codebase.

```Ruby
class Article < ActiveRecord::Base
  has_many :views, -> { ViewsQuery.new.organic_views  }
  scope :published, -> { ArticlesQuery.new.published  }

  # model implementation ...
end
```

## Implementation

Each Query Object implementation should resemble the following example:

```Ruby
class ArticlesQuery
  attr_reader :relation

  def initialize(relation = Article.all)
    @relation = relation
  end

  def published
    # method implementation ...
  end

  private

  def custom_sql
    # custom SQL query ...
  end
end
```

## Examples

We have an Article model with the following fields:

* author_id
* title
* content
* view_count
* published_at
* created_at
* updated_at

It also implements a belongs_to relation 'author' that returns an instance of
the User model with the following fields:

* first_name
* last_name

## Bad solution

The usual bad solution is to keep everything either in a controller or in a
model, thus making them 'fat'.

Controller solution:


```Ruby
class ArticlesController < ApplicationController
  def index
    @articles = Article
                  .joins('LEFT OUTER JOIN users ON users.id = articles.author_id')
                  .where(published: true)
                  .where('view_count > ?', params[:min_view_count])
                  .where('users.first_name LIKE ?', "#{params[:author_name]}%")
end
```

Model solution:

```Ruby
class Article < ActiveRecord::Base
  scope :published, -> { where(published: true)  }

  def with_view_count_greater_than(min_view_count)
    where('view_count > ?', min_view_count)
  end

  def with_author_with_first_name_like(first_name)
    joins('LEFT OUTER JOIN users ON users.id = articles.author_id')
      .where('users.first_name LIKE ?', "#{first_name}%")
  end
end
```

It doesn't matter if we put this code in a controller or in a model, it simply
doesn't belong there. A model's job isn't to handle querying logic, query
logic in a controller isn't reusable and it makes the controller 'fat'.

## Good solution

Create a Query Object in the `app/queries` directory.
It's implementation should resemble the following:

```Ruby
class ArticlesQuery
  attr_reader :relation

  def initialize(relation = Article.all)
    @relation = relation
  end

  def published
    relation.where(published: true)
  end

  def minimal_view_count(view_count)
    return relation unless view_count.present?
    relation.where('view_count > ?', view_count)
  end

  def author_first_name_like(first_name)
    return relation unless first_name.present?
    with_authors
      .where('users.first_name LIKE ?', "#{first_name}%")
  end

  private

  def with_authors
    relation.joins('LEFT OUTER JOIN users ON users.id = articles.author_id')
  end
end
```

Then you would use it like this:

```Ruby
class ArticlesController < ApplicationController
  def index
    @articles = articles_scope
  end

  private

  def articles_scope
    published_articles = ArticlesQuery.new.published
    articles_with_view_count = ArticlesQuery.new(published_articles)
                                .minimal_view_count(params[:view_count])
    ArticlesQuery.new(articles_with_view_count)
                  .author_first_name_like(params[:author_name])
  end
end
```

## Further reading

* [Delegating to Query Objects through ActiveRecord scopes](http://craftingruby.com/posts/2015/06/29/query-objects-through-scopes.html)
* [7 Patterns to Refactor Fat ActiveRecord Models](http://blog.codeclimate.com/blog/2012/10/17/7-ways-to-decompose-fat-activerecord-models/)
* [ActiveRecord Eager Loading with Query Objects and Decorators](https://robots.thoughtbot.com/active-record-eager-loading-with-query-objects-and-decorators)

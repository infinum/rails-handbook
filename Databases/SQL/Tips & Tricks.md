## N-queries

You should avoid making (generating) multiple DB queries from the application if you can fetch the needed data in a single query. Always check what queries are executed in your (Rails) log.

Also, when you work on a complex SQL calculation, avoid pulling intermediate results to your application, which will be used for the next step in your query. It can be a very resource-intensive operation.


## UNION (ALL)

When you use the `UNION` operator for combining result sets of two or more query statements, double-check if you should use `UNION` or `UNION ALL`.

The `UNION` operator is *less performant* because it **removes all duplicate rows** from the combined data set.
To retain the duplicate rows, you use the `UNION ALL` instead.


## VIEWs

A database `VIEW` is a pretty useful SQL mechanism. You can use it for:
  * hiding the complexity of queries
  * reducing complexity by naming a part of the complex query
  * exposing a data subset to the outer world

A plain `VIEW` contains only a definition - that means the query will be executed each time you fetch data from the view.

The results can be "cached" by using a `MATERIALIZED VIEW`. Basically, a `MATERIALIZED VIEW` acts as a read-only plain table. To update data in materialized views, use the `REFRESH` statement.

`REFRESH` completely replaces the content of a materialized view. During the refresh process, the view is locked. To refresh a materialized view without locking, use `REFRESH MATERIALIZED VIEW CONCURRENTLY`.


### Plain or materialized views?

So, the rule of thumb when you have to choose between plain or materialized VIEW is the frequency of data updates.
`VIEWs` are generally used when data is to be accessed infrequently and data in the table get updated frequently.
On the other hand, `MATERIALIZED VIEWs` are used when data is to be accessed frequently and data in the table does not get updated frequently.


> You can add indexes to materialized views!


### Handling views from Rails

When using Rails and `ActiveRecord`, views behave like a plain models:

```ruby
class Report < ApplicationRecord
  belongs_to :property

  def readonly?
    true
  end
end

Report.where(property: property)
      .group(:date)
      .sum(:revenue)
```

Also, for managing (materialized) views in your Rails application, it's recommended to use [Scenic gem](https://github.com/scenic-views/scenic).
The gem adds methods for easier management (create, delete, update, and refresh) of database views.


## PostGIS

When you want to work with geographical data, most likely you'll end up using [PostGIS](https://postgis.net/). PostGIS is an extension for the PostgreSQL database.

In the official [documentation](https://postgis.net/docs/), you can find more information about data types, functions, and operators.
Here's the list of some useful functions and geometry constructors, which can be your starting point when you'll start playing with PostGIS:

  * [`ST_Point`](https://postgis.net/docs/ST_Point.html)
  * [`ST_ClosestPoint`](https://postgis.net/docs/ST_ClosestPoint.html)
  * [`ST_Contains`](https://postgis.net/docs/ST_Contains.html)
  * [`ST_Covers`](https://postgis.net/docs/ST_Covers.html)
  * [`ST_Collect`](https://postgis.net/docs/ST_Collect.html)
  * [`ST_Centroid`](https://postgis.net/docs/ST_Centroid.html)
  * [`ST_DWithin`](https://postgis.net/docs/ST_DWithin.html)
  * [`ST_ClusterDBSCAN`](https://postgis.net/docs/ST_ClusterDBSCAN.html)

But, sometimes, PostGIS might be overkill for your use case - read more here [why you (probably) don't need PostGIS](https://blog.rebased.pl/2020/04/07/why-you-probably-dont-need-postgis.html).

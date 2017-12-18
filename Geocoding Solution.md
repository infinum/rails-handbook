# Geokit-rails

Despite geocoder having a bigger community and giving the impression that it's better in all aspects, the majority of our developers had less trouble using geokit-rails.

# Gem Comparisons

## Repo state

|  | [Geocoder](https://github.com/alexreisner/geocoder)   | [Geokit-Rails](https://github.com/geokit/geokit-rails)  |
| ------------------ |:--------:| ----------:|
| Stars                       |    4802  |          1286|
| Issues                     |    38    |          58     |
| Open pull requests|    19    |          1     |
| Closed pull requests|    512    |          39     |
| First commit|    15.4.2009.    |          18.12.2008.     |
| Last commit|    17.11.2017.    |          9.8.2017.     |
| Popularity|   9.4 and growing    |          7.2 stable    |
| Activity |   7.5 stable    |          3.9    |

## Features

|  | [Geocoder](https://github.com/alexreisner/geocoder)   | [Geokit-Rails](https://github.com/geokit/geokit-rails)  |
| ------------------ |:--------:| ----------:|
| Postgis solution     |    Yes     |             No|
| Geocoding by street *    |    Yes     |             Yes|
| Geocoding by IP address*     |    Yes     |             Yes|
| Reverse geocoding*     |    Yes     |             Yes|
| Distance queries     |    Yes     |             Yes|

### Legend:
- geocoding by street: get latitude and longitude based on the address (string)
- geocoding by IP address: get latitude and longitude based on the ip address
- reverse geocoding: finding street address based on given coordinates (lat/lng)
- distance queries: how far is an object from another in meters (the objects need to have lat/lng attributes)

## Database support
|  | [Geocoder](https://github.com/alexreisner/geocoder)   | [Geokit-Rails](https://github.com/geokit/geokit-rails)  |
| ------------------ |:--------:| ----------:|
| MySQL                       |   Yes   |    Yes      |
| Postgresql                       |   Yes   |     Yes     |
| SQLite                       |    Yes  |     No     |
| MongoDB                       |    Yes  |     No     |


## Differences in code

Given the next example of a project, the goal is to fetch a particular type of users (limit 50) that were close to a job (`user` and `job` are names of AR models):

_geocoder version_
```ruby
MAX_USER_DISTANCE = 200
MAX_SET_OF_USERS = 50

def call
  User.near(job_location, MAX_USER_DISTANCE)
      .with_user_type_assignee
      .limit(MAX_SET_OF_USERS)
end

def job_location
  [job.latitude, job.longitude].compact
end
```
Running this the following sql is produced:
```
SELECT  users.*, 3958.755864232 * 2 * ASIN(SQRT(POWER(SIN((20.0 - users.last_latitude) * PI() / 180 / 2), 2) + COS(20.0 * PI() / 180) * COS(users.last_latitude * PI() / 180) * POWER(SIN((-15.0 - users.last_longitude) * PI() / 180 / 2), 2))) AS distance, MOD(CAST((ATAN2( ((users.last_longitude - -15.0) / 57.2957795), ((users.last_latitude - 20.0) / 57.2957795)) * 57.2957795) + 360 AS decimal), 360) AS bearing FROM \"users\" WHERE (users.last_latitude BETWEEN 17.10536433778304 AND 22.89463566221696 AND users.last_longitude BETWEEN -18.08040693114738 AND -11.91959306885262 AND (3958.755864232 * 2 * ASIN(SQRT(POWER(SIN((20.0 - users.last_latitude) * PI() / 180 / 2), 2) + COS(20.0 * PI() / 180) * COS(users.last_latitude * PI() / 180) * POWER(SIN((-15.0 - users.last_longitude) * PI() / 180 / 2), 2)))) BETWEEN 0.0 AND 200) AND \"users\".\"user_type\" = 'assignee' ORDER BY distance ASC LIMIT 50
```

However, the following error occurred:
```
ActiveRecord::StatementInvalid: PG::UndefinedColumn: ERROR:  column "distance" does not exist
LINE 1: ....0 AND 200) AND "users"."user_type" = $1 ORDER BY distance A...
                                                             ^
: SELECT COUNT(count_column) FROM (SELECT  1 AS count_column FROM "users" WHERE (users.last_latitude BETWEEN 17.10536433778304 AND 22.89463566221696 AND users.last_longitude BETWEEN -18.08040693114738 AND -11.91959306885262 AND (3958.755864232 * 2 * ASIN(SQRT(POWER(SIN((20.0 - users.last_latitude) * PI() / 180 / 2), 2) + COS(20.0 * PI() / 180) * COS(users.last_latitude * PI() / 180) * POWER(SIN((-15.0 - users.last_longitude) * PI() / 180 / 2), 2)))) BETWEEN 0.0 AND 200) AND "users"."user_type" = $1 ORDER BY distance ASC LIMIT $2) subquery_for_count
from /Users/cilim/.rbenv/versions/2.4.1/lib/ruby/gems/2.4.0/gems/activerecord-5.1.3/lib/active_record/connection_adapters/postgresql_adapter.rb:614:in `async_exec'
```

Substituting geocoder with geokit-rails was easy:

_geokit-rails version_
```ruby
MAX_USER_DISTANCE = 200
MAX_SET_OF_USERS = 50

def call
  User.within(MAX_USER_DISTANCE, origin: job)
      .with_user_type_assignee
      .limit(MAX_SET_OF_USERS)
end
```

And all worked normally.

SQL query:
```
"SELECT  \"users\".* FROM \"users\" WHERE (users.last_latitude IS NOT NULL AND users.last_longitude IS NOT NULL) AND (users.last_latitude>17.10860294292818 AND users.last_latitude<22.89139705707182 AND users.last_longitude>-18.07661470455701 AND users.last_longitude<-11.923385295442989) AND ((\n" +
"          (ACOS(least(1,COS(0.3490658503988659)*COS(-0.2617993877991494)*COS(RADIANS(users.last_latitude))*COS(RADIANS(users.last_longitude))+\n" +
"          COS(0.3490658503988659)*SIN(-0.2617993877991494)*COS(RADIANS(users.last_latitude))*SIN(RADIANS(users.last_longitude))+\n" +
"          SIN(0.3490658503988659)*SIN(RADIANS(users.last_latitude))))*3963.1899999999996)\n" +
"          <= 200)) AND \"users\".\"user_type\" = 'assignee' LIMIT 50"
```

without errors 👍

### References:
- https://forum.shakacode.com/t/picking-a-geocoding-gem/377/5
- https://ruby.libhunt.com/project/geocoder/vs/geokit

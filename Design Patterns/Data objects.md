Data objects are used for encapsulating (complex) data-structures into objects rather than into primitives such as arrays or hashes.

Data objects give us:  

  * Better insight into what our data structure holds
  * Flexibility to manipulate data in a uniform place
  * Convenience of adding additional behavior (i.e., methods)

## Example

We have a list of cities in a CSV format, and we have to store them in our database.

Each city comes with the following information: country code, city code, and a name:

```csv
US, NYC, new york city
US, LA, los angeles
US, SF, san francisco
```

The CSV format does not exactly meet our needs:  

  * we need all of the words in a city's name to be capitalized
  * we need different codes—we want to store them concatenated—`USNYC` instead of `US` & `NYC`
  * the CSV is malformed, and some of the cities came without a name—we have to ignore those

## Bad solution

We want to retrieve the cities from the CSV in Ruby. Natively, they would come as an array of arrays:

```ruby
[['US', 'NYC', 'New York City'],['US', 'LA', 'Los Angeles'],['US', 'SF', 'San Francisco']]
```

Let's write a simple `CityImporter` class that would handle this:

```ruby
class CityImporter
  attr_accessor :csv_file_path

  def initialize(csv_file_path)
    @csv_rows = CSV.read(csv_file_path)
  end

  def import
    csv_rows.each do |csv_row|
      if csv_row[2].present?
        City.create(name: csv_row[2].titleize,
                    code: "#{csv_row[0].upcase}#{csv_row[1].upcase}")
      end
    end
  end
end
```

1. Can you tell what the #import method does **without looking** at our CSV structure? The #import method is unreadable. And bear in mind that **this is an oversimplified version of a real-world example.**
2. What would happen if we reorganized the order of our columns? We would have to calculate the column position multiple times and possibly in multiple places.

## Good solution

We could ditch our unreadable arrays and create an object that plays nicely with our CSV data:

```ruby
class CSVCity
  attr_reader :country_code, :location_code, :name

  def initialize(csv_row)
    @country_code = csv_row[0].upcase
    @city_code    = csv_row[1].upcase
    @name         = csv_row[2].titleize
  end

  def code
    "#{country_code}#{city_code}"
  end

  def name_present?
    name.present?
  end
end
```

With the CSV city holding all our data, our CityImporter class is much more readable:

```ruby
class LocationImporter
  attr_accessor :codes_csv_path, :csv_rows, :csv_cities

  def initialize(csv_file_path)
    @csv_rows = CSV.read(csv_file_path)
    @csv_cities = csv_rows.map { |csv_row| CSVCity.new(csv_row) }
  end

  def import
    csv_cities.select(&:name_present?).each do |csv_city|
      City.create(name: csv_city.name, code: csv_city.code)
    end
  end
end
```

## Questions

**Couldn't this go into a model?**

A possible solution **(but a really bad one)** would be to create a "factory" class method called `from_csv` in our `City` model:

```ruby
class City < ActiveRecord::Base
  def self.from_csv(csv_file)
    CSV.read(csv_file_path).each do |csv_row|
      if csv_row[2].present?
        City.create(name: csv_row[2].titleize,
                    code: "#{csv_row[0].upcase}#{csv_row[1].upcase}")
      end
    end
  end
end
```

This is not a good solution because our `City` model should not be responsibile for importing CSV files. It should handle only database persistence.

Although this is just one method, our model would become bloated with a more complex example and with more similar tasks.

**Where to put the `CSVCity` and `CityImporter` class?**

There are multiple places where you can put this kind of code. A good place may be:  

  * Create a folder `app/csv_importers`, add the `CityImporter` there and `CSVCity` inside the same class.
  * If you're using this in something like a rake task, you can write that content in the rake task file itself.

## Further reading

  * [Be nice to each other, use Data Objects](http://brewhouse.io/2015/07/31/be-nice-to-others-and-your-future-self-use-data-objects.html)

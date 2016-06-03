# Null Objects

The intent of a Null Object is to encapsulate the absence of an object by providing a substitutable alternative that offers suitable default behavior.

With Null objects we can:
  - Encapsulate the absence of an object by providing a substitutable alternative
  - Remove conditionals from our code (try, nil?, &&..)
  - Keep all Null objects structured in one place

## Example

### Assignment

We have an address model where we store all the address information like city, zipcode, country, etc...

```ruby
class Address
  validates :address, presence: true
  validates :city, presence: true
  validates :zipcode, presence: true
  validates :country, presence: true
end
```

Also we have a company model which can have multiple addresses but it can also have no address data.

We want to print out first (primary) address.

### Bad Solution

We get the first addressthis way:

```ruby
@address = @company.addresses.first
```

Now `@address` can be `nil` so we have to check if address is present before we print it and give a fallback scenario:

```ruby
- if @address.present?
  %p= @address.address
  %p= @address.city
  %p= @address.zipcode
  %p= @address.country
- else
  %p= '< no address entered >'
  %p= '< no city entered >'
  %p= '< no zipcode entered >'
  %p= '< no country entered >'
```

### Good Solution

We create a NullAddress class with it's default behaviours:

```ruby
class NullAddress
  def address
    '< no address entered >'
  end
  
  def city
    '< no city entered >'
  end
  
  def zipcode
    '< no zipcode entered >'
  end
  
  def country
    '< no country entered >'
  end
end
```

Then we instantiate it if needed:

```ruby
@address = @company.addresses.first || NullAddress.new
```

And we can remove the conditional now:
```ruby
%p= @address.address
%p= @address.city
%p= @address.zipcode
%p= @address.country
```

### Questions

### Where to put my Null objects classes

Create a folder `app/nulls` and put all your Null object classes there

### Further reading

- [Handling Associations on Null Objects](https://robots.thoughtbot.com/handling-associations-on-null-objects)
- [Rails Refactoring Example: Introduce Null Object](https://robots.thoughtbot.com/rails-refactoring-example-introduce-null-object)

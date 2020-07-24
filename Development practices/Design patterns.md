
## Classic design patterns

Design patterns are typical solutions to common problems in software design.
Each pattern is like a customizable blueprint used to solve a particular design problem in your code.
They define a language that efficiently improves team communication.

They can also be used to improve code readability and cleanness, as well as to simplify the design.

[Refactoring.guru](https://refactoring.guru) is a great online place to learn more about them.

## Rails patterns

### Factory
Class that has one creation method with a large conditional.

#### Problem
You have a method which instantiates a different type of object depending on a particular parameter.

#### Solution
Read more [here](https://www.sihui.io/design-pattern-factory/).

### Form object
Move form-specific logic away from your ActiveRecord models into a separate class.

#### Problem
* Your controllers are huge and you want to make them more readable by extracting some of their logic, but you don't want to clutter your models
* Your opinion is that models shouldn't have validations
* You have a virtual attribute that works only in a couple of places, but it doesn't make sense to add `attr_accessor` to the model

#### Solution
Read more [here](https://thoughtbot.com/blog/activemodel-form-objects).
_Note_: Form objects work great with [active_type](https://github.com/makandra/active_type).

### Value object
A small simple object, like a date range or location, whose equality isn’t based on identity.

#### Problem
The problem can be spotted by finding:
* attributes that don't make sense on their own
* logic that is tightly coupled with primitives (attributes with behaviour - methods)

#### Solution
Read more [here](https://revs.runtime-revolution.com/value-objects-in-ruby-on-rails-9df64bc8db34).
[dry-rb](https://dry-rb.org/gems/dry-initializer/3.0/) is a Ruby gem that can be used for value objects.

### Null object
Instead of returning null, or some odd value, return a Special Case that has the same interface as what the caller expects.

#### Problem
You have a lot of conditionals scattered around many classes or view files that check the same thing. It's usually a low-level error handling part that covers calls to `nil`s.

#### Solution
Read more [here](https://thoughtbot.com/blog/handling-associations-on-null-objects).

### Query object
Represent a database query or a set of database queries related to the same table.

#### Problem
* You want to simplify a class that fetches some data from your database using a complex query
* You want to simplify your model scope to use another class, instead of keeping the query in the caller
* You have a complex SQL query that is used in multiple callers

#### Solution
Read more [here](https://medium.flatstack.com/query-object-in-ruby-on-rails-56ea434365f0).

### Service object
Concentrate the core logic of a request operation into a separate object.

#### Problem
You have a complex set of operations that need to be executed in a sequence synchronously or asynchronously.

#### Solution
Read more [here](http://brewhouse.io/blog/2014/04/30/gourmet-service-objects.html).
_Note_: Service objects can easily become a code smell if not handled properly. If you find yourself with a service object that has too many methods and/or becomes generally unreadable and hard to understand, you might have to find a [bounded context](https://blog.carbonfive.com/bring-clarity-to-your-monolith-with-bounded-contexts/) or use one of the design patterns mentioned in this chapter.

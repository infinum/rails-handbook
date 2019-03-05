The [decorator pattern](https://en.wikipedia.org/wiki/Decorator_pattern) is
used to wrap an object and extend its functionality without modifying the
object itself. It's similar to the presenter and adapter patterns, which also
wrap an object (or multiple objects), but it provides a different functionality
when compared to those two patterns.

In Rails, the decorator pattern is generally used to format
view-specific data, as well as to handle simple view logic.

## Examples

We have a User model, which has a ```#first_name```, ```#last_name```
, ```#birthday```, ```#email```, and ```#email_private?``` fields. We need to create
a user profile page which will display the user's full name (first name
and last name), formatted birthday, and the email address if it isn't private.

## Bad solutions

We can do a couple of things here:

**1. Keep the logic in the view**

The simplest solution would be to simply do those things in the view:  

``` slim
Name:
= @user.first_name + ' ' @user.last_name

Email:
- if @user.email_private?
  'Private'
- else
  = @user.email

Birthday:
= @user.birthday.strftime('%d. %m. %Y.')
```

This is a bad solution because views shouldn't be concerned with how to display
data, or what to display under what conditions. Code like this tends to
multiply, so views quickly become an unreadable mess filled with logic all
around.


**2. Use helper methods**

Another possibility is putting the methods in helpers:

``` ruby
module UserHelper
  def full_name(user)
    user.first_name + ' ' + user.last_name
  end

  def email(user)
    return 'Private' if user.email_private?
    user.email
  end

  def formatted_birtday(user)
    user.birthday.strftime('%d. %m. %Y.')
  end
end
```

View:

```
Name:
= full_name(@user)

Email:
= email(@user)

Birthday:
= formatted_birthday(@user)
```

This solution has two main drawbacks:

  * The methods aren't tied to a specific object, which they should be since
    they're methods clearly tied to the User model. Also, you need to pass the
    user as an argument to the methods, instead of simply calling the methods
    on the relevant object.
  * Methods defined in helpers are available to all views, which can cause name
    collisions if you, for example, have another model which has an email field, but
    uses different logic to see if it needs to display the email or not.

**3. Add the methods to the model**

Considering the drawbacks of using helper methods, you might decide to define
the methods in the User model itself:

``` ruby
class User < ActiveRecord::Base

  ...

  def protected_email
    return 'Private' if email_private?
    self[:email]
  end

  def full_name
    first_name + ' ' + last_name
  end

  def formatted_birthday
    birthday.strftime('%d. %m. %Y.')
  end
end
```

View:

```
Name:
= @user.full_name

Email:
= @user.protected_email

Birthday:
= @user.formatted_birthday
```

While this fixes all issues we had with defining the methods in helpers,
it brings back the problems we had when we defined those things in the view—these kinds of methods will appear quickly and often, which will lead to fat
models and cause those models to become unmaintainable. Also, the User model should not be concerned with how
and when to display stuff in the view.

## Good solutions

**1. Make a decorator using SimpleDelegator**

[SimpleDelegator](http://ruby-doc.org/stdlib-2.2.3/libdoc/delegate/rdoc/SimpleDelegator.html)
is a Ruby class that provides a means to easily delegate all method calls to an object passed
to the constructor. A simple implementation of a decorator using SimpleDelegator looks
something like this:

``` ruby
class UserDecorator < SimpleDelegator
  def full_name
    first_name + " " + last_name
  end

  def protected_email
    return "Private" if email_private?
    email
  end

  def formatted_birthday
    birthday.strftime("%d %b %Y")
  end
end
```

Controller:

``` ruby
class UserController < ApplicationController
  def show
    user = User.find(params[:id])
    @user = UserDecorator.new(user)
  end
end
```

View:

``` ruby
Name:
= @user.full_name

Email:
= @user.protected_email

Birthday
= @user.formatted_birthday
```

**2. Use Draper**

[Draper](https://github.com/drapergem/draper/) is a gem that simplifies the creation of
decorators and adds some additional sugar on top of the SimpleDelegator decorators.

One of the benefits of using Draper is that it provides the view context inside of the
decorator, so you can easily use view-specific methods in your decorator. This isn't
something too desirable, so make sure to use it only for simple conditional renders.

``` ruby
class UserDecorator < Draper::Decorator
  # Using decorates_associaton always returns a decorated object or a collection
  # when calling the association on the already decorated object, e.g. user.comments
  decorates_association :comments

  # You can delegate either specific methods to the underlying object, or use delegate_all
  # to delegate all methods sent to the decorator to the underlying object
  delegate :first_name, :last_name, :birthday, :email_private?, :email

  def full_name
    first_name + ' ' + last_name
  end

  def formatted_birthday
    birthday.strftime('%d. %m. %Y')
  end

  def protected_email
    return 'Private' if email_private?

    # You can also use view helpers in Draper decorators
    h.mail_to email
  end
end
```

Controller:

``` ruby
class UserController < ApplicationController
  def show
    user = User.find(params[:id])
    @user = UserProfileDecorator.new(user)
  end
end
```

View:

``` ruby
Name:
= @user.full_name

Email:
= @user.protected_email

Birthday:
= @user.formatted_birthday

# This method will be delegated to the User instance:
= @user.first_name
```

Be sure to read the documentation, since Draper offers a lot more than what's
been shown here.

## Further reading

* [7 Patterns to Refactor Fat ActiveRecord Models](http://blog.codeclimate.com/blog/2012/10/17/7-ways-to-decompose-fat-activerecord-models/)
* [Refactoring Fat Models with Patterns by Bryan Helmkamp](https://www.youtube.com/watch?v=5yX6ADjyqyE)—A talk going through all the patterns from the 7 Patterns to the Refactor Fat ActiveRecord Models blog post
* [Evaluating Alternative Decorator Implementations](https://robots.thoughtbot.com/evaluating-alternative-decorator-implementations-in) —Some other ways to implement the decorator pattern
* [What I dislike about Draper](http://thepugautomatic.com/2014/03/draper/)—A critique of Draper

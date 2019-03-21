We strive to write code that conforms to the four rules for developers by [Sandi Metz](http://www.sandimetz.com/):

1. Classes can be no longer than one hundred lines of code.
2. Methods can be no longer than five lines of code.
3. Pass no more than four parameters into a method. Hash options are parameters.
4. Controllers can instantiate only one object. Therefore, views should only know about one instance variable.

Please read an [article](https://robots.thoughtbot.com/sandi-metz-rules-for-developers) on these four rules.

You may find it difficult to conform to the fourth rule. In general, whenever you think you might be breaking a rule, follow this:

1. Don't violate a guideline without a good reason.
2. A reason is good when you can convince a teammate.

Since it's often difficult, just remember that it was created because of controller actions like [this one](https://gist.github.com/DamirSvrtan/889133405cf02e4327e0).

Other guidelines

* [Avoid writing class methods](http://blog.codeclimate.com/blog/2012/11/14/why-ruby-class-methods-resist-refactoring/)—they resist refactoring and are better as extracted objects with one public method and multiple privates than as a single class method.
Class methods should be used only for describing a rule that's applied for all instances:

```Ruby
class Order
  def self.policy_class
    OrderPolicy
  end
end
```

* Avoid writing local variables—most local variables are better as extracted methods.

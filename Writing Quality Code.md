We strive to write code that conforms to the 4 rules for developers by [Sandi Metz](http://www.sandimetz.com/):

1. Classes can be no longer than one hundred lines of code.
2. Methods can be no longer than five lines of code.
3. Pass no more than four parameters into a method. Hash options are parameters.
4. Controllers can instantiate only one object. Therefore, views should only know about one instance variable.

Please read an [article](https://robots.thoughtbot.com/sandi-metz-rules-for-developers) on those 4 rules.

The 4th rule can be a handful to conform to. Whenever in doubt of breaking any rule generally, follow this:

1. Don't violate a guideline without a good reason.
2. A reason is good when you can convince a teammate.

Since it's often a handful, just note that the reason it's created is because of controller actions like [this one](https://gist.github.com/DamirSvrtan/889133405cf02e4327e0).

Other guidelines

* [Avoid writing class methods](http://blog.codeclimate.com/blog/2012/11/14/why-ruby-class-methods-resist-refactoring/) - they resist refactoring and are better as extracted objects with one public method and multiple privates then as a single class method.
Class methods should only be used for describing a rule that's applied for all instances:

```Ruby
class Order
  def self.policy_class
    OrderPolicy
  end
end
```

* Avoid writing local variables - most local variables are better off as extracted methods.

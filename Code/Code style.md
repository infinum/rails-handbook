The Infinum Backend Team uses the [community guidelines](https://github.com/bbatsov/ruby-style-guide#naming) for naming and syntax conventions. Please read them.

We use a Ruby style-checking analyzer called [Rubocop](https://github.com/bbatsov/rubocop) to preserve syntax consistency in our applications. It's packaged as a Ruby gem and comes with a CLI to check style consistency.
We run the checks through an [Overcommit](https://github.com/sds/overcommit) hook on almost all our projects (if it's missing on a legacy project, check with your colleagues whether it makes sense to install it).

We also use it as a plugin in our editors to get real time in-editor warnings just as we're writing code. Whether you're using Vim, Atom or Sublime, please check that it's properly installed after you've run our installation scripts.

Rubocop is highly configurable, and we deviate only slightly from the suggested guidelines.
Our configuration can be found [here](https://github.com/infinum/default_rails_template/blob/master/.rubocop.yml).
Each project must have its own copy of rubocop.yml where project specific configuration can be added.


Some exceptions from our config file:

* We tend not to write documentation for every class or module. It doesn't make sense to document a User, Post, or a Comment class. However, sometimes applications tend to have a Domain-Specific Business Logic that just needs to be documented since, without a few comments, it's often not clear what's going on. If that's the case, please write up a few comments explaining the interactions.

* The community guidelines suggest to write arrays with string elements with the [percent notation](https://en.wikibooks.org/wiki/Ruby_Programming/Syntax/Literals#The_.25_Notation):

  ```
  developers = %w(Lucas John Tommy)
  ```

  instead of

  ```
  developers = ['Lucas', 'John', 'Tommy']
  ```

  We find it OK to write string arrays the second way for two reasons:  

  * We never remember to write them with the percent notation in the first place, so we always have to change them afterwards.
  * The percent notation looks odd when you have multiple words in one string:

  ```
  developers = %w(Lucas John Tommy Herman\ Zvonimir)
  ```

# Naming and Syntax Conventions

Infinum Rails Team uses the [community guideline](https://github.com/bbatsov/ruby-style-guide#naming) for naming and syntax conventions. Please read it.

To preserve syntax consistency in our applications we use a Ruby style-checking analyzer called [Rubocop](https://github.com/bbatsov/rubocop) - it's packaged as a Ruby gem and it comes with a CLI to check style consistency. Although that's nice, we prefer to use it as a plugin for our favorite editors to get realtime in-editor warnings just as we're writing code. Wether you're using Vim, Atom or Sublime, please check if it's properly installed after you've runned our installation scripts.

Rubocop is highly configurable and we only slightly steer from the suggested guidelines.
Our configuration can be found [here](https://github.com/infinum/guides/blob/master/rails/.rubocop.yml).

* We use a max of 100 characters per line instead of 80 characters.
* We tend not to write documentation for every class or module. It doesn't make sense to document a User, Post or a Comment class. However, sometimes applications tend to have Domain Specific Business Logic that just needs to be documented since it's often not clear on what's going on without a few comments. If that's the case, feel free to write up a few comments explaining the interactions.
* The community guidelines suggests to write arrays with string elements with the [Percentage Notation](https://en.wikibooks.org/wiki/Ruby_Programming/Syntax/Literals#The_.25_Notation):

```
developers = %w(Lucas John Tommy)
```

instead of

```
developers = ['Lucas', 'John', 'Tommy']
```

We find it OK to write string arrays in the second manner for 2 reasons:  

* we never remember to write them with the percentage notation in the first place so we always have to change them afterwards.
* the percentage notation looks odd when you have multiple words in one string:

```
developers = %w(Lucas John Tommy Herman\ Zvonimir)
```

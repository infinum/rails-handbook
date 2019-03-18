* [Check the return value of "save", otherwise use "save!"](http://rails-bestpractices.com/posts/2012/11/02/check-the-return-value-of-save-otherwise-use-save) The same goes for create & update.
* Never use `has_and_belongs_to_many`—rather use `has_many :through`. The first one has unexpected hidden behaviors and, if you find out that you need an extra column in the intermediate table, converting it to the second macro requires a fair amount of work.
* [Never rescue the Exception class](http://rails-bestpractices.com/posts/2012/11/01/don-t-rescue-exception-rescue-standarderror/)
* Use `ENV.fetch` for environment variables instead of `ENV[]`so that unset
  environment variables are detected on deploy.
* Avoid bypassing validations with methods like `save(validate: false)`,
  `update_attribute`, `update_column`, `update_columns`, and `toggle`.
* Don't change a migration after it has been merged into master if the desired
  change can be solved with another migration.
* Don't reference a model class directly from a view.
* Don't use instance variables in partials. Pass local variables to partials
  from view templates.
* If there are default values, set them in migrations.
* Validate the associated `belongs_to` object (`user`), not the database column
  (`user_id`).
* Use private instead of protected when defining controller methods.
* Name date columns with `_on` suffixes.
* Name datetime columns with `_at` suffixes.
* Name time columns (referring to a time of day with no date) with `_time`
  suffixes.
* Name initializers for their gem name.
* Order ActiveRecord associations above ActiveRecord validations.
* Order the controller contents: filters, public methods, private methods.
* Order i18n translations alphabetically by key name.
* Put application-wide partials in the [`app/views/application`] directory.
* Use the default `render 'partial'` syntax over `render partial: 'partial'`.
* Avoid the `:except` option in routes. Use the `:only` option to explicitly state exposed routes.
* Use the user's name in the `From` header and email in the `Reply-To` when [delivering an email on behalf of the app's users]( http://robots.thoughtbot.com/post/3215611590/recipe-delivering-email-on-behalf-of-users)
* Don't use before_actions for setting instance variables. Use them only for changing the application flow, such as redirecting if a user is not authenticated.

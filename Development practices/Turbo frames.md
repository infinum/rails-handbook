Turbo frames are one of the [Turbo](https://turbo.hotwired.dev/) techniques which allow predefined parts of a web page to be updated on request.
They are independent pieces of a web page that can be appended, prepended, replaced, or removed without a complete page refresh and writing a single line of JavaScript.

With Turbo Frames, you can treat a subset of the page as its own component, where links and form submissions replace only that part.

## How it works

Whenever the link inside a frame is clicked or form inside a frame is submitted, the frame content will automatically be updated after receiving a response.

Example:

If we have a part of the page defined like this:

```Html
...
<turbo-frame id='id'>
  <a href='/users/new'>New user</a>
</turbo-frame>
...
```

Once the link to add a new user is clicked, the response might look like this:

```Html
<body>
  <h1> New user </h1>

  <turbo-frame id='id'>
    <form action='/users'>
      <input name='user[name]' type='text' value='Name'>
      <input type="submit">
    </form>
  </turbo-frame>
</body>
```

In this case, the frame from the response will be extracted and it will replace the frame with the same id from where the click originated.
That means that the form for adding a new user will replace the link that was clicked and the rest of the page will remain intact.

## Installation
To be able to use the turbo frames in our application, the [turbo-rails](https://github.com/hotwired/turbo-rails) gem needs to be installed.

The gem is automatically configured for applications made with Rails 7+ unless --skip-hotwire was passed to the generator.
Any new project that uses the [default-rails-template](https://github.com/infinum/default_rails_template) should have Turbo set up and ready to go.

It can also be installed manually:
1. Add the turbo-rails gem to your Gemfile: `gem 'turbo-rails'`
2. Run `./bin/bundle install`
3. Run `./bin/rails turbo:install`
4. Add `import '@hotwired/turbo-rails'` to the Javascript entrypoint file, which is usually located in `app/javascript/application.js` or `app/webpack/packs/application.js` for apps using webpacker if it hasn't been automatically added in the previous step.

If you installed turbo to an existing application, it might be a good idea to disable it globally, and only enable it when needed.

To disable turbo, add the following to one of the js files:
```javascript
import { Turbo } from "@hotwired/turbo-rails";
Turbo.session.drive = false;
```

Then if you want to use turbo somewhere, just add `data-turbo="true"` to the relevant link or form.

## Usage
For this example we have an authors index page:
```ruby
h2 = 'Authors'

- @authors.each do |author|
  .row
    .col
      = author.first_name
    .col
      = author.last_name
    .col
      = link_to 'Edit', edit_author_path(author.id)
```

There is a list of authors and next to each author there is an edit button.
Now let's say that we want to be able to edit the authors inline, without redirecting to a new page.

What would usually require some javascript code, can easily be done using turbo frames with just a few changes on the existing code:
```ruby
h2 = 'Authors'

- @authors.each do |author|
  = turbo_frame_tag "edit_author_#{author.id}"
    .row
      .col
        = author.first_name
      .col
        = author.last_name
      .col
        = link_to 'Edit', edit_author_path(author.id)
```

The only required change was adding this line for each author:
```ruby
= turbo_frame_tag "edit_author_#{author.id}"
```

The `turbo_frame_tag` helper method will wrap the content in a `<turbo-frame>` tag.

For example, for the author with the `id=1` it will wrap the block of code in `<turbo-frame id="edit_author_1">` tag.

The view for the #edit action may look like this:
```ruby
= turbo_frame_tag "edit_author_#{@author.id}"
  = simple_form_for @author do |f|
    .row
      .col
        = f.input :first_name
      .col
        = f.input :last_name
      .col
        = f.button :submit
```

The whole form is wrapped inside the same turbo frame that was used for the index page.

Those are all the changes needed in the views for the inline editing to work.

Once the link to edit the author is clicked, the response provided by the #edit action will have its turbo-frame segment extracted, and the content will replace the frame from where the click originated.

### dom_id helper

The `dom_id` helper can be used to convert the given object into a unique id like this:
```ruby
# If the author is persisted and its id is 1:
dom_id(@author) # => 'author_1'

# If the author is a new record:
dom_id(Author.new) # => 'new_author'

# optional prefix argument
dom_id(Author.new, 'prefix') # => 'prefix_new_author'
```

So we could rewrite the view for the #edit action from the previous example like this:
```ruby
= turbo_frame_tag [:edit, dom_id(@author)]
```

However, since the [`turbo_frame_tag`](https://github.com/hotwired/turbo-rails/blob/ea00f3732e21af9c2156cf74dabe95524b17c361/app/helpers/turbo/frames_helper.rb#L38) helper automatically passes the given object to `dom_id`, this can be further shortened to:
```ruby
= turbo_frame_tag :edit, @author
```

Both of those solutions work correctly and generate the same HTML:
```Html
<turbo-frame id='edit_author_1'>
```

### Controllers

One more thing that needs to be configured is the controller. Since we are wrapping the form in a turbo-frame, we want the validation errors to appear if the object we are creating is invalid.

To do that, the server needs to respond with `422 Unprocessable Entity`, otherwise the whole page might break if there were any validation errors.

Example controller code:
```ruby
def edit
  @author = Author.find(params[:id])
end

def update
  @author = Author.find(params[:id])

  if @author.update(author_params)
    redirect_to authors_path
  else
    render :edit, status: :unprocessable_entity
  end
end
```

NOTE: If you are using slim, make sure to name the views like `'index.html.slim'` instead of just `'index.slim'` because otherwise the form submissions won't work correctly!

## Targeting navigation into or out of a frame
Sometimes we may want the most links to operate within the frame context, but not all of them.

### data-turbo-frame
Let's say that we also want to be able to add a new author on the index page, without redirecting to a completely new page.
This time we don't want to replace anything that's visible on the index page.
We actually want to "append" the form for creating a new author to the page.

If we used the same approach as for the inline editing, we would replace the link for creating a new user with the form. Maybe we don't want to do this because we don't want the form to appear at the same position where the button is, but somewhere else on the page. Luckily it is also done very easily with turbo frames.

Index page:
```ruby
h2 = 'Authors'
= link_to 'Add author', new_author_path, data: { 'turbo-frame': dom_id(Author.new) }

= turbo_frame_tag Author.new
- @authors.each do |author|
  = turbo_frame_tag "edit_author_#{author.id}"
    .row
      .col
        = author.first_name
      .col
        = author.last_name
      .col
        = link_to 'Edit', edit_author_path(author.id)
```

The first change to our index page is the link to the new_author_path. This link is different to the one for the edit_author_path, because `data: { 'turbo-frame': dom_id(Author.new) }` was added to it.

The `data-turbo-frame` attribute can be added on non-frame elements to control from which frame was the link clicked or the form submitted.

In our example, we want the link to act as if it was clicked inside of a turbo-frame with the `new_author` id.

And that turbo-frame is the second change for the index page. It is just an empty frame which will be replaced with the response from the #new action, which may look like this:
```ruby
= turbo_frame_tag Author.new
  = simple_form_for @author do |f|
    .row
      .col
        = f.input :first_name
      .col
        = f.input :last_name
      .col
        = f.button :submit
```

The relevant controller actions may look like this:
```ruby
def new
  @author = Author.new
end

def edit
  @author = Author.find(params[:id])
end

def create
  @author = Author.new(author_params)

  if @author.save
    redirect_to authors_path
  else
    render :new, status: :unprocessable_entity
  end
end
```

So we click on the link to add a new author, it acts as if it was clicked inside of a frame with the `new_author` id because of the `data-turbo-frame` attribute. The response provided by the #new action will have its turbo-frame segment extracted and the form for creating a new author will replace the empty frame with the same id from the index page.

However, there is one issue when we try to submit the form with the valid attributes.
The author gets created in the database, but it is not visible on the index page until we refresh the page.

That's because the form was submitted from the frame with the `new_author` id.
Now the AuthorsController#create action renders the Authors#index page that contains an empty frame with the same `new_author` id.

So basically the only change on the page is that the form for creating a new author will disappear because it is replaced with the empty frame.
The rest of the page, including the authors list will remain intact and that's why the newly created author won't be visible until the page is refreshed.

There is a workaround for that, we can wrap the whole index page inside of another turbo frame:
```ruby
# index.html.slim

= turbo_frame_tag 'main'
  h2 = 'Authors'
  = link_to 'Add author', new_author_path, data: { 'turbo-frame': dom_id(Author.new) }

  = turbo_frame_tag Author.new
  - @authors.each do |author|
    = turbo_frame_tag "edit_author_#{author.id}"
      .row
        .col
          = author.first_name
        .col
          = author.last_name
        .col
          = link_to 'Edit', edit_author_path(author.id)
```

Then the view for the #new action can be updated like this:
```ruby
# new.html.slim

= turbo_frame_tag Author.new
  = simple_form_for @author, data: { turbo_frame: 'main' } do |f|
    .row
      .col
        = f.input :first_name
      .col
        = f.input :last_name
      .col
        = f.button :submit
```

This way, when we submit the form with the valid attributes, it will act as if it has been submitted from the `main` turbo frame because of the `data-turbo-frame` attribute.
The AuthorsController#create action then renders the Authors#index page, and replaces the content from the `main` turbo frame, which almost acts like the whole page has been refreshed.
Now the newly created author should be visible in the authors list.

Another way of achieving this behaviour is by using [Turbo streams](https://turbo.hotwired.dev/handbook/streams).

### target
The documentation for the turbo frames describes the `target` attribute like this:

*By default, navigation within a frame will target just that frame. This is true for both following links and submitting forms. But navigation can drive the entire page instead of the enclosing frame by setting the target to _top. Or it can drive another named frame by setting the target to the ID of that frame.*

Setting the target to `_top` does actually drive the entire page:
```Html
<turbo-frame id='authors' target='_top'>
  <a href='/authors/1'>
    Following link will replace the whole page, not just this frame.
  </a>
</turbo-frame>
```

However, the second part (*Or it can drive another named frame by setting the target to the ID of that frame.*) doesn't seem to work in development without using turbo streams ([discussion](https://discuss.hotwired.dev/t/target-attribute-on-turbo-frame-tags/1959)) and `data-turbo-frame` attribute can be used to achieve the required functionality.

## Useful links:
- [rails-infinum-hotwire-example](https://github.com/infinum/rails-infinum-hotwire-example)
- [turbo handbook](https://turbo.hotwired.dev/handbook/introduction)
- [hotwire discussion](https://discuss.hotwired.dev/)

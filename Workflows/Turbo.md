Turbo is the heart of [Hotwire](https://hotwired.dev/).
As stated in the official documentation, it is "a set of complementary techniques for speeding up page changes and form submissions, dividing complex pages into components and stream partial page updates over WebSocket. All without writing any javascript at all."

To simplify, it enables server side rendering to deliver a modern user experience and allows for partial page updates and real-time interactions.

With Turbo, all the logic lives on the server, and the browser deals just with the final HTML.

It does this through 3 key concepts:
- [Turbo Drive](#turbo-drive)
- [Turbo Frames](#turbo-frames)
- [Turbo Streams](#turbo-streams)

Most of the code examples used in this chapter can be found in the [example app](https://github.com/infinum/rails-jsonapi-example-app).

## Installation and set up
To be able to use Turbo in our application, the [turbo-rails](https://github.com/hotwired/turbo-rails) gem needs to be installed.

The gem is automatically configured for applications made with Rails 7+ unless `--skip-hotwire` was passed to the generator.
Any new project that uses the [default-rails-template](https://github.com/infinum/default_rails_template) should already have Turbo set up and ready to go.

All of the installation steps are listed in the [docs](https://github.com/hotwired/turbo-rails?tab=readme-ov-file#installation).

## Turbo Drive
[Turbo Drive](https://turbo.hotwired.dev/handbook/drive) intercepts regular browser navigation (link clicks and form submissions), performs an AJAX request instead and replaces full page reloads with dynamic updates to the <body> element of the page.

Once Turbo is installed, this behaviour will be applied to the whole application so each link click and form submission will be performed in the background by default.

This can be disabled globally:
```javascript
import { Turbo } from "@hotwired/turbo-rails";
Turbo.session.drive = false;
```

Or it can also be disabled/enabled on a per-element basis by setting the `data-turbo` attribute to true/false, e.g.:
```ruby
= link_to 'New', new_admin_flight_path, data: { turbo: false }
```

Another thing to notice is that prefetching links on hover is enabled by default since Turbo v8 which sends a request on `mouseenter` events before the user clicks the link which results in effectively instant navigation in most cases.

However, this can also be disabled on a global level by adding this meta tag:
```ruby
meta name="turbo-prefetch" content="false"
```

Or on a per-element basis:
```ruby
= link_to 'New', new_admin_flight_path, data: { turbo_prefetch: false }
```

Note: `data-turbo-prefetch` attribute can also be set on a parent element which will result in enabling/disabling of prefetching on all of it's child elements.

Lastly, one thing to look out for when migrating the existing app to Turbo, the `data-confirm` attribute on links has to be updated with `data-turbo-confirm` and `data-turbo-method` attributes, e.g.:
```ruby
# before Turbo
= link_to 'Delete', admin_flight_path(company), method: :delete, data: { confirm: 'Are you sure?' }

# after enabling Turbo:
= link_to 'Delete', admin_flight_path(company), data: { turbo_method: :delete, turbo_confirm: 'Are you sure?' }
```

## Turbo Frames
[Turbo Frames](https://turbo.hotwired.dev/handbook/frames) allow predefined parts of a web page to be updated on request.
They are independent pieces of a web page that can be appended, prepended, replaced, or removed without a complete page refresh and writing a single line of JavaScript.

With Turbo Frames, you can treat a subset of the page as its own component, where links and form submissions replace only that part.

### How it works
Each `<turbo-frame>` acts as a self-contained area that can be updated independently.
When a link or form inside a frame is clicked or submitted, only that frame is updated based on the server's response.

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
To be more precise, the frame's content is replaced, the actual frame element remains intact.

### Rails helpers
There are a couple of helpers that are most often used when working with Turbo Frames in a rails app.

`turbo_frame_tag` helper is the most important one, it is provided by the `turbo-rails` gem and is used to create a `<turbo-frame>` element:
```ruby
= turbo_frame_tag 'id'
```

And the other one is the `dom_id` helper which is used to convert the given object into a unique id like this:
```ruby
# If the location is persisted and it's id is 1:
dom_id(@location) # => 'location_1'

# If the location is a new record:
dom_id(Location.new) # => 'new_location'

# optional prefix argument
dom_id(Location.new, 'prefix') # => 'prefix_new_location'
```

So in our view, we can do something like this:
```ruby
= turbo_frame_tag [:edit, dom_id(@location)]
```

However, since the [`turbo_frame_tag`](https://github.com/hotwired/turbo-rails/blob/ea00f3732e21af9c2156cf74dabe95524b17c361/app/helpers/turbo/frames_helper.rb#L38) helper automatically passes the given object to `dom_id`, this can be further shortened to:
```ruby
= turbo_frame_tag :edit, @location
```

Both of those solutions work correctly and generate the same HTML:
```Html
<turbo-frame id='edit_location_1'>
```

### Inline editing
One common use case for Turbo Frames is inline editing.

With Turbo Frames, that behaviour can be easily implemented without any use of javascript.

Since we usually have some sort of a list/table of records on the index page, each record representation can be wrapped inside of it's own turbo frame, e.g.:
```ruby
- locations.each do |location|
  = turbo_frame_tag :edit, dom_id(location)
    .row
      .col = location.id
      .col = location.name
      .col = link_to 'Edit', edit_admin_location_path(location), class: 'btn btn-info btn-sm'
```

The above code wraps each row inside of a turbo frame based on the record's id (`edit_location_1`, `edit_location_2` etc.).

Since we want to achieve inline editing, the idea is to replace the whole row with the form for updating a record, so the view for the #edit action should include the turbo frame that matches the id of the corresponding turbo frame on the index page:
```ruby
= turbo_frame_tag :edit, dom_id(@location)
  = simple_form_for [:admin, @location] do |f|
    .row
      .col
      .col
        = f.input :name
      .col
        = f.submit 'Save'
```

Once the link to edit the location is clicked, the response provided by the #edit action will have its turbo-frame segment extracted and the content will replace the frame from where the click originated.

NOTE: this won't work if the records are inside of a table as table rows since tables only accept table elements as children.
However, there is a [workaround](https://www.reddit.com/r/rails/comments/15imy2j/til_you_can_actually_use_table_with_turbo_frame/?rdt=45642) for it if needed.

One more thing that needs to be configured is the controller. Since we are wrapping the form in a turbo-frame, we want the validation errors to appear if the object we are creating is invalid.

To do that, the server needs to respond with `422 Unprocessable Entity`, otherwise the whole page might break if there are any validation errors.

Example controller code:
```ruby
def update
  @location = Location.find(params[:id])

  if @location.update(location_params)
    redirect_to locations_path
  else
    render :edit, status: :unprocessable_entity
  end
end
```

NOTE: If you are using slim, make sure to name the views like `'index.html.slim'` instead of just `'index.slim'` because otherwise the [form submissions won't work correctly](https://github.com/hotwired/turbo-rails/issues/168)!

### Targeting navigation into or out of a frame
By default, navigation within a frame will target just that frame.
However, it's possible to define which frame will be targeted on link clicks and form submissions.

This can be done in 2 ways:
1. `target` attribute on frame elements
2. `data-turbo-frame` attribute on non-frame elements

E.g.
```ruby
# index.html.slim
= turbo_frame_tag :main
  = link_to 'New', new_admin_company_path, data: { turbo_frame: :form_modal }
  
  = render 'companies', companies: @companies

  = turbo_frame_tag :form_modal, target: :main
```

The idea here is to have a form for creating a new company inside of a modal.
Once the link is clicked, we want the modal to appear on the index page, that's why we target the empty `form_modal` turbo frame - the empty frame will be replaced with the frame from the response which contains the form modal:
```ruby
# new.html.slim
= turbo_frame_tag :form_modal, target: :main
  = simple_form_for [:admin, @company] do |f|
    .modal
      .modal-content
        .modal-header
          h5.modal-title
            = title
        .modal-body
          = f.input :name
        .modal-footer
          = f.submit 'Save'
```

`target` has been set to `main` because we want to "refresh" all of the content from the index view (everything is wrapped inside of the `main` turbo frame) - that way the newly created company will appear on the page after a successful form submission.

`target` can also be set to `_top`, in which case the navigation will drive the entire page, however we couldn't really use it in the above example because otherwise the validation errors wouldn't be rendered correctly.

### Eager and lazy loading frames
Frames don't have to be populated as soon as the page is loaded.
If a `src` attribute is present on the `turbo-frame` tag, the referenced URL will automatically be loaded as soon as the tag appears on the page.

This can be used e.g. for showing some sort of a placeholder while the data is being loaded:
```ruby
= turbo_frame_tag 'statistics', src: '/admin/flights/statistics'
  .spinner-border
```

The `statistics` frame will initially only contain a boostrap spinner when the page loads and fire a request to `/admin/flights/statistics`.
The response must contain a corresponding frame element as in the original example:
```ruby
= turbo_frame_tag :statistics
  .container-md
    p | Some data
```

The initial spinner placeholder will then be replaced with the actual data.

It is also possible to mark the frame with `loading=lazy`, in that case the changes to the src attribute will defer navigation until the element is visible in the viewport.

## Turbo Streams
Turbo Streams enable real-time updates by sending specially formatted HTML responses that describe actions to perform on the DOM.

There are nine possible stream actions, and all of them are explained in the [docs](https://turbo.hotwired.dev/handbook/streams#stream-messages-and-actions).

Using the [Broadcastable](https://github.com/hotwired/turbo-rails/blob/main/app/models/concerns/turbo/broadcastable.rb) concern mixed into Active Record, it is possible to trigger broadcast updates directly from the model.

To demonstrate this, we want to update the number of bookings made today accross all clients in real time - whenever a new booking is created/destroyed, the counter should change.

One way to achieve this is the following:
1. subcribe clients to a broadcast channel, e.g. `booking_today`
2. add an `after_commit` callback to the Booking model which will trigger the broadcast update whenever a booking is created/destroyed
3. create a partial for the counter (the partial is used to display the count value) - make sure the counter is wrapped inside of an element with a unique ID, e.g. `bookings_today_count`

Subscribing the client to a broadcast channel can be done in the view:
```ruby
# index.html.slim

# subscribe the client to a bookings_today stream
= turbo_stream_from 'bookings_today'

.container-md
  # render the bookings_today_count partial to display the initial value
  = render 'bookings_today_count', count: Booking.today.count
```

Partial view should include an element with the bookings_today_count ID:
```ruby
# _bookings_today_count.html.slim

p#bookings_today_count
  | Number of bookings today: #{count}
```

Track the count changes in the Booking model by using the after_commit callback and triggering the broadcast update:
```ruby
class Booking < ApplicationRecord
  ...
  scope :today, -> { where(created_at: Time.zone.now.all_day) }

  after_commit :broadcast_bookings_today_count, on: [:create, :destroy]

  private

  def broadcast_bookings_today_count
    count = Booking.today.count

    Turbo::StreamsChannel.broadcast_update_to 'bookings_today',
                                              target:  'bookings_today_count',
                                              partial: 'admin/bookings/bookings_today_count',
                                              locals:  { count: count }
  end
end
```

- `bookings_today` - broadcast stream identifier -> all of the clients subscribed to this stream will receive an update
- `target: 'bookings_today_count'` - the ID of the element that should be updated -> when the stream message is received, the element with this ID will be updated with the new content rendered from the partial
- `partial: 'admin/bookings/bookings_today_count'` - which partial will be rendered

Note that Action Cable should be properly set up in order to use the broadcast streams:
- in `application.rb` make sure to require either `rails/all` or `action_cable/engine`
- add a `cable.yml` file, e.g. from Example app:
  ```yml
    development:
    adapter: async

    test:
      adapter: async

    production:
      adapter: redis
      url: <%= ENV.fetch("REDIS_URL") { "redis://localhost:6379/1" } %>
  ```

## Userful resources
- [Turbo handbook](https://turbo.hotwired.dev/handbook/introduction)
- [Turbo reference](https://turbo.hotwired.dev/reference/drive)
- [Hotwire examples](https://hotwiredcases.dev/)
- [Hotrails](https://www.hotrails.dev/)
- [Hotwire discussion](https://discuss.hotwired.dev/)

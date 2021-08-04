> The first concern of the architect is to make sure that the house is usable; it is not to ensure that the house is made of brick. 
> — Uncle Bob

# What it looks like now
Ruby on Rails is an opinionated framework and forces developers into a specific folder structure. 
Most gems also follow the Rails guidelines of how the folder structure should look like.
On smaller projects this can be fine, but on bigger monoliths the files can get out of hand and it becomes too difficult to find the right class.

Here is a typical Rails folder structure:

```
app
├── assets         # sprockets
├── channels       # active_cable
├── components     # view_component
├── controllers    # Rails 
├── decorators     # draper
├── enumerations   # enumerations
├── forms          # reform/active_type
├── helpers        # Rails
├── javascript     # webpacker
├── jobs           # active_job/sidekiq
├── mailers        # Rails
├── models         # Rails
├── policies       # pundit/active_policy
├── serializers    # AMS/jsonapi-serializer
├── services       # PORO
├── uploaders      # shrine
└── views          # Rails
```

This type of folder structure usually encourages a flat hierarchy of classes, which, again, is fine for a smaller projects but 
falls short when the application grows.
It is said that you can discern the application's purpose just by looking at files in your `services` folder. 

Here is an example of a project services folder. Can you tell me what this application does?

<details>
<summary>Services</summary>

<pre>
<code>
app/services
├── add_points_to_user.rb
├── aes_crypt.rb
├── assign_geo_keywords.rb
├── assign_location_attributes.rb
├── braintree
│   ├── customer_and_token_creator.rb
│   └── webhook_handler.rb
├── calculate_bearing.rb
├── developers
│   └── api
│       └── v1
│           └── process_occupancy_params.rb
├── devise
│   └── strategies
│       └── jwt.rb
├── export_service
│   ├── config.rb
│   ├── generator.rb
│   ├── model.rb
│   └── options_formatter.rb
├── exporter
│   ├── creator.rb
│   └── generator.rb
├── exporters
│   └── csv
│       ├── base.rb
│       └── generator.rb
├── feature.rb
├── filters
│   ├── creator.rb
│   └── updater.rb
├── generate_friends_report.rb
├── indigo
│   ├── cancel_reservation.rb
│   └── create_reservation.rb
├── jwt_serializer.rb
├── locations
│   └── actions.rb
├── loyalty_points_awarder.rb
├── mailgun_notify_email_forwarder.rb
├── merge_parking_places.rb
├── migrations
│   └── file_mover.rb
├── notifications
│   ├── amazon_notifications.rb
│   ├── firebase_notifications.rb
│   ├── sender.rb
│   └── shared_actions.rb
├── parking_place_awarder.rb
├── parking_place_exporter.rb
├── parking_places
│   ├── category_updater.rb
│   ├── country_code_mapper.rb
│   ├── creator.rb
│   ├── exporter.rb
│   ├── generators
│   │   ├── icons
│   │   │   ├── combiner.rb
│   │   │   └── type.rb
│   │   └── icons.rb
│   ├── icon_types.rb
│   ├── icons
│   │   └── updater.rb
│   ├── icons.rb
│   ├── occupancies
│   │   └── setter.rb
│   ├── security_rating.rb
│   └── updater.rb
├── payment_exporter.rb
├── payments
│   └── exporter.rb
├── promiles
│   ├── add_driving_distance.rb
│   ├── build_polyline.rb
│   ├── corridor_search.rb
│   ├── create_or_update_parking_places.rb
│   ├── create_or_update_promiles_locations.rb
│   └── get_route.rb
├── provider_occupancies_service.rb
├── providers
│   ├── client
│   │   └── mdm.rb
│   └── parser
│       ├── a63_france.rb
│       └── mdm.rb
├── request_logs
│   ├── archive_partition.rb
│   ├── create_archive.rb
│   └── create_partition.rb
├── reservations
│   ├── booking_id_generator.rb
│   ├── charger.rb
│   ├── creator.rb
│   ├── emails.rb
│   ├── integrations_caller.rb
│   ├── mails_sender.rb
│   ├── notifications_sender.rb
│   ├── settler.rb
│   ├── status_setter.rb
│   ├── updater.rb
│   ├── users_setter.rb
│   └── voider.rb
├── shared
│   └── filter_suggested.rb
├── stats_maker.rb
├── subscriptions
│   └── actions.rb
├── tln
│   └── number_checker.rb
├── translators
│   └── google.rb
├── units.rb
├── update_user_to_developer.rb
├── user_loyalty_points.rb
├── users
│   ├── actions.rb
│   ├── creator.rb
│   ├── destroyer.rb
│   ├── exporter.rb
│   ├── o_auth_creator.rb
│   ├── retrieve_all_data
│   │   ├── base.rb
│   │   ├── favorites.rb
│   │   ├── friends_list.rb
│   │   ├── notifications.rb
│   │   ├── personal_information.rb
│   │   └── reviews.rb
│   ├── retrieve_all_data.rb
│   └── settings.rb
├── vat_checker.rb
├── version_converter.rb
└── x_server
    ├── add_detour_distance.rb
    ├── add_driving_distance.rb
    ├── base.rb
    ├── build_polyline.rb
    ├── corridor_search.rb
    ├── get_detour_distance.rb
    ├── get_driving_distance.rb
    ├── get_geo_keywords.rb
    ├── get_location_attributes.rb
    ├── get_route.rb
    └── leave_reachable.rb
</code>
</pre>
</details>

At some point this folder became a dumpall folder without rhyme or reason.
There is also an issue of boundaries: there are none. Each class is used from anywhere in the application and that can
cause headaches.

# Desired structure

Here is a folder structure we should strive for

```
app
├── controllers  # Rails 
├── frontend     # webpacker
├── domains      # custom
├── models       # Rails
├── views        # Rails
└── web          # custom
```

Let's first talk about Rails-specific folders (`controllers`, `views` and `models`).

The `views` folder needs to exist because it is hardcoded in the Rails itself (https://github.com/rails/rails/blob/main/railties/lib/rails/engine.rb#L610)

The `controllers` folder is there because of the way Rails finds controller names (it appends `"Controller"` to the resource name) (https://github.com/rails/rails/blob/main/actionpack/lib/action_dispatch/http/request.rb#L88)

`models` is a folder where we put all our ActiveRecord models. It is technically not necessary but we will talk about it in the [Further Improvements](#further-improvements) chapter.


# `domains` vs `web` folder
`web` is for classes used in controller actions: forms, policies, serializers, queries and similar.
`domains` is for business logic.

## `web` folder
The `web` folder structure should follow your routes. Having these routes:

```ruby
namespace: :dasboard do
  resources :projects
end
namespace :api do
  namespace :v1 do
    resources :projects
  end
end
```

it would have this folder structure:

```
web
├── dashboard
│   ├── projects
│   │   ├── presenter.rb
│   │   ├── decorator.rb
│   │   ├── form.rb
│   │   └── policy.rb
├── api
│   ├── v1
│   │   ├── v1
│   │   │   ├── projects
│   │   │   │   ├── policy.rb
│   │   │   │   ├── query.rb
│   │   │   │   └── serializer.rb
├── mailers
│   ├── projects
│   │   ├── mailer.rb
│   │   └── decorator.rb
```

## `domains` folder
The `domains` folder should be organized into domains. 

The main idea is to create cohesive groups of domain objects that are involved in the same business use case(s). The same way we try to build classes that are loosely coupled but highly cohesive we should build our domains as well.
Here is an example of domains folder:

```
domains
├── base
│   ├── decorator.rb
│   ├── mailer.rb
│   ├── policy.rb
│   ├── presenter.rb
│   ├── query.rb
│   └── serializer.rb
├── exports
│   ├── formatters
│   ├── csvs
│   └── spreadsheets
├── external
│   └── zapier.rb
├── projects
│   ├── counter.rb
│   └── presister.rb
└── utils
    ├── cache.rb
    └── slug.rb
```

## Further improvements
* Move mailers into domains folder, where they are actually used. There might be an issue of template files discovery.
* Move controllers to web folder as well
* Move models to domains and web folders (build DTOs)
* Move views to web folder
* Move frontend to web folder

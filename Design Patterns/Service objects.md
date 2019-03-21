## What is a service object?

*A service object implements the user’s interactions with the application*.
Inspecting the services folder of an application should tell the programmer what the application really does, which is not always obvious when looking at controllers or models.
The benefit of using services is concentrating the core logic of the application in a separate object, instead of scattering it around controllers and models.

## Conventions

* Services go under the app/services directory. Use subdirectories for business logic-heavy domains. For instance,
the `app/services/order/creator.rb` file will define `Order::Creator`
* Services end with an actor (and do not end with Service): `TransactionApprover`, `TestNewsletterSender`, `UsersImporter`
* A service object should have only *one* public method
* Services should, in most cases, respond to a `call` method—using another verb makes it a bit redundant: `TransactionApprover.approve`

## Example

You have a Parking Place model, and you need to merge two records into one record.

## Bad solution

Do everything in a controller.

```ruby
class ParkingPlaceController < ApplicationController
  def merge
    primary_parking_place = ParkingPlace.find_by(id: params[:id])
    secondary_parking_place = ParkingPlace.find_by(id: params[:secondary_parking_place_id])
    #merge attributes
    primary_parking_place.attributes = secondary_parking_place.attributes.except("id", "created_at", "updated_at", "description").delete_if { |k, v| v.blank? }
    #merge comments
    secondary_parking_place.comments.update_all(parking_place_id: primary_parking_place.id)
    secondary_parking_place.ratings.update_all(parking_place_id: primary_parking_place.id)
    #merge filters
    primary_parking_place.update_filters(secondary_parking_place.location_filters)
    primary_parking_place.save
  end
end
```

## Still bad solution

Move the business logic in the model.

```ruby
# app/controllers/parking_place_controller.rb
class ParkingPlaceController < ApplicationController
  def merge
    primary_parking_place = ParkingPlace.find_by(id: params[:id])
    secondary_parking_place = ParkingPlace.find_by(id: params[:secondary_parking_place_id])
    primary_parking_place.merge(secondary_parking_place)
  end
end

# app/model/parking_place.rb

def merge(secondary_parking_place)
  #merge attributes
  attributes = secondary_parking_place.attributes.except("id", "created_at", "updated_at", "description").delete_if { |k, v| v.blank? }
  #merge comments
  secondary_parking_place.comments.update_all(parking_place_id: id)
  #merge filters
  update_filters(secondary_parking_place.location_filters)
  save
end
```

## Good solution

Move the logic to a service object.

```ruby
# app/controllers/parking_place_controller.rb
class ParkingPlaceController < ApplicationController
  def merge
    ParkingPlaceMergerer.new(
      params[:primary_parking_place_id],
      params[:secondary_parking_place_id]
    ).call
  end
end

# app/services/parking_place_mergerer.rb

class ParkingPlaceMergerer
  def initialize(primary_parking_place_id, secondary_parking_place_id)
    @primary_parking_place = ParkingPlace.find_by(id: primary_parking_place_id)
    @secondary_parking_place = ParkingPlace.find_by(id: secondary_parking_place_id)
  end

  def call
    merge_attributes
    merge_comments
    merge_filters
    primary_parking_place.save
  end

  private

  attr_reader :primary_parking_place, :secondary_parking_place

  def merge_attributes
    primary_parking_place.attributes = secondary_parking_place_attributes
  end

  def merge_comments
    secondary_parking_place.comments.update_all(parking_place_id: primary_parking_place.id)
  end

  def merge_filters
    primary_parking_place.update_filters(secondary_parking_place.location_filters)
  end

  def secondary_parking_place_attributes
    secondary_parking_place
      .attributes
      .except("id", "created_at", "updated_at", "description")
      .delete_if { |k, v| v.blank? }
  end
end
```

## Further reading

[Keeping your Rails controllers dry with services](https://blog.engineyard.com/2014/keeping-your-rails-controllers-dry-with-services)

[Gourmet Service Objects](http://brewhouse.io/blog/2014/04/30/gourmet-service-objects.html)

[Railscast Service Objects](http://railscasts.com/episodes/398-service-objects)

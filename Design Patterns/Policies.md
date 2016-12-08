# Policies

Policy objects are plain old ruby ruby classes that encapsulates complex read operations. One definition says that policy objects are similar to Service objects, but the difference is that Service objects are used for write operations and Policy objects for reads. Also, they are different from Query objects as query objects focus on SQL reads and Policy objects operate on data already loaded in memory.

Most common case to use Policy objects is for authorization when you need to check combination of rules before allowing user to execute some action. Sometimes those rules are complex and it is better to extract this logic in it's own class, rather than putting it in controllers.

The most used gem based on policy objects is [Pundit](https://github.com/elabs/pundit). In our examples we'll focus on this gem, because it shows a very good way of using policy objects for authorization.

## Example

We have an Order model that has this fields: `company_id`, `due_date`, `active`, `offers_count`. Field `company_id` tells us which company created the order. An Order can have multiple Offers from **other companies**. The Offer model consist of `order_id`, `company_id`, `amount`.

Let's say that we have an OffersController with basic CRUD actions. Before every action we need to perform authorization checks.

For offer creation, these are the rules:

* A User can give offers only to orders of other companies.
* Offers can be made only to active orders.

For updating the order, the rules are as follows:

* User cannot edit other companies offers.
* Offer cannot be updated after order's due date.

## Bad solution

We will put authorization logic in controller actions.

```ruby
class OffersController < ApplicationController
  def new
    # ...
  end

  def create
    @offer = Offer.new(offer_params)

    if @offer.company_id == current_user.company_id || !@offer.order.active?
      flash[:error] = 'You are not authorized to perform this action.'
      redirect_to root_path
    end

    @offer.save
    respond_with @offer
  end

  def edit
    # ...
  end

  def update
    @offer = Offer.find(params[:id])

    if @offer.company_id != current_user.company_id || @offer.order.due_date >= Date.today
      flash[:error] = 'You are not authorized to perform this action.'
      redirect_to root_path
    end

    @offer.update(offer_params)
    respond_with @offer
  end

  def delete
    # ...
  end

  private

  def offer_params
    params.require(:offer).permit(:order_id, :company_id, :amount)
  end
end
```

## Good solution

We will use Pundit to extract authorization logic into a separate ruby class.

We will include Pundit in the Application controller (or some other base controller) and tell it how to handle unauthorized actions.

```ruby
class ApplicationController < ActionController::Base
  include Pundit
  rescue_from Pundit::NotAuthorizedError, with: :user_not_authorized

  def user_not_authorized
    flash[:error] = 'You are not authorized to perform this action.'
    redirect_to general_root_path
  end
end
```

In our controller, we only need to call the authorize method and give it a resource object as an argument.

```ruby
class OffersController < ApplicationController
  def new
    # ...
  end

  def create
    @offer = Offer.new(offer_params)
    authorize @offer

    @offer.save
    respond_with @offer
  end

  def edit
    # ...
  end

  def update
    @offer = Offer.find(params[:id])
    authorize @offer

    @offer.update(offer_params)
    respond_with @offer
  end

  def delete
    # ...
  end

  private

  def offer_params
    params.require(:offer).permit(:order_id, :company_id, :amount)
  end
end
```

Our authorization logic will be in the OfferPolicy class. A good place for our policy classes is the `app/policies` folder.

```ruby
class OfferPolicy
  attr_reader :user, :offer, :order

  def initialize(user, offer)
    @user = user
    @offer = offer
    @order = offer.order
  end

  def create?
    order.company != user.company && order.active?
  end

  def update?
    offer.company == user.company && order.due_date >= Date.today
  end
end
```

That's it.

## Questions

**How Pundit's `authorize` method actually works?**

From Pundit's docs:
The authorize method automatically infers that Offer class will have a matching OfferPolicy class - it instantiates the OfferPolicy class, passing in the current user and the given record. It then infers from the action name, that it should call update? on this instance of the policy. In this case, you can imagine that authorize would have done something like this:

```ruby
raise "not authorized" unless OfferPolicy.new(current_user, @offer).update?
```

## Further reading

* [Rails - the Missing Parts - Policies](http://eng.joingrouper.com/blog/2014/03/20/rails-the-missing-parts-policies/)
* [Straightforward Rails Authorization with Pundit](http://www.sitepoint.com/straightforward-rails-authorization-with-pundit/)

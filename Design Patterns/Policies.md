Policy Objects are plain old Ruby classes that encapsulate complex read operations. One definition says that Policy Objects are similar to Service Objects, but the difference is that Service Objects are used for write operations and Policy Objects for reads. Also, they are different from Query Objects because Query Objects focus on SQL reads, while Policy Objects operate on data already loaded in memory.

The most common case of using Policy Objects is for authorization when you need to check a combination of rules before allowing the user to execute some action. Sometimes those rules are complex, and it is better to extract this logic in its own class, rather than put it in controllers.

[Pundit](https://github.com/elabs/pundit) is the most commonly used gem based on Policy Objects. We'll focus on this gem in our examples because it shows a very good way of using Policy Objects for authorization.

## Example

We have an Order model with these fields: `company_id`, `due_date`, `active`, and `offers_count`. The `company_id` field tells us which company created the order. An Order can have multiple Offers from **other companies**. The Offer model consist of `order_id`, `company_id`, and `amount`.

Let's say that we have an OffersController with basic CRUD actions. We need to carry out authorization checks before every action.

These are the rules for creating an Offer:

* A User can give Offers only to Orders of other companies.
* Offers can be made only to active Orders.

These are the rules for updating an Offer:

* A User cannot edit other companies' Offers.
* An Offer cannot be updated after the Order's due date.

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

We will use Pundit to extract authorization logic into a separate Ruby class.

We will include Pundit in the Application Controller (or some other base controller) and tell it how to handle unauthorized actions.

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

Our authorization logic will be in the OfferPolicy class. The `app/policies` folder is a good place for our policy classes.

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

**How does Pundit's `authorize` method actually work?**

From Pundit's docs:
The authorize method automatically infers that the Offer class will have a matching OfferPolicy class, and it instantiates the OfferPolicy class, handing in the current user and the given record. It then infers from the action name that it should call update? on this instance of the policy. In this case, you can imagine that authorize would have done something like this:

```ruby
raise "not authorized" unless OfferPolicy.new(current_user, @offer).update?
```

## Further reading

* [Rails—the Missing Parts—Policies](http://eng.joingrouper.com/blog/2014/03/20/rails-the-missing-parts-policies/)
* [Straightforward Rails Authorization with Pundit](http://www.sitepoint.com/straightforward-rails-authorization-with-pundit/)

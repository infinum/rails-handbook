Form Objects are used for removing form specific logic away from your ActiveRecord models into a separate class.

With Form Objects we can:

  * Decouple form logic from your ActiveRecord models (i.e. refactor fat models)
  * Update multiple ActiveRecord models with a single form submission
  * Respect the single responsibility principle
  * Easily add additional behaviour to forms (i.e. methods)
  * Reuse logic between multiple forms
  * Use with formbuilders like SimpleForm or Formtastic

## Example

We have a registration form where we ask users to give us the following data:

  * full name
  * company name
  * phone number
  * email

`full_name` and `email` are associated with `User` model and `company_name` and `phone_number` are associated with `Company` model and we want to create both models on form submission. Additionally `User` model has `first_name` and `last_name` columns so we have to split the `full_name` attribute.

## Bad solution

There are couple of bad ways in which we could do this. One bad way is by adding missing attributes to one model, lets say User:

```ruby
class User < ActiveRecord::Base
  attr_accessor :company_name
  attr_accessor :phone
  attr_accessor :full_name

  validates :email, presence: true, email: true
  validates :company_name, presence: true
  validates :full_name, presence: true
end
```

This way we pollute our `User` class with logic that is not directly related to it which breaks the single responsibility principle. But without adding this logic to our `User` class we wouldn't be able to use a formbuilder on an instance of `User` class like we want to:

```ruby
= simple_form_for(@user, url: registrations_path) do |f|
  = f.input :full_name, placeholder: 'John Doe', autofocus: true
  = f.input :company_name, placeholder: "eg. Apple"
  = f.input :phone, label: 'Phone number', placeholder: "eg. + 1 555 330-1212"
  = f.input :email, placeholder: 'john.doe@apple.com'
  = f.button :submit, 'Start your 60 day trial', class: 'btn btn-green'
```

Also a lot of form logic has to be in our controller class:

```ruby
class RegistrationsController < ApplicationController
  def new
    @user = User.new
  end

  def create
    @user = User.new(user_params)
    if @user.valid?
      @user.first_name = @user.full_name.split(' ').first
      @user.last_name = @user.full_name.split(' ')[1..-1].to_a.join(' ')
      @user.save
      @company = Company.create(name: @user.company_name, phone: @user.phone)
      @company.users << @user
      redirect_to root_path
    else
      render :new
    end
  end
  
  def user_params
    params.require(:user).permit(:full_name, :company_name, :phone, :email)
  end
end
```

## Good solution

We create a Form Object to represent this specific form:

```ruby
class RegistrationForm
  include ActiveModel::Model

  attr_accessor :full_name
  attr_accessor :company_name
  attr_accessor :phone
  attr_accessor :email

  validates :full_name, presence: true
  validates :company_name, presence: true
  validates :email, presence: true, email: true

  def save
    return false unless valid?
    company = Company.create(name: company_name, phone: phone)
    company.users.create(first_name: user_first_name, last_name: user_last_name, email: email)
  end

  private

  def user_first_name
    full_name.split(' ').first
  end

  def user_last_name
    full_name.split(' ')[1..-1].to_a.join(' ')
  end
end
```

Including `ActiveModel::Model` gives us access to model name introspections, conversions, translations and validations (e.g. we can call the method `valid?` to check validations).

Now we use this Form Object in our view and controller:

```ruby
= simple_form_for(@form, url: registrations_path) do |f|
  = f.input :full_name, placeholder: 'John Doe', autofocus: true
  = f.input :company_name, placeholder: "eg. Apple"
  = f.input :phone, label: 'Phone number', placeholder: "eg. + 1 555 330-1212"
  = f.input :email, placeholder: 'john.doe@apple.com'
  = f.button :submit, 'Start your 60 day trial', class: 'btn btn-green'
```

```ruby
class RegistrationsController < ApplicationController
  def new
    @form = RegistrationForm.new
  end

  def create
    @form = RegistrationForm.new(registration_params)
    if @form.save
      redirect_to root_path
    else
      render :new
    end
  end

  private

  def registration_params
    params.require(:registration_form).permit(:full_name, :company_name, :phone, :email)
  end
end
```

If the form is valid we create both models and redirect to success path. On the other hand if there are any validation errors, we render the `:new` partial with errors. Everything here behaves like we are using an ActiveRecord model.

## Questions

**Can I use I18n with Form Objects?**

Sure, the same way you use I18n with ActiveRecord objects. Just use `activemodel` instead of `activerecord` key in your locale files:

```yml
en:
  activemodel:
    attributes:
      registration_form:
        phone: "Phone number"
    errors:
      models:
        registration_form:
          attributes:
            phone:
              present: "Phone number can't be blank"
```

**Where to put the Form Object classes?**

Create a folder `app/forms` and put all your Form Object classes there

## Further reading
  * [ActiveModel Form Objects](https://robots.thoughtbot.com/activemodel-form-objects)
  * [Reform](https://github.com/apotonick/reform)
  * [Active Type](https://github.com/makandra/active_type)
  * [Virtus](https://github.com/solnic/virtus)

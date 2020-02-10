# Local development

We don't want to send real emails in development so we use Mailer previews and Letter opener in development.

As for the content formatting goes - sometimes we use [simple html templates](https://github.com/mailgun/transactional-email-templates), other times we slice custom templates by design. If you're not sure which one is suited for your project, consult with your team lead and project manager.

If somebody from an other team will be working on emails, please make sure you first set up the letter opener and instruct your colleague on how to use Mailer previews.

## Mailer previews
Rails has a built-in feature for [previewing emails](https://guides.rubyonrails.org/action_mailer_basics.html#previewing-emails), and it's the most efficient way to preview email templates in development. Please read the instructions.

Here's an example of a preview class:
```ruby
# Preview all emails at http://localhost:3000/rails/mailers/devise_mailer
class UserMailerPreview < ActionMailer::Preview
  def storage_limit_warning_email
    UserMailer.storage_limit_warning_email(user)
  end

  private

  def user
    User.new(email: 'rails@is.best', username: 'mockyMock', space_usage: 450)
  end
end
```

The important thing is not to load objects from your database here, because you will have problems when you delete or change the data - and other developers probably won't have the same data in their database. Instantiate virtual objects in the mailer class with all attributes and relationships used in the template.


## Letter opener

[Letter opener](https://github.com/ryanb/letter_opener) is very useful when clicking around the app. Instead of sending the email, it will render the email in a new tab.


# Production and staging

In production/staging/uat we use [Mailgun](https://mailgun.com) to send emails from our apps.

Someone from the DevOps team should open a Mailgun account and set up everything for production / staging and store credentials in corresponding secrets files.

To set up Mailgun in the app, add the official [mailgun-ruby](https://github.com/mailgun/mailgun-ruby#rails) gem and set it up per environment as described in the Readme.

Mailgun also has webhooks, if you, for example, need to track failed deliveries due to having non-existing/non-verified emails in the database.


## Sending emails on behalf of users
Sometimes you will need to send email on behalf of the app's users, i.e. Contact form submissions. **Never set from field to user's address - use Reply-To.**. Mailgun won't deliver emails with from address with a different domain than the one configured in Mailgun settings.

## Intercepting emails
In most apps you should intercept emails in staging, to prevent sending emails to real users or avoid Mailgun failures if you have fake/non-existing emails in the database. 

There's a [mail_interceptor](https://github.com/bigbinary/mail_interceptor) gem which intercepts and forwards emails to a given address, and also sends the emails only to whitelisted addresses.

Non-production environments should have prefixed subjects, i.e. `[Staging] Confirmation Instructions`.
https://guides.rubyonrails.org/action_mailer_basics.html#intercepting-emails

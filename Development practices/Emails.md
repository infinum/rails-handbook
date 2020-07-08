# Guidelines

## Templates
In most cases we don't need a custom design for the emails. We use [simple HTML templates](https://github.com/mailgun/transactional-email-templates) and modify them slightly with project's theme colors, and add a logo if needed.

For implementing a fully custom template we use [MJML](https://mjml.io/) with the [mjml-rails gem](https://github.com/sighmon/mjml-rails). Before setting it up on a new project, check with someone from the JS team whether this still is the best tool for the job.

If somebody from the other team is expected to work on emails, please make sure you first set up the Letter opener, prepare the previews for all mailers and instruct your colleague on how to use the [mailer previews](#mailer-previews).

## Send emails asynchronously
Emails should be sent asynchronously from a background job:
- so they don't block the current HTTP request longer than needed
- if the email server is temporarily unavailable, the background job will fail and re-try again (check your retry options!).

See the [Background jobs chapter](/books/rails/development-practices/background-and-scheduled-jobs) on how to configure those.

If you're using Devise, don't forget to configure it for ActiveJob as described [here](https://github.com/heartcombo/devise#activejob-integration).

# Local development

We usually don't need nor want to send real emails to users in development, so we use [Letter opener](https://github.com/ryanb/letter_opener). Instead of sending an email, it will render the email in a new tab.

## Mailer previews
Rails has a built-in feature for [previewing emails](https://guides.rubyonrails.org/action_mailer_basics.html#previewing-emails). Using it allows you to see what the email looks like without sending a real email. Please read the setup instructions on the previous link.

We typically add mailer preview classes to the `spec/mailers/previews` folder.

Here's an example of a preview class:
```ruby
# Preview all emails at http://localhost:3000/rails/mailers
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

The important thing is not to load existing objects from your development database here, because you will have problems when you delete or change the data. Other developers working on the project probably won't have the same data in their database.
A better solution would be to just build the necessary objects with all attributes and relationships used in the template.


# Production and staging

In production/staging/uat we mainly use [Mailgun](https://mailgun.com) to send emails from our apps.

Someone from the DevOps team should open a Mailgun account and set up everything for production/staging and store credentials in [Vault](../handling-secrets).

To set up Mailgun in the app, add the official [mailgun-ruby](https://github.com/mailgun/mailgun-ruby#rails) gem and set it up per environment as described in the gem's Readme.

Mailgun has [webhooks](https://documentation.mailgun.com/en/latest/api-webhooks.html), which are useful if you need to track failed deliveries caused by non-existing/non-verified emails in the database. We've created the [mailgun_catcher](https://github.com/infinum/rails-infinum-mailgun_catcher) gem to simplify webhook integration in our apps.

## Sending emails on behalf of users
Sometimes we need to send an email on behalf of the app's users, i.e. Contact form submissions. **Never set _from_ field to user's email address - use Reply-To.**. Mailgun won't deliver emails with _from_ address with a different domain than the one configured in Mailgun settings.

## Intercepting emails
Emails in non-production environments should have prefixed subjects, i.e. `[Staging] Confirmation Instructions`. Creating a custom interceptor is simple, for more info read [here](https://guides.rubyonrails.org/action_mailer_basics.html#intercepting-emails).

In some apps you should intercept emails in staging to prevent sending them to real users, or to avoid Mailgun failures if you have fake/non-existing emails in the database.
There's the [mail_interceptor](https://github.com/bigbinary/mail_interceptor) gem which intercepts and forwards emails to a given address, and also sends the emails only to whitelisted addresses.

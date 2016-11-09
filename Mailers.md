# Mailers

## Production

We are using [mailgun](https://mailgun.com) to send mails from our apps.

### Mailgun setup

1. Check Last Pass if a client already has a mailgun account.
2. Create an account if not. Email template is: mailgun+{client}@infinum.hr
3. Create a domain for the app.
4. Contact your friendly neighborhood devops to add DNS records for domain verification.
5. Add `mailgun_rails` gem to gemfile
6. Add smtp settings to your app. Example:

        config.action_mailer.delivery_method = :mailgun
        config.action_mailer.mailgun_settings = {
                api_key: Rails.application.secrets.mailgun[:api_key],
                domain:  Rails.application.secrets.mailgun[:domain]
        }
7. Profit

### Mailgun HTTP API advantages

The HTTP API has some advantages over SMTP:

1. It’s faster.
2. Better for large scale sending.
3. You don’t have to deal with MIME because Mailgun will assemble it on their side.
4. Request libraries are available for your language of choice.


## Staging

### Email interceptor setup

When staging and production databases are in sync, users and their emails are also in sync.
Sometimes, we trigger an action which could send emails to real users. To prevent this,
we should intercept all emails which are sent to real users from staging environment.

We are using [recipient_interceptor](https://github.com/croaky/recipient_interceptor) for emails restriction at staging.

1. Add `recipient_interceptor` gem to gemfile
2. In config/environments/staging.rb add gem settings. Example:

        Mail.register_interceptor(
          RecipientInterceptor
          .new(Rails.application.secrets.permitted_emails,
               subject_prefix: '[STAGING]')
        )
3. In secrets.yml add permitted_emails key under the staging section.
4. If you want to set more then one email, then emails should be separated with comma. (Example: 'test1@infinum.co,test2@infinum.co')

If everything is configured, all emails will be delivered at specified emails.

## Development

### Letter opener setup

Preview email in the default browser instead of sending it. This is great because of:

1. You don't need to set up email delivery in development environment.
2. There is no risk of accidentally sending a test email to real user when developing and testing.

We are using [letter_opener](https://github.com/ryanb/letter_opener) for previewing emails in development.

1. Add `letter_opener` gem to gemfile(in development group)
2. In config/environments/development.rb set delivery method. Example:

      config.action_mailer.delivery_method = :letter_opener

### Rails built in email previewing

Rails also has built in feature for previewing emails.[Previewing emails](http://guides.rubyonrails.org/action_mailer_basics.html#previewing-emails)
With this approach, it is possible to see how email looks without sending real email.

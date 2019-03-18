## Production

We use [Mailgun](https://mailgun.com) to send emails from our apps.

### Mailgun setup

1. Check Last Pass if the client already has a Mailgun account.
2. Create an account if they don't. The email template is: mailgun+{client}@infinum.hr
3. Create a domain for the app.
4. Contact your friendly neighborhood devops to add DNS records for domain verification.
5. Add the `mailgun_rails` gem to the gemfile.
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

### Mailgun webhooks setup

If you want to track your emails for more than 7 days (which is the retainer period for free Mailgun), you need to setup webhooks to our email aggregation service.

![alt tag](https://s3.amazonaws.com/infinum.productive.production/attachments/files/000/092/933/original/Webhooks_-_Mailgun___2016-11-10_10-41-26.png?1478770924)

## Staging

### Email interceptor setup

When the staging and production databases are in sync, the users and their emails are also in sync.
Sometimes, we trigger an action which could send emails to real users. To prevent this,
we should intercept all emails that are sent to real users from the staging environment.

We use [recipient_interceptor](https://github.com/croaky/recipient_interceptor) to restrict emails at staging.

1. Add the `recipient_interceptor` gem to the gemfile
2. Add the gem settings in config/environments/staging.rb. Example:

        Mail.register_interceptor(
          RecipientInterceptor
          .new(Rails.application.secrets.permitted_emails,
               subject_prefix: '[STAGING]')
        )
3. Add the permitted_emails key in secrets.yml under the staging section.
4. If you want to set more then one email address, then they should be separated with a comma. (Example: 'test1@infinum.co,test2@infinum.co')

If everything is configured, all emails will be delivered to specified email addresses.

## Development

### Letter opener setup

Preview an email in the default browser instead of sending it. This is great for two reasons:

1. You don't need to set up email delivery in the development environment.
2. There is no risk of accidentally sending a test email to a real user when developing and testing.

We are using [letter_opener](https://github.com/ryanb/letter_opener) for previewing emails in development.

1. Add the `letter_opener` gem to the gemfile(in the development group)
2. Set the delivery method in config/environments/development.rb. Example:

      config.action_mailer.delivery_method = :letter_opener

### Rails built-in email previewing

Rails also has a built-in feature for [previewing emails](http://guides.rubyonrails.org/action_mailer_basics.html#previewing-emails).
Using it allows you to see how the email looks without sending a real email.

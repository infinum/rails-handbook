# Mailers In production

We are using [mailgun](https://mailgun.com) to send mails from our apps.

## Setup

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

## Logging

Infinum Rails Team uses the [Graylog](https://www.graylog.org/) for logging all requests which are processed by our apps.

### Setup
To use Graylog in a Rails project you need to do the following:

* Append your `Gemfile` with following gems:

  * Example

    ```Ruby
      gem 'gelf'
      gem 'rails_semantic_logger'
    ```

  * [Gelf gem](https://github.com/Graylog2/gelf-rb) allows you to send GELF messages to Graylog server instances.

  * The [rails_semantic_logger gem](https://github.com/rocketjob/rails_semantic_logger) replaces the default Rails logger with Semantic Logger. It also reduces Rails logging output in production to almost a single line for every Controller-Action call. Basically, it is full fledged logging framework with [numerous options](http://rocketjob.github.io/semantic_logger/rails).

* Add `graylog_host` in config for all environments.
  * production.rb and staging.rb

    ```Ruby
      config.graylog_host = Rails.application.secrets.graylog_host
      config.rails_semantic_logger.add_file_appender = false  # if you want to disable logging to files
    ```

  * development.rb and test.rb

    ```Ruby
      config.graylog_host = nil
    ```

* Add `semantic_logger.rb` in initializers folder.

  * Example

    ```Ruby
      if Rails.application.config.graylog_host.present?
        SemanticLogger.add_appender(
          appender:    :graylog,
          url:         Rails.application.config.graylog_host,
          application: "#{application_name}-#{Rails.env}",
          level:       :info,
          gelf_options: {
            tls: {
              cert: '/etc/ssl/private/graylog/graylog.crt',
              key: '/etc/ssl/private/graylog/graylog.key'
            }
          }
        )
      end
    ```
  * With these settings, you need to use TCP, keep that in mind when defining the URL (ask the DevOps team)

* Customize payload

  * Add following code in the controllers which logs you wish to alter.

    ```Ruby
      def append_info_to_payload(payload)
        super
        payload[:ip_address] = request.remote_ip if request.remote_ip.present?
        payload[:user] = current_user.username if current_user.present?
      end
    ```
* Consult with PM, and define time range for keeping historical data.

* Notify our devops before turning logging on because they should handle streams, permissions, backup options etc...

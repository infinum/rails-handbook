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
  * staging.rb

    ```Ruby
      config.graylog_host = :luscic
    ```

  * production.rb

    ```Ruby
      config.graylog_host = :dreznik
    ```

  * development.rb and test.rb

    ```Ruby
      config.graylog_host = nil
    ```

* Add `semantic_logger.rb` in initializers folder.

  * Example

    ```Ruby
      SemanticLogger.add_appender(
        appender:    :graylog,
        url:         "udp://#{Rails.application.config.graylog_host}.infinum.co:12201",
        application: "application_name-#{Rails.env}",
        level:       :info
      )
    ```

* Customize payload

  * Example

    ```Ruby
      def append_info_to_payload(payload)
        super
        payload[:ip_address] = request.remote_ip if request.remote_ip.present?
        payload[:user] = current_user.username if current_user.present?
      end
    ```

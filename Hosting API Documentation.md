# Summary

This is the flow we use to generate and host our documentation:

- Write tests using rspec and annotate them using [dox](https://github.com/infinum/dox)
- Once tests on [semaphore](https://semaphoreci.com) pass we move to deploy
- In the deploy phase we do the following:
  * deploy the application using [mina](https://github.com/mina-deploy/mina)
  * run only dox tagged specs (request specs) to generate the documentation
  * copy the generated documentation to the application server inside `public` folder

# Why

Hosting the API documentation directly from the application server is convenient and does not cost extra.
Solutions like apiary.io require having one project for each environment, can't easily set
a desired domain and can't easily hide behind some kind of authentication (like Basic Authentication).

To ensure that the documentation is always up to date it is copied only after a successful deploy

By using the `public` folder nginx takes care of serving the static file and the requests don't go through the whole Rails stack.

# How

## Dox generating task

Here is an advanced example of a rake task that generates the documentation:
It has the feature to write ERB code inside your accompanying .md files.

```ruby
namespace :dox do
  task :md, [:version, :docs_path, :host] do |_, args|
    require 'rspec/core/rake_task'

    version = args[:version] || :v1
    docs_path = args[:docs_path] || "api/#{version}/docs"

    RSpec::Core::RakeTask.new(:api_spec) do |t|
      t.pattern = "spec/requests/#{version}/"
      t.rspec_opts =
        "-f Dox::Formatter --order defined -t dox -o public/#{docs_path}/apispec.md.erb"
    end

    Rake::Task['api_spec'].invoke
  end

  task :html, [:version, :docs_path, :host] => :erb do |_, args|
    version = args[:version] || :v1
    docs_path = args[:docs_path] || "api/#{version}/docs"
    `node_modules/.bin/aglio -i public/#{docs_path}/apispec.md -o public/#{docs_path}/index.html --theme-full-width --theme-variables flatly`
  end

  task :open, [:version, :docs_path, :host] => :html do |_, args|
    version = args[:version] || :v1
    docs_path = args[:docs_path] || "api/#{version}/docs"

    `open public/#{docs_path}/index.html`
  end

  task :erb, [:version, :docs_path, :host] => [:environment, :md] do |_, args|
    require 'erb'

    host = args[:host] || 'http://localhost:3000'
    version = args[:version] || :v1
    docs_path = args[:docs_path] || "api/#{version}/docs"

    File.open("public/#{docs_path}/apispec.md", 'w') do |f|
      f.write(ERB.new(File.read("public/#{docs_path}/apispec.md.erb")).result(binding))
    end
  end
end
```

```markdown
FORMAT: 1A
HOST: <%= host %>


# My app API
An API documentation for my app
```

```sh
rake 'api:doc:html[v1, api/v1/docs, https://example.com]'
```

The above example can be simplified if some of the features are not needed.

## Deploy script

The mina-dox plugin includes the `dox:publish` task, which runs `rake dox:html` and copies the generated documentation to the server.

```sh
echo "=========== bundle install ==========="
bundle install --deployment --path vendor/bundle
echo "=========== deploy ==========="
bundle exec mina production ssh_keyscan_domain deploy

echo "=========== setup for tests ==========="
bundle exec secrets pull -e development -y
bundle exec rake db:setup
bundle exec rake db:test:prepare

echo "=========== install aglio and themes ==========="
yarn install

echo "=========== publish api doc ==========="
bundle exec mina production dox:publish
```

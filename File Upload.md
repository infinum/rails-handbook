There are several file upload gems out there. We use
[Shrine](https://github.com/janko-m/shrine) because of its flexibility and
easy-to-extend nature.

Table of contents:

1. [General](#general)
2. [Setup](#setup)
3. [Plugins](#plugins)
4. [Migrating from other uploaders](#migrating-from-other-uploaders)

## General

Shrine is a general-purpose file uploader gem. As it has no direct dependency,
__it can be used in both Rails and non-Rails projects.__ Shrine features a rich
plugin system which makes it incredibly easy to change its behavior to suit
your needs. All this power is packaged in an easy-to-use form factor.

It is the youngest of all popular file uploaders, but it maintains an issue
count of 0 and has fixed all the issues that other gems have.

If you wish to read more about Shrine's design, you can do so
[here](http://twin.github.io/introducing-shrine/).

There is also [a demo application](https://github.com/erikdahlstrand/shrine-rails-example). It's not recommended to reference it as it's fairly simplistic and hides many configuration options.

## Setup

To use Shrine in a Rails project, you need to do the following:

* Add it to your `Gemfile` together with its image processing and cloud storage
libraries

```Ruby
gem 'shrine'

# Only add this gem if you are going to use Amazon S3 as storage
gem 'aws-sdk', '~> 2.1'

# Only add these gems if you are going to handle image uploads
gem 'mini_magick'
gem 'image_processing'
gem 'fastimage'
```

* Run `bundle install`
* Create a `shrine.rb` file in your `config/initializers` folder to configure
Shrine

```Ruby
# config/initializers/shrine.rb

require 'shrine'
require 'shrine/storage/s3'
require 'shrine/storage/file_system'

# Load application secrets
secrets = Rails.application.secrets

# NOTE:
# The following code expects a secrets structure similar to the following
# and uses some Ruby 2.3 features
#
# development:
#   aws:
#     s3:
#       access_key_id: 'ashkjdsahjas...'
#       secret_access_key: 'ashkjdsahjas...'
#       region: 'ashkjdsahjas...'
#       bucket: 'ashkjdsahjas...'


# Load S3 credentials from secrets
s3_options = {
  access_key_id: secrets.dig(:aws, 's3', 'access_key_id'),
  secret_access_key: secrets.dig(:aws, 's3', 'secret_access_key'),
  region: secrets.dig(:aws, 's3', 'region'),
  bucket: secrets.dig(:aws, 's3', 'bucket')
}

# Load storage options
storages = {
  cache: nil,
  store: nil
}

# Set storage options to S3 if S3 credentials have been provided, else use the
# filesystem
storages.keys.each do |store|
  storages[store] = if secrets.dig(:aws, :s3)
                      Shrine::Storage::S3.new(prefix: store, **s3_options)
                    else
                      Shrine::Storage::FileSystem.new(
                        'public',
                        prefix: "uploads/#{store}"
                      )
                    end
end

# Assign storage options
Shrine.storages = storages

# Load ORM integration
Shrine.plugin :activerecord
# Shrine.plugin :sequel # if you are using Sequel
```

* Require image processing libraries in your `application.rb` file by
prepending the following line

```Ruby
require 'image_processing/mini_magick'
```

* Create an uploader in your `app/uploaders` folder

```Ruby
# app/uploaders/avatar_uploader.rb

class AvatarUploader < Shrine
end
```

* Mount the uploader in your model

Shrine's uploaders are mounted on a model using the `include` keyword.
You can use the square brackets method on the uploader class to specify the attribute the uploader will be mounted to.

```Ruby
class User < ActiveRecord::Base
  # The following line will mount an instance of AvatarUploader to the user's
  # avatar attribute
  include AvatarUploader[:avatar]
end
```

* Create the appropriate column in your model's table.

Shrine assumes that it should use a column bearing the name of the attribute it
was mounted on suffixed with `_data`. For example, if you mount an uploader on the
`avatar` attribute of the `User` model, Shrine will store information about the
uploaded file in the `avatar_data` column of the `users` table. The column's
type should be `text`, `json`, or `jsonb`.

```BASH
rails g migration add_avatar_data_to_users avatar_data:jsonb
```

* Create a file field in your view

```SLIM
= simple_form_for @user do |f|
  .row
    .col-xs-12
      = f.input :avatar, as: :hidden, input_html: { value: @user.avatar_data }
      = f.input :avatar, as: :file
```

If you use simple_form, add the following file to your `app/inputs` folder

```Ruby
# app/inputs/shrine_file_input.rb

class ShrineFileInput < SimpleForm::Inputs::Base
  def input(wrapper_options)
    merged_input_options = merge_wrapper_options(
      input_html_options, wrapper_options
    )
    raw_input_html(merged_input_options)
  end

  private

  def raw_input_html(options)
    %(
      #{hidden_field}
      #{file_field}
    ).html_safe
  end

  def data_attribute
    "#{attribute_name}_data".to_sym
  end

  def hidden_field
    @builder.hidden_field(
      attribute_name,
      value: object.send(data_attribute).to_json
    )
  end

  def file_field
    @builder.file_field(
      attribute_name,
      input_html_options
    )
  end
end
```

Then you can use it in your views as follows:

```SLIM
= simple_form_for @user do |f|
  .row
    .col-xs-12
      = f.input :avatar, as: :shrine_file
```

## Plugins

Shrine features quite a few useful out-of-the-box plugins.

* Versions

The Versions plugin allows you to create different versions of an uploaded
image, for example, a small, medium, and large version. Read more about it
[here](https://github.com/janko-m/shrine#versions).

Here is an example of how to use it:

```Ruby
class ImageUploader < Shrine
  include ImageProcessing::MiniMagick
  plugin :versions, names: [:original, :large, :medium, :small]

  def process(io, context)
    return super(io, context) if context[:phase] != :store

    original = io.download
    size_1200 = resize_to_limit(original, 1200, 1200)
    size_600 = resize_to_limit(size_1200, 600, 600)
    size_300 = resize_to_limit(size_600, 300, 300)

    { original: original, large: size_1200, medium: size_600, small: size_300 }
  end
end
```

* Validations

This plugin allows you to check the dimensions and file type of uploaded images.
Check it out [here](https://github.com/janko-m/shrine#validations).

* Metadata

The Metadata plugin is used to store information about uploaded images.
You can read more about it [here](https://github.com/janko-m/shrine#metadata).

* Background jobs

Shrine supports background image processing and deletion. You can read how to set it up 
[here](https://github.com/janko-m/shrine#background-jobs).

## Migrating from other uploaders

Here are two articles that explain how to migrate to
Shrine from other uploaders:

* [Shrine for CarrierWave Users](http://shrinerb.com/rdoc/files/doc/carrierwave_md.html)
* [Shrine for Paperclip Users](http://shrinerb.com/rdoc/files/doc/paperclip_md.html)

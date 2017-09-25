## General
Our APIs always have an endpoint for mobile clients to check what the latest and minimum version of their SDK is.

They are using a library called [Prince of Versions](https://github.com/infinum/Android-Prince-of-Versions) to achieve
that, and if the current version is deprecated the library will notify the user to update their app.

This file is only used in production, since we keep our staging builds on [Infinum Labs](https://labs.infinum.co/).

## Name
Our current convention for naming that file is `mobile-versions.json`

## Location
The file should live in the `public` folder since it's only a JSON file and there's no business logic behind it.

## Initial content
Use these contents when you're creating the file. The mobile team will contact you when you should upgrade a mobile SDK version.
  ```
    {
      "ios": {
        "minimum_version": "1.0.0",
        "latest_version": {
          "version": "1.0.0",
          "notification_type": "ONCE"
        }
      },
      "android": {
        "minimum_version": "1.0.0",
        "minimum_version_min_sdk": 1,
        "latest_version": {
          "version": "1.0.0",
          "notification_type": "ONCE",
          "min_sdk": 1
        }
      }
    }
  ```

## Validate it!
You should always validate the `mobile-versions.json` file after making a change (https://jsonlint.com/) and before
pushing the change and deploying it to production.

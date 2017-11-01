## General
[Prince of Versions](https://github.com/infinum/Android-Prince-of-Versions) is a library used by mobile API clients to determine the available application versions. It can see if there is an update available or if the client has an outdated application still running on their device.

Prince of Versions contains the minimum and the latest version of the application that is suitable to use with the current API server. It is useful for mobile clients when mandatory updates are required due to security purposes or API incompatibilities.

The library contacts the server requesting the file with the version information. It is used only in production, since we keep our staging builds on [Infinum Labs](https://labs.infinum.co/).

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

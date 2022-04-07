## Environment Synchronization
A synchronization of environments, or more precisely, a synchronization of the data used in different environments,
is usually necessary when you need to work in an environment that is as production-like as possible, but you don't have
access to production itself, for example:

* when you need to debug a more complex issue occuring in production
* when you need data that is as close to production as possible (both when it comes to type, relationships,
value distribution and amount) for development and/or testing purposes

Access to production data is normally limited in order to ensure end user privacy.
The same should be ensured once this data is copied to another environment. In order to limit the number
of developers who have access to raw production data, it would be best if that data was anonymized by someone
from the DevOps team before it is given to the developer. To achieve this, the Rails team has agreed with
the DevOps team that the synchronization process should look like this:
1. a completely new environment is set up
2. the production database is copied to this new environment
3. anonymization is run on the copied production database
4. the anonymized database is dumped
5. the anonymized database dump is copied to a destination of choice (e.g. S3 bucket - but not a public one!)
6. the anonymized database dump is downloaded to the destination environment (staging, development, etc.)
7. a dump of the local database is generated as a backup
8. the local database schema is reset
9. the anonymized database dump is imported/restored to the local database

Keep in mind that all the steps up until and including step #5 require the involvement of the DevOps team.
The remaining steps can be automated using the [copy_bot](https://github.com/infinum/copy_bot) gem,
which is explained later in this chapter.

In addition to this, for the devops team to be able to anonymize the production database, you first need to provide
them with a way to do that. One such solution is described in the following section.

### Data anonymization
Data anonymization is a form of information sanitization intended to protect private information contained in
the database. It can be performed when it is necessary to work with production data during testing and/or debugging of
an issue, but also to fulfill GDPR requirements. In the first case, personal information is anonymized only when it is
needed (e.g. after copying the production database) and only in non-production environments. However, in the case of
GDPR compliance, it may need to be done in regular intervals in the production environment.

The required final result may also be different depending on the use case.
It may be enough to replace the sensitive data with random strings, but sometimes this is not a good solution if it
prevents the app from functioning properly. In the latter case it is more appropriate to replace real data with
synthetic data: synthetic names to replace the names of real persons, synthetic phone numbers
that are in the same format as the real phone numbers, etc. - this is called *pseudonymization*.

Since each project has different models and attributes, and consequently different personally identifiable
information that needs to be anonymized, as well as different needs for anonymization and pseudonymization,
there is no single universal solution for all of them. However, there is a proposed solution you can check out:
the `anonymize.rb` script in the JSONAPI Example App.

The suggested solution uses the [remont gem](https://github.com/infinum/remont) that we have developed to provide us
with a practical DSL for implementing anonymization by writing a simple script. It iterates over the records in
the specified scope and updates each record as it is defined in the script.
In the Example App we have also used the [faker gem](https://github.com/faker-ruby/faker)
to generate pseudo-anonymized data, but if your project requires more complex anonymization logic,
the remont gem also accepts [custom processors](https://github.com/infinum/remont#attributes).

You should implement some form of anonymization, whether it is a script based on the remont gem or a simple SQL query
replacing sensitive information with hash strings. You should apply the same care to private information that gets
to non-production environments in any other way, e.g. through the import of a file provided by the client.

Once you add anonymization to your project, also explain in the README file how and when it should be performed.

### Data synchronization
As mentioned, this step in the environment synchronization process can be automated with the help of
the [copy_bot gem](https://github.com/infinum/copy_bot).

To use this gem, you need to define at least one file with the required step definitions. It is possible to have
multiple step definition files, each for the synchronization of different environment pairs.

An example of the step definitions file:

```yaml
steps:
  download_remote_db_dump:
    s3_credentials:
      access_key_id: <%= Rails.application.secrets.aws_access_key_id %>
      access_key: <%= Rails.application.secrets.aws_secret_access_key %>
      region: <%= Rails.application.secrets.aws_region %>
      bucket: <%= Rails.application.secrets.aws_s3_bucket %>
    source_file_path: '/staging_db_dump.sql'
    destination_file_path: './tmp/downloaded_db_dump.sql'
  create_local_db_backup:
    destination_file_path: './tmp/development_backup.sql'
  drop_local_db_tables:
  import_remote_db_to_local_db:
    remote_db_dump_file_path: './tmp/downloaded_db_dump.sql'
  run_migrations_on_local_db:
  delete_remote_db_dump:
    remote_db_dump_file_path: './tmp/downloaded_db_dump.sql'
  execute_custom_command:
    command: "RAILS_ENV=staging bundle exec rake \"remont[db/s3_sync.rb]\""
```

Not all steps are mandatory. For example, you may have obtained the database dump in some other way,
i.e. not by downloading it from the S3 bucket, or you may not want to delete the downloaded DB dump file.
The option of adding a final custom command can be used to run a rake task for 
file synchronization or any other shell command.

This gem has also been added to the [JSONAPI Example App](https://github.com/infinum/rails-jsonapi-example-app).

### File synchronization
File synchronization is an optional step that not everyone may need when syncing environments,
but it can also be a standalone task that is run whenever necessary.

In addition to that, there could be very different requirements as to what files should be synchronized and how.
For example, you may want to sync entire AWS S3 buckets, or you may want to just copy a couple of files from one
bucket to another, or a file may have several different versions and you may want to handle that in a particular way.
You may want to use the AWS CLI tool or the [aws-sdk-ruby gem](https://github.com/aws/aws-sdk-ruby)
with options specific to your use case.

Before you decide on the implementation, take some time to consider if syncing files is even a good idea.
For instance, if you want to sync entire buckets, keep in mind that the size of the files you are copying to
the destination bucket will affect your AWS costs. Also, if you are copying files from the production environment,
they may contain sensitive data, such as proprietary and/or personally identifiable information that you cannot easily
anonymize.

If you decide to proceed with file synchronization, you can choose to implement the solution we have included in
the JSONAPI Example App. It contains a template (the `s3_sync.rb` script with accompanying classes)
for copying files from one bucket to another for a specified set of records in your database.
The remont gem is a flexible tool that has been reused here and the corresponding rake task can be added as
the final step to the data sync step definitions required by the copy_bot gem or it can be run separately.

The proposed solution uses the remont gem DSL to iterate over records for which you want to copy files from one bucket
to another. For each record the `EnvSync::FileSync::Processor` is called, and this processor calls
the `EnvSync::FileSync::Synchronizer`. You will notice that in the `EnvSync::FileSync::Synchronizer`
we are just copying one file from one bucket to another. However, you can use this example and adjust it to your needs
by simply changing the code in the `copy_file` method or by adding a completely different method, just be sure
to update the `EnvSync::FileSync::Processor` if necessary.

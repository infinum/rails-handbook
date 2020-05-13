If you want to use postgres extensions (for example postgis or uuid) on Amazons RDS Postgres database add this migration to your application:

```ruby
def change
  if Rails.env.in? ['development', 'test']
    enable_extension 'postgis'
  else
    STDERR.puts 'TELL DEVOPS TO ENABLE Postgis ON DATABASE'
  end
end
```

This is because of restrictions on Amazon RDS databases. To be able to enable extension on RDS you need to be a superuser (to have access to scripts on disk). The users we are using on RDS to read and write in our databases do not have superuser rights. That is why you will need to humbly ask our devops to enable it for you.

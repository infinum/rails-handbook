# Geokit-rails

Despite geocoder having a bigger community and giving the impression that it's better in all aspects, the majority of our developers had less trouble using geokit-rails.

`Geocoder` queries use aliases for computing geo informations(eg. `distance`). When used with PostgreSQL database, this implementation detail causes bugs in the client code in case where results need to be sorted by the computed geo information(because PostgreSQL doesn't allow ordering by aliased columns). Until maintainers show initiative to address [this problem](https://github.com/alexreisner/geocoder/issues/1205#issuecomment-327323301) we have decided to use `Geokit-rails`.

For more detailed information about the state of the gems, please take a look at the [gem comparisons](https://gist.github.com/cilim/a19aece2b954a837dc87ac67c67c92ad)

### References:
- https://forum.shakacode.com/t/picking-a-geocoding-gem/377/5
- https://ruby.libhunt.com/project/geocoder/vs/geokit

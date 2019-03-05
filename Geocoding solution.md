# Geokit-rails

Despite the Geocoder having a bigger community and giving the impression that it's better in all aspects, the majority of our developers have had less trouble using geokit-rails.

The `Geocoder` queries use aliases for computing geo-information (e.g., `distance`). When used with the PostgreSQL database, this implementation detail causes bugs in the client code in cases when results need to be sorted by the computed geo-information (because PostgreSQL doesn't allow ordering by aliased columns). We have decided to use `Geokit-rails` until the maintainers show initiative to address [this problem](https://github.com/alexreisner/geocoder/issues/1205#issuecomment-327323301).

For more detailed information about the state of the gems, please have a look at these [gem comparisons](https://gist.github.com/cilim/a19aece2b954a837dc87ac67c67c92ad)

### References:
- https://forum.shakacode.com/t/picking-a-geocoding-gem/377/5
- https://ruby.libhunt.com/project/geocoder/vs/geokit

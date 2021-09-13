Project requirements are prone to changes. Even with a good, stable API design, the new requirement might force the API to change as well, which breaks client code unless it's adapted. Sometimes, it's not possible for all clients to adopt the change (like mobile applications which depend on users to update them), but we still have to push the change to other clients. At that point, we have to backward support one version of an endpoint and simultaneously release a new one. This is called versioning.

In technical terms, API versioning is a strategy for serving the same resource in multiple, incompatible formats. We will define what "incompatible" means later on.

To give an example of a breaking change, imagine you serve an endpoint which returns country data:

```json
// GET api/v1/countries/hr
{
  "code": "HR", // alpha-2 code
  "name": "Croatia"
}
```

This endpoint fulfills all needs — for the time being. After a while, a new requirement is to serialize alpha-3 country code along with the existing alpha-2 code. You might be tempted to just add a new key `alpha_3_code` to the response, but why should `code` by default imply alpha-2 code? Now that you have to serialize multiple country codes, wouldn't it make more sense to have keys `alpha_2_code` and `alpha_3_code` so it's immediately obvious what each of them represents?

If the answer to that question is affirmative, and you know that some clients depend on the `code` key and that they won't be able to rename it to `alpha_2_code`, but you still want to add the new alpha-3 code, then you'll have to preserve the current endpoint and create a new one with the updated format:

```json
// GET api/v2/countries/hr
{
  "alpha_2_code": "HR",
  "alpha_3_code": "HRV",
  "name": "Croatia"
}
```

Now you're serving the same resource in multiple formats, or, in other words, you have a versioned API.

*Side note*: notice that if the key for alpha-2 code was named `alpha_2_code` from the start, a breaking change wouldn't be necessary. That is, the only change would be the addition of a new key `alpha_3_code`, which is not breaking anything. This is one of the reasons why it's important to carefully think about API design and naming in particular.

## How to version

There are a number of strategies for API versioning. This is still a widely discussed and contentious topic, but this chapter won't go into details (if you're interested, see [this blog post](https://www.troyhunt.com/your-api-versioning-is-wrong-which-is/)). Rather, we'll propose a versioning strategy which works for us.

We version APIs by adding a version number in the URL (prefixed with `v`). For example, an API for retrieving countries might have endpoints `/api/v1/countries` and `/api/v2/countries`.

We include the version in APIs *from the beginning*, starting with `v1`. This choice is a pragmatic one even if you'll never have to release a second version. The cost of adding a version to the URL initially is imperceptible and once it's set up you don't have to think about it anymore. Adding new versions is also easy because you won't have to change/refactor code from version `v1` in order to support the new version.

In Rails, this kind of versioning has the following structure in routes:

```ruby
# config/routes.rb
namespace :api do
  namespace :v1 do
    resource :countries
  end

  namespace :v2 do
    resource :countries
  end
end
```

Controllers are then also namespaced under API version: `Api::V1::CountriesController` for `/api/v1/countries`, and `Api::V2::CountriesController` for `/api/v2/countries`.

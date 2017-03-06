# Intro

Building an API is different than building a regular HTML application. There are
no HTML views to worry about, but instead you should take extra care about status responses, response format and especially documentation since your application will be used
by other developers.

## Creating an application

To generate a Rails application with only the elements necessary for an API
application use the `--api` flag e.g.
`rails new my_application --api`

We write our APIs in JSON format, more specifically using the [JSON API standard](http://jsonapi.org/).
This allows those who consume our APIs to know what to expect regarding the
document structure.

## Gems

We use a few gems that make our lives easier when writing APIs:
* [json_api_responders](https://github.com/infinum/json_api_responders)
* Simplifies the code in controllers, no need to write `if @object.save` etc. to
display errors.
* [jsonapi-rails](https://github.com/jsonapi-rb/jsonapi-rails)
* Takes care of serialization/deserialization of Ruby objects into/out of JSON
objects, all according to the JSON API standard, so you don't have to think about
those details.
* [rspec](https://github.com/rspec/rspec-rails)
* Testing is always important when building software, but doubly so when writing
an API since we can use tests to generate documentation.
* [dox](https://github.com/infinum/dox)
* Generates documentation from rspec tests in api blueprint format.

## Documentation

You can generate documentation from your controller or request specs, and it's very
important that you cover all the possible cases and return codes e.g. a valid response,
response when there are errors and any other outcome of the action. Also document any filtering parameters and enumerations that may be used to refine the results of the API call.

https://apiary.io/ is a service that can be used to host the API documentation.

## Versioning

APIs should be versioned from the start, that way in case of large changes clients can still use the older version of the API while adjusting their application to the new API specification. Otherwise if the API is not versioned, changes to the API would break clients applications frequently and the client would have to take a lot of care in tracking the changes.

You can make different API versions obvious by namespacing them in the URL e.g. ```api/V1/resource``` where the new version would look like ```api/V2/resource```

## Pagination

It's often possible that the results of the API call are huge e.g. hundreds of thousands of records, and that's impractical if the client only needs a small part of the results. That's why we can limit the amount of results returned with pagination, which allows the client to choose which part of the results they want to see in the request e.g results 500-600 out of 10000.

## HTTP status codes and headers
* Clients must set the content-type to `application/json` in their request headers
* Read about which HTTP status codes exist and what they are used for [here](https://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html)

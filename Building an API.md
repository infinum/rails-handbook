# Intro

Building an API is different than building a regular HTML application. There are
no HTML views to worry about, but instead you should take extra care about response statuses, response format and especially documentation since your application will be used by other developers.

## Creating an application

To generate a Rails application with only the elements necessary for an API
application, use the `--api` flag, e.g.,
`rails new my_application --api`.

Use the `--api` flag only if you're sure that your application will be exclusively an API. If you'll have
an admin backend, or use the view layer in some capacity, the Rails API mode will not suffice, and it's best to
use the standard `rails new` command.

We write our APIs using the JSON format, more specifically, we use the [JSON API standard](http://jsonapi.org/).
This allows those who consume our APIs to know what to expect regarding the
document structure, and they can use libraries which enable them to build their clients faster and easier.

## Gems

We use several gems that make our lives easier when writing APIs:

[json_api_responders](https://github.com/infinum/json_api_responders)
* Simplifies the code in controllers, e.g.,
```
  if resource.valid?
    render json: resource, status: 200
  else
    render error: errors, status: 422
  end
```
becomes
```
  respond_with resource
```

[jsonapi-rails](https://github.com/jsonapi-rb/jsonapi-rails)
 * Takes care of the serialization/deserialization of Ruby objects into/out of JSON
objects, all according to the JSON API standard, so you don't have to think about
those details.

[rspec](https://github.com/rspec/rspec-rails)
 * Testing is always important when building software, but doubly so when writing
an API since we can use tests to generate documentation.

[dox](https://github.com/infinum/dox)
*  Generates documentation from the RSpec tests in the API Blueprint format.

## Documentation

You can generate documentation from your controller or request specs. It's very
important that you cover all the possible cases and return codes, e.g., a valid response,
response when there are errors, and any other outcome of the action. Also, document pagination, enumerations, filtering parameters, sorting parameters, and anything else that may be used to refine the results of the API call.

Take special care to document crucial parts of your API, such as authentication and session management.

Generating documentation should be a part of the Continuous Integration process so that you can be sure your documentation is always up-to-date. You can use a service like
[Apiary](https://apiary.io/) to host the documentation.

## Versioning

APIs must be versioned from the start. In that way, in case of large changes, clients can still use the older version of the API while adjusting their application to the new API specification. Otherwise, if the API is not versioned, changes to the API would break the  client's applications frequently, and the client would have to take a lot of care to track the changes.

Namespace different API versions in the URL, e.g., a new version of ```api/v1/resource``` would look like ```api/v2/resource```. Also, make sure that your application code, that is, your controllers, serializers, tests, and documentation are also namespaced.

## Pagination

It's often possible that the API call provides a huge number of results—it could be hundreds of thousands of records. That's impractical if the client needs only a small part of the results. That's why we can limit the number of results returned with pagination, which allows the client to choose which part of the results they want to see in the request, for example, results 500-600 out of 10000.

Pagination is required for every index action, and should be described in the documentation with information on which query parameters are used, what the default and max page size is, etc.

## HTTP headers and status codes

If the JSON API standard is used, clients have to set the `content-type` to `application/vnd.api+json` in their request headers. In other cases, when using the JSON format, `application/json` will suffice.

Some of the more commonly used HTTP status codes are:
#### Success codes
* `200 OK`—request has succeeded
* `201 Created`—new resource has been created
* `204 No Content`—no content needs to be returned (e.g., when deleting a resource)

#### Client error codes
* `400 Bad Request`—request is malformed in some way (e.g., wrong formatting of JSON)
* `401 Unauthorized`—authentication failed
* `403 Forbidden`—user does not have permissions for the resource
* `404 Not Found`—resource could not be found
* `422 Unprocessable Entity`—resource hasn't be saved (e.g., validations on the resource failed)

#### Server error codes
* `500 Internal Server Error`—something exploded in your application


There are many more HTTP status codes. You can go [here](https://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html) to read the whole specification.

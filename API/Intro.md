APIs are sets of defined protocols enabling applications to communicate with other systems, that can also be applications. In the context of the Web, APIs are a way for Web clients (browsers, mobile apps, terminals, servers, and other platforms) to communicate with Web servers.

Our APIs use the HTTP protocol and are usually in JSON format, but if necessary, we might implement file uploads, XML or something else the project requires.

The following chapters provide information on our practices regarding JSON APIs. Although some gems are mentioned throughout, primary focus is on aspects surrounding API design and delivery.

## API design principles

Before talking about specific topics like authentication and pagination, we should outline a set of principles all APIs should follow. Since it's challenging to understand generic definitions, all of the principles will be introduced with an example showing what happens when they're *not* followed to try to illustrate the importance of following them.

*Note*: all the examples below are inspired by situations developers have been in when dealing with terrible APIs.

### Predictable

What we mean by "predictable API" is that it never aims to surprise the client. As an example, let's take an imaginary API endpoint for retrieving song details:

```json
// GET api/songs/123
{
  "id": 123,
  "name": "Modern Love",
  "artist": "David Bowie",
  "album": "Let's Dance"
}
```

There is nothing wrong here, the endpoint returns data which you use in your application. Everything works fine, until one day your application breaks. You debug the code, find that it has to do with the endpoint, so you fetch it, and get the following result:

```json
// GET api/songs/123
{
  "id": 123,
  "name": "Modern Love",
  "artist_id": 491,
  "album_id": 528
}
```

The API developer decided that instead of serializing artist and album names, they want to return their IDs. Technically, this is not a bad decision, it's quite a common change actually. However, what they failed to do is communicate this change to their clients in advance, so that clients can update their code to support the new response.

Clients have expectations about APIs. If you tell clients that an endpoint returns data in a particular format, they will start to build their code around it. An API changed without prior notice breaks those assumptions and client applications which depend on them.

Fortunately, the remedy is simple: communicate with your clients. Anticipate that a change might break client applications and tell them about it in advance.

### Consistent design

Let's imagine an API endpoint which returns country data:

```json
// GET api/countries/hrv
{
  "Ime": "Croatia",
  "nativeName": "Hrvatska",
  "top_level_domain": ".hr"
}
```

What's the issue here? As you can see, the API is using different naming conventions for attributes:

* `Ime` - Croatian PascalCase
* `nativeName` - English camelCase
* `top_level_domain` - English snake_case

The problem is not the choice of cases (every option is good), nor the language, but the inconsistency. When building an API, pick one style and stick with it.

This doesn't apply just to request and response bodies, URLs are also subject to consistency rules, e.g. endpoints `/api/country_data` and `/api/currency-data` are bad design because they're mixing snake_case and kebab-case.

### Following conventions and standards

Here is an example of an endpoint which returns album data:

```json
// GET api/albums/123
{
  "name": "Elephant",
  "explicit": "false"
}
```

The API serializes the attribute `explicit` which tells the client if the album contains strong language. This has been implemented as a boolean field, however the API developer decided to stringify the boolean, so instead of returning `false`, they return `"false"`.

### Performant

As the project grows, the user base will also grow, thus increasing the amount of requests sent to your server. Without a fast and optimized API, the response times will sky-rocket and the user experience will plummet, which might cause users to delete their accounts and abandon your product.

There are many ways to improve the performance of an API, from caching to database sharding, but please be aware that the amount of information fetched from the database and wrapped in an API response should be as small as it can be. Increasing the speed of SQL queries and avoiding any [performance issues](https://infinum.com/handbook/books/devproc/general-coding-practices/api-design#performance-issues) are crucial measures that should always be undertaken.

### Robust

A strong, healthy, and flexible API is a delight to work with. This goes for the producers as well as the consumers of the API.
To have a robust API, you'll need to ensure:

- a nice interface: existing behaviour is easily applicable throughout the system (e.g.: new filter or sort option)
- bad requests don't cause problems for subsequent usages
- integration tests that go through every endpoint and supported behaviour before deploying new builds

### Debuggable

Delivering a feature usually entails many steps that have to be resolved before shipping to production. When the project is split into *backend* and *frontend*, it often results in time spent communicating how an API is working. This time increases, especially if the API's consumer doesn't understand something and wastes time debugging a certain endpoint or a collection of endpoints.

The debugging time can be decreased by using appropriate HTTP status codes and providing descriptive error messages in API responses.

Common HTTP statuses for client-related errors are:

- _Bad Request (400)_ - user must use valid and supported format in API communication
- _Unauthorized (401)_ - user needs to be logged in in order to use an endpoint
- _Forbidden (403)_ - user needs to be permitted to perform a certain action
- _Not Found (404)_ - user can only fetch a resource that exists
- _Unprocessable entity (422)_ - user tries to create/update a resource, but has provided incomplete or invalid data

The HTTP status codes help the consumer distinguish if the error is *data-related* (for which access to the database is needed, providing valid credentials, updating permissions, finding a resource to work with), or if it's *request-related* (for which the HTTP request itself would need further investigation).

Request-related errors can be improved so that the frontend developer has a painless experience figuring out the problem. Imagine an endpoint where a user can update a song. The *Song* model validates that the _name_ attribute is always present, meaning that a song can't exist in the database without its name. Attempting to update a song with an invalid name would look like this:

```JSON
PATCH api/v1/songs/123
{
  "name": "",
  "artist": "David Bowie",
  "album": "Let's Dance"
}
```

A good error structure should consist of:

- title - short categorization of the error, usually the name of the HTTP status code
- detail - coherent and explicit description of the error (e.g.: _"The name of the song must be present."_)
- code - usually HTTP status code, but if you have a requirement for errors to be categorized in one way or another, this can be a good attribute to place the values from the error legend
- source - object that tells the API consumer what the *name* of the parameter with the error is and its *path* in the prior request

```JSON
{
  "errors": [
    {
      "title": "Unprocessable entity",
      "detail": "Song must contain a name",
      "code": 422,
      "source": {
        "parameter": "name",
        "pointer": "/name"
      }
    }
  ]
}
```

### Documented

Transparency is key to not losing any data or information that might be beneficial to anyone on the project. Human beings are faulty, and we're not capable of keeping everything at the forefront of our minds. Over time, some insights might get lost or forgotten and spending time debugging and reminding ourselves is costly.

An API should always live alongside its documentation. Read more in the [API Documentation](Documentation) chapter.

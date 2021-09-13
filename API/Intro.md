APIs are sets of defined protocols enabling applications to communicate with other systems, that can also be applications. In the context of the Web, APIs are a way for Web clients (browsers, mobile apps, terminals, servers, and other platforms) to communicate with Web servers.

Our APIs use the HTTP protocol and are usually in JSON format, but if necessary we might implement file uploads, XML or something else the project requires.

The following chapters provide information on our practices regarding JSON APIs. Although some gems are mentioned throughout, primary focus is on aspects surrounding API design and delivery.

## API design principles

Before talking about specific topics like authentication and pagination, we should outline a set of principles all APIs should follow. Since it's challenging to understand generic definitions, all of the principles will be introduced with an example showing what happens when they're *not* followed, to try to illustrate the importance of following it.

*Note*: all of the examples below are inspired by situations developers have been in when dealing with terrible APIs.

### Predictable

When we call an API "predictable", what we mean is that it never aims to surprise the client. As an example, let's take an imaginary API endpoint for retrieving song details:

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

Clients have expectations about APIs. If you tell clients that an endpoint returns data in a particular format, they will start to build their code around it. An API changed without prior notice breaks those assumptions and client application which depend on them.

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

### Robust



### Following conventions and standards

Here is an example of an endpoint which returns album data:

```json
// GET api/albums/123
{
  "name": "Elephant",
  "explicit": "false",
}
```

The API serializes an attribute `explicit` which tells the client if the album contains strong language. This has been implemented as a boolean field, however the API developer decided to stringify the boolean, so instead of returning `false`, they return `"false"`.

### Debuggable

# TODO: explain that when errors happen they have to help the user resolve them, i.e. they shouldn't be vague

### Documented

# TODO: explain the need for documentation and point to Documentation chapter

### Performant

# TODO: link to Jakas article about not including everything

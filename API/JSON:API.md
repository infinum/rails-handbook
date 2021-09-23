JSON:API is a specification for tailoring APIs that respond with JSON. Its purpose is to take the API `design` out of the equation so that developers can focus on API `delivery`. Due to its structure, tooling can be built around the specification for handling the bulk of the work regarding parsing requests and formatting responses.

If your work consists of building or working with JSON:API APIs, read the [specification](https://jsonapi.org/) to understand the concepts behind it. If you're interested in what the community has to say about JSON:API, check out these blog posts:

- [Standardizing RESTful JSON APIs with OpenAPI Spec](https://oozou.com/blog/standardizing-restful-json-apis-with-openapi-spec-53)
- [The Benefits of Using JSON API](https://nordicapis.com/the-benefits-of-using-json-api/)

### Standardized APIs

Developers at Infinum have implemented dozens of APIs and the [JSON:API query builder](https://github.com/infinum/jsonapi-query_builder) is a collection of the most required features a modern API has to offer.

The gem allows a developer to use a nice DSL for the so-called *query classes*, which are classes that contain JSON:API related logic for resource manipulation (filtering, sorting, pagination etc...).

If you're just starting a new project, install the gem and make your life easier.

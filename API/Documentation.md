An API without accompanying documentation practically doesn't even exist. Furthermore, without good, thorough documentation, no API can be good either. As the developer behind an API, you are naturally familiar with what you've created and how to use it. However, your clients are not. Documentation exists to transfer that knowledge to clients.

Documentation writing requires thinking about what the API client needs to set up their environment, what they're able to do with the API, what to send in requests and which response scenarios to expect. All of this naturally requires providing lots of information and structuring it in a comprehensible way.

Writing isn't easy and developers tasked with writing documentation sometimes do a poor job. Poorly-written documentation necessitates further communication and explanations from the API developer — imagine then what would happen if you had to explain your API to a thousand confused clients. Therefore, to save time in the long run, you have to invest enough time to write proper documentation from the start.

## Essentials

The following checklist contains documentation essentials. This is not an exhaustive list — APIs always have specifics which cannot be included in such lists and you should document them as well. A good rule of thumb is, if someone would ask about it, it should be documented.

- **Overview** - Placed at the top of the documentation, the overview should explain: what the API does, architecture (e.g. REST), format (e.g. JSON:API or GraphQL) and it should provide general information (base URL, response types, etc.).
- **Format requirements** - If you use JSON:API, GraphQL or some other format, they typically come with requirements, like setting `Content-Type`/`Accept` headers, which the clients must abide by.
- **CORS** - If applicable, document your CORS policy (allowed origins, methods, exposed headers, credentials, etc.).
- **Authentication** - Applicable if endpoints are authenticated. Explain the authentication method(s) used (API key, cookies, JWT, etc.) and how to obtain credentials.
- **Authorization** - If the outcome of requests depends on the user's permissions, then that should also be documented. If authorization logic varies between endpoints, the specific logic should be explained for each endpoint.
- **Errors** - All APIs are subject to errors. Errors are roadblocks, preventing clients from doing what they need to do. When a roadblock appears with either no explanation or a vague one (`PC LOAD LETTER`), frustration ensues.<br />
Document <u>all</u> common errors and supply descriptions. Descriptions should explain what happened and what the client can do to remove the error. Document even the 5xx messages, no matter how obvious they seem.
- **Changelog** - Major changes to the API should be documented for historical reference. It's an optional step if the API will be used by a limited number of users to whom you can communicate changes directly, though it's useful to have it even then.
- **Endpoints** - The main body of the documentation, each endpoint should: describe what it is, document available HTTP verbs, and show example requests and responses for happy and sad paths. If a response error is specific to an endpoint, document it.

*Bonus points:*

- **Quickstart guide** - Onboard clients quickly by providing an example request (e.g. in cURL) which they can copy/paste to test out and modify. This approach is the fastest way to get clients to learn how to use the API.
- **Diagrams** - Complex flows are hard to explain with just words. If the API, for example, controls a state machine, words will struggle to explain its states and transitions. Diagrams, even simple ones created in tools like [draw.io](https://draw.io), will help clients tremendously in understanding the underlying logic.<br />
*Tip*: keep raw, editable files in the repository along with exported formats (like PNG). That way other developers can later update existing diagrams instead of having to create new ones.

## Hall of fame

Everything written about documentation practices in the preceding paragraphs sets a basis for writing API documentation. But it's just that — a basis. Real-world APIs are big and complex and to write comprehensible documentation for them means to consider many aspects not pointed out here. Instead of trying to put everything into words, here are some examples of API documentation that prove that it's possible to manage complexity and present it in an understandable and user-friendly way to the clients.

- [Stripe API](https://stripe.com/docs/api)
- [Productive](https://developer.productive.io/)
- [Example App](https://cekila.byinfinum.co/api/v1/docs/)

## Automating documentation

Documentation writing is laborious. Most of it has to be done by hand — nothing can replace that — but some things can be automated. One of the things we can automate are example requests and responses.

### Dox

Infinum's very own gem that does the majority of work in regards to API documentation for the developer. It's super easy to set up and with just a couple of changes to your existing request specs, you get an updated API documentation after the specs finish running.
Check the dox documentation by following [this link](https://github.com/infinum/dox).

# Authentication

Authentication is the process of verifying someone's identity. There are many ways to verify identities over HTTP APIs, the most popular of which will be described below.

When we talk about API authentication, it's important to distinguish its two components:
1. the mechanism — the medium which the authentication protocol utilizes
2. the method — the protocol for authenticating the user

These two work in tandem and sometimes they're indistinguishable, but we'll try to explain each one separately and then show how they work together. Let's look at available mechanisms first.

## Mechanisms

As was mentioned above, a mechanism is a medium utilized by the authentication protocol. The architecture of HTTP allows for several authentication mechanisms.

### Cookies

Cookies are pieces of data exchanged between the client and the server through headers. The server sends one or more cookies via a `Set-Cookie` response header and then the browser saves them and includes them in subsequent requests to the server in a `Cookie` header. The browser can also send cookies to the server without receiving any first. 

When setting cookies, the server can apply one or more restrictions, like expiration date and domains where they're supposed to be used. You can learn more about how cookies work [here](https://developer.mozilla.org/en-US/docs/Web/HTTP/Cookies).

There are two important cookie attributes you should know about:
- `Secure` — When set, cookie will only be sent to the server and saved in the browser if the protocol through which they're sent/received is `https`. Insecure `http` protocol can't access cookies with that attribute. If you're communicating over `https`, make sure to set this attribute.
- `HttpOnly` - These cookies cannot be accessed by JavaScript code. They are exchanged only between the browser and the server. If the cookie is used to store server-side session (a method described below), this attribute should be enabled because the cookie should be saved and used only by the browser, not read or changed by the user through JS. Note that this attribute doesn't prevent tampering of cookies as users can still edit them manually (for example, in a developer console).

### Authorization header

HTTP supports sending a standard `Authorization` header in requests. The header consists of a type (indicating what kind of a credential it is) and a value separated by whitespace. An example header can look like this: `Basic aHR0cHM6Ly95b3V0dS5iZS9kUXc0dzlXZ1hjUQ==`.

You can find the list of types [here](https://developer.mozilla.org/en-US/docs/Web/HTTP/Authentication#authentication_schemes). The one most commonly seen in APIs is the so-called `Bearer` token. As explained in the [Bearer Token Usage RFC](https://datatracker.ietf.org/doc/html/rfc6750#section-1.2), a bearer token is a:
> A security token with the property that any party in possession of the token (a "bearer") can use the token in any way that any other party in possession of it can.
When the server generates a token, it should make it opaque (meaning they can't be interpreted by anyone other than the server).

`Authorization` header is the preferred mechanism if your API will be used in platforms other than the browser. It's easier to control than cookies outside of browsers, and it doesn't come with security concerns around using query strings (described below).

### Basic auth

Basic auth is the simplest authentication method. It's a type of `Authorization` header authentication, where the "type" value is `Basic`.

### Query string

Query string is a part of the URL used to provide additional parameters. Query string starts after question mark character (`?`) and delimits multiple parameters with amspersands (`&`). Each parameter consists of a name and a string value separated by an equal sign (`=`).

For example, URL `example.com?api_key=123&token=abc` has two query params: `api_key` with value `123`, and `token` with value `abc`.

Query string can be used to transmit keys and tokens in requests to the server, similar to cookies and `Authorization` header. At a first glance, this might seem insecure, but if it's sent over the `https` protocol, query string is encrypted with the rest of the request and it's visible only by the client and the server.

However, there are other concerns when using query strings to transmit sensitive data:
- URLs (including query strings) are visible in server logs (although, you can [filter parameters](https://guides.rubyonrails.org/action_controller_overview.html#parameters-filtering) like these in Rails)
- URLs are saved in browser history (when used in navigation)
- you have to include the key/token in every request, but normally you want to reserve the query string for other parameters (like filters, pagination, sorting, etc.)

Consider using query strings for authentication only when other methods are unavailable.

## Methods

### API keys

API keys are unique strings used to identify clients.

API keys in most cases aren't used to authenticate the users themselves, but rather external access to user's resources. In those scenarios, users authenticate through other means (like server-side session) and then they request one or more keys for API access.

Regarding the format of API keys, they usually consist of alphanumeric characters and have a minimum length to prevent brute-force attacks.

If your system allows access via API keys, it should also provide a way to revoke them, in case they're stolen.

Remind users of security measures they can take to decrease the likelihood of a malicious party accessing their keys and resources:
- never store an API key in a public place (private repos on Github count as public too) or embed it directly in code (unless the key is meant to be public)
- regenerate API keys periodically
- delete a key if you're not using it anymore

One option for generating API keys is [`SecureRandom`](https://ruby-doc.org/stdlib-3.0.0/libdoc/securerandom/rdoc/SecureRandom.html), a Ruby standard library.

### Access and refresh tokens

# TODO: explain

### Server-side session

# TODO: explain

### JWT

# TODO: explain

## On reinventing authentication

For whatever reasons, you may feel like building your own authentication solution. Pursue this motivation, but not with the intention to use it in a production environment.

Authentication is a complex beast encompassing many aspects, most of which you won't be aware until you start building your own solution, and some of which you'll only learn about when someone bypasses its security.

There are megapopular public libraries available for all common authentication methods, which come with the assurance:
- that they have been tested in real, production environments by thousands if not millions of users,
- that they had their implementation examined by security experts,
- that they had suffered and fixed serious security breaches.
If you think your custom solution will live up to those expectations in the time you have to implement it, then feel free to build it. However, there are better and more productive ways to use your time, solving problems which haven't been solved already.

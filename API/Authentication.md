Authentication is the process of verifying someone's identity. There are many ways to verify identities over HTTP APIs, the most popular of which will be described below.

When we talk about API authentication, it's important to distinguish its two components:

1. the mechanism — the medium utilized by the authentication protocol
2. the method — the protocol for authenticating the user

These two work in tandem and sometimes they're indistinguishable, but we'll explain each one separately and show how they work together.

## Mechanisms

As mentioned above, a mechanism is a medium utilized by the authentication protocol. The architecture of HTTP allows for several authentication mechanisms.

### Cookies

Cookies are pieces of data exchanged between the client and the server through headers. The server sends one or more cookies via a `Set-Cookie` response header and then the browser saves them and includes them in subsequent requests to the server in a `Cookie` header. The browser can also send cookies to the server without receiving any first.

When setting cookies, the server can apply one or more restrictions, like expiration date and domains where they're supposed to be used. You can learn more about how cookies work on [MDN](https://developer.mozilla.org/en-US/docs/Web/HTTP/Cookies).

There are two important cookie attributes you should know about:
- `Secure` — When set, the cookie will only be sent to the server and saved in the browser if the protocol through which they're sent/received is `https`. Insecure `http` protocol can't access cookies with that attribute. If you're communicating over `https`, make sure to set this attribute.
- `HttpOnly` - These cookies cannot be accessed by JavaScript code. They are exchanged only between the browser and the server. If the cookie is used to store a server-side session (a method described below), this attribute should be enabled because the cookie should be saved and used only by the browser, not read or changed by the user through JS. Note that this attribute doesn't prevent the tampering of cookies as users can still edit them manually (for example, in a developer console).

_Note_: A cookie with the `HttpOnly` flag is the safest option for storing sensitive information in a browser, so if you have a JS-based frontend application that will consume your APIs, and you need to keep the secrets really secret, consider using this technique.

### Authorization header

HTTP supports sending a standard `Authorization` header in requests. The header consists of a type (indicating what kind of a credential it is) and a value separated by whitespace. An example header can look like this: `Basic aHR0cHM6Ly95b3V0dS5iZS9kUXc0dzlXZ1hjUQ==`.

[MDN](https://developer.mozilla.org/en-US/docs/Web/HTTP/Authentication#authentication_schemes) has a nice list of types. The one most commonly seen in APIs is the so-called `Bearer` token. As explained in the [Bearer Token Usage RFC](https://datatracker.ietf.org/doc/html/rfc6750#section-1.2), a bearer token is:

> A security token with the property that any party in possession of the token (a "bearer") can use the token in any way that any other party in possession of it can.
When the server generates a token, it should make it opaque (meaning they can't be interpreted by anyone other than the server).

The `Authorization` header is the preferred mechanism if your API is to be used by platforms other than the browser (browsers using it are susceptible to XSS attacks). It's easier to control than cookies outside of browsers, and it doesn't come with security concerns around using query strings (described below).  If your API will be consumed by both browsers and mobile platforms, you should provide for both cookies and the `Authorization` header as cookies can complicate the implementation on mobile platforms, and JS clients can't store/exchange authorization information securely with the server, other than through a cookie.

#### Basic auth

Basic auth is the simplest authentication method. It's a type of `Authorization` header authentication, where the "type" value is `Basic`.

### Query string

The query string is the part of the URL used to provide additional parameters. The query string starts after the question mark character (`?`) and delimits multiple parameters with ampersands (`&`). Each parameter consists of a name and a string value separated by an equal sign (`=`).

For example, the URL `example.com?api_key=123&token=abc` has two query params: `api_key` with the value `123`, and `token` with the value `abc`.

The query string can be used to transmit keys and tokens in requests to the server, similar to cookies and the `Authorization` header. At the first glance, this might seem insecure, but if it's sent over the `https` protocol, the query string is encrypted with the rest of the request and it's visible only to the client and the server.

However, there are other concerns when using query strings to transmit sensitive data:

- URLs (including query strings) are visible in server logs (although, you can [filter parameters](https://guides.rubyonrails.org/action_controller_overview.html#parameters-filtering) like these in Rails)
- URLs are saved in browser history (when used in navigation)
- you have to include the key/token in every request, but normally you want to reserve the query string for other parameters (like filters, pagination, sorting, etc.)
- XSS attacks
- length constraints (if you want to ensure your apps work on most modern browsers, exceeding 2047 characters isn't suggested)

Consider using query strings for authentication only when other methods are unavailable.

## Methods

### API keys

API keys are unique strings used to identify clients. Think of them like normal passwords, and you should also treat them as such.

API keys in most cases aren't used to authenticate the users themselves, but rather external access to user's resources. In those scenarios, users authenticate through other means (like a server-side session) and then they request one or more keys for API access.

Regarding the format of API keys, they usually consist of alphanumeric characters and have a minimum length to prevent brute-force attacks.

If your system allows access via API keys, it should also provide a way to revoke them, in case they're stolen.

Remind users of security measures they can take to decrease the likelihood of a malicious party accessing their keys and resources:
- never store an API key in a public place (private repos on Github count as public too) and never embed it directly in code (unless the key is meant to be public)
- regenerate API keys periodically
- delete a key if you're not using it anymore

One option for generating API keys is [`SecureRandom`](https://ruby-doc.org/stdlib-3.0.0/libdoc/securerandom/rdoc/SecureRandom.html), a Ruby standard library.

### Access and refresh tokens

Access tokens and refresh tokens are artifacts used in the OAuth 2.0 authorization protocol. This protocol enables a user to grant permissions for accessing resources from a 3rd party application. It is most widely used for features like Single Sign On (SSO) via a variety of social media platforms (eg: Facebook, Instagram, LinkedIn).

Once the user logs into the authorization server and grants access to their personal data, an `access token` is returned to the client server. This token can be used for subsequent communication with the server on behalf of the user. For that reason, it's critical to have security strategies that minimize the risk of compromising access tokens, for example creating access tokens with a short lifespan. The downside of short-lived access tokens is the fact that the client application must prompt the user to log in again, which makes for bad UX. A better solution would be to use `refresh tokens`.

Refresh tokens are artifacts that let the client application refresh an access token without asking the user to log in. They usually livie longer than access tokens, but that also means that if the refresh token is stolen, the malicious user has the power to create new access tokens, so additional security techniques need to be in place, like refresh token rotation or automatic reuse detection.

### JWT

The JSON Web Token standard is a method for creating the mentioned access tokens or any other artifact used for transmitting information between 2 servers. You can read more about it on [jwt.io](https://jwt.io/introduction).

### Server-side sessions

Believe it or not, cookies aren't the answer sometimes. Here are some usual arguments against them:

- file size (only about 4kb of data can be saved into a cookie)
- they are sent with every request making the requests slower if the cookies are bigger
- storing wrong data inside a cookie can be insecure ([replay attacks](https://guides.rubyonrails.org/security.html#replay-attacks-for-cookiestore-sessions))

When you can't store your session data inside a cookie, Rails has other options that are easily configurable:

_config/initializers/session_store.rb_

```ruby
Rails.application.config.session_store :cookie_store
```

Two of the most widely used alternatives for storing session data are:

- `redis_store` - toggle storage to be backed by Redis
- `active_record_store` - toggle storage to be backed by ActiveRecord

Both solutions come out of the box with their corresponding dependencies: [redis-rails](https://github.com/redis-store/redis-rails) & [activerecord-session_store](https://github.com/rails/activerecord-session_store).


## On reinventing authentication

For whatever reasons, you may feel like building your own authentication solution. Pursue this motivation, but not with the intention to use it in a production environment.

Authentication is a complex beast encompassing many aspects, most of which you won't be aware of until you start building your own solution, and some of which you'll only learn about when someone bypasses its security.

There are megapopular public libraries available for all common authentication methods, which come with the assurance:

- that they have been tested in real, production environments by thousands if not millions of users,
- that they had their implementation examined by security experts,
- that they had suffered and fixed serious security breaches.

If you think your custom solution will live up to those expectations in the time you have to implement it, then feel free to build it. However, there are better and more productive ways to use your time: solving problems which haven't been solved already.

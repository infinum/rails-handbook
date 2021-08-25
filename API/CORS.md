CORS (Cross-Origin Resource Sharing) is a mechanism for allowing web requests between domains (origins). Cross-origin requests are by default prevented by the browser [same-origin policy](https://developer.mozilla.org/en-US/docs/Web/Security/Same-origin_policy) — CORS exists to lift that restriction when required.

An excellent guide to CORS is available [here](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS). This mechanism concerns frontend and backend developers alike and should be understood by both. The omnipresence of CORS cannot be understated — if your work concerns the Web, CORS will crop up eventually. Therefore, don't hesitate to understand how it works, take your time and read the guide.

## Misconceptions

Assuming you have read the guide, you still might misunderstand how it works. This is not uncommon with CORS, it is one of the most misunderstood aspects of the Web. Therefore, the following list of misconceptions was compiled to help you understand it better:
- "CORS is enforced by the server"
  - Servers only supply CORS headers, the enforcer is actually **the browser**. It sends a preflight request to discover server's CORS policy, and based on that info decides whether to execute the request or raise an error.<br />
  This is also the reason why CORS errors don't show up in tools like cURL or Postman — they don't send a preflight request or enforce the server's headers.
- "I can just allow all origins with `*` and be done with it"
  - Depends. Setting `*` only works when the request doesn't include credentials (cookies or Authorization header). If you allow all origins *and* include credentials, the browser will [raise an error](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS/Errors/CORSNotSupportingCredentials). When you need to include credentials, `Access-Control-Allow-Origin` header must have a concrete origin value.
- "I can copy `Origin` header to `Access-Control-Allow-Origin` to satisfy CORS"
  - In certain situations (like hosting public APIs) this is okay, but in others you can expose your users to malicious websites which can execute requests on the user's behalf to your server with potentially disastrous results for the user. Read [this](https://portswigger.net/research/exploiting-cors-misconfigurations-for-bitcoins-and-bounties) article for more info.
- "Browser clients can use `no-cors` mode to skip CORS"
  - In a way, yes, but that option comes with severe limitations. The only methods allowed are `HEAD`, `GET` and `POST`, with the constraint that all headers are [simple](https://fetch.spec.whatwg.org/#simple-header) and that they **cannot** read the response in JS. [[Reference](https://developer.mozilla.org/en-US/docs/Web/API/Request/mode#value)]

To learn more about CORS misconceptions and misconfigurations, read [this](https://web-in-security.blogspot.com/2017/07/cors-misconfigurations-on-large-scale.html) article which shows how even the most popular sites fail to properly serve CORS headers.

## How to set up headers

For Rack-based applications (like Ruby on Rails), [`rack-cors`](https://github.com/cyu/rack-cors) is the most popular choice for CORS headers management. Follow the gem's README and make sure to read the [Common Gotchas](https://github.com/cyu/rack-cors#common-gotchas) chapter.

If you're setting up CORS for environments which are accessed locally by frontend developers (like staging), add `localhost` to the list of allowed domains so they can send requests from their local environment.

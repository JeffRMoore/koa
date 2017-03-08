# Response

  A Koa `Response` object is an abstraction on top of node's vanilla response object,
  providing additional functionality that is useful for every day HTTP server
  development.

## Overview

[Response Body](#response-body-overview) •
[Status Line](#status-line-overview) •
[Header Accessors](#header-accessor-overview) •
[HTTP Caching](#http-caching-overview) •
[Response Routing](#response-routing-overview) •
[Utilities](#utilities-overview) •
[HTTP Control](#http-control) •
[Content Negotiation](#content-negotiation) •
[Response Context](#response-context) •
[Security and Privacy](#security-and-privacy) •
[Range Responses](#range-responses) •
[Authentication Challenges](#authentication challenges) •
[Cookies](#cookies)

### Response Body Overview

[Response Body Reference](#response-body)

Property / Method | Type | Description
------------------| ---- | -----------
[`body`](#responsebody) | `String`<br> `Buffer`<br> `Object`<br> `Stream` | The response body
[`request.body`](#responsebody)` =` | `String`<br> `Buffer`<br> `Object`<br> `Stream` | Set the response body
[`request.length`](#responselength) | `Number` | The value of the `Content-Length` header as a number when present
[`request.length`](#responselength)` =` | `Number` | Set response `Content-Length` to the given value
[`request.type`](#responsetype) | `String` | The value of the `Content-Type` header void of parameters such as "charset"
[`request.type`](#responsetype)` =` | `String` | Set response `Content-Type` header
[`request.is`](#responseis)`()` | `String`<br> `false` | Check whether the response `Content-Type` matches one of the supplied types.
[`request.attachment`](#responseattachment)`()` | `undefined` | Set `Content-Disposition` to "attachment" to signal the client to prompt for download.

### Status Line Overview

[Status Line Reference](#status-line)

Property / Method | Type | Description
------------------| ---- | -----------
[`request.status`](#responsestuatus) | `Number` | The response status
[`request.status`](#responsestuatus) = | `Number` | Set the response status
[`request.message`](#response) | `String` | The response status message
[`request.message`](#response) = | `String` | Set the response status message

### Header Accessor Overview

[Basic Header Accessor Reference](#header-accessors)

Property / Method | Type | Description
------------------| ---- | -----------
[`request.header`](#responseheader) | `Object` | All response headers as an object with header name as the key.
[`request.headers`](#responseheaders) | `Object` | Alias of `header`
[`request.get`](#responseget)() | `String` | Get a response header value by name
[`request.set`](#responseset)() | `undefined` | Set a response header value by name (replacing existing header)
[`request.append`](#responseappend)() | `undefined` | Append additional header.  Like `set` but without replacing.
[`request.remove`](#responseremove)() | `undefined` | Remove response header by name
[`request.flushHeaders`](#responseflushheaders)() | `undefined` | Flush any set headers, and begin the body.
[`request.headerSent`](#responseheaderssent)() | `Boolean` | Check if a response header has already been sent. 

### HTTP Caching Overview

[HTTP Caching Reference](#http-caching)

Property / Method | Type | Description
------------------| ---- | -----------
[`request.lastModified`](#responselastmodified) | `Date` | The value of the `Last-Modified` header if it exists
[`request.lastModified`](#responselastmodified)` =` | `String`<br> `Date` | Set the `Last-Modified` header as an appropriate UTC string
[`request.etag`](#responseetag) | `String` | The value of the `Etag` header if it exists
[`request.etag`](#responseetag)` =` | `String` | Set the value of the `Etag` header
[`request.vary`](#responsevary)`()` | `undefined` | Issue a `Vary` header

### Response Routing Overview

[Response Routing Reference](#response-routing)

Property / Method | Type | Description
------------------| ---- | -----------
[`request.redirect`](#responseredirect)`()` | `undefined` | Issue a `Location` header for a 30x redirect.

### Utilities Overview

[Utilities Reference](#utilities)

#### Readable Properties

Property / Method | Type | Description
------------------| ---- | -----------
[`request.socket`](#responsesocket) | [`net.Socket`](https://nodejs.org/api/net.html#net_class_net_socket) | The socket associated with the response
[`request.inspect`](#responseinspect)`()` | `Object` | Not documented
[`request.toJSON`](#responsetojson)`()` | `Object` | Not documented
[`request.writable`](#responsetojson)`()` | `Boolean` | Not documented

### HTTP Control Overview

[Connection Management and HTTP Control Reference](#http-control)

### Content Negotiation Overview

[Content Negotiation Reference](#content-negotiation)

### Response Context Overview

[Response Context Reference](#response-context)

### Security and Privacy Overview

[Security and Privacy Reference](#security-and-privacy)

### Range Responses Overview

[Range Response Reference](#range-responses)

### Authentication Challenge Overview

[Authentication Challenge Reference](#authentication-challenges)

### Cookies Overview

[Cookies Reference](#cookies)

## API Reference

### Response Body

See also [Request Body](request.md#request-body).

#### `response.body`

  Get response body.

#### `response.body = String | Buffer | Stream | Object`

  Set response body to one of the following:

  - `string` written
  - `Buffer` written
  - `Stream` piped
  - `Object` json-stringified
  - `null` no content response

If `response.status` has not been set, Koa will automatically set the status to `200` or `204`.

##### String

  The Content-Type is defaulted to text/html or text/plain, both with
  a default charset of utf-8. The Content-Length field is also set.

##### Buffer

  The Content-Type is defaulted to application/octet-stream, and Content-Length
  is also set.

##### Stream

  The Content-Type is defaulted to application/octet-stream.

  Whenever a stream is set as the response body, `.onerror` is automatically added as a listener to the `error` event to catch any errors.
  In addition, whenever the request is closed (even prematurely), the stream is destroyed.
  If you do not want these two features, do not set the stream as the body directly.
  For example, you may not want this when setting the body as an HTTP stream in a proxy as it would destroy the underlying connection.

  See: https://github.com/koajs/koa/pull/612 for more information.

  Here's an example of stream error handling without automatically destroying the stream:

```js
const PassThrough = require('stream').PassThrough;

app.use(function * (next) {
  ctx.body = someHTTPStream.on('error', ctx.onerror).pipe(PassThrough());
});
```

##### Object

  The Content-Type is defaulted to application/json.

#### `response.length`

  Return response Content-Length as a number when present, or deduce
  from `ctx.body` when possible, or `undefined`.

#### `response.length = Number`

  Set response Content-Length to the given value.

#### `response.type`

  Get response `Content-Type` void of parameters such as "charset".

```js
const ct = ctx.type;
// => "image/png"
```

#### `response.type = String`

  Set response `Content-Type` via mime string or file extension.

```js
ctx.type = 'text/plain; charset=utf-8';
ctx.type = 'image/png';
ctx.type = '.png';
ctx.type = 'png';
```

  Note: when appropriate a `charset` is selected for you, for
  example `response.type = 'html'` will default to "utf-8", however
  when explicitly defined in full as `response.type = 'text/html'`
  no charset is assigned.

#### `response.is(types...)`

  Very similar to `ctx.request.is()`.
  Check whether the response type is one of the supplied types.
  This is particularly useful for creating middleware that
  manipulate responses.

  For example, this is a middleware that minifies
  all HTML responses except for streams.

```js
const minify = require('html-minifier');

app.use(function * minifyHTML(next) {
  yield next;

  if (!ctx.response.is('html')) return;

  let body = ctx.body;
  if (!body || body.pipe) return;

  if (Buffer.isBuffer(body)) body = body.toString();
  ctx.body = minify(body);
});
```
#### `response.attachment([filename])`

  Set `Content-Disposition` to "attachment" to signal the client
  to prompt for download. Optionally specify the `filename` of the
  download.

#### Response Body Headers

and Representation Metadata

Header Field | Specification | Koa Support
------------ | ------------- | -----------
[`Content-Length`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Length) | [RFC 7230 sec 3.3.2](https://tools.ietf.org/html/rfc7230#section-3.3.2) | [body](#responsebody) or [length](#responselength)
[`Content-Type`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Type) | [RFC 7231 sec 3.1.1.5](https://tools.ietf.org/html/rfc7231#section-3.1.1.5) | [body](#responsebody) or [type](#responsetype) or [is](#responseis)
[`Content-Disposition`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Disposition) | [RFC 6286](https://tools.ietf.org/html/rfc6266) | [attachment](#responseattachment)
[`Content-Encoding`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Encoding) | [RFC 7231 sec 3.1.2.2](https://tools.ietf.org/html/rfc7231#section-3.1.2.2) | basic [header accessors](#header-accessors)
[`Content-Language`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Language) | [RFC 7231 sec 3.1.3.2](https://tools.ietf.org/html/rfc7231#section-3.1.3.2) | basic [header accessors](#header-accessors)
[`Content-Location`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Location) | [RFC 7231 sec 3.1.4.2](https://tools.ietf.org/html/rfc7231#section-3.1.4.2) | basic [header accessors](#header-accessors)

See also [Request Body Headers](request.md#request-body-headers).

### Status Line

#### `response.status`

  Get response status. By default, `response.status` is set to `404` unlike node's `res.statusCode` which defaults to `200`.

#### `response.status = Number`

  Set response status via numeric code:

  - 100 "continue"
  - 101 "switching protocols"
  - 102 "processing"
  - 200 "ok"
  - 201 "created"
  - 202 "accepted"
  - 203 "non-authoritative information"
  - 204 "no content"
  - 205 "reset content"
  - 206 "partial content"
  - 207 "multi-status"
  - 300 "multiple choices"
  - 301 "moved permanently"
  - 302 "moved temporarily"
  - 303 "see other"
  - 304 "not modified"
  - 305 "use proxy"
  - 307 "temporary redirect"
  - 400 "bad request"
  - 401 "unauthorized"
  - 402 "payment required"
  - 403 "forbidden"
  - 404 "not found"
  - 405 "method not allowed"
  - 406 "not acceptable"
  - 407 "proxy authentication required"
  - 408 "request time-out"
  - 409 "conflict"
  - 410 "gone"
  - 411 "length required"
  - 412 "precondition failed"
  - 413 "request entity too large"
  - 414 "request-uri too large"
  - 415 "unsupported media type"
  - 416 "requested range not satisfiable"
  - 417 "expectation failed"
  - 418 "i'm a teapot"
  - 422 "unprocessable entity"
  - 423 "locked"
  - 424 "failed dependency"
  - 425 "unordered collection"
  - 426 "upgrade required"
  - 428 "precondition required"
  - 429 "too many requests"
  - 431 "request header fields too large"
  - 500 "internal server error"
  - 501 "not implemented"
  - 502 "bad gateway"
  - 503 "service unavailable"
  - 504 "gateway time-out"
  - 505 "http version not supported"
  - 506 "variant also negotiates"
  - 507 "insufficient storage"
  - 509 "bandwidth limit exceeded"
  - 510 "not extended"
  - 511 "network authentication required"

__NOTE__: don't worry too much about memorizing these strings,
if you have a typo an error will be thrown, displaying this list
so you can make a correction.

#### `response.message`

  Get response status message. By default, `response.message` is
  associated with `response.status`.

#### `response.message = String`

  Set response status message to the given value.

### Header Accessors

See also [Request Header Accessors](request.md#header-accessors).

#### `response.header`

  Response header object.

#### `response.headers`

  Response header object. Alias as `response.header`.

#### `response.headerSent`

  Check if a response header has already been sent. Useful for seeing
  if the client may be notified on error.

#### `response.get(field)`

  Get a response header field value with case-insensitive `field`.

```js
const etag = ctx.get('ETag');
```

#### `response.set(field, value)`

  Set response header `field` to `value`:

```js
ctx.set('Cache-Control', 'no-cache');
```

#### `response.set(fields)`

  Set several response header `fields` with an object:

```js
ctx.set({
  'Etag': '1234',
  'Last-Modified': date
});
```

#### `response.append(field, value)`

  Append additional header `field` with value `val`.

```js
ctx.append('Link', '<http://127.0.0.1/>');
```

#### `response.remove(field)`

  Remove header `field`.

#### `response.flushHeaders()`

  Flush any set headers, and begin the body.

### HTTP Caching

See also [Request HTTP Caching](request.md#http-caching).

#### `response.lastModified`

  Return the `Last-Modified` header as a `Date`, if it exists.

#### `response.lastModified = String | Date`

  Set the `Last-Modified` header as an appropriate UTC string.
  You can either set it as a `Date` or date string.

```js
ctx.response.lastModified = new Date();
```

#### `response.etag = String`

  Set the ETag of a response including the wrapped `"`s.
  Note that there is no corresponding `response.etag` getter.

```js
ctx.response.etag = crypto.createHash('md5').update(ctx.body).digest('hex');
```

#### `response.vary(field)`

  Vary on `field`.

#### HTTP Caching Headers

Header Field | Specification | Koa Support
------------ | ------------- | -----------
[`ETag`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/ETag) | [RFC 7232 sec 2.3](https://tools.ietf.org/html/rfc7232#section-2.3) | [etag](#responseetag)
[`Vary`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Vary) | [RFC 7231 sec 7.1.4](https://tools.ietf.org/html/rfc7231#section-7.1.4) | [vary](#responsevary)
[`Age`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Age) | [RFC 7234 sec 5.1](https://tools.ietf.org/html/rfc7234#section-5.1) | basic [header accessors](#header-accessors)
[`Cache-Control`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Cache-Control) | [RFC 7234 sec 5.2](https://tools.ietf.org/html/rfc7234#section-5.2) | basic [header accessors](#header-accessors)
[`Expires`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Expires) | [RFC 7234 sec 5.3](https://tools.ietf.org/html/rfc7234#section-5.3) | basic [header accessors](#header-accessors)
[`Last-Modified`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Last-Modified) | [RFC 7232 sec 2.2](https://tools.ietf.org/html/rfc7232#section-2.2) | basic [header accessors](#header-accessors)
[`Warning`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Warning) | [RFC 7234 sec 5.5](https://tools.ietf.org/html/rfc7234#section-5.5) | basic [header accessors](#header-accessors)
[`Date`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Date) | [RFC 7231 sec 7.1.1.2](https://tools.ietf.org/html/rfc7231#section-7.1.1.2) | basic [header accessors](#header-accessors)
[`Pragma`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Pragma) | No standard | basic [header accessors](#header-accessors)

See also [Request HTTP Caching Headers](request.md#http-caching-headers).

### Response Routing

See also [Request Routing](request.md#request-routing).

#### `response.redirect(url, [alt])`

  Perform a [302] redirect to `url`.

  The string "back" is special-cased
  to provide Referrer support, when Referrer
  is not present `alt` or "/" is used.

```js
ctx.redirect('back');
ctx.redirect('back', '/index.html');
ctx.redirect('/login');
ctx.redirect('http://google.com');
```

  To alter the default status of `302`, simply assign the status
  before or after this call. To alter the body, assign it after this call:

```js
ctx.status = 301;
ctx.redirect('/cart');
ctx.body = 'Redirecting to shopping cart';
```

#### Response Routing Headers

Header Field | Specification | Koa Support
------------ | ------------- | -----------
[`Location`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Location) | [RFC 7231 sec 7.1.2](https://tools.ietf.org/html/rfc7231#section-7.1.2) | [redirect](#responseredirect) 
[`Via`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Via) | [RFC 7230 sec 5.7.1](https://tools.ietf.org/html/rfc7230#section-5.7.1) | basic [header accessors](#header-accessors)

See also [Request Routing Headers](request.md#request-routing-headers).

### Utilities

#### `response.socket`

  Request socket.

### HTTP Control

See also [Request Connection Management and HTTP Control](request.md#http-control).

#### HTTP Control Headers

Header Field | Specification | Koa Support
------------ | ------------- | -----------
[`Connection`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Connection) | [RFC 7230 sec 6.1](https://tools.ietf.org/html/rfc7230#section-6.1) | basic [header accessors](#header-accessors)
`Upgrade` | [RFC 7230 sec 6.7](https://tools.ietf.org/html/rfc7230#section-6.7) | basic [header accessors](#header-accessors)
[`Retry-After`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Retry-After) | [RFC 7231 sec 7.1.3](https://tools.ietf.org/html/rfc7231#section-7.1.2) | basic [header accessors](#header-accessors)
[`Transfer-Encoding`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Transfer-Encoding) | [RFC 7230 sec 3.3.1](https://tools.ietf.org/html/rfc7230#section-3.3.1) | basic [header accessors](#header-accessors)
[`TE`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/TE) | [RFC 7230 sec 4.3](https://tools.ietf.org/html/rfc7230#section-4.3) | basic [header accessors](#header-accessors)
[`Trailer`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Trailer) | [RFC 7230 sec 4.4](https://tools.ietf.org/html/rfc7230#section-4.4) | basic [header accessors](#header-accessors)

See also [Request Connection Management and HTTP Control Headers](request.md#http-control-headers).

### Content Negotiation

See also [Request Content Negotiation](request.md#content-negotiation).

### Response Context

See also [Request Context](request.md#request-context).

#### Response Context Headers

Header Field | Specification | Koa Support
------------ | ------------- | -----------
[`Allow`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Allow) | [RFC 7231 sec 7.4.1](https://tools.ietf.org/html/rfc7231#section-7.4.1) | basic [header accessors](#header-accessors)
[`Server`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Server) | [RFC 7231 sec 7.4.1](https://tools.ietf.org/html/rfc7231#section-7.4.2) | basic [header accessors](#header-accessors)

See also [Request Context Headers](request.md#request-context-headers).

### Security and Privacy

See also [Request Security and Privacy](request.md#security-and-privacy).

#### Security and Privacy Headers

Header Field | Specification | Koa Support
------------ | ------------- | -----------
[`X-Frame-Options`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Frame-Options) | [RFC 7034](https://tools.ietf.org/html/rfc7034) | basic [header accessors](#header-accessors)
[`Strict-Transport-Security`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Strict-Transport-Security) | [RFC 6797](https://tools.ietf.org/html/rfc6797) | basic [header accessors](#header-accessors)
[`Access-Control-Max-Age`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Access-Control-Max-Age) | [CORS Reccomendation](https://www.w3.org/TR/cors/) | basic [header accessors](#header-accessors)
[`Access-Control-Expose-Headers`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Access-Control-Expose-Headers) | [CORS Reccomendation](https://www.w3.org/TR/cors/) | basic [header accessors](#header-accessors)
[`Access-Control-Allow-Methods`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Access-Control-Allow-Methods) | [CORS Reccomendation](https://www.w3.org/TR/cors/) | basic [header accessors](#header-accessors)
[`Access-Control-Allow-Headers`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Access-Control-Allow-Headers) | [CORS Reccomendation](https://www.w3.org/TR/cors/) | basic [header accessors](#header-accessors)
[`Access-Control-Allow-Credentials`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Access-Control-Allow-Credentials) | [CORS Reccomendation](https://www.w3.org/TR/cors/) | basic [header accessors](#header-accessors)
[`Access-Control-Allow-Origin`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Access-Control-Allow-Origin) | [CORS Reccomendation](https://www.w3.org/TR/cors/) | basic [header accessors](#header-accessors)
[`Content-Security-Policy`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy) | [CSP Draft](https://w3c.github.io/webappsec-csp/) | basic [header accessors](#header-accessors)
[`Content-Security-Policy-Report-Only`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy-Report-Only) | [CSP Draft](https://w3c.github.io/webappsec-csp/) | basic [header accessors](#header-accessors)
[`Public-Key-Pins`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Public-Key-Pins) | [RFC 7469](https://tools.ietf.org/html/rfc7469) | basic [header accessors](#header-accessors)
[`Public-Key-Pins-Report-Only`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Public-Key-Pins-Report-Only) | [RFC 7469](https://tools.ietf.org/html/rfc7469) | basic [header accessors](#header-accessors)
[`X-Content-Type-Options`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Content-Type-Options) | [Fetch Standard](https://fetch.spec.whatwg.org/#x-content-type-options-header) | basic [header accessors](#header-accessors)
[`X-XSS-Protection`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-XSS-Protection) | No standard | basic [header accessors](#header-accessors)

### Range Responses

See also [Range Requests](request.md#range-requests).

#### Range Response Headers

Header Field | Specification | Koa Support
------------ | ------------- | -----------
[`Accept-Ranges`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Accept-Ranges) | [RFC 7233 sec 2.3](https://tools.ietf.org/html/rfc7233#section-2.3) | basic [header accessors](#header-accessors)
[`Content-Range`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Range) | [RFC 7233 4.2](https://tools.ietf.org/html/rfc7233#section-4.2) | basic [header accessors](#header-accessors)

See also [Range Request Headers](request.md#range-request-headers).

### Authentication Challenges

See also [Request Authentication Credentials](request.md#authentication-credentials).

#### Authentication Challenge Headers

Header Field | Specification | Koa Support
------------ | ------------- | -----------
[`WWW-Authenticate`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/WWW-Authenticate) | [RFC 7235 sec 4.1](https://tools.ietf.org/html/rfc7235#section-4.1) | basic [header accessors](#header-accessors)
[`Proxy-Authenticate`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Proxy-Authenticate) | [RFC 7235 sec 4.3](https://tools.ietf.org/html/rfc7235#section-4.3) | basic [header accessors](#header-accessors)

See also [Request Authentication Credential Headers](request.md#authentication-credential-headers).

### Cookies

Support for request cookies is located at [context.Cookies](context#cookies).

#### Cookie Headers

[`Set-Cookie`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Set-Cookie) | [RFC 6265](https://tools.ietf.org/html/rfc6265) | [context.Cookies](context#cookies)

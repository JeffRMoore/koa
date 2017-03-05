# Response

  A Koa `Response` object is an abstraction on top of node's vanilla response object,
  providing additional functionality that is useful for every day HTTP server
  development.

## Overview

### Message Body Overview

#### Readable Properties

Property | Type | Description
---------| ---- | -----------
[body](#responsebody) | mixed | The response body
[length](#responselength) | `Number` | The value of the `Content-Length` header as a number when present
[type](#responsetype) | `String` | The value of the `Content-Type` header void of parameters such as "charset"

#### Writable Properties

Property | Type | Description
---------| ---- | -----------
[body](#responsebody) | `String` or `Buffer` or `Object` or `Stream` | Set the response body
[length](#responselength) | `Number` | Set response `Content-Length` to the given value
[type](#responsetype) | `String` | Set response `Content-Type` header

#### Methods

Method | Return Type | Description
------ | ------- | -----------
[is](#responseis) | `String` or `false` | Check whether the response `Content-Type` matches one of the supplied types.
[attachment](#responseattachment) | `undefined` | Set `Content-Disposition` to "attachment" to signal the client to prompt for download.

### Status Line Overview

#### Readable Properties

Property | Type | Description
---------| ---- | -----------
[status](#responsestuatus) | `Number` | The response status
[message](#response) | `String` | The response status message

#### Writable properties

Property | Type | Description
---------| ---- | -----------
[status](#responsestuatus) | `Number` | Set the response status
[message](#response) | `String` | Set the response status message

### Basic Header Accessor Overview

#### Readable Properties

Property | Type | Description
---------| ---- | -----------
[header](#responseheader) | `Object` | All response headers as an object with header name as the key.
[headers](#responseheaders) | `Object` | Alias of `header`

#### Methods

Method | Return Type | Description
------- | ------- | -----------
[get](#responseget) | `String` | Get a response header value by name
[set](#responseset) | `undefined` | Set a response header value by name (replacing existing header)
[append](#responseappend) | `undefined` | Append additional header.  Like `set` but without replacing.
[remove](#responseremove) | `undefined` | Remove response header by name
[flushHeaders](#responseflushheaders) | `undefined` | Flush any set headers, and begin the body.
[headerSent](#responseheaderssent) | `Boolean` | Check if a response header has already been sent. 

### Cache Header Overview

#### Readable Properties

Property | Type | Description
---------| ---- | -----------
[lastModified](#responselastmodified) | `Date` | The value of the `Last-Modified` header if it exists
[etag](#responseetag) | `String` | The value of the `Etag` header if it exists

#### Writable Properties

Property | Type | Description
---------| ---- | -----------
[lastModified](#responselastmodified) | `String` or `Date` | Set the `Last-Modified` header as an appropriate UTC string
[etag](#responseetag) | `String` | Set the value of the `Etag` header

#### Methods

Method | Return Type | Description
------ | ------- | -----------
[vary](#responsevary) | `undefined` | Issue a `Vary` header

### Response Routing Overview

#### Methods

Method | Return Type | Description
------ | ------- | -----------
[redirect](#responseredirect) | `undefined` | Issue a `Location` header for a 30x redirect.

### Utility Overview

#### Readable Properties

Property | Type | Description
---------| ---- | -----------
[socket](#responsesocket) | [net.Socket](https://nodejs.org/api/net.html#net_class_net_socket) | The socket associated with the response

#### Methods

Method | Type | Description
------ | ---- | -----------
[inspect](#responseinspect) | `Object` | Not documented
[toJSON](#responsetojson) | `Object` | Not documented
[writable](#responsetojson) | `Boolean` | Not documented

## API

### Message Body

#### response.body

  Get response body.

#### response.body = `string` | `Buffer` | `Stream` | `Object`

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

#### response.length

  Return response Content-Length as a number when present, or deduce
  from `ctx.body` when possible, or `undefined`.

#### response.length = `Number`

  Set response Content-Length to the given value.

#### response.type

  Get response `Content-Type` void of parameters such as "charset".

```js
const ct = ctx.type;
// => "image/png"
```

#### response.type = `String`

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

#### response.is(types...)

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
#### response.attachment([filename])

  Set `Content-Disposition` to "attachment" to signal the client
  to prompt for download. Optionally specify the `filename` of the
  download.

#### Message Body and Representation Metadata Header Reference

Header | Specification | Support
------ | ------------- | -------
`Content-Length` | [RFC 7230 sec 3.3.2](https://tools.ietf.org/html/rfc7230#section-3.3.2) | [body](#responsebody) or [length](#responselength)
`Content-Type` | [RFC 7231 sec 3.1.1.5](https://tools.ietf.org/html/rfc7231#section-3.1.1.5) | [body](#responsebody) or [type](#responsetype) or [is](#responseis)
`Transfer-Encoding` | [RFC 7230 sec 3.3.1](https://tools.ietf.org/html/rfc7230#section-3.3.1) | [body](#responsebody) 
`Content-Disposition` | [RFC 6286](https://tools.ietf.org/html/rfc6266) | [attachment](#responseattachment)
`Content-Encoding` | [RFC 7231 sec 3.1.2.2](https://tools.ietf.org/html/rfc7231#section-3.1.2.2) | basic [header accessors](#basicheaderaccessors)
`Content-Language` | [RFC 7231 sec 3.1.3.2](https://tools.ietf.org/html/rfc7231#section-3.1.3.2) | basic [header accessors](#basicheaderaccessors)
`Content-Location` | [RFC 7231 sec 3.1.4.2](https://tools.ietf.org/html/rfc7231#section-3.1.4.2) | basic [header accessors](#basicheaderaccessors)
`TE` | [RFC 7230 sec 4.3](https://tools.ietf.org/html/rfc7230#section-4.3) | basic [header accessors](#basicheaderaccessors)
`Trailer` | [RFC 7230 sec 4.4](https://tools.ietf.org/html/rfc7230#section-4.4) | basic [header accessors](#basicheaderaccessors)

### Status Line

#### response.status

  Get response status. By default, `response.status` is set to `404` unlike node's `res.statusCode` which defaults to `200`.

#### response.status = `Number`

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

#### response.message

  Get response status message. By default, `response.message` is
  associated with `response.status`.

#### response.message = `String`

  Set response status message to the given value.

### Basic Header Accessors

#### response.header

  Response header object.

#### response.headers

  Response header object. Alias as `response.header`.

#### response.headerSent

  Check if a response header has already been sent. Useful for seeing
  if the client may be notified on error.

#### response.get(field)

  Get a response header field value with case-insensitive `field`.

```js
const etag = ctx.get('ETag');
```

#### response.set(field, value)

  Set response header `field` to `value`:

```js
ctx.set('Cache-Control', 'no-cache');
```

#### response.set(fields)

  Set several response header `fields` with an object:

```js
ctx.set({
  'Etag': '1234',
  'Last-Modified': date
});
```

#### response.append(field, value)

  Append additional header `field` with value `val`.

```js
ctx.append('Link', '<http://127.0.0.1/>');
```

#### response.remove(field)

  Remove header `field`.

#### response.flushHeaders()

  Flush any set headers, and begin the body.

### Cache Headers

#### response.lastModified

  Return the `Last-Modified` header as a `Date`, if it exists.

#### response.lastModified = `String` | `Date`

  Set the `Last-Modified` header as an appropriate UTC string.
  You can either set it as a `Date` or date string.

```js
ctx.response.lastModified = new Date();
```

#### response.etag = `String`

  Set the ETag of a response including the wrapped `"`s.
  Note that there is no corresponding `response.etag` getter.

```js
ctx.response.etag = crypto.createHash('md5').update(ctx.body).digest('hex');
```

#### response.vary(field)

  Vary on `field`.

#### Caching Header Reference

Header | Specification | Support
------ | ------------- | -------
`ETag` | [RFC 7232 sec 2.3](https://tools.ietf.org/html/rfc7232#section-2.3) | [etag](#responseetag)
`Vary` | [RFC 7231 sec 7.1.4](https://tools.ietf.org/html/rfc7231#section-7.1.4) | [vary](#responsevary)
`Age` | [RFC 7234 sec 5.1](https://tools.ietf.org/html/rfc7234#section-5.1) | basic [header accessors](#basicheaderaccessors)
`Cache-Control` | [RFC 7234 sec 5.2](https://tools.ietf.org/html/rfc7234#section-5.2) | basic [header accessors](#basicheaderaccessors)
`Expires` | [RFC 7234 sec 5.3](https://tools.ietf.org/html/rfc7234#section-5.3) | basic [header accessors](#basicheaderaccessors)
`Last-Modified` | [RFC 7232 sec 2.2](https://tools.ietf.org/html/rfc7232#section-2.2) | basic [header accessors](#basicheaderaccessors)
`Warning` | [RFC 7234 sec 5.5](https://tools.ietf.org/html/rfc7234#section-5.5) | basic [header accessors](#basicheaderaccessors)
`Date` | [RFC 7231 sec 7.1.1.2](https://tools.ietf.org/html/rfc7231#section-7.1.1.2) | basic [header accessors](#basicheaderaccessors)

### Response Routing

#### response.redirect(url, [alt])

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

#### Message Routing Header Reference

Header | Specification | Support
------ | ------------- | -------
`Location` | [RFC 7231 sec 7.1.2](https://tools.ietf.org/html/rfc7231#section-7.1.2) | [redirect](#responseredirect) 
`Via` | [RFC 7230 sec 5.7.1](https://tools.ietf.org/html/rfc7230#section-5.7.1) | basic [header accessors](#basicheaderaccessors)

### Utility

#### response.socket

  Request socket.

### Connection Management and HTTP Control

#### Connection Management and HTTP Control Header Reference

Header | Specification | Support
------ | ------------- | -------
`Connection` | [RFC 7230 sec 6.1](https://tools.ietf.org/html/rfc7230#section-6.1) | basic [header accessors](#basicheaderaccessors)
`Upgrade` | [RFC 7230 sec 6.7](https://tools.ietf.org/html/rfc7230#section-6.7) | basic [header accessors](#basicheaderaccessors)
`Retry-After` | [RFC 7231 sec 7.1.3](https://tools.ietf.org/html/rfc7231#section-7.1.2) | basic [header accessors](#basicheaderaccessors)

### Content Negotiation

#### Content Negotiation Header Reference

NA for response

### Response Context

### Response Context Header Reference

Header | Specification | Support
------ | ------------- | -------
`Allow` | [RFC 7231 sec 7.4.1](https://tools.ietf.org/html/rfc7231#section-7.4.1) | basic [header accessors](#basicheaderaccessors)
`Server` | [RFC 7231 sec 7.4.1](https://tools.ietf.org/html/rfc7231#section-7.4.2) | basic [header accessors](#basicheaderaccessors)

### Security and Privacy

### Security and Privacy Header Reference

Header | Specification | Support
------ | ------------- | -------
`X-Frame-Options` | [RFC 7034](https://tools.ietf.org/html/rfc7034) | basic [header accessors](#basicheaderaccessors)
`Strict-Transport-Security` | [RFC 6797](https://tools.ietf.org/html/rfc6797) | basic [header accessors](#basicheaderaccessors)

### Range

### Range Header Reference

Header | Specification | Support
------ | ------------- | -------
`Accept-Ranges` | [RFC 7233 sec 2.3](https://tools.ietf.org/html/rfc7233#section-2.3) | basic [header accessors](#basicheaderaccessors)
`Content-Range` | [RFC 7233 4.2](https://tools.ietf.org/html/rfc7233#section-4.2) | basic [header accessors](#basicheaderaccessors)

### Authentication Challenges

### Authentication Challenge Header Reference

Header | Specification | Support
------ | ------------- | -------
`WWW-Authenticate` | [RFC 7235 sec 4.1](https://tools.ietf.org/html/rfc7235#section-4.1) | basic [header accessors](#basicheaderaccessors)
`Proxy-Authenticate` | [RFC 7235 sec 4.3](https://tools.ietf.org/html/rfc7235#section-4.3) | basic [header accessors](#basicheaderaccessors)

### Cookies

### Cookie Header Reference

Header | Specification | Support
------ | ------------- | -------
`Cookie` | [RFC 6265](https://tools.ietf.org/html/rfc6265) | [context.Cookies](context#cookies)

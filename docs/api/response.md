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

Read and set the body of the HTTP response,
as well as basic representation metadata headers.

Property / Method | Type | Description
------------------| ---- | -----------
[`response.body`](#responsebody) | `String`<br> `Buffer`<br> `Object`<br> `Stream`<br> `null` | The response body
[`response.body`](#responsebody)` =` | `String`<br> `Buffer`<br> `Object`<br> `Stream`<br> `null` | Set the response body
[`response.length`](#responselength) | `Number` | The value of the `Content-Length` header as a number when present
[`response.length`](#responselength)` =` | `Number` | Set response `Content-Length` to the given value
[`response.type`](#responsetype) | `String` | The value of the `Content-Type` header void of parameters such as "charset"
[`response.type`](#responsetype)` =` | `String` | Set response `Content-Type` header
[`response.is`](#responseis)`()` | `String`<br> `false` | Check whether the response `Content-Type` matches one of the specified types.
[`response.attachment`](#responseattachment)`()` | `undefined` | Set `Content-Disposition` to `attachment` to signal the client to prompt for download.

### Status Line Overview

[Status Line Reference](#status-line)

Read and set the HTTP response status and proper corresponding status message.

Property / Method | Type | Description
------------------| ---- | -----------
[`response.status`](#responsestuatus) | `Number` | The response status
[`response.status`](#responsestuatus) = | `Number` | Set the response status
[`response.message`](#response) | `String` | The response status message
[`response.message`](#response) = | `String` | Set the response status message

### Header Accessor Overview

[Basic Header Accessor Reference](#header-accessors)

Basic support for emitting and manipulating HTTP header fields.

Property / Method | Type | Description
------------------| ---- | -----------
[`response.header`](#responseheader) | `Object` | All response headers as an object with header name as the key.
[`response.headers`](#responseheaders) | `Object` | Alias of [`header`](#responseheader)
[`response.get`](#responseget)() | `String` |  Get the value of a response header field by the specified name
[`response.set`](#responseset)() | `undefined` | Set a response header field value by field name (replacing existing header)
[`response.append`](#responseappend)() | `undefined` | Append additional header field.  Like [`set`](#responseset) but without replacing
[`response.remove`](#responseremove)() | `undefined` | Remove response header field by field name
[`response.flushHeaders`](#responseflushheaders)() | `undefined` | Flush any headers associated with the response, and begin the body
[`response.headerSent`](#responseheaderssent)() | `Boolean` | Check if a response header field has already been sent 

### HTTP Caching Overview

[HTTP Caching Reference](#http-caching)

Emit basic caching related HTTP headers.

Property / Method | Type | Description
------------------| ---- | -----------
[`response.lastModified`](#responselastmodified) | `Date` | The value of the `Last-Modified` header field if it exists
[`response.lastModified`](#responselastmodified)` =` | `String`<br> `Date` | Set the value of the `Last-Modified` header field as an appropriate UTC string
[`response.etag`](#responseetag) | `String` | The value of the `Etag` header field if it exists
[`response.etag`](#responseetag)` =` | `String` | Set the value of the `Etag` header field
[`response.vary`](#responsevary)`()` | `undefined` | Issue a `Vary` header field

### Response Routing Overview

[Response Routing Reference](#response-routing)

Property / Method | Type | Description
------------------| ---- | -----------
[`response.redirect`](#responseredirect)`()` | `undefined` | Issue a `Location` header for a 30x redirect.

### Utilities Overview

[Utilities Reference](#utilities)

Property / Method | Type | Description
------------------| ---- | -----------
[`response.ctx`](#responsectx) | [`Context`](context.md) | The context object that this response is associated with
[`response.res`](#responseres) | [`http.ServerResponse`](#https://nodejs.org/api/http.html#http_class_http_serverresponse) | The node `ServerResponse` object managed by this response object
[`response.socket`](#responsesocket) | [`net.Socket`](https://nodejs.org/api/net.html#net_class_net_socket) | The socket associated with the response
[`response.inspect`](#responseinspect)`()` | `Object` | Not documented
[`response.toJSON`](#responsetojson)`()` | `Object` | Not documented
[`response.writable`](#responsetojson)`()` | `Boolean` | Not documented

## API Reference

### Response Body

See also [Request Body](request.md#request-body).

Koa provides basic support for generating the body of the HTTP response,
as well support for basic representation metadata headers for the body.
The Koa API helps middleware to coordinate changes to the body and supports
the node Stream and Buffer data types.

#### `response.body`

  Get response body.

  An alias is available on the Context object for `response.body` allowing
  `ctx.response.body` to be abbreviated as `ctx.body`.

#### `response.body = String | Buffer | Stream | Object| null`

  Set response body to one of the following types:

  - `string` will be written to the body
  - `Buffer` will be written to the body
  - `Stream` will be piped to the body
  - `Object` will be json-stringified and written to the body
  - `null` indicates no content in the body of the response

If [`response.status`](#responsestatus) has not been set, Koa will automatically set the status to `200` or `204`.
_When is it 200 and when is it 204?_

_What happens if a Number or Boolean is used?_

_When does writing begin?_

  An alias is available on the Context object for `response.body` allowing
  `ctx.response.body` to be abbreviated as `ctx.body`.

##### String

  The value of the `Content-Type` header field is defaulted to `text/html` or `text/plain`, both with
  a default charset of `utf-8`. The `Content-Length` header field is also set
  to the length of the body string.

##### Buffer

  The `Content-Type` header field is defaulted to `application/octet-stream`, and the 
  `Content-Length` header field is also set.

##### Stream

  The `Content-Type` header field is defaulted to `application/octet-stream`.

  Whenever a stream is set as the response body, `.onerror` is automatically added as a listener to the `error` event to catch any errors.
  In addition, whenever the request is closed (even prematurely), the stream is destroyed.
  If you do not want these two features, do not set the stream as the body directly.
  For example, you may not want this when setting the body as an HTTP stream in a proxy as it would destroy the underlying connection.

  See: https://github.com/koajs/koa/pull/612 for more information.

  Here's an example of stream error handling without automatically destroying the stream:

```js
const PassThrough = require('stream').PassThrough;

app.use(function * (next) {
  ctx.response.body = someHTTPStream.on('error', ctx.onerror).pipe(PassThrough());
});
```

##### Object

  The `Content-Type` header field is defaulted to `application/json` and
  the object is serialized as a json string and written to the body.

#### `response.length`

  Return response `Content-Length` as a number when present, or deduce
  from `ctx.response.body` when possible, or `undefined`.

  An alias is available on the Context object for `response.length` allowing
  `ctx.response.length` to be abbreviated as `ctx.length`.

#### `response.length = Number`

  Set response `Content-Length` to the given value.

  An alias is available on the Context object for `response.length` allowing
  `ctx.response.length` to be abbreviated as `ctx.length`.

#### `response.type`

  Get response `Content-Type` void of parameters such as "charset".

```js
const ct = ctx.response.type;
// => "image/png"
```

  An alias is available on the Context object for `response.type` allowing
  `ctx.response.type` to be abbreviated as `ctx.type`.

#### `response.type = String`

  Set response `Content-Type` via mime string or file extension.

```js
ctx.response.type = 'text/plain; charset=utf-8';
ctx.response.type = 'image/png';
ctx.response.type = '.png';
ctx.response.type = 'png';
```

  Note: when appropriate a `charset` is selected for you, for
  example `response.type = 'html'` will default to "utf-8", however
  when explicitly defined in full as `response.type = 'text/html'`
  no charset is assigned.
  
  Powered by [content-type](https://github.com/jshttp/content-type)

  An alias is available on the Context object for `response.type` allowing
  `ctx.response.type` to be abbreviated as `ctx.type`.

#### `response.is(types...)`

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

  let body = ctx.response.body;
  if (!body || body.pipe) return;

  if (Buffer.isBuffer(body)) body = body.toString();
  ctx.response.body = minify(body);
});
```
  Very similar to [`request.is`](request.md#requestis).
  Powered by [type-is](https://github.com/jshttp/type-is).

#### `response.attachment([filename])`

  Set `Content-Disposition` to "attachment" to signal the client
  to prompt for download. Optionally specify the `filename` of the
  download.
  Powered by [content-disposition](https://github.com/jshttp/content-disposition).
  
  An alias is available on the Context object for `response.attachment` allowing
  `ctx.response.attachment` to be abbreviated as `ctx.attachment`.

#### Response Body Headers

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

Koa has support for setting the HTTP response status and message and
determining the proper message to correspond to a given status.

#### `response.status`

  Get response status. By default, `response.status` is set to `404` unlike node's `res.statusCode` which defaults to `200`.
  
  An alias is available on the Context object for `response.status` allowing
  `ctx.response.status` to be abbreviated as `ctx.status`.

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

  An alias is available on the Context object for `response.status` allowing
  `ctx.response.status` to be abbreviated as `ctx.status`.

#### `response.message`

  Get response status message. By default, `response.message` is
  associated with [`response.status`](#response.status).

  An alias is available on the Context object for `response.message` allowing
  `ctx.response.message` to be abbreviated as `ctx.message`.

#### `response.message = String`

  Set response status message to the given value.

  An alias is available on the Context object for `response.message` allowing
  `ctx.response.message` to be abbreviated as `ctx.message`.

### Header Accessors

See also [Request Header Accessors](request.md#header-accessors).

Basic methods for accessing the request header and the unparsed value of request
header fields.  There is support for adding and removing headers from the response.

Headers are buffered until the entire header for the response is written.  _After that?_

#### `response.header`

  Response header object.

#### `response.headers`

  Response header object. Alias as `response.header`.

#### `response.headerSent`

  Check if a response header has already been sent. Useful for seeing
  if the client may be notified on error.

  An alias is available on the Context object for `response.headerSent` allowing
  `ctx.response.headerSent` to be abbreviated as `ctx.headerSent`.

#### `response.get(field)`

  Get a response header field value with case-insensitive `field`.

```js
const etag = ctx.respnse.get('ETag');
```

#### `response.set(field, value)`

  Set response header `field` to `value`:

```js
ctx.response.set('Cache-Control', 'no-cache');
```

  An alias is available on the Context object for `response.set` allowing
  `ctx.response.set` to be abbreviated as `ctx.set`.

#### `response.set(fields)`

  Set several response header `fields` with an object:

```js
ctx.response.set({
  'Etag': '1234',
  'Last-Modified': date
});
```

#### `response.append(field, value)`

  Append additional header `field` with value `value`.

```js
ctx.response.append('Link', '<http://127.0.0.1/>');
```

  An alias is available on the Context object for `response.append` allowing
  `ctx.response.append` to be abbreviated as `ctx.append`.

#### `response.remove(field)`

  Remove header `field`.

  An alias is available on the Context object for `response.remove` allowing
  `ctx.response.remove` to be abbreviated as `ctx.remove`.

#### `response.flushHeaders()`

  Flush any set headers, and begin the body.

  An alias is available on the Context object for `response.flushHeaders` allowing
  `ctx.response.flushHeaders` to be abbreviated as `ctx.flushHeaders`.

### HTTP Caching

See also [Request HTTP Caching](request.md#http-caching).

Koa has basic support for issuing response headers related to caching.
For more comprehensive support, check [jshttp](https://jshttp.github.io) or [npm](https://www.npmjs.com) for
lower level support, or the [Koa middleware community](https://github.com/koajs/koa/wiki).

#### `response.lastModified`

  Return the `Last-Modified` header as a `Date`, if it exists.

  An alias is available on the Context object for `response.lastModified` allowing
  `ctx.response.lastModified` to be abbreviated as `ctx.lastModified`.

#### `response.lastModified = String | Date`

  Set the `Last-Modified` header as an appropriate UTC string.
  You can either set it as a `Date` or date string.

```js
ctx.response.lastModified = new Date();
```

  An alias is available on the Context object for `response.lastModified` allowing
  `ctx.response.lastModified` to be abbreviated as `ctx.lastModified`.

#### `response.etag`

NOT DOCUMENTED

  An alias is available on the Context object for `response.etag` allowing
  `ctx.response.etag` to be abbreviated as `ctx.etag`.

#### `response.etag = String`

  Set the `ETag` header field of a response including the wrapped `"`s.

```js
ctx.response.etag = crypto.createHash('md5').update(ctx.response.body).digest('hex');
```

  An alias is available on the Context object for `response.etag` allowing
  `ctx.response.etag` to be abbreviated as `ctx.etag`.

#### `response.vary(field)`

  Vary on `field`.  Powered by [vary](https://github.com/jshttp/vary).
  
  An alias is available on the Context object for `response.vary` allowing
  `ctx.response.vary` to be abbreviated as `ctx.vary`.


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

  Perform a `302` redirect to `url`.

  The string `"back"` is special-cased
  to provide support for the `Referrer` header field.
  When `Referrer` is not present, `alt` will be used and if
  `alt` is not specified, `"/"` is used.

```js
ctx.response.redirect('back');
ctx.response.redirect('back', '/index.html');
ctx.response.redirect('/login');
ctx.response.redirect('http://google.com');
```

  To alter the default status of `302`, simply assign the status
  before or after this call. To alter the body, assign it after this call:

```js
ctx.response.status = 301;
ctx.response.redirect('/cart');
ctx.response.body = 'Redirecting to shopping cart';
```

  An alias is available on the Context object for `response.redirect` allowing
  `ctx.response.redirect` to be abbreviated as `ctx.redirect`.

#### Response Routing Headers

Header Field | Specification | Koa Support
------------ | ------------- | -----------
[`Location`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Location) | [RFC 7231 sec 7.1.2](https://tools.ietf.org/html/rfc7231#section-7.1.2) | [redirect](#responseredirect) 
[`Via`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Via) | [RFC 7230 sec 5.7.1](https://tools.ietf.org/html/rfc7230#section-5.7.1) | basic [header accessors](#header-accessors)

See also [Request Routing Headers](request.md#request-routing-headers).

### Utilities

#### `request.ctx`

  MISSING DOCUMENTATION

#### `request.res`

  MISSING DOCUMENTATION

#### `response.socket`

  Request socket.

#### `inspect`

  MISSING DOCUMENTATION

#### `toJSON`

  MISSING DOCUMENTATION

#### `response.writable`

  MISSING DOCUMENTATION

  An alias is available on the Context object for `response.writable` allowing
  `ctx.response.writable` to be abbreviated as `ctx.writable`.

### HTTP Control

See also [Request Connection Management and HTTP Control](request.md#http-control).

HTTP Control headers are usually handled by node's [http](https://nodejs.org/api/http.html) module.

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

### Response Context

See also [Request Context](request.md#request-context).

Koa does not bundle specific support for response context headers.
Check [jshttp](https://jshttp.github.io) or [npm](https://www.npmjs.com) for
lower level support, or the [Koa middleware community](https://github.com/koajs/koa/wiki).

#### Response Context Headers

Header Field | Specification | Koa Support
------------ | ------------- | -----------
[`Allow`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Allow) | [RFC 7231 sec 7.4.1](https://tools.ietf.org/html/rfc7231#section-7.4.1) | basic [header accessors](#header-accessors)
[`Server`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Server) | [RFC 7231 sec 7.4.1](https://tools.ietf.org/html/rfc7231#section-7.4.2) | basic [header accessors](#header-accessors)

See also [Request Context Headers](request.md#request-context-headers).

### Security and Privacy

See also [Request Security and Privacy](request.md#security-and-privacy).

Koa does not bundle specific support for HTTP security features such as CORS, 
Strict Transport Security, Content Security Policy, etc.
Check [jshttp](https://jshttp.github.io) or [npm](https://www.npmjs.com) for
lower level support, or the [Koa middleware community](https://github.com/koajs/koa/wiki).

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

Koa does not bundle support for range requests, check [jshttp](https://jshttp.github.io) or [npm](https://www.npmjs.com) for
lower level support, or the [Koa middleware community](https://github.com/koajs/koa/wiki).

#### Range Response Headers

Header Field | Specification | Koa Support
------------ | ------------- | -----------
[`Accept-Ranges`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Accept-Ranges) | [RFC 7233 sec 2.3](https://tools.ietf.org/html/rfc7233#section-2.3) | basic [header accessors](#header-accessors)
[`Content-Range`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Range) | [RFC 7233 4.2](https://tools.ietf.org/html/rfc7233#section-4.2) | basic [header accessors](#header-accessors)

See also [Range Request Headers](request.md#range-request-headers).

### Authentication Challenges

See also [Request Authentication Credentials](request.md#authentication-credentials).

Koa does not bundle support for [HTTP authentication](https://developer.mozilla.org/en-US/docs/Web/HTTP/Authentication), check [jshttp](https://jshttp.github.io) or [npm](https://www.npmjs.com) for
lower level support, or the [Koa middleware community](https://github.com/koajs/koa/wiki).

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

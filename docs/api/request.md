# Request

  A Koa `Request` object is an abstraction on top of node's vanilla request object,
  providing additional functionality that is useful for every day HTTP server
  development.

## Overview

[Request Method](#request-method-overview) •
[Request Target](#request-target-overview) •
[Header Accessors](#header-accessor-overview) •
[HTTP Caching](#http-caching-overview) •
[Content Negotiation](#content-negotiation-overview) •
[Request Body](#request-body-overview) •
[Request Context](#request-context-overview) •
[Utility](#utility-overview) •
[HTTP Control](#http-control) •
[Security and Privacy](#security-and-privacy) •
[Range Requests](#range-requests) •
[Authentication Credentials](#authentication-credentials) •
[Cookies](#cookies)

### Request Method Overview

[Request Method Reference](#request-method)

Access the HTTP method from the [request line](https://tools.ietf.org/html/rfc7230#section-3.1.1).

Property / Method | Type | Description
------------------| ---- | -----------
[`request.method`](#requestmethod) | `String` | The request's HTTP method (e.g. `GET` or `POST`)
[`request.method`](#requestmethod)` = ` | `String` | Set the request's HTTP method
[`request.idempotent`](#requestidempotent) | `Boolean` | Is the request method idempotent?

### Request Target Overview

[Request Target Reference](#request-target)

Access the request target from the [request line](https://tools.ietf.org/html/rfc7230#section-3.1.1)
as well as the `Host` header and the protocol used for the request.
This functionality is used for url construction and routing.

Property / Method | Type | Description
------------------| ---- | -----------
[`request.protocol`](#requestprotocol) | `String` | The request protocol, (e.g. `"https"` or `"http"`)
[`request.secure`](#requestsecure) | `Boolean` | `true` if the request was issued via TLS
[`request.host`](#requesthost) | `String` | The value of the `Host` header
[`request.hostname`](#requesthostname)| `String` | The [`host`](#requesthost) without the port
[`request.subdomains`](#requestsubdomains) | `Array` | The [`hostname`](#requesthostname), split into subdomains 
[`request.url`](#requesturl) | `String` | The request URL (full request target)
[`request.url`](#requesturl)`= ` | `String` | Set the full request url, including query parameters
[`request.originalUrl`](#requestoriginalurl) | `String` | The value of [`url`](#requesturl) prior to any modifications of the request
[`request.path`](#requestpath) | `String` | The requested path
[`request.path`](#requestpath)` =` | ` String` | Set request path
[`request.search`](#requestsearch) | `String` | The raw query string including the `?`
[`request.search`](#requestsearch)` =` | `String` | Set raw query string with (must include `?`)
[`request.querystring`](#requestquerystring) | `String` | The raw query string void of `?`
[`request.querystring`](#requestquerystring)`= ` | `String` | Set raw query string (do not include `?`)
[`request.query`](#requestquery) | `Object` | The parsed query string
[`request.query`](#requestquery)`= ` | `Object` | Use the given object to set the [`querystring`](#querystring)
[`request.origin`](#requestorigin) | `String` | The [`protocol`](#requestprotocol) and the [`host`](#requesthost) in `protocol:\\host` format
[`request.href`](#requesthref) | `String` | The full [`url`](#requesturl) prefixed by [`origin`](#requestorigin)

### Header Accessor Overview

[Header Accessors Reference](#header-accessors)

General methods and properties for accessing request headers.

Property / Method | Type | Description
------------------| ---- | -----------
[`request.header`](#requestheader) | `Object` | All request headers as an object with header field name as the key.
[`request.headers`](#requestheaders) | `Object` | An alias of [`header`](#requestheader)
[`request.get`](#requestget)() | `String` | Get the value of a header field by the specified name

### HTTP Caching Overview

[HTTP Caching Reference](#http-caching)

Property / Method | Type | Description
------------------| ---- | -----------
[`request.fresh`](#requestfresh) | `Boolean` | Check if a request cache is "fresh"
[`request.stale`](#requeststale) | `Boolean` | Inverse of `request.fresh`

### Content Negotiation Overview

[Content Negotiation Reference](#content-negotiation)

Property / Method | Type | Description
------------------| ---- | -----------
[`request.accepts`](#requestaccepts)`()` | `Array`<br> `String`<br> `false` | Check if the specified MIME types are acceptable to the client
[`request.acceptsEncodings`](#requestacceptsencodings)`()` | `Array`<br> `String` | Check if the specified encodings are acceptable to the client
[`request.acceptsCharsets`](#requestacceptscharsets)`()` | `Array`<br> `String` | Check if character sets are acceptable to the client
[`request.acceptsLanguages`](#requestacceptslanguages)`()` | `Array`<br> `String` | Check if specified languages are acceptable to the client

### Request Body Overview

[Request Body Reference](#request-body)

Access the [request body](https://tools.ietf.org/html/rfc7230#section-3.3) and
`Content-Length` headers.

Property / Method | Type | Description
------------------| ---- | -----------
[`request.type`](#requesttype) | `String` | The `Content-Type` header field void of parameters such as `"charset"`
[`request.charset`](#requestcharset) | `String` | The charset from the `Content-Type` header field when present
[`request.length`](#requestlength) | `Number` | Return the value of the `Content-Length` header field as a number when present
[`request.is`](#requestis)`()` | `Boolean` |Check if the incoming request contains the `Content-Type` header field, and it contains any of the given MIME types. 

### Request Context Overview

[Request Context Reference](#request-context)

Property / Method | Type | Description
------------------| ---- | -----------
[`request.ip`](#requestip) | `String` |  The remote IP address of the request accounting for proxies via the `X-Forwarded-For` header field 
[`request.ips`](#requestips) | `Array` | The list of IPs from the `X-Forwarded-For` header field

### Utilities Overview

[Utilities Reference](#utilities)

Property / Method | Type | Description
------------------| ---- | -----------
[`request.ctx`](#requestctx) | [`Context`](context.md) | The context object that this request is associated with
[`request.req`](#requestreq) | [`http.IncomingMessage`](https://nodejs.org/api/http.html#http_class_http_incomingmessage) | The node `IncomingMessage` object managed by this request object
[`request.socket`](#requestsocket) | [`net.Socket`](https://nodejs.org/api/net.html#net_class_net_socket) | The socket associated with the request
[`request.inspect`](#requestinspect)`()`| `Object` | Not documented
[`request.toJSON`](#requesttojson)`()` | `Object` | Not documented

## API Reference

### Request Method

Access the http method from the [request line](https://tools.ietf.org/html/rfc7230#section-3.1.1).

#### request.method

  Request method.

  An alias is available on the Context object for `request.method` allowing
  `ctx.request.method` to be abbreviated as `ctx.method`.

#### request.method = `String`

  Set request method, useful for implementing middleware
  such as `methodOverride()`.

  An alias is available on the Context object for `request.method` allowing
  `ctx.request.method` to be abbreviated as `ctx.method`.

#### request.idempotent

  Check if the request is idempotent.

  An alias is available on the Context object for `request.idempotent` allowing
  `ctx.request.idempotent` to be abbreviated as `ctx.idempotent`.

### Request Target

Access the request target from the [request line](https://tools.ietf.org/html/rfc7230#section-3.1.1)
as well as the `Host` header and the protocol used for the request.
This functionality is used for url construction and routing.

Koa does not bundle support for routing requests beased on the request target.
Check [jshttp](https://jshttp.github.io) or [npm](https://www.npmjs.com) for
lower level support, or the [Koa middleware community](https://github.com/koajs/koa/wiki).

#### `request.protocol`

  Return request protocol, "https" or "http". Supports `X-Forwarded-Proto`
  when `app.proxy` is __true__.

  An alias is available on the Context object for `request.protocol` allowing
  `ctx.request.protocol` to be abbreviated as `ctx.protocol`.

#### `request.secure`

  Shorthand for `ctx.request.protocol == "https"` to check if a request was
  issued via TLS.

  An alias is available on the Context object for `request.secure` allowing
  `ctx.request.secure` to be abbreviated as `ctx.secure`.

#### `request.host`

  Get host (hostname:port) when present. Supports `X-Forwarded-Host`
  when `app.proxy` is __true__, otherwise `Host` is used.

  An alias is available on the Context object for `request.host` allowing
  `ctx.request.host` to be abbreviated as `ctx.host`.

#### `request.hostname`

  Get hostname when present. Supports `X-Forwarded-Host`
  when `app.proxy` is __true__, otherwise `Host` is used.

  An alias is available on the Context object for `request.hostname` allowing
  `ctx.request.hostname` to be abbreviated as `ctx.hostname`.

#### `request.subdomains`

  Return subdomains as an array.

  Subdomains are the dot-separated parts of the host before the main domain of
  the app. By default, the domain of the app is assumed to be the last two
  parts of the host. This can be changed by setting `app.subdomainOffset`.

  For example, if the domain is `"tobi.ferrets.example.com"`:
  If `app.subdomainOffset` is not set, `ctx.request.subdomains` is `["ferrets", "tobi"]`.
  If `app.subdomainOffset` is 3, `ctx.request.subdomains` is `["tobi"]`.

  An alias is available on the Context object for `request.subdomains` allowing
  `ctx.request.subdomains` to be abbreviated as `ctx.subdomains`.

#### `request.url`

  Get request URL.

  An alias is available on the Context object for `request.url` allowing
  `ctx.request.url` to be abbreviated as `ctx.url`.

#### `request.url = String`

  Set request URL, useful for url rewrites.

  An alias is available on the Context object for `request.url` allowing
  `ctx.request.url` to be abbreviated as `ctx.url`.

#### `request.originalUrl`

  Get request original URL.

#### `request.path`

  Get request pathname.

  An alias is available on the Context object for `request.path` allowing
  `ctx.request.path` to be abbreviated as `ctx.path`.

#### `request.path = String`

  Set request pathname and retain query-string when present.

  An alias is available on the Context object for `request.path` allowing
  `ctx.request.path` to be abbreviated as `ctx.path`.

#### `request.search`

  Get raw query string with the `?`.

  An alias is available on the Context object for `request.search` allowing
  `ctx.request.search` to be abbreviated as `ctx.search`.

#### `request.search = String`

  Set raw query string.

  An alias is available on the Context object for `request.search` allowing
  `ctx.request.search` to be abbreviated as `ctx.search`.

#### `request.querystring`

  Get raw query string void of `?`.

  An alias is available on the Context object for `request.querystring` allowing
  `ctx.request.querystring` to be abbreviated as `ctx.querystring`.

#### `request.querystring = String`

  Set raw query string.

  An alias is available on the Context object for `request.querystring` allowing
  `ctx.request.querystring` to be abbreviated as `ctx.querystring`.

#### `request.query`

  Get parsed query-string, returning an empty object when no
  query-string is present. Note that this getter does _not_
  support nested parsing.

  For example `"color=blue&size=small"`:

```js
{
  color: 'blue',
  size: 'small'
}
```

  An alias is available on the Context object for `request.query` allowing
  `ctx.request.query` to be abbreviated as `ctx.query`.

#### `request.query = Object`

  Set query-string to the given object. Note that this
  setter does _not_ support nested objects.

```js
ctx.request.query = { next: '/login' }
```

  An alias is available on the Context object for `request.query` allowing
  `ctx.request.query` to be abbreviated as `ctx.query`.

#### `request.origin`

  Get origin of URL, include `protocol` and `host`. Unrelated to the CORS 'Origin' header.

```js
ctx.request.origin
// => http://example.com
```

  An alias is available on the Context object for `request.origin` allowing
  `ctx.request.origin` to be abbreviated as `ctx.origin`.

#### `request.href`

  Get full request URL, include `protocol`, `host` and `url`.

```js
ctx.request.href
// => http://example.com/foo/bar?q=1
```

  An alias is available on the Context object for `request.href` allowing
  `ctx.request.href` to be abbreviated as `ctx.href`.

### Request Routing Headers

Header Field | Specification | Koa Support
------------ | ------------- | -----------
[`Host`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Host) | [RFC 7230 sec 5.4](https://tools.ietf.org/html/rfc7230#section-5.4) | [host](#requesthost)
[`X-Forwarded-Host`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Forwarded-Host) | de facto standard | [host](#requesthost)
[`X-Forwarded-Proto`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Forwarded-Proto) | de facto standard | [protocol](#requestprotocol)
[`X-Forwarded-For`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Forwarded-For) | de facto standard | [ips](#requestips)
[`Forwarded`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Forwarded) | [RFC 7239](https://tools.ietf.org/html/rfc7239) | basic [header accessors](#header-accessors)
`Max-Forwards` | [RFC 7231 sec 5.1.2](https://tools.ietf.org/html/rfc7231#section-5.1.2) | basic [header accessors](#header-accessors)
[`Via`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Via) | [RFC 7230 sec 5.7.1](https://tools.ietf.org/html/rfc7230#section-5.7.1) | basic [header accessors](#header-accessors)

See also [Response Routing Headers](response.md#response-routing-headers)

### Header Accessors

See also [Response Header Accessors](response.md#http-caching-headers)

Basic methods for accessing the request header and the unparsed value of request
header fields.

#### `request.header`

 Request header object.

  An alias is available on the Context object for `request.header` allowing
  `ctx.request.header` to be abbreviated as `ctx.header`.

#### `request.headers`

 Request header object. Alias of [`request.header`](#requestheader).

  An alias is available on the Context object for `request.headers` allowing
  `ctx.request.headers` to be abbreviated as `ctx.headers`.

#### `request.get(field)`

  Return request header.

  An alias is available on the Context object for `request.get` allowing
  `ctx.request.get` to be abbreviated as `ctx.get`.

### HTTP Caching

See also [Response HTTP Caching](response.md#http-caching)

Koa has basic support for checking conditional headers related to caching.
For more comprehensive support, check [jshttp](https://jshttp.github.io) or [npm](https://www.npmjs.com) for
lower level support, or the [Koa middleware community](https://github.com/koajs/koa/wiki).

#### `request.fresh`

  Check if a request cache is "fresh", aka the contents have not changed. This
  method is for cache negotiation between `If-None-Match` / `ETag`, and `If-Modified-Since` and `Last-Modified`. It should be referenced after setting one or more of these response headers.

```js
// freshness check requires status 20x or 304
ctx.response.status = 200;
ctx.response.set('ETag', '123');

// cache is ok
if (ctx.request.fresh) {
  ctx.response.status = 304;
  return;
}

// cache is stale
// fetch new data
ctx.response.body = yield db.find('something');
```

Powered by [Fresh](https://github.com/jshttp/fresh).

  An alias is available on the Context object for `request.fresh` allowing
  `ctx.request.fresh` to be abbreviated as `ctx.fresh`.

#### `request.stale`

  Inverse of `request.fresh`.

  An alias is available on the Context object for `request.stale` allowing
  `ctx.request.stale` to be abbreviated as `ctx.stale`.

#### HTTP Caching Headers

Header Field | Specification | Koa Support
------------ | ------------- | -----------
[`If-None-Match`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/If-None-Match)  |  [RFC 7232 sec 3.2](https://tools.ietf.org/html/rfc7232#section-3.2) | [fresh](#requestfresh) or [stale](#requeststale)
[`If-Modified-Since`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/If-Modified-Since)  |  [RFC 7232 sec 3.3](https://tools.ietf.org/html/rfc7232#section-3.3) | [fresh](#requestfresh) or [stale](#requeststale)
[`Cache-Control`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Cache-Control) | [RFC 7234 sec 5.2](https://tools.ietf.org/html/rfc7234#section-5.2) | basic [header accessors](#header-accessors)
[`If-Match`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/If-Match) | [RFC 7232 sec 3.1](https://tools.ietf.org/html/rfc7232#section-3.1) | basic [header accessors](#header-accessors)
[`If-Unmodified-Since`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/If-Unmodified-Since)  |  [RFC 7232 sec 3.4](https://tools.ietf.org/html/rfc7232#section-3.4) | basic [header accessors](#header-accessors)
[`Pragma`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Pragma) | [RFC 7434 Sec 5.4](https://tools.ietf.org/html/rfc7234#section-5.3) | basic [header accessors](#header-accessors)

See also [Response HTTP Caching Headers](response.md#http-caching-headers)

### Content Negotiation

  Koa's `request` object includes helpful content negotiation utilities powered by [accepts](http://github.com/expressjs/accepts) and [negotiator](https://github.com/federomero/negotiator). These utilities are:

- `request.accepts(types)`
- `request.acceptsEncodings(types)`
- `request.acceptsCharsets(charsets)`
- `request.acceptsLanguages(langs)`

If no types are supplied, __all__ acceptable types are returned.

If multiple types are supplied, the best match will be returned. If no matches are found, a `false` is returned, and you should send a `406 "Not Acceptable"` response to the client.

In the case of missing accept headers where any type is acceptable, the first type will be returned. Thus, the order of types you supply is important.

#### `request.accepts(types)`

  Check if the given `type(s)` is acceptable, returning the best match when true, otherwise `false`. The `type` value may be one or more mime type string
  such as `"application/json"`, the extension name
  such as `"json"`, or an array `["json", "html", "text/plain"]`.

```js
// Accept: text/html
ctx.request.accepts('html');
// => "html"

// Accept: text/*, application/json
ctx.request.accepts('html');
// => "html"
ctx.request.accepts('text/html');
// => "text/html"
ctx.request.accepts('json', 'text');
// => "json"
ctx.request.accepts('application/json');
// => "application/json"

// Accept: text/*, application/json
ctx.request.accepts('image/png');
ctx.request.accepts('png');
// => false

// Accept: text/*;q=.5, application/json
ctx.request.accepts(['html', 'json']);
ctx.request.accepts('html', 'json');
// => "json"

// No Accept header
ctx.request.accepts('html', 'json');
// => "html"
ctx.request.accepts('json', 'html');
// => "json"
```

  You may call `ctx.reqeust.accepts()` as many times as you like,
  or use a switch:

```js
switch (ctx.request.accepts('json', 'html', 'text')) {
  case 'json': break;
  case 'html': break;
  case 'text': break;
  default: ctx.throw(406, 'json, html, or text only');
}
```

  An alias is available on the Context object for `request.accepts` allowing
  `ctx.request.accepts` to be abbreviated as `ctx.accepts`.

#### `request.acceptsEncodings(encodings)`

  Check if `encodings` are acceptable, returning the best match when true, otherwise `false`. Note that you should include `identity` as one of the encodings!

```js
// Accept-Encoding: gzip
ctx.request.acceptsEncodings('gzip', 'deflate', 'identity');
// => "gzip"

ctx.request.acceptsEncodings(['gzip', 'deflate', 'identity']);
// => "gzip"
```

  When no arguments are given all accepted encodings
  are returned as an array:

```js
// Accept-Encoding: gzip, deflate
ctx.request.acceptsEncodings();
// => ["gzip", "deflate", "identity"]
```

  Note that the `identity` encoding (which means no encoding) could be unacceptable if the client explicitly sends `identity;q=0`. Although this is an edge case, you should still handle the case where this method returns `false`.

  An alias is available on the Context object for `request.acceptsEncodings` allowing
  `ctx.request.acceptsEncodings` to be abbreviated as `ctx.acceptsEncodings`.

#### `request.acceptsCharsets(charsets)`

  Check if `charsets` are acceptable, returning
  the best match when true, otherwise `false`.

```js
// Accept-Charset: utf-8, iso-8859-1;q=0.2, utf-7;q=0.5
ctx.request.acceptsCharsets('utf-8', 'utf-7');
// => "utf-8"

ctx.request.acceptsCharsets(['utf-7', 'utf-8']);
// => "utf-8"
```

  When no arguments are given all accepted charsets
  are returned as an array:

```js
// Accept-Charset: utf-8, iso-8859-1;q=0.2, utf-7;q=0.5
ctx.request.acceptsCharsets();
// => ["utf-8", "utf-7", "iso-8859-1"]
```

  An alias is available on the Context object for `request.acceptsCharsets` allowing
  `ctx.request.acceptsCharsets` to be abbreviated as `ctx.acceptsCharsets`.

#### `request.acceptsLanguages(langs)`

  Check if `langs` are acceptable, returning
  the best match when true, otherwise `false`.

```js
// Accept-Language: en;q=0.8, es, pt
ctx.request.acceptsLanguages('es', 'en');
// => "es"

ctx.request.acceptsLanguages(['en', 'es']);
// => "es"
```

  When no arguments are given all accepted languages
  are returned as an array:

```js
// Accept-Language: en;q=0.8, es, pt
ctx.request.acceptsLanguages();
// => ["es", "pt", "en"]
```

  An alias is available on the Context object for `request.acceptsLanguages` allowing
  `ctx.request.acceptsLanguages` to be abbreviated as `ctx.acceptsLanguages`.

### Content Negotiation Headers

Header Field | Specification | Koa Support
------------ | ------------- | -----------
[`Accept`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Accept) | [RFC 7231 sec 5.3.2](https://tools.ietf.org/html/rfc7231#section-5.3.2) | [accepts](#requestaccepts)
[`Accept-Charset`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Accept-Charset) | [RFC 7231 sec 5.3.3](https://tools.ietf.org/html/rfc7231#section-5.3.3) | [acceptsCharsets](#requestacceptscharsets)
[`Accept-Encoding`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Accept-Encoding) | [RFC 7231 sec 5.3.4](https://tools.ietf.org/html/rfc7231#section-5.3.4) | [acceptsEncodings](#requestacceptsencodings)
[`Accept-Language`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Accept-Language) | [RFC 7231 sec 5.3.5](https://tools.ietf.org/html/rfc7231#section-5.3.5) | [acceptsLanguages](#requestacceptslanguages)

### Request Body

See also [Response Body](response.md#response-body)

Koa has support for setting representation metadata headers, but does not bundle support
for parsing the body of a request, such as for submitted forms or REST API
payloads.  check [jshttp](https://jshttp.github.io) or [npm](https://www.npmjs.com) for
lower level support, or the [Koa middleware community](https://github.com/koajs/koa/wiki).

#### `request.length`

  Return request `Content-Length` as a number when present, or `undefined`.

#### `request.type`

  Get request `Content-Type` void of parameters such as `"charset"`.

```js
const ct = ctx.request.type
// => "image/png"
```

#### `request.charset`

  Get request charset when present, or `undefined`:

```js
ctx.request.charset
// => "utf-8"
```

#### `request.is(types...)`

  Check if the incoming request contains the `Content-Type`
  header field, and it contains any of the give mime `types`.
  If there is no request body, `null` is returned.
  If there is no content type, or the match fails `false` is returned.
  Otherwise, it returns the matching content-type.

```js
// With Content-Type: text/html; charset=utf-8
ctx.request.is('html'); // => 'html'
ctx.request.is('text/html'); // => 'text/html'
ctx.request.is('text/*', 'text/html'); // => 'text/html'

// When Content-Type is application/json
ctx.request.is('json', 'urlencoded'); // => 'json'
ctx.request.is('application/json'); // => 'application/json'
ctx.request.is('html', 'application/*'); // => 'application/json'

ctx.request.is('html'); // => false
```

  For example if you want to ensure that
  only images are sent to a given route:

```js
if (ctx.request.is('image/*')) {
  // process
} else {
  ctx.throw(415, 'images only!');
}
```

  An alias is available on the Context object for `request.get` allowing
  `ctx.request.get` to be abbreviated as `ctx.get`.

#### Request Body Headers
 
Header Field | Specification | Koa Support
------------ | ------------- | -----------
[`Content-Length`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Length) | [RFC 7230 sec 3.3.2](https://tools.ietf.org/html/rfc7230#section-3.3.2) | [body](#responsebody) or [length](#responselength)
[`Content-Type`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Type) | [RFC 7231 sec 3.1.1.5](https://tools.ietf.org/html/rfc7231#section-3.1.1.5) | [body](#responsebody) or [type](#responsetype) or [is](#responseis)
[`Content-Encoding`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Encoding) | [RFC 7231 sec 3.1.2.2](https://tools.ietf.org/html/rfc7231#section-3.1.2.2) | basic [header accessors](#header-accessors)
[`Content-Language`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Language) | [RFC 7231 sec 3.1.3.2](https://tools.ietf.org/html/rfc7231#section-3.1.3.2) | basic [header accessors](#header-accessors)
[`Content-Location`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Location) | [RFC 7231 sec 3.1.4.2](https://tools.ietf.org/html/rfc7231#section-3.1.4.2) | basic [header accessors](#header-accessors)

See also [Response Body and Representation Metadata Headers](response.md#response-body-headers)

### Request Context

See also [Response Context](response.md#response-context)

Request context describes the issuer of the request.

#### `request.ip`

  Request remote address. Supports `X-Forwarded-For` when `app.proxy`
  is __true__.

  An alias is available on the Context object for `request.ip` allowing
  `ctx.request.ip` to be abbreviated as `ctx.ip`.

#### `request.ips`

  When `X-Forwarded-For` is present and `app.proxy` is enabled an array
  of these ips is returned, ordered from upstream -> downstream. When disabled
  an empty array is returned.

  An alias is available on the Context object for `request.ips` allowing
  `ctx.request.ips` to be abbreviated as `ctx.ips`.

#### Request Context Headers

Header Field | Specification | Koa Support
------------ | ------------- | -----------
[`Referer`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Referer) | [RFC 7231 sec 5.5.2](https://tools.ietf.org/html/rfc7231#section-5.5.2) | [response.redirect](response#redirect)
[`User-Agent`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/User-Agent) | [RFC 7231 sec 5.5.3](https://tools.ietf.org/html/rfc7231#section-5.5.3) | basic [header accessors](#header-accessors)
[`From`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/From) | [RFC 7231 sec 5.5.1](https://tools.ietf.org/html/rfc7231#section-5.5.1) | basic [header accessors](#header-accessors)

See also [Response Context Headers](response.md#response-context-headers)

### Utilities

#### `request.ctx`

  MISSING DOCUMENTATION

#### `request.req`

  MISSING DOCUMENTATION

#### `request.socket`

  Return the request socket.

  An alias is available on the Context object for `request.socket` allowing
  `ctx.request.socket` to be abbreviated as `ctx.socket`.

#### `inspect()`

MISSING DOCUMENTATION

#### `toJSON()`

MISSING DOCUMENTATION

### HTTP Control

See also [Response Connection Management and HTTP Controls](response.md#http-control).

HTTP Control headers are usually handled by node's [http](https://nodejs.org/api/http.html) module.

#### HTTP Control Headers

Header Field | Specification | Koa Support
------------ | ------------- | -----------
[`Connection`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Connection) | [RFC 7230 sec 6.1](https://tools.ietf.org/html/rfc7230#section-6.1) | basic [header accessors](#header-accessors)
`Upgrade` | [RFC 7230 sec 6.7](https://tools.ietf.org/html/rfc7230#section-6.7) | basic [header accessors](#header-accessors)
[`Expect`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Expect) | [RFC 7231 sec 5.1.1](https://tools.ietf.org/html/rfc7231#section-5.1.1) | basic [header accessors](#header-accessors)
[`Transfer-Encoding`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Transfer-Encoding) | [RFC 7230 sec 3.3.1](https://tools.ietf.org/html/rfc7230#section-3.3.1) | basic [header accessors](#header-accessors)
[`TE`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/TE) | [RFC 7230 sec 4.3](https://tools.ietf.org/html/rfc7230#section-4.3) | basic [header accessors](#header-accessors)
[`Trailer`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Trailer) | [RFC 7230 sec 4.4](https://tools.ietf.org/html/rfc7230#section-4.4) | basic [header accessors](#header-accessors)

See also [Response HTTP Control Headers](response.md#http-control-headers).

### Security and Privacy

See [Response Security and Privacy](response.md#security-and-privacy)

Koa does not bundle support for CORS or upgrading SSL requests, check [jshttp](https://jshttp.github.io) or [npm](https://www.npmjs.com) for
lower level support, or the [Koa middleware community](https://github.com/koajs/koa/wiki).

#### Security and Privacy Headers

Header Field | Specification | Koa Support
------------ | ------------- | -----------
[`Origin`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Origin) | [CORS Reccomendation](https://www.w3.org/TR/cors/) | basic [header accessors](#header-accessors)
[`Access-Control-Request-Method`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Access-Control-Request-Method) | [CORS Reccomendation](https://www.w3.org/TR/cors/) | basic [header accessors](#header-accessors) 
[`Access-Control-Request-Headers`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Access-Control-Request-Headers) | [CORS Reccomendation](https://www.w3.org/TR/cors/) | basic [header accessors](#header-accessors)
[`Upgrade-Insecure-Requests`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Upgrade-Insecure-Requests) | [Candidate Recommendation](https://w3c.github.io/webappsec-upgrade-insecure-requests/) | basic [header accessors](#header-accessors)

See also [Response Security and Privacy Headers](response.md#security-and-privacy-headers)

### Range Requests

See also [Range Responses](response.md#range-responses)

Koa does not bundle support for range requests, check [jshttp](https://jshttp.github.io) or [npm](https://www.npmjs.com) for
lower level support, or the [Koa middleware community](https://github.com/koajs/koa/wiki).

#### Range Request Headers

Header Field | Specification | Koa Support
------------ | ------------- | -----------
[`Range`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Range) | [RFC 7233 Sec 3.1](https://tools.ietf.org/html/rfc7233#section-3.1) | basic [header accessors](#header-accessors)
[`If-Range`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/If-Range)  |  [RFC 7233 sec 3.2](https://tools.ietf.org/html/rfc7233#section-3.2) | basic [header accessors](#header-accessors)
[`Content-Range`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Range) | [RFC 7233 4.2](https://tools.ietf.org/html/rfc7233#section-4.2) | basic [header accessors](#header-accessors)

See also [Range Response Headers](response.md#range-response-headers)

### Authentication Credentials

See also [Response Authentication Challenges](response.md#authentication-challenges)

Koa does not bundle support for [HTTP authentication](https://developer.mozilla.org/en-US/docs/Web/HTTP/Authentication), check [jshttp](https://jshttp.github.io) or [npm](https://www.npmjs.com) for
lower level support, or the [Koa middleware community](https://github.com/koajs/koa/wiki).

#### Authentication Credential Headers

Header Field | Specification | Koa Support
------------ | ------------- | -----------
[`Authorization`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Authorization) | [RFC 7235 sec 4.2](https://tools.ietf.org/html/rfc7235#section-4.2) | basic [header accessors](#header-accessors)
[`Proxy-Authorization`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Proxy-Authenticate) | [RFC 7235 sec 4.4](https://tools.ietf.org/html/rfc7235#section-4.4) | basic [header accessors](#header-accessors)

See also [Response Authentication Challenge Headers](response.md#authentication-challenge-headers).

### Cookies

Support for request cookies is located on the context object and documentated at [context.Cookies](context#cookies).

#### Cookie Headers

Header Field | Specification | Koa Support
------------ | ------------- | -----------
[`Cookie`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Cookie) | [RFC 6265](https://tools.ietf.org/html/rfc6265) | [context.Cookies](context#cookies)

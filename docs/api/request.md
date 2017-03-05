# Request

  A Koa `Request` object is an abstraction on top of node's vanilla request object,
  providing additional functionality that is useful for every day HTTP server
  development.

## Overview

### Request Method Overview

Access the http method from the [request line](https://tools.ietf.org/html/rfc7230#section-3.1.1).

#### Readable Properties

Property | Type | Description
---------| ---- | -----------
[method](#requestmethod) | `String` | The request's HTTP method (e.g. GET or POST)
[idempotent](#requestidempotent) | `Boolean` | Is the request method idempotent?

#### Writable Properties

Property | Type | Description
---------| ---- | -----------
[method](#requestmethod) | `String` | Set the request's HTTP method

### Request Target Overview

Access the request target from the [request line](https://tools.ietf.org/html/rfc7230#section-3.1.1)
as well as the `Host` header and the protocol used for the request.
This functionality is used for url construction and routing.

#### Readable Properties

Property | Type | Description
--------- | -------- | -----------
[protocol](#requestprotocol) | `String` | The request protocol, "https" or "http".
[secure](#requestsecure) | `Boolean` | `true` if the request was issued via TLS
[host](#requesthost) | `String` | The value of the `Host` header.
[hostname](#requesthostname) | `String` | The `host` without the port.
[subdomains](#requestsubdomains) | `Array` | The `hostname`, split into subdomains 
[url](#requesturl) | `String` | The request URL (full request target)
[originalUrl](#requestoriginalurl) | `String` | The value of `url` prior to any request modifications
[path](#requestpath) | `String` | The requested path
[search](#requestsearch) | `String` | The raw query string with the `?`
[querystring](#requestquerystring) | `String` | The raw query string void of `?`
[query](#requestquery) | `Object` | The parsed query string
[origin](#requestorigin) | `String` | The `protocol` and the `host` in `protocol:\\host` format
[href](#requesthref) | `String` | The full `url`, prefixed by `origin`

#### writable Properties

Property | Type | Description
-------- | ------- | -----------
[url](#requesturl) | `String` | Set the full request url, including query parameters
[path](#requestpath) | `String` | Set request path
[search](#requestsearch) | `String` | Set raw query string with (must include '?')
[querystring](#requestquerystring) | `String` | Set raw query string (do not include '?')
[query](#requestquery) | `String` | Set query-string to the given object

### Basic Header Accessor Overview

General methods and properties for accesing request headers.

#### Readable Properties

Property | Type | Description
-------- | ---- | -----------
[header](#requestheader) | `Object` | All request headers as an object with header name as the key.
[headers](#requestheaders) | `Object` | Alias of `header`

#### Methods

Method | Return Type | Description
------- | ------- | -----------
[get](#requestget) | `String` | Get a header value by name

### Caching Overview

#### Readable Properties

Property | Type | Description
-------- | ---- | -----------
[fresh](#requestfresh) | `Boolean` | Check if a request cache is "fresh"
[stale](#requeststale) | `Boolean` | Inverse of `request.fresh`

### Content Negotiation Overview

#### Methods

Method | Return Type | Description
-------| ----------- | -----------
[accepts](#requestaccepts) | `Array` or `String` or `false` | Check if the given MIME types are acceptable
[acceptsEncodings](#requestacceptsencodings) | `Array` or `String` | Check if `encodings` are acceptable
[acceptsCharsets](#requestacceptscharsets) | `Array` or `String` | Check if `charsets` are acceptable
[acceptsLanguages](#requestacceptslanguages) | `Array` or `String` | Check if `langs` are acceptable

### Request Body Overview

Access the [request body](https://tools.ietf.org/html/rfc7230#section-3.3) and
`Content-Length` headers.

#### Readable Properties

Property | Type | Description
-------- | ---- | -----------
[type](#requesttype) | `String` | The `Content-Type` header void of parameters such as "charset"
[charset](#requestcharset) | `String` | The charset from the `Content-Type` header when present, or `undefined`
[length](#requestlength) | `Number` | Return `Content-Length` header as a number when present, or `undefined`

#### Methods

Method | Type | Description
------ | ---- | -----------
[is](#requestis) | `Boolean` |Check if the incoming request contains the `Content-Type` header field, and it contains any of the given MIME types. 

### Request Context

Property | Type | Description
---------| ---- | -----------
[ip](#requestip) | `String` |  Request remote address. Supports `X-Forwarded-For`
[ips](#requestips) | `Array` | The list of IPs from the `X-Forwarded-For` header  

### Utility

#### Readable Properties

Property | Type | Description
---------| ---- | -----------
[socket](#requestsocket) | [net.Socket](https://nodejs.org/api/net.html#net_class_net_socket) | The socket associated with the request

#### Methods

Method | Type | Description
------ | ---- | -----------
[inspect](#requestinspect) | `Object` | Not documented
[toJSON](#requesttojson) | `Object` | Not documented

## API

### Request Method

Access the http method from the [request line](https://tools.ietf.org/html/rfc7230#section-3.1.1).

#### request.method

  Request method.

#### request.method = `String`

  Set request method, useful for implementing middleware
  such as `methodOverride()`.

#### request.idempotent

  Check if the request is idempotent.

### Request Target

Access the request target from the [request line](https://tools.ietf.org/html/rfc7230#section-3.1.1)
as well as the `Host` header and the protocol used for the request.
This functionality is used for url construction and routing.

#### request.protocol

  Return request protocol, "https" or "http". Supports `X-Forwarded-Proto`
  when `app.proxy` is __true__.

#### request.secure

  Shorthand for `ctx.protocol == "https"` to check if a request was
  issued via TLS.

#### request.host

  Get host (hostname:port) when present. Supports `X-Forwarded-Host`
  when `app.proxy` is __true__, otherwise `Host` is used.

#### request.hostname

  Get hostname when present. Supports `X-Forwarded-Host`
  when `app.proxy` is __true__, otherwise `Host` is used.

#### request.subdomains

  Return subdomains as an array.

  Subdomains are the dot-separated parts of the host before the main domain of
  the app. By default, the domain of the app is assumed to be the last two
  parts of the host. This can be changed by setting `app.subdomainOffset`.

  For example, if the domain is "tobi.ferrets.example.com":
  If `app.subdomainOffset` is not set, `ctx.subdomains` is `["ferrets", "tobi"]`.
  If `app.subdomainOffset` is 3, `ctx.subdomains` is `["tobi"]`.

#### request.url

  Get request URL.

#### request.url = `String`

  Set request URL, useful for url rewrites.

#### request.originalUrl

  Get request original URL.

#### request.path

  Get request pathname.

#### request.path = `String`

  Set request pathname and retain query-string when present.

#### request.search

  Get raw query string with the `?`.

#### request.search = `String`

  Set raw query string.

#### request.querystring

  Get raw query string void of `?`.

#### request.querystring = `String`

  Set raw query string.

#### request.query

  Get parsed query-string, returning an empty object when no
  query-string is present. Note that this getter does _not_
  support nested parsing.

  For example "color=blue&size=small":

```js
{
  color: 'blue',
  size: 'small'
}
```

#### request.query = `Object`

  Set query-string to the given object. Note that this
  setter does _not_ support nested objects.

```js
ctx.query = { next: '/login' }
```

#### request.origin

  Get origin of URL, include `protocol` and `host`.

```js
ctx.request.origin
// => http://example.com
```

#### request.href

  Get full request URL, include `protocol`, `host` and `url`.

```js
ctx.request.href
// => http://example.com/foo/bar?q=1
```

### Message Routing Header Reference

Header | Specification | Support
------ | ------------- | -------
`Host` | [RFC 7230 sec 5.4](https://tools.ietf.org/html/rfc7230#section-5.4) | [host](#requesthost)
`X-Forwarded-Host` | de facto standard | [host](#requesthost)
`X-Forwarded-Proto` | de facto standard | [protocol](#requestprotocol)
`X-Forwarded-For` | de facto standard | [ips](#requestips)
`Forwarded` | [RFC 7239](https://tools.ietf.org/html/rfc7239) | basic [header accessors](#basicheaderaccessors)
`Max-Forwards` | [RFC 7231 sec 5.1.2](https://tools.ietf.org/html/rfc7231#section-5.1.2) | basic [header accessors](#basicheaderaccessors)
`Via` | [RFC 7230 sec 5.7.1](https://tools.ietf.org/html/rfc7230#section-5.7.1) | basic [header accessors](#basicheaderaccessors)

### Basic Header Accessors

#### request.header

 Request header object.

#### request.headers

 Request header object. Alias as `request.header`.

#### request.get(field)

  Return request header.

### Caching

#### request.fresh

  Check if a request cache is "fresh", aka the contents have not changed. This
  method is for cache negotiation between `If-None-Match` / `ETag`, and `If-Modified-Since` and `Last-Modified`. It should be referenced after setting one or more of these response headers.

```js
// freshness check requires status 20x or 304
ctx.status = 200;
ctx.set('ETag', '123');

// cache is ok
if (ctx.fresh) {
  ctx.status = 304;
  return;
}

// cache is stale
// fetch new data
ctx.body = yield db.find('something');
```

#### request.stale

  Inverse of `request.fresh`.

#### Caching Header Reference

Header | Specification | Support
------ | ------------- | -------
`If-None-Match`  |  [RFC 7232 sec 3.2](https://tools.ietf.org/html/rfc7232#section-3.2) | [fresh](#requestfresh) or [stale](#requeststale)
`If-Modified-Since`  |  [RFC 7232 sec 3.3](https://tools.ietf.org/html/rfc7232#section-3.3) | [fresh](#requestfresh) or [stale](#requeststale)
`Cache-Control` | [RFC 7234 sec 5.2](https://tools.ietf.org/html/rfc7234#section-5.2) | [fresh](#requestfresh) or [stale](#requeststale)
`If-Match` | [RFC 7232 sec 3.1](https://tools.ietf.org/html/rfc7232#section-3.1) | basic [header accessors](#basicheaderaccessors)
`If-Unmodified-Since`  |  [RFC 7232 sec 3.4](https://tools.ietf.org/html/rfc7232#section-3.4) | basic [header accessors](#basicheaderaccessors)
`Pragma` | [RFC 7434 Sec 5.4](https://tools.ietf.org/html/rfc7234#section-5.3) | basic [header accessors](#basicheaderaccessors)

### Content Negotiation

  Koa's `request` object includes helpful content negotiation utilities powered by [accepts](http://github.com/expressjs/accepts) and [negotiator](https://github.com/federomero/negotiator). These utilities are:

- `request.accepts(types)`
- `request.acceptsEncodings(types)`
- `request.acceptsCharsets(charsets)`
- `request.acceptsLanguages(langs)`

If no types are supplied, __all__ acceptable types are returned.

If multiple types are supplied, the best match will be returned. If no matches are found, a `false` is returned, and you should send a `406 "Not Acceptable"` response to the client.

In the case of missing accept headers where any type is acceptable, the first type will be returned. Thus, the order of types you supply is important.

#### request.accepts(types)

  Check if the given `type(s)` is acceptable, returning the best match when true, otherwise `false`. The `type` value may be one or more mime type string
  such as "application/json", the extension name
  such as "json", or an array `["json", "html", "text/plain"]`.

```js
// Accept: text/html
ctx.accepts('html');
// => "html"

// Accept: text/*, application/json
ctx.accepts('html');
// => "html"
ctx.accepts('text/html');
// => "text/html"
ctx.accepts('json', 'text');
// => "json"
ctx.accepts('application/json');
// => "application/json"

// Accept: text/*, application/json
ctx.accepts('image/png');
ctx.accepts('png');
// => false

// Accept: text/*;q=.5, application/json
ctx.accepts(['html', 'json']);
ctx.accepts('html', 'json');
// => "json"

// No Accept header
ctx.accepts('html', 'json');
// => "html"
ctx.accepts('json', 'html');
// => "json"
```

  You may call `ctx.accepts()` as many times as you like,
  or use a switch:

```js
switch (ctx.accepts('json', 'html', 'text')) {
  case 'json': break;
  case 'html': break;
  case 'text': break;
  default: ctx.throw(406, 'json, html, or text only');
}
```

#### request.acceptsEncodings(encodings)

  Check if `encodings` are acceptable, returning the best match when true, otherwise `false`. Note that you should include `identity` as one of the encodings!

```js
// Accept-Encoding: gzip
ctx.acceptsEncodings('gzip', 'deflate', 'identity');
// => "gzip"

ctx.acceptsEncodings(['gzip', 'deflate', 'identity']);
// => "gzip"
```

  When no arguments are given all accepted encodings
  are returned as an array:

```js
// Accept-Encoding: gzip, deflate
ctx.acceptsEncodings();
// => ["gzip", "deflate", "identity"]
```

  Note that the `identity` encoding (which means no encoding) could be unacceptable if the client explicitly sends `identity;q=0`. Although this is an edge case, you should still handle the case where this method returns `false`.

#### request.acceptsCharsets(charsets)

  Check if `charsets` are acceptable, returning
  the best match when true, otherwise `false`.

```js
// Accept-Charset: utf-8, iso-8859-1;q=0.2, utf-7;q=0.5
ctx.acceptsCharsets('utf-8', 'utf-7');
// => "utf-8"

ctx.acceptsCharsets(['utf-7', 'utf-8']);
// => "utf-8"
```

  When no arguments are given all accepted charsets
  are returned as an array:

```js
// Accept-Charset: utf-8, iso-8859-1;q=0.2, utf-7;q=0.5
ctx.acceptsCharsets();
// => ["utf-8", "utf-7", "iso-8859-1"]
```

#### request.acceptsLanguages(langs)

  Check if `langs` are acceptable, returning
  the best match when true, otherwise `false`.

```js
// Accept-Language: en;q=0.8, es, pt
ctx.acceptsLanguages('es', 'en');
// => "es"

ctx.acceptsLanguages(['en', 'es']);
// => "es"
```

  When no arguments are given all accepted languages
  are returned as an array:

```js
// Accept-Language: en;q=0.8, es, pt
ctx.acceptsLanguages();
// => ["es", "pt", "en"]
```

### Content Negotiation Header Reference

Header | Specification | Support
------ | ------------- | -------
`Accept` | [RFC 7231 sec 5.3.2](https://tools.ietf.org/html/rfc7231#section-5.3.2) | [accepts](#requestaccepts)
`Accept-Charset` | [RFC 7231 sec 5.3.3](https://tools.ietf.org/html/rfc7231#section-5.3.3) | [acceptsCharsets](#requestacceptscharsets)
`Accept-Encoding` | [RFC 7231 sec 5.3.4](https://tools.ietf.org/html/rfc7231#section-5.3.4) | [acceptsEncodings](#requestacceptsencodings)
`Accept-Language` | [RFC 7231 sec 5.3.5](https://tools.ietf.org/html/rfc7231#section-5.3.5) | [acceptsLanguages](#requestacceptslanguages)

### Request Body

#### request.length

  Return request Content-Length as a number when present, or `undefined`.

#### request.type

  Get request `Content-Type` void of parameters such as "charset".

```js
const ct = ctx.request.type
// => "image/png"
```

#### request.charset

  Get request charset when present, or `undefined`:

```js
ctx.request.charset
// => "utf-8"
```

#### request.is(types...)

  Check if the incoming request contains the "Content-Type"
  header field, and it contains any of the give mime `type`s.
  If there is no request body, `null` is returned.
  If there is no content type, or the match fails `false` is returned.
  Otherwise, it returns the matching content-type.

```js
// With Content-Type: text/html; charset=utf-8
ctx.is('html'); // => 'html'
ctx.is('text/html'); // => 'text/html'
ctx.is('text/*', 'text/html'); // => 'text/html'

// When Content-Type is application/json
ctx.is('json', 'urlencoded'); // => 'json'
ctx.is('application/json'); // => 'application/json'
ctx.is('html', 'application/*'); // => 'application/json'

ctx.is('html'); // => false
```

  For example if you want to ensure that
  only images are sent to a given route:

```js
if (ctx.is('image/*')) {
  // process
} else {
  ctx.throw(415, 'images only!');
}
```

#### Message Body and Representation Metadata Header Reference

Header | Specification | Support
------ | ------------- | -------
`Content-Length` | [RFC 7230 sec 3.3.2](https://tools.ietf.org/html/rfc7230#section-3.3.2) | [length](#requestlength) 
`Content-Type` | [RFC 7231 sec 3.1.1.5](https://tools.ietf.org/html/rfc7231#section-3.1.1.5) | [type](#requesttype), [charset](#requestcharset), or [is](#requestis) 
`Content-Encoding` | [RFC 7231 sec 3.1.2.2](https://tools.ietf.org/html/rfc7231#section-3.1.2.2) | basic [header accessors](#basicheaderaccessors)
`Content-Language` | [RFC 7231 sec 3.1.3.2](https://tools.ietf.org/html/rfc7231#section-3.1.3.2) | basic [header accessors](#basicheaderaccessors)
`Content-Location` | [RFC 7231 sec 3.1.4.2](https://tools.ietf.org/html/rfc7231#section-3.1.4.2) | basic [header accessors](#basicheaderaccessors)
`Transfer-Encoding` | [RFC 7230 sec 3.3.1](https://tools.ietf.org/html/rfc7230#section-3.3.1) | basic [header accessors](#basicheaderaccessors)
`TE` | [RFC 7230 sec 4.3](https://tools.ietf.org/html/rfc7230#section-4.3) | basic [header accessors](#basicheaderaccessors)
`Trailer` | [RFC 7230 sec 4.4](https://tools.ietf.org/html/rfc7230#section-4.4) | basic [header accessors](#basicheaderaccessors)

### Request Context

#### request.ip

  Request remote address. Supports `X-Forwarded-For` when `app.proxy`
  is __true__.

#### request.ips

  When `X-Forwarded-For` is present and `app.proxy` is enabled an array
  of these ips is returned, ordered from upstream -> downstream. When disabled
  an empty array is returned.

#### Request Context Headers

Header | Specification | Support
------ | ------------- | -------
`Referer` | [RFC 7231 sec 5.5.2](https://tools.ietf.org/html/rfc7231#section-5.5.2) | [response.redirect](response#redirect)
`User-Agent` | [RFC 7231 sec 5.5.3](https://tools.ietf.org/html/rfc7231#section-5.5.3) | basic [header accessors](#basicheaderaccessors)
`From` | [RFC 7231 sec 5.5.1](https://tools.ietf.org/html/rfc7231#section-5.5.1) | basic [header accessors](#basicheaderaccessors)

### Utility

#### request.socket

  Return the request socket.

#### inspect()

#### toJSON()

### Connection Management and HTTP Control Headers

Header | Specification | Support
------ | ------------- | -------
`Connection` | [RFC 7230 sec 6.1](https://tools.ietf.org/html/rfc7230#section-6.1) | basic [header accessors](#basicheaderaccessors)
`Upgrade` | [RFC 7230 sec 6.7](https://tools.ietf.org/html/rfc7230#section-6.7) | basic [header accessors](#basicheaderaccessors)
`Expect` | [RFC 7231 sec 5.1.1](https://tools.ietf.org/html/rfc7231#section-5.1.1) | basic [header accessors](#basicheaderaccessors)

### Security and Privacy Headers

NA for requests.

### Range Headers

Header | Specification | Support
------ | ------------- | -------
`Range` | [RFC 7233 Sec 3.1](https://tools.ietf.org/html/rfc7233#section-3.1) | basic [header accessors](#basicheaderaccessors)
`If-Range`  |  [RFC 7233 sec 3.2](https://tools.ietf.org/html/rfc7233#section-3.2) | basic [header accessors](#basicheaderaccessors)
`Content-Range` | [RFC 7233 4.2](https://tools.ietf.org/html/rfc7233#section-4.2) | basic [header accessors](#basicheaderaccessors)

### Authentication Credential Headers

Header | Specification | Support
------ | ------------- | -------
`Authorization` | [RFC 7235 sec 4.2](https://tools.ietf.org/html/rfc7235#section-4.2) | basic [header accessors](#basicheaderaccessors)
`Proxy-Authorization` | [RFC 7235 sec 4.4](https://tools.ietf.org/html/rfc7235#section-4.4) | basic [header accessors](#basicheaderaccessors)

### Cookie Header

Header | Specification | Support
------ | ------------- | -------
`Cookie` | [RFC 6265](https://tools.ietf.org/html/rfc6265) | [context.Cookies](context#cookies)

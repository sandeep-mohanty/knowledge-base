# HTTP caching, a refresher · Dan Cătălin Burzo

This is a reading of [RFC 9111](https://www.rfc-editor.org/rfc/rfc9111) (2022), the latest iteration of the HTTP Caching standard.

It defines the [`Cache-Control`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Cache-Control) HTTP header as a way to prescribe how caches should store and reuse HTTP responses, with regards to not just the browser cache, but to any other intermediary caches, such as proxies and content delivery networks, that may exist between the client and the origin server.

![A crude illustration depicting a browser with its private cache, two intermediary services with their shared caches, and the origin server](https://danburzo.ro/img/http-caching-refresher/cache-chain.png)

The `Cache-Control` header accepts a set of comma-separated directives, some of which are meant to be added to HTTP requests, and others to HTTP responses. A typical response header:

```
HTTP/2 200
Cache-Control: max-age=0, must-revalidate
```

Some of these directives specifically target _shared caches_, that is intermediary caches that serve the same cached responses to many users, while others also apply to _private caches_ such as the browser cache.

## What’s fresh?

Whenever the cache receives a request, it must figure out if the cached response is still **fresh** and can therefore be reused without incurring the performance tax of an HTTP request, or whether it has gone **stale** and should be validated with the server.

To decide on freshness, the cache compares the age of the response to the response’s so-called freshness timeline.

The age of a cached response is the time elapsed since it was last generated or revalidated by the origin server. To the time spent in its own cache, the browser will add any [`Age: <seconds>`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Age) header received from intermediary caches.

The freshness timeline is a duration beyond which the cached response is to be considered stale. It’s usually signaled by the server via the appropriate response headers, but may also be guesstimated by the cache in the absence of explicit, valid cues. In order of precedence:

-   the server establishes a freshness timeline, in seconds, with the [`Cache-Control: max-age=<number>`](#max-age-response) directive on the response; otherwise,
-   the cache falls back to computing the interval between the `Expires: <date>` and `Date: <date>` response headers, if available; otherwise,
-   if there’s no `Expires` header, the response lacks an explicit expiration, and a heuristic freshness based on the `Last-Modified` response header might be applicable.

For shared caches, the special [`s-maxage=<number>`](#s-maxage-response) directive takes precedence over all others.

## Going past expiration

Just because a response has gone stale, it doesn’t mean it needs to be thrown out.

When it receives a request for a stale cached response, the cache should validate it with its upstream server. Although validation always generates an HTTP request, it avoids a data transfer when there’s no newer version of the cached response on the server, so it can still be faster than a regular request.

Validation uses a mechanism known as a conditional HTTP request, which includes one or more special headers called preconditions:

-   if the precondition with the highest precedence is met, the server responds with [HTTP 200 OK](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Status/200) and an updated response body; otherwise,
-   it responds with [HTTP 304 Not Modified](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Status/304) and an empty body, confirming the existing response can be reused.

To generate the preconditions needed for these conditional requests, which the server uses to compare the cached response to the freshest version available, responses must be tagged in a way that’s unique to each version:

-   historically, this was done with the [`Last-Modified: <date>`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Last-Modified) header, corresponding to the latest update to the content;
-   a more flexible and robust alternative is the [`ETag: "<value>"`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/ETag) header, which stores an arbitrary ASCII string that uniquely identifies the response. This string is usually a hash incorporating one or more aspects: the modification time, the file size, and the file content.

When performing the validation, the cached response headers are mirrored as preconditions for the conditional request:

-   `Last-Modified: <date>` becomes [`If-Modified-Since: <date>`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/If-Modified-Since);
-   `ETag: "<value>"` becomes [`If-None-Match: "<value>"`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/If-None-Match).

When both preconditions are present, only `If-None-Match` is evaluated.

Regardless of the result of the validation request, the cached response headers are updated with the new values received from the server, and the fresh-o-meter on the cached response is reset.

Certain caches may be set up to serve stale responses in some circumstances, such as when losing the connection to the server or in the event of an HTTP 5xx server error. There are also `Cache-Control` directives that influence how stale responses are handled, covered in the next section.

## Cache-Control response directives

### 

`max-age=<number>`

The `max-age` response directive defines the response’s freshness timeline in seconds, after which the response should be considered stale. [☞ RFC 9111 § 5.2.2.1](https://www.rfc-editor.org/rfc/rfc9111#name-max-age-2)

### 

`must-revalidate`

The `must-revalidate` response directive indicates that the cache must not reuse a stale response until it’s been successfully validated by the origin server. [☞ RFC 9111 § 5.2.2.2](https://www.rfc-editor.org/rfc/rfc9111#name-must-revalidate)

If the server throws an error, the cache must surface that instead of reusing a stale response. If the cache is disconnected, it must produce an error with the [HTTP 504 Gateway Timeout](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Status/504) status code, or another more applicable error code.

**Side effects:** `must-revalidate` is one of the directives, along with `s-maxage` and `public`, that allow shared caches to [store and reuse a response to a request containing an `Authorization` header](#caching-authenticated-responses), which they are generally prohibited from doing.

### 

`no-cache`

The `no-cache` response directive indicates that the cache must not reuse _any_ response until it’s successfully validated by the origin server. [☞ RFC 9111 § 5.2.2.4](https://www.rfc-editor.org/rfc/rfc9111#name-no-cache-2)

This is similar to `must-revalidate` but refers to all cached responses, not just stale ones. In effect, `no-cache` is a sort of `max-age=0, must-revalidate`.

### 

`no-store`

The `no-store` response directive indicates that private and shared caches must not store any part of the request or the response, and to never reuse the response. The standard is quick to warn that the effect is not guaranteed:

> “MUST NOT store” in this context means that the cache MUST NOT intentionally store the information in non-volatile storage and MUST make a best-effort attempt to remove the information from volatile storage as promptly as possible after forwarding it. This directive is not a reliable or sufficient mechanism for ensuring privacy. In particular, malicious or compromised caches might not recognize or obey this directive, and communications networks might be vulnerable to eavesdropping. [☞ RFC 9111 § 5.2.2.4](https://www.rfc-editor.org/rfc/rfc9111#name-no-store-2)

**Side effects:** The directive can also influence non-HTTP caches. Most browsers will [exclude from the back/forward cache](https://web.dev/articles/bfcache#minimize-no-store) pages having the `no-store` response directive. Chrome, however, has recently started to make _some_ of these pages [eligible for bfcache](https://developer.chrome.com/docs/web-platform/bfcache-ccns) when the browser deems it safe.

### 

`must-understand`

The `must-understand` response directive indicates that the cache shouldn’t store or reuse responses with HTTP status codes whose semantics the cache doesn’t understand and conform to. The directive is meant to future-proof existing implementations from status codes that might have special requirements in regards to caching. [☞ RFC 9111 § 5.2.2.3](https://www.rfc-editor.org/rfc/rfc9111#name-must-understand)

It’s recommended to use `must-understand, no-store` together as a fallback, and caches are encouraged to ignore the `no-store` directive if they do understand the semantics of the HTTP status code. This ensures older caches that don’t recognize the `must-understand` directive don’t cache the response at all, although by 2025 this should be an exceedingly rare sight.

### 

`private`

The `private` response directive indicates that the response is meant for a single user. [☞ RFC 9111 § 5.2.2.7](https://www.rfc-editor.org/rfc/rfc9111#name-private).

Therefore:

-   a shared cache must not store the response; and
-   a private cache may store the response even if the response wouldn’t otherwise be heuristically cacheable.

The `private` directive can be used to guard against other directives that might inadvertently make [authenticated responses available to shared caches](#caching-authenticated-responses).

### 

`public`

The `public` response directive indicates two things:

-   a shared cache may [store and reuse a response to a request containing an `Authorization` header](#caching-authenticated-responses), which it’s generally prohibited from doing; and
-   a private cache may store the response even if the response wouldn’t otherwise be heuristically cacheable.

[☞ RFC 9111 § 5.2.2.9](https://www.rfc-editor.org/rfc/rfc9111#name-public)

### 

`s-maxage=<number>`

The `s-maxage=<number>` response directive is analogous to `max-age`, but only affects shared caches. [☞ RFC 9111 § 5.2.2.10](https://www.rfc-editor.org/rfc/rfc9111#name-s-maxage).

The directive also incorporates the semantics of the `proxy‑revalidate` response directive, in that a shared cache must not use a stale response until it has been successfully validated with the origin server.

**Side effects:** `s-maxage` is one of the directives, along with `must-revalidate` and `public`, that allow shared caches to [store and reuse a response to a request containing an `Authorization` header](#caching-authenticated-responses), which they are generally prohibited from doing.

### 

`proxy-revalidate`

The `proxy-revalidate` response directive is analogous to `must-revalidate`, but only affects shared caches. [☞ RFC 9111 § 5.2.2.8](https://www.rfc-editor.org/rfc/rfc9111#name-proxy-revalidate).

### 

`no-transform`

The `no-transform` response directive indicates that intermediaries, regardless of whether they implement a cache or not, must not transform the response content, such as optimizing images or compressing stylesheets and scripts. [☞ RFC 9111 § 5.2.2.6](https://www.rfc-editor.org/rfc/rfc9111#name-no-transform-2).

### 

`stale-while-revalidate=<number>`

The `stale-while-revalidate` response directive was defined in [RFC 5861: HTTP Cache-Control Extensions for Stale Content](https://www.rfc-editor.org/rfc/rfc5861) (2010). It indicates that the cache may use a cached response if it hasn’t exceeded its freshness lifetime by more than the specified number of seconds.

Whenever the presence of this directive causes a stale response to be served, the cache should trigger a background revalidation of the response.

The author of the RFC, Mark Nottingham, [has written a rationale](https://www.mnot.net/blog/2014/06/01/chrome_and_stale-while-revalidate) for this directive.

### 

`stale-if-error=<number>`

Also defined in the RFC 5861 extension, the `stale-if-error` response directive indicates that the cache may use a cached response if it hasn’t exceeded its freshness lifetime by more than the specified number of seconds, if the attempt to validate the stale response results in an error.

[HTTP Caching Tests](https://cache-tests.fyi/) suggests this directive is not well supported.

## Cache-Control request directives

As web developers, we most often deal with `Cache-Control` in HTTP responses, but this header can also be included on HTTP requests. Browsers, for example, use them when the user refreshes the page.

When used in HTTP requests, `Cache-Control` directives express the client’s preference in regards to the freshness or age of the response. Caches reconcile these requests with the `Cache-Control` response directives of its cached responses.

> Note: the [`cache` option](https://developer.mozilla.org/en-US/docs/Web/API/RequestInit#cache) for `fetch()` has a separate set of values that map to `Cache-Control` request directives, but the mappings are not always intuitive. For example, `cache: 'no-cache'` maps to `Cache-Control: max-age=0`. For the curious, the mappings are [defined here](https://fetch.spec.whatwg.org/#http-network-or-cache-fetch). You can always set your `Cache-Control` headers directly with the `headers` option.

### `max-age=<number>`

The `max-age` request directive indicates that the client wants a fresh response whose age is less than or equal to the specified number of seconds. When combined with `max-stale`, the client will accept some stale responses. [☞ RFC 9111 § 5.2.1.1](https://www.rfc-editor.org/rfc/rfc9111#name-max-age).

### `max-stale=<number>`

The `max-stale` request directive indicates that the client will accept a stale response that has exceeded its freshness lifetime by no more than the specified number of seconds. When used without an argument, `max-stale` indicates that the client will accept any stale response, no matter how old. [☞ RFC 9111 § 5.2.1.2](https://www.rfc-editor.org/rfc/rfc9111#section-5.2.1.2)

### `min-fresh=<number>`

The `min-fresh` request directive indicates that the client prefers a response that still has at least the specified number of seconds of freshness left. [☞ RFC 9111 § 5.2.1.3](https://www.rfc-editor.org/rfc/rfc9111#name-min-fresh)

### `no-cache`

The `no-cache` request directive indicates that the client prefers caches not to use a stored response without successfully validating it with the origin server. [☞ RFC 9111 § 5.2.1.4](https://www.rfc-editor.org/rfc/rfc9111#name-no-cache)

### `no-store`

The `no-store` request directive indicates that a cache must not store any part of either this request or any response to it. The same caveats as to [its response counterpart](#no-store-response) apply. [☞ RFC 9111 § 5.2.1.5](https://www.rfc-editor.org/rfc/rfc9111#name-no-store)

If a cache serves this request with a response that was previously stored, the `no-store` request directive doesn’t cause the cache to remove the response after serving it.

### `no-transform`

The `no-transform` request directive indicates that the client is asking for intermediaries to avoid transforming the content. [☞ RFC 9111 § 5.2.1.6](https://www.rfc-editor.org/rfc/rfc9111#name-no-transform)

### `only-if-cached`

The `only-if-cached` request directive indicates that the client only wants a stored response. Caches should respond with either a stored response that satisfies all the other constraints, or an HTTP 504 Gateway Timeout status code. [☞ RFC 9111 § 5.2.1.7](https://www.rfc-editor.org/rfc/rfc9111#name-only-if-cached)

### `stale-if-error=<number>`

Similarly to [its response counterpart](#stale-if-error-response), the `stale-if-error` request directive indicates that the client will accept a stale response that has exceeded its freshness lifetime by no more than the specified number of seconds, if an attempt to validate it resulted in a server error.

## Browser refresh mechanisms

Browsers typically offer two refresh mechanisms:

-   _soft reloads_, triggered by the reload button, a corresponding menu item and keyboard shortcut, and the pull-to-refresh gesture in mobile browsers, are meant to get an updated representation of the page, for example getting the latest posts on a social media timeline.
-   _hard reloads_, enabled with a modifier key, skip the cache altogether and are meant to fix interrupted loads, outdated cached responses, and other broken states.

Here’s how some browsers on macOS implement these behaviors.

### Soft reloads

Triggered by Ctrl + R on Windows/Linux and Command + R on macOS.

Firefox triggers a conditional request to revalidate the cached response for the main resource (the HTML file). Sub-resources such as stylesheets, scripts, and images are reloaded as usual, according to their cache directives.

Chrome behaves similarly, with the difference that the validation request for the main resource also includes a `Cache-Control: max-age=0` directive (which can’t hurt).

Instead of revalidating its cached response, Safari performs a non-conditional request for the main resource, then loads sub-resources as usual.

### Hard reloads

Triggered by Ctrl + Shift + R on Windows/Linux and Command + Shift + R on macOS except Safari, which uses Command + Option + R. (If you’ve applied your muscle memory to Safari before, you know all too well that the common shortcut opens Reader Mode instead…)

On a hard reload, all three browsers trigger non-conditional requests with the `Cache-Control: no-cache` directive on the HTML page and its sub-resources.

Curiously, once you perform a hard reload in Safari, subsequent soft reloads will still use the `Cache-Control: no-cache` request directive to fetch the main resource, which is probably an unintended, but otherwise benign behavior.

### The `immutable` response directive

Reloading a web page didn’t always work like this. Historically, when performing a soft reload, all the sub-resources would be revalidated along with the main resource, in effect freshening up the cache for the current page.

Circa 2015, Facebook was seeing several HTTP 304 Not Modified responses on long-lived resources like scripts and stylesheets whenever a user would refresh their feed page with the browser’s reload button.

To address this issue, Patrick McManus from Mozilla [proposed](https://bitsup.blogspot.com/2016/05/cache-control-immutable.html) the `immutable` response directive, which later became [RFC 8246: HTTP Immutable Responses](https://www.rfc-editor.org/rfc/rfc8246).

The directive indicates that the origin server won’t update a resource during the freshness lifetime of the cached response, so a cache shouldn’t issue conditional requests for responses that are still fresh when the user reloads the page, unless the user really, really wants an updated response (e.g. a hard reload).

Around the time that [support for `immutable`](https://hacks.mozilla.org/2017/01/using-immutable-caching-to-speed-up-the-web/) landed in Firefox 49 and Facebook began to use it to great effect, Chrome introduced a [new way to perform reloads](https://blog.chromium.org/2017/01/reload-reloaded-faster-and-leaner-page_26.html) that solved the problem without introducing additional directives: instead of revalidating everything on a soft reload, just revalidate the main resource and load sub-resources as usual. Safari switched over to the new reload policy soon after \[[Webkit#169756](https://bugs.webkit.org/show_bug.cgi?id=169756)\], and Firefox eventually did with [Firefox 100](https://www.firefox.com/en-US/firefox/100.0/releasenotes/).

That leaves the `immutable` directive in an awkward place. Safari added support \[[Webkit#167497](https://bugs.webkit.org/show_bug.cgi?id=167497)\] but Chrome representatives remain unconvinced that it offers a significant benefit on top of the current reload behavior \[[Chromium#41253661](https://issues.chromium.org/issues/41253661)\].

## Caching responses to authenticated requests

One of the more confusing aspects of HTTP caching is how various `Cache-Control` response directives affect the way shared caches treat responses to requests that contain an [`Authorization` header](https://www.rfc-editor.org/rfc/rfc9110#name-authorization), which are understood as specific to a single user.

As per [RFC 9111 § 3.5](https://www.rfc-editor.org/rfc/rfc9111#name-storing-responses-to-authen), shared caches are not allowed to store these responses unless the response contains a Cache-Control field with a response directive that allows it to be stored by a shared cache, and the cache conforms to the requirements of that directive for that response.

The three directives that enable shared caches to store authenticated responses, and which must therefore be carefully evaluated before deploying, are:

-   [`public`](#public-response)
-   [`s-maxage=<number>`](#s-maxage-response)
-   [`must-revalidate`](#must-revalidate-response)

Conversely, a [`private`](#private-response) directive prevents any other directive from making authenticated responses eligible to shared caches.

## Conclusion

I wrote this article to clarify for myself what the various cache directives stand for and how they overlap and interact. It only covers the main ideas, without delving into the more obscure corners of HTTP semantics. I approached the subject with a “clear cache”, and mainly used the normative references (RFC 9111 and its extensions), aided by various guides from different eras:

-   [Caching Tutorial for Web Authors and Webmasters](https://www.mnot.net/cache_docs/) (1998—) by Mark Nottingham;
-   [Caching best practices & max-age gotchas](https://jakearchibald.com/2016/caching-best-practices/) (2016) by Jake Archibald;
-   [Prevent unnecessary network requests with the HTTP Cache](https://web.dev/articles/http-cache) (2018) by Ilya Grigorik and Jeff Posnik;
-   [Cache control for civilians](https://csswizardry.com/2019/03/cache-control-for-civilians/) (2019–2025) and [Why Do We Have a Cache-Control Request Header?](https://csswizardry.com/2025/03/why-do-we-have-a-cache-control-request-header/) (2025) by Harry Roberts;
-   [Cache-Control Recommendations](https://grayduck.mn/2021/09/13/cache-control-recommendations/) (2021) by April King;
-   [Web Caching](https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/Caching) on MDN.

What’s interesting about these guides is that the recommendations don’t just encode an interpretation of the specs, but also incorporate safeguards against non-conformant or outdated browser caches and intermediares.

In light of developments as recent as 2022, it would be cool to figure out to what extent things have improved, and which of these safeguards can be discarded. [HTTP Caching Tests](https://cache-tests.fyi/) seems to be a good resource for assessing the situation.
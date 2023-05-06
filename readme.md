# Web User Tracking Techniques

## Disclaimer

This document is intended to help someone who wishes to learn enough about how users are tracked on the web. The reader is assumed to be technical but not deeply experienced with web development. Much of this is speculative but based on my experience as a web developer and things I've read over the years. 

This hasn't really been edited. I've basically just quickly written down a bunch of things. I'll be happy to add topics or details, so feel free to ask.

# Cookies

## Relevant Basics of Cookies

The web browser sends a request to the web server and the web server sends back a response. Both requests and responses include metadata in headers. They’re called headers because they are at the beginning of the request or response.

## How Cookies Work
When a web server includes “Set-Cookie: Name=Value; path=/path/of/cookie; domain=.example.com; expires=Tue, 02 May 2023 16:35:18 GMT” in the response, the web browser will include the “Cookie: …” header in future requests. 

⚠️TODO: break out example request/response lines into separate blocks formatted as code

#### Side Note about Cookie Values

The value of the example cookie above is “Name=Value”. That seems a little confusing but a request can include multiple cookies and the “Name=” prefix helps software developers find a particular cookie.

### Path

If the path is not set, it defaults to the path of the URL that set the cookie – not the web page that was on-screen, but the URL of the specific HTTP request to which the server was responding. 

If the path ends with `/`, the browser will include the cookie in requests to any URL path that starts with the cookie’s path. ⚠️TODO: fact check

In many cases, the path is simply set to the root path, `/`, so the browser will include the cookie in requests regardless of the URL’s path.

### Domain

The domain defaults to the domain of the URL that set the cookie. This means that a cookie set by `www.example.com` would not normally be included in a request to `ads.example.com`. In the example above, we see the domain is set to `.example.com`. Note the dot (`.`) at the beginning of the domain. This tells the web browser to include the cookie in any “subdomain” of `example.com`. Since `ads.example.com` is considered a subdomain

#### Side Note about Domains

Browsers don’t allow just any domain in a `Set-Cookie` header. Basically `www.example.com` is allowed to set the domain to:
 `www.example.com` (but this is unnecessary because it is the default), 
`.www.example.com` (which would cover, for example, `www27.www.example.com`, but I don’t know that anybody actually does this), or 
`.example.com`. 
A `Set-Cookie` with a domain of `.com` is not allowed.

### Session Cookies

Unless a cookie includes an “expires” attribute, the browser forgets it when the user closes the browser. 

### Persistent Cookies

If the cookie specifies a future expiration, the browser will save the cookie and remember it when the browser next opens their browser. A site will sometimes set the value of an existing cookie but with a past expiration date or an expiration of “-1”. These have the effect of telling the browser to delete the cookie.

## First-Party Cookies

First-party just means the cookie was set by the same domain as the page being viewed.

## Third-Party Cookies

Third-party cookies are cookies set by requests to other domains. This can be part of the response to an image request, a script request, a frame/iframe request, or an XHR request that loads data.

# Browser Storage
Cookies are bad at storing larger data and each cookie adds to the size of the HTTP request sent to the server. Browsers now implement various JavaScript APIs for storing data in the browser. Similar to cookies, there is session storage and persistent local storage and data is compartmentalized so one site can't access another's data. When a web site adds an analytics script to their pages, the script can add an iframe that loads a page from their site. This could use local storage to check whether the user has been tagged or store an identifying tag. 

# Identifying Users

For the most part, they don't care about your personal identifying information, because they just want to build a profile of which web pages you've browsed etc. so they can figure out how to sell more lucrative web ads. On the other hand, some of those web sites you visit will know your name and some of those sites will share information about you.

They can use script in the web page to send information about you from your web browser directly to the analytics server.

The site's page can communicate to the iframe which can then send the data from your browser to the analytics server.

The analytics company's server could communicate directly with the servers of the site you're browsing. This would not be visible to the user. All that is necessary for this scenario is that they both have a common identifier. 

IP address and the browser's user-agent (which typically includes information about the hardware and operating system) string can be used as an identifier.

They can use JavaScript fingerprinting techniques to attempt to determine browser type, browser features, and which browser extensions are installed. If this browser fingerprint is fairly unique, it can be used to recognize you. Even if it is not especially unique, it can be combined with your IP address to improve recognition. 

# Profiling Users

They can use your IP address to get a general idea of the location of your internet connection. Maybe your home IP address is even in a database somewhere that someone has been building. If they have access to such a database, then your IP address gives a likely location.

If tracker/analytics JavaScript is in the page or in an iframe the page loads, that JavaScript can use the HTML5 APIs to ask the browser for your precise location. If the user has not disabled this entirely or configured the browser to always allow access, then the browser would prompt the user to allow or deny the web page access to their location.

When multiple websites use the same tracking service, that third party gets to see that you visit those websites and can build a personality profile of you from which websites it knows you do and don't visit. 

If you are logged into any of those sites and they share your personal information with the tracking company, then the tracking company can use other sources to "enrich" your profile by buying data like your phone number, address, etc. from data brokers.

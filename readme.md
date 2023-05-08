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

ISPs and VPN services are uniquely situated in the network to identify you and monitor your web browsing. Also, "web accelerator" services that claim to speed up web browsing over slower connections often require users to install their software and use their web proxies, which allows them to identify you and monitor your web browsing.

Early mobile phone web browsers that made all web requests through a special gateway owned by the phone company sometimes added a unique device identifier to HTTP request headers. Unfortunately, this sometimes included your phone number. This had the effect of sending your phone number to every website you visited.
⚠️TODO: add links to examples

Various techniques can be used to attempt to describe the user's machine machine, operating system, browser type, and extensions. If this description is fairly unique, it can be used to recognize you. This is called fingerprinting and is described in its own section below.

# Profiling Users

They can use your IP address to get a general idea of the location of your internet connection. Maybe your home IP address is even in a database somewhere that someone has been building. If they have access to such a database, then your IP address gives a likely location.

If tracker/analytics JavaScript is in the page or in an iframe the page loads, that JavaScript can use the HTML5 APIs to ask the browser for your precise location. If the user has not disabled this entirely or configured the browser to always allow access, then the browser would prompt the user to allow or deny the web page access to their location.

When multiple websites use the same tracking service, that third party gets to see that you visit those websites and can build a personality profile of you from which websites it knows you do and don't visit. 

If you are logged into any of those sites and they share your personal information with the tracking company, then the tracking company can use other sources to "enrich" your profile by buying data like your phone number, address, etc. from data brokers.

When information about your machine or browser is not sufficiently unique to identify you (see the Fingerprinting section, below), it can still be used to categorize you by statistical association with behaviors. For example, users of the absolute latest iPhones may be more likely to buy at higher prices than users of older iPhones or an older or low-end Android device.

If they can build a list of devices associated with your home's IP address, they can estimate how many people are using how many devices in a given household. This is potentially valuable information about lifestyle and income. The tracker that has the best visibility of this sort of information is your ISP.

# Fingerprinting

## Network-based Fingerprinting

Other then the most obvious example of your IP address, the behavior of your network traffic can reveal information about your operating system and web browser. The mathematical patterns (or lack thereof) in your TCP packet sequence numbers can sometimes be used to recognize the fact that you're running (for example) an old Windows XP box or a certain range of Linux versions. The quirks of your SSL/TLS handshake can show which cryptographic ciphers your browser supports. If TCP sequences indicate Linux system but TLS handshake indicates Windows, then it's reasonable to say you might be running a Windows box through a Linux web proxy. 
⚠️TODO: add links to more information about network-based fingerprinting

## Client-side Browser Fingerprinting

By client-side, I mean that a web page, web page content (e.g. style sheets), or code loaded on your machine as part of a web page.

They can use JavaScript fingerprinting techniques to attempt to determine browser type, browser features, and which browser extensions are installed. If this browser fingerprint is fairly unique, it can be used to recognize you. Even if it is not especially unique, it can be combined with your IP address to improve recognition. 

CSS is a declarative programming language used for telling the browser how the content of a page should look. It has also been used to detect extract information from a rendered page and send it back to the server. I don't remember for sure whether this has been used for extracting browsing history but it has been used to send text a user types into a text field. 
⚠️TODO: find examples of CSS abuse

Here are links to a few examples of client-side browser fingerprinting:
* [Fingerprinting Visit History](https://privacycheck.sec.lrz.de/active/fp_vh/fp_visit_history.html)
* [Intro to Chrome addons hacking: fingerprinting](http://blog.kotowicz.net/2012/02/intro-to-chrome-addons-hacking.html)
* [Dirty Browser Enumeration Tricks – Using chrome:// and about: to Detect Firefox & Addons](https://thehackerblog.com/dirty-browser-enumeration-tricks-using-chrome-and-about-to-detect-firefox-plugins/)

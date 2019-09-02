---
layout: post
title: Bug Hunting: Understanding and detecting Open Redirections
---
# Bug Hunting: Understanding and detecting Open Redirections

I recently started giving my first shots on Bug Hunting, even though I have no previous experience on security apart from HTB, CTFs and learning in general. And nevertheless, I came across my first bug: an Open Redirection.

## What's an Open Redirection vulnerability?
According to [hdivsecurity.com](https://hdivsecurity.com/owasp-unvalidated-redirects-and-forwards):
```
Unvalidated redirects and forwards (or Open Redirections) are possible when a web
application accepts untrusted input that could cause the web application to redirect the
request to a URL contained within untrusted input.
```
An Open Redirection seems innocent and maybe not even a flaw on the website's security. But an attacker can use this bug to redirect users to a malicious site by sending them an already prepared link, which can lead to credential stealing.It allows the attacker to leverage the reputation of the business for a start. Great for phishing.

And it is true, as not so many people do really examine every link they click on. Open Redirection was placed 10th on OWASP Top 10 for 2010 and 2013 as "[Unvalidated Redirects and Forwards](https://www.owasp.org/index.php/Top_10_2013-A10-Unvalidated_Redirects_and_Forwards)". The following discovery of mine is the most typical example of this type of vulnerability.

## Detection
I started simply by browsing the site and having Burp Suite logging the traffic between my computer and the server. After some searching and clicking around, BurpSuite intercepted a GET request from clicking the "Google play" button where the URL had a "url" parameter and it had as value the Google Play address of the company's Android app.

```
https://website.com/track?dp=/&ea=redirect&ec=Store&el=GooglePlay&
params={"from":"Web","campaign":"website_home"}&
url=https://play.google.com/store/apps/details?id=com.website.app&referrer
=utm_campaign=website_home&utm_source=website
```

([Here](https://github.com/fuzzdb-project/fuzzdb/blob/master/attack/redirect/redirect-urls-template.txt) you can find some other typical Open Redirection URL parameters.)

So, by changing the "url" parameter to any other website (let's say http://malicioussite.com), the user gets redirected to this website (in this case malicioussite.com). And that is actually the vulnerability: a hacker could take advantage of this bug and redirect users of the site to a malicious site, owned by him, and copy the look and feel of the original site to confuse the victims on entering their credentials.

This bug may also be combined with [other techniques](https://twitter.com/LooseSecurity/status/1120638007760117760):
```
Open Redirect + Miconfigured OAuth App => OAuth Token Stealing
Open Redirect + Filtered SSRF => SSRF
Open Redirect + CRLFi => XSS
Open Redirect + javascript URI => XSS
```
Ways to [prevent](https://www.credera.com/blog/technology-insights/java/top-10-web-security-risks-unvalidated-redirects-forwards-10/) Open Redirections:
1. Avoid using them.

2. Don’t use user input to determine the destination.

3. If destination parameters can’t be avoided, ensure that the supplied value is valid and authorized for the user via whitelisting or blacklisting.

Thanks for reading!

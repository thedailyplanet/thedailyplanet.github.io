---
layout: post
published: true
title: Further Enumeration of BT block lists
---

Over breakfast this morning I kicked off a run through feeding the Alexa top 10k hosts to BT's nameservers to detect blocking. Idly watching the output whilst munching breakfast I noticed some apparently innocuous sites resolve to the BT blocking HTTP servers, so decided to investigate further...

Oracle, those wonderful people responsible for giving us such delights as Java and Solaris, seemed an odd choice to block, but sure enough, BT's name servers point you at the two proxies responsible for protecting our innocent youth from the horrors of freedom of expression:

```
$ host -ta oracle.com
oracle.com has address 213.120.234.109
oracle.com has address 213.120.234.146
```

Strange.. let's poke a little furhter

```
$ nc -v 213.120.234.109 80
proxy.sspi.bt.net [213.120.234.109] 80 (www) open
GET / HTTP/1.1
Host: oracle.com

HTTP/1.1 301 Moved Permanently
Connection: Keep-Alive
Content-Length: 0
Location: http://www.oracle.com/
Server: BigIP

^C
$ host -ta www.oracle.com
www.oracle.com has address 2.21.118.140
```

So wait, a DNS redirect to those hosts does not necessarily mean the site is blocked. Following the HTTP redirect we see:

```
$ host -ta www.oracle.com
www.oracle.com has address 2.21.118.140
```

At which point, your traffic goes straight to oracle. Let's try another host:

```
$ nc -v 213.120.234.109 80
sspi02.sspi.bt.net [213.120.234.109] 80 (www) open
GET / HTTP/1.1
Host: www.gtv8.org

HTTP/1.1 200 OK
Content-Length: 454
Content-Type: text/html
Date: Fri, 02 May 2014 07:34:56 GMT
Server: Apache/2.2.19 (Unix) PHP/5.3.6 mod_ssl/2.2.19 OpenSSL/1.0.0d
X-Powered-By: PHP/5.3.6

<p>Welcome to gtv8.org; there's nothing to see here.</p>
```


So that's nice, they've proxied my traffic with my asking.

A blocked site will produce the following:

```
$ nc -v 213.120.234.109 80
proxy.sspi.bt.net [213.120.234.109] 80 (www) open
GET / HTTP/1.1
Host: www.facebook.com

HTTP/1.1 302 Found
Location: http://blockpage.bt.com/pcstaticpage/blocked.html?policy=Z2xvYmFsLXN0cmljdC0yNDIyYWQzMC01OTAyLTRhY2YtOTkzZC1kNDNmY2E4ODEyNzI=;&view=MjQyMmFkMzAtNTkwMi00YWNmLTk5M2QtZDQzZmNhODgxMjcy;&list=BT-Strict&originalUrl=aHR0cDovL3d3dy5mYWNlYm9vay5jb20v
Connection: close
```


So what does it mean? It means that BT are transparently proxying some [1] of your HTTP traffic. Further analysis shows that a fair number of domains are resolved to the BT proxies, and content permitted, not blocked.

Good news for censorship, not so great news for your privacy.


[1] Where 'some' means 'a yet to be determined subset'





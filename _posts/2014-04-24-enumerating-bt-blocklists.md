---
layout: post
published: true
---

## Enumerating BT block lists

I had a thought this morning. Back in the day I used to filter ads at the DNS level by running a caching name server and feeding it a zone file drawn from a list of dodgy ad sites. This list is published with this in mind, but is in the format of an /etc/hosts file. A little tweaking and it serves nicely as zone file usable by BIND.

Having been thinking about the topic of ISP-level internet filtering a great deal of late, I'd been wondering how to reverse engineer the block lists, and this morning I remembered the existence of this [hosts file](http://winhelp2002.mvps.org/hosts.txt).. BT censor internet domains by returning an A record pointing to a webserver which redirects to their 'blocked' page, so why not feed the list of hosts contained therein to BT's name servers and grep for responses indicating the site is blocked?

A line of code later...

```
for i in `grep ^0 hosts.txt | awk '{print $2}'`; 
  do host -t a $i |grep 213.120.234.1[40][69] > blocked.txt ; 
done && awk '{print $1}' blocked.txt | sort -u > blocked-hosts.txt
```

No error checking, but this is quick n dirty proof of concept... We run this three times, having adjusted the level of filtering between each run. A further addition of a few hundred social networking sites and we have four output files. Diffs against them reveal which hosts are added at each increment in blocking level.

```
$ wc -l *
  1345 blocked-light.hosts.txt
  1722 blocked-moderate.hosts.txt
   377 blocked-moderate-increment.txt
   105 blocked-social-strict.hosts.txt
  3096 blocked-strict.hosts.txt
  1379 blocked-strict.increment.txt
  8024 total
```

Wow! That's about 3200 domains proven to be blocked, across the varying filter levels... Now, where's that Alexa top million sites list...





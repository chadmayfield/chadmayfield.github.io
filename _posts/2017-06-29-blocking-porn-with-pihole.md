---
layout:     post
title:      Blocking porn with Pi-hole
date:       2017-06-29 11:12:01
summary:    Using my new project called pihole-blocklists it is possible to create custom blocklists to block porn (among other things) using a pihole.  More lists and creation scripts are coming soon to block a number of possibly unwanted sites.
#categories: pihole blocklist porn-blocking filtering raspberrypi
---

## Introduction
Most people I know are connected to the Internet more than they are disconnected, for better or worse.  I, for example, am connected all day at work, then as I travel and commute I'm connected by my smartphone, and at home I am also connected via my workstations, servers, and smart devices.  If you have total control over your network, like I do in my home, then you start to monitor usage.  You can see malicious traffic, highest bandwidth users/devices, and where that bandwidth is going.  You start to think... should I be blocking some of this traffic?

### Pi-Hole
Enter, the [pi-hole](http://pi-hole.net).  I first saw this project at the end of 2015, and I was intrigued.  I watched the project, waiting for it to mature to a point that I knew it was a viable option for my home network.  (Up until that time I was filtering directly with my firewall via [i-blocklist.com](https://www.iblocklist.com/) blocklists, which was okay, but the interface was lacking).  At the beginning of this year I finally put it into play on a [Raspberry Pi 3](https://www.raspberrypi.org/) and have **loved** it ever since.  The pi-hole will block ads and other [unwanted traffic](https://github.com/pi-hole/pi-hole/wiki/What-is-Pi-hole%3F-A-simple-explanation) from your network by taking over as your network's DNS server filtering out any query that it finds on it's blacklist. (Yes, it's still possible for a savvy user to get around this, but that is for another discussion).  I know in certain corners of the Internet it's controversial to block ads.  Some of those arguments are strong and valid, however, I like to control what comes in and out of my network, especially since I have Elementary to High School aged kids that I want to "protect" from things I don't want them to access.

### Unwanted Traffic
I want to clarify; the term *unwanted traffic* can have different meanings to people.  For some, unwanted traffic might be gambling, dating, and [warez](https://en.wikipedia.org/wiki/Warez) sites that employees or users shouldn't visit lest they in legal trouble using corporate assets.  For others, like governments, it might be entire country codes like [.cn](https://en.wikipedia.org/wiki/.cn), [.ru](https://en.wikipedia.org/wiki/.ru), [.su](https://en.wikipedia.org/wiki/.su), and [.us](https://en.wikipedia.org/wiki/.us), that you don't want your users to access.  As for my home network, I set out to simply block ads in the beginning.  I always had the idea I could create custom blocklists like I had on my firewall.   I made a list of *possible* site categories that I could block;

* pornography/adult/mature content
* bots/spiders
* hijacked/known exploit sites/spammers
* proxies
* known pedophile sites
* malware/spyware
* microsoft
* countries (cn/ru/su)
* illegal drug usage
* gambling
* dating 
* violence/hate/racism

## Project: pihole-blocklists
With those lists in mind I set out to create [PoC](https://en.wikipedia.org/wiki/Proof_of_concept#Software_development) code that would gather open source lists and collate them into a single larger category list that I would then block using pihole.

The product of that PoC now lives in my project repository named[pihole-blocklists](https://github.com/chadmayfield/pihole-blocklists) (hosted on GitHub).  Currently it has just a single perl script (with more coming soon) called [create_blocklist_porn.pl](https://raw.githubusercontent.com/chadmayfield/pihole-blocklists/master/create_blocklist_porn.pl) which only blocks porn sites.  It does a few things to create the list;

1. Downloads the Alexa Top 1 Million Site csv file from S3.
2. Downloads the adult site filter list from Université Toulouse 1 Capitole.
3. Expands each archive.
4. Removes IP addresses and non-domains from the adult domains list (from #2 above).
5. Compares each of the adult sites to each Alexa Top 1 Million Site domains to see which of the top 1 million sites are adult or pornography sites, as it finds a correlation in the line it writes it to a "light" list.  This list ends up containing ~20,000 domains out of the top 1 million sites.
6. It also writes out a list of all domains (minus IP addresses) contained in the adults domains list (from #2 above) to a "heavy" list which is ~1.7 million domains.

The generated "heavy" list is about 45MB;

```
pi@pihole:~ $ ls -lh /etc/pihole/list.*.githubusercontent*
-rw------- 1 root   root   1.1M Jun 26 21:41 list.0.raw.githubusercontent.com.domains
-rw------- 1 root   root    45M Jun 26 21:42 list.7.raw.githubusercontent.com.domains
```

Either the "light" or "heavy" list can be used on a Raspberry Pi 1 Model B+ or better hardware.  Currently I am using the "heavy" list with ~1.7 million domains on my Raspberry Pi 3.  On that Raspberry Pi 3 with 50+ devices all doing DNS queries it barely breaks a sweat, the load average isn't even 1.0!

*(NOTE: this pihole even has [Docker](https://docker.com/) running with 3 containers!*;

<a href="http://i.imgur.com/4LKgton.png"><img src="http://i.imgur.com/4LKgtonl.png"></a><br />

### Installing Custom Lists
I'm not going to explain how to install the blocklists in detail, because Installing a custom list is quite simple, [and is covered by the pihole wiki](https://github.com/pi-hole/pi-hole/wiki/Customising-Sources-for-Ad-Lists).  In simple steps, you modify the `/etc/pihole/adlists.list` and add the URL of the pihole list created by [create_blocklist_porn.pl](https://raw.githubusercontent.com/chadmayfield/pihole-blocklists/master/create_blocklist_porn.pl) (that is served via a webserver accessible to your pihole) and run `pihole -g` to generate a new blocklist for the pihole to use.

For ease of adding lists to your pihole I have hosted my generated lists that I create on Github for your use;
* Light @ 327KB (~20,000 domains): [Chad Mayfield (from Top 1M)](https://raw.githubusercontent.com/chadmayfield/pihole-blocklists/master/lists/pi_blocklist_porn_top1m.list)
* Heavy @ ~45MB (~1.7m domains): [Chad Mayfield (All)](https://raw.githubusercontent.com/chadmayfield/pihole-blocklists/master/lists/pi_blocklist_porn_all.list)

### Other Lists & Projects
There are [list collections](https://wally3k.github.io/) that have [listed my pornography blocklist](https://github.com/WaLLy3K/wally3k.github.io/commit/3c006da42570a871f40067e10d80abfb6ae85cd5) as well as many other lists, along with download links to easily update your pihole.  This [list collection](https://wally3k.github.io/) is courtesy of [@WaLLy3K](https://twitter.com/WaLLy3K);

<a href="http://i.imgur.com/r2yzb9W.png"><img src="http://i.imgur.com/r2yzb9Wl.png"></a><br />

There are also projects like [piadvanced](https://github.com/deathbybandaid/piadvanced/) from [@deathbybandaid](https://twitter.com/deathbybandaid) that have used  [my list](https://github.com/deathbybandaid/piadvanced/blob/6b67c0b47e869501863f9ddbc481ddc9578e618e/modules/piholetweakmodules/pihole-chadmayfieldpornblock.sh) and others in [several](https://github.com/deathbybandaid/piholeparser/search?utf8=%E2%9C%93&q=chadmayfield&type=) of it's setup scripts.  I encourage you to look as this great project that has many features for customizing your pihole.

## Conclusion
In conclusion, using a pihole can easily help block unwanted traffic from your network.  Using the scripts contained in this project you can begin to create and use custom blocklists to protect those on your network.  [More blocklist creation scripts will be created and released soon with an install script that can update them automatically, so stay tuned!  As always, for questions or comments please [contact me via email](https://chadmayfield.com/contact/) or [via twitter](https://twitter.com/chadrmayfield).<br />
<br />
<br />
### TODO
Currently the `create_blocklist_porn.pl` script is pulling down the [Alexa Top 1 Million](https://aws.amazon.com/alexa-top-sites/) list.  However, recently Amazon disabled API access to the list without paying a fee, [which would be about $300/mo](https://twitter.com/paul_pearce/status/800780539204538370), which I won't pay.  As such there are [other lists](https://gist.github.com/chilts/7229605) lists that can be used to the same effect;

1. [Stavoo](https://statvoo.com/top/sites) Top Sites List, a list of 1 million sites based on Stavoo's own collection.  In their own words;  "Statvoo has been collecting information on over 300 million websites since the beginning of 2013."
    * Download [Stavoo Top 1 Million](https://statvoo.com/dl/top-1million-sites.csv.zip)
2. [Cisco Umbrella Popularity List](https://s3-us-west-1.amazonaws.com/umbrella-static/index.html): which [according to Cisco](https://umbrella.cisco.com/blog/2016/12/14/cisco-umbrella-1-million/), "...contains our most queried domains based on passive DNS usage across our Umbrella global network.." which "...Unlike Alexa, the metric is not based on only browser based 'http' requests from users but rather takes in to account the number of unique client IPs invoking this domain relative to the sum of all requests to all domains."
    * Download [Umbrella Top 1 Million](http://s3-us-west-1.amazonaws.com/umbrella-static/top-1m.csv.zip)
    * Download [Umbrella Top TLDs](http://s3-us-west-1.amazonaws.com/umbrella-static/top-1m-TLD.csv.zip)
3. [Majestic Million](https://blog.majestic.com/development/alexa-top-1-million-sites-retired-heres-majestic-million/) which has a [daily](https://blog.majestic.com/development/majestic-million-csv-daily/) list
    * Download [Majestic Million Top 1 Million](http://downloads.majestic.com/majestic_million.csv)
    *  Also has a great [Public Interface](https://majestic.com/reports/majestic-million).

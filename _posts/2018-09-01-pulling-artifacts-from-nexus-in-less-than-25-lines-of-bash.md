---
layout:     post
title:      Pulling artifacts from Nexus 3 in 25 lines of bash
date:       2018-09-01 13:57:01
summary:    Pulling artifacts from Nexus 3 can be very easy using the provided API. This is an example of how to pull (and check the integrity of) an artifact in less than 25 lines of bash.
#categories: nexus nexus3 artifacts api bash curl
---

# Introduction

A few months ago, at work, I was tasked to create a build system for several different projects including Docker containers (multi-stage) and a web fronend.  At the time artifacts were pushed to a [Nexus 3](https://www.sonatype.com/nexus-repository-oss) maven repository.  The Docker containers are built on a Linux system and delivered to a private Docker registry.  Those containers need to pull other artifacts, so I needed a way to pull artifacts to include in the Docker containers on a Linux system.  So I wrote a quick and dirty bash script to pull those artifacts from Nexus, that now lives in my company's git repository.  Several weeks ago I needed to do the same thing at home, so I decided to do something similar but better, this is the result.

## Nexus

[What is Nexus?](https://blog.sonatype.com/2010/04/why-nexus-for-the-non-programmer/) Nexus is a Repository Manager that allows you to proxy, collect, publish, and manage all of your dependencies and artifacts.  It can be managed through a web interface and supports Maven/Java, npm, NuGet, RubyGems, Docker, P2, OBR, APT and YUM and more.

## The script

```bash
#!/bin/bash

url="http://fileserver/service/rest/beta/search/assets?repository=maven-releases&name=dot-files"

artifact=( $(curl -s -X GET --header 'Accept: application/json' \
    "$url" | grep -Po '"downloadUrl" : ".*?[^\\]*.(zip.sha1|zip)",' | \
    awk -F '"' '{print $4}' | sort -Vr | head -n2) )

((${#artifact[@]})) || echo "ERROR! No artifacts found at provided url!"

for i in "${artifact[@]}"; do
    if [[ $i =~ (sha1) ]]; then
        checksum=$(curl -s "$i" | awk '{print $1}')
    else
        file="$(echo "$i" | awk -F "/" '{print $NF}')"
        curl -sO "$i" || { echo "ERROR: Download failed!"; exit 1; }

        if [ "$(sha1sum "$file" | awk '{print $1}')" != "$checksum" ]; then
            echo "ERROR: Checksum validation on $file failed!"; exit 1;
        else
            printf "Downloaded : %s\nChecksum   : %s\n" "$file" "$checksum"
        fi
    fi
done
#EOF
```

## Explanation

Let's breakdown the script and discuss each step.  It's obviously a very simple script that accomplishes a small task in a container build project.  There are two main steps;

1. Construct the known artifact list.
2. Download the artifact and check it's integrity.

### Construct the artifact list

In order to download we'll interact with the Nexus API.  It's well documented [here](https://help.sonatype.com/repomanager3/rest-and-integration-api). To view it, navigate to `https://$nexus/swagger-ui` where `$nexus` is your hostname or IP address.  (As a side note, it uses [swagger tools](https://swagger.io/) as the user interface to the API). You'll see something similar;

<a href="https://i.imgur.com/yKYOPHU.png"><img src="https://i.imgur.com/yKYOPHUl.png"></a><br />

Explore the API interface, there are many different ways to search for and pull artifacts.  The main thing to be concerned with is the asset names to download.  Looking at the `curl` command in the script you can see I am pulling the artifact named `dot-files` from the repository named `maven-releases` (which is a compressed zip file of my [dot-files](https://github.com/chadmayfield/dot_files) stored in a maven repo in Nexus).  

The API provides all the information necessary to build a URL to request the artifacts.

<a href="https://i.imgur.com/mWLUiEs.png"><img src="https://i.imgur.com/mWLUiEsl.png"></a><br />

So I'll breakdown the [`curl`](https://curl.haxx.se/) command to understand what it is doing.

```bash
curl -s -X GET --header 'Accept: application/json' \
    "$url" | grep -Po '"downloadUrl" : ".*?[^\\]*.(zip.sha1|zip)",' | \
    awk -F '"' '{print $4}' | sort -Vr | head -n2)
```

First, I [silence (-s)](https://curl.haxx.se/docs/manpage.html#-s) curl while making the [GET (-X)](https://curl.haxx.se/docs/manpage.html#-X) request and I include the [extra header (--header)](https://curl.haxx.se/docs/manpage.html#-H) to request a JSON response from the server.  The `$url` is defined from what we discovered interacting with the Nexus API.  That is enough to get a JSON list of artifacts similar to this;

```bash
{
  "items" : [ {
    "downloadUrl" : "http://fileserver/repository/maven-releases/configs/1.0.1-16-g3b321b9/dot-files-0.0.1-16-g3b321b9.zip",
    "path" : "configs/0.0.1-16-g3b321b9/dot-files-0.0.1-16-g3b321b9.zip",
    "id" : "Vm0xd1IyRnRVWGxWV0dSUFZteHdUMVpzWkc5V2JHeDBaVVYwV0ZKdGVIcFhhMk0",
    "repository" : "maven-releases",
    "format" : "maven2",
    "checksum" : {
      "sha1" : "c1c4ed2deadbeefcafebabe2d311b075df39e591",
      "md5" : "6f31dfbdeadbeefcafebabe62dff2f34"
    }
  }, {
    "downloadUrl" : "http://fileserver/repository/maven-releases/configs/1.0.1-16-g3b321b9/dot-files-0.0.1-16-g3b321b9.zip.md5",
    "path" : "configs/0.0.1-16-g3b321b9/dot-files-0.0.1-16-g3b321b9.zip.md5",
    "id" : "VmpGYWIxUXlSWGhpUm14VllsaFNhRmxzVm1GT2JHUjBXWHBzVVZWVU1Eaz0isak",
    "repository" : "maven-releases",
    "format" : "maven2",
    "checksum" : {
      "sha1" : "0314aaadeadbeefcafebabef6ed1861e857632de",
      "md5" : "9b2154deadbeefcafebabedb08c7d0a6"
    }
  }, {
    "downloadUrl" : "http://fileserver/repository/maven-releases/configs/1.0.1-16-g3b321b9/dot-files-0.0.1-16-g3b321b9.zip.sha1",
    "path" : "configs/0.0.1-16-g3b321b9/dot-files-0.0.1-16-g3b321b9.zip.sha1",
    "id" : "VmpGYWIxUXlSWGhqU0ZKVFltNUNhRlZxUm5kaU1WWkhVbFJzVVZWVU1Eaz0jnsj",
    "repository" : "maven-releases",
    "format" : "maven2",
    "checksum" : {
      "sha1" : "cc27709deadbeefcafebabeb14959f27a82ac649",
      "md5" : "4b9098deadbeefcafebabe07cd359c17"
    }
  }, {
    "downloadUrl" : "http://fileserver/repository/maven-releases/configs/1.0.1-19-g214a6ed/dot-files-0.0.1-19-g214a6ed.zip",
    "path" : "configs/0.0.1-19-g214a6ed/dot-files-0.0.1-19-g214a6ed.zip",
    "id" : "RlppVkZaeVdWWlZlRll4WkhKaApSbHBwVW10d05sWnNXbUZXTVZwV1RWVldhQXB",
    "repository" : "maven-releases",
    "format" : "maven2",
    "checksum" : {
      "sha1" : "c1df66fcdeadbeefcafebabe1aa282bd4cbb52f3",
      "md5" : "9beb64deadbeefcafebabe32722543b3"
    }
  }, {
    "downloadUrl" : "http://fileserver/repository/maven-releases/configs/1.0.1-19-g214a6ed/dot-files-0.0.1-19-g214a6ed.zip.md5",
    "path" : "configs/0.0.1-19-g214a6ed/dot-files-0.0.1-19-g214a6ed.zip.md5",
    "id" : "Vm0weGQxSXhiRmhUV0doV1YwZDRWRll3Wkc5alZsVjNWMnQwVjFKdGVEQlViRlp",
    "repository" : "maven-releases",
    "format" : "maven2",
    "checksum" : {
      "sha1" : "bd16addeadbeefcafebabece888505203128a5c0",
      "md5" : "bc9badeadbeefcafebabe8c3f1555dc8"
    }
  },
```

We just need to parse down the information to find what we want, a URL.  First we [pipe](https://en.wikipedia.org/wiki/Pipeline_(Unix)) the JSON document we received from the server to [grep](http://linuxcommand.org/lc3_man_pages/grep1.html).  We use the [PCRE option (-P)](https://www.gnu.org/software/grep/manual/grep.html#grep-Programs-1) option and print [only matched lines (-o)](https://www.gnu.org/software/grep/manual/grep.html#General-Output-Control).  We'll match any line that has the following 'downloadUrl' pattern; `"downloadUrl" : ".*?[^\\]*.(zip.sha1|zip)",`. Which returns lines like this;

```bash
"downloadUrl" : "http://fileserver/repository/maven-releases/configs/1.0.1-16-g3b321b9/dot-files-1.0.1-16-g3b321b9.zip",
"downloadUrl" : "http://fileserver/repository/maven-releases/configs/1.0.1-16-g3b321b9/dot-files-1.0.1-16-g3b321b9.zip.sha1",
"downloadUrl" : "http://fileserver/repository/maven-releases/configs/1.0.1-19-g214a6ed/dot-files-1.0.1-19-g214a6ed.zip",
"downloadUrl" : "http://fileserver/repository/maven-releases/configs/1.0.1-19-g214a6ed/dot-files-1.0.1-19-g214a6ed.zip.sha1",
```

We then parse the line with `awk` using double-quote as the [field seperator (-F)](https://www.gnu.org/software/gawk/manual/html_node/Field-Separators.html).  Then pass the list to `sort`, [reversing (-r)](https://shapeshed.com/unix-sort/#how-to-sort-in-reverse-order) the sort based on a natural [versions (-V)](http://man7.org/linux/man-pages/man1/sort.1.html) and grab the first two lines using `head -n2` (which will be the latest .zip and .zip.sha1);

```bash
http://fileserver/repository/maven-releases/configs/1.0.1-19-g214a6ed/dot-files-1.0.1-19-g214a6ed.zip.sha1
http://fileserver/repository/maven-releases/configs/1.0.1-19-g214a6ed/dot-files-1.0.1-19-g214a6ed.zip
```

Both files are then added to our array, `${artifacts[@]}`.  The next line `((${#artifact[@]}))` checks to make sure that we have elements in the array.  If not the script dies since our curl command failed.  If it succeeds, we now have two URLs to pull from Nexus.  One the artifact and one the checksum that was created by maven when the artifact was uploaded.

### Download and Integrity Check

The download then happens inside a simple for loop, again using curl.  We iterate on each elemenet in the array mentioned above, checking to see if the URL has a match of `sha1` in it.  If it does we download it and assign the checksum as a variable to be used to check the integrity: `checksum=$(curl -s "$i" | awk '{print $1}')`.

If it is not the sha1 file then we just download it and run `sha1sum` (since we're running on a Linux system) on the downloaded file and compare the Nexus reported sha1 hash in `$checksum` to the checksum of the file after it is downloaded; `"$(sha1sum "$file" | awk '{print $1}')"`.  If they match we get a success message;

```bash
(0) [chad@Chads-MBP:~] $ ./artifact_pull.sh 
Downloaded : dot-files-1.0.1-19-g214a6ed.zip
Checksum   : c1df66fcdeadbeefcafebabe1aa282bd4cbb52f3
```

Easy right? The code for this post can be found here; [pull-nexus-artifact](https://github.com/chadmayfield/pull-nexus-artifact).

## Conclusion

The script could be extened to include the many different checks and additional pulls of artifacts.  This isn't ground breaking work but it was fun throwing a script together, which took about 5 minutes (and the blog post took an order of magnitude longer).  Many people look down on bash as not worth of their attention, however it can be a very powerful and surgical tool if used correctly.

As always, for questions or comments please [contact me via email](https://chadmayfield.com/contact/) or [via twitter](https://twitter.com/chadrmayfield).<br />

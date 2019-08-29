---
layout:     post
title:      Upgrading Go in 40 lines of Bash
date:       2019-08-29 16:41:00
summary:    A simple shell script to download and install the latest featured Go updated on a Linux system.
#categories: golang go github development bash curl
---

# Introduction
I enjoy quickly [automating tasks](https://github.com/chadmayfield/scriptlets) that I do often.  I usually use Bash because it's quick and simple. I feel like I'm actually saving time and sometimes it helps me to take a break and look at something new.  Recently I started learning [Golang](https://golang.org).  I had setup my Linux environment several months ago and wanted to [upgrade](https://golang.org/doc/install#install) my installation.  After reading the docs and upgrading it quickly, I decided to make a command line tool that would do it for me even quicker in the future.  So this script was born.  It's relatively simple and takes no time to run the command and update my install.  I thought it might save some time for someone else in the future.

Requirements for this script can be found in the [README.md](https://github.com/chadmayfield/upgrade-go/blob/master/README.md) file in it's [repo](https://github.com/chadmayfield/upgrade-go).  Basically it uses curl/tar/gzip to download and extract the archive into the default recommended path of `/usr/local/`.

Let's look at it.

## The script

```bash
#!/bin/bash

fail() {
    echo "ERROR: $@"; exit 1;
}

if [[ $OSTYPE =~ "linux" ]]; then

    command -v curl >/dev/null 2>&1 || \
        { echo >&2 "ERROR: curl is required, but it's not installed!"; exit 1; }

    read -t 10 -p "This will upgrade go... are you sure? (Y/N) " -n 1 -r

    gpath="/usr/local/"
    url="$(curl -s https://golang.org/dl/ | grep 'downloadBox.*linux' | grep -Po '(?<=href=")[^"]*(?=")')"

    if [ -d "${gpath}/go" ]; then
        if [ -n "${url+x}" ]; then
            printf "\nFound: %s\nDownloading now...\n" "$url"
            curl -nsS "$url" -O -w "Downloaded %{size_download} bytes in %{time_total}s\n" || fail "Download failed!"

            printf "Uncompressing: %s\n" "${url##*/}"
            sudo rm -rf "$gpath/go" || fail "Unable to remove prior install!"
            sudo tar xzf "${url##*/}" -C "$gpath" || fail "Failed to uncompress ${url##*/}!"

            printf "Updated to %s!\n" "$($gpath/go/bin/go version)" || fail "Uh-oh, go version failed! Is $GOPATH set?"
            rm -f "${url##*/}"
        else
            fail "URL not found!"
        fi
    else
        fail "Go is not installed at ${gpath}/go"
    fi
else
    fail "This must be run on a Linux system!"
fi
```

Pretty simple and straightforward.  It does some checking on paths, commands, and OS required.  Then it scapes the offical golang download page for the lastest featured version and fetches it with curl.  We then remove the old install and uncompress the new install into `/usr/local/`.

### Output
```
chad@dev-vm:~$ ./update_go.sh
This will upgrade go... are you sure? (Y/N) y
Found: https://dl.google.com/go/go1.12.9.linux-amd64.tar.gz
Downloading now...
Downloaded 127961523 bytes in 4.606516s
Uncompressing: go1.12.9.linux-amd64.tar.gz
Updated to go version go1.12.9 linux/amd64!
```

## Conclusion
Easy right? Now using a script will take me only as long as a download to upgrade my go installation to latest version!

The code for this post can be found here; [upgrade-go](https://github.com/chadmayfield/upgrade-go) or <a href="https://i.imgur.com/fGzeoBt.png">as an image</a>.  As always, for questions or comments please [contact me via email](https://chadmayfield.com/contact/) or [via twitter](https://twitter.com/chadrmayfield).<br />

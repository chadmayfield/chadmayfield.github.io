---
layout:     post
title:      Docker usability tip #1: Change the prompt!
date:       2016-06-17 14:00:00
summary:    Extending the usability of Docker when dealing with many containers, it helps to change the bash prompt so you always know where you are.
#categories: linux admin docker containers
---


I started using [Docker](https://en.wikipedia.org/wiki/Docker_(software)) in 2014, then took a <em>small</em> break to do other things.  By the time I had come back to docker in 2015, [the community was exploding!](https://blog.docker.com/2016/02/docker-hub-two-billion-pulls/).  Now, I use Docker everyday.  I build and administer containers both professionally and personally.  I build apps for containers.  But above all, I save time by using Docker.

Often times I need to [attach](https://docs.docker.com/engine/reference/commandline/attach/) (or [exec](https://docs.docker.com/engine/reference/commandline/exec/)) to a Docker container to debug or troubleshoot my misbehaving app(s).  More than once I have gotten lost in all my containers, not finding a config file or the correct app in a container.  Only after those frustrating times did I find that I was in the wrong container or never left the host!

So on all my containers I now [change](http://www.tldp.org/HOWTO/Bash-Prompt-HOWTO/x329.html) the [bash command prompt](http://bashrcgenerator.com/) (and sometimes the hostname) inside the container so I always know where I am.  Most times my hosts are bare installs and have no customization so it's difficult, when you're working fast, to differentiate between the host and a container quickly.  At some point the complexity of your app and all of it's containers will require debugging, this helps. (Some people may never need to do this since they never attach to a container but I do, **a lot**.) 

To illustrate this, we'll build a very simple container on a freshly deployed host with no prompt customization.  Normally this would cause some heart-ache since both the host and container have the same dull white prompt.  But not with the new container, here's the Dockerfile;

<pre>FROM ubuntu:latest

RUN echo 'PS1="\[$(tput setaf 3)$(tput bold)[\]appname@\\h$:\\w]#\[$(tput sgr0) \]"' >> /root/.bashrc

CMD ["/bin/bash"]</pre>

You'll notice the line that changes the prompt;

<pre>RUN echo 'PS1="\[$(tput setaf 3)$(tput bold)[\]appname@\\h$:\\w]#\[$(tput sgr0) \]"' >> /root/.bashrc</pre>

This changes the prompt to bold yellow, which is hard to miss when you are attached to a container.

While this is dead simple it is very effective if you have several containers running (and you've changed the hostnames to be pretty rather than the stock UUIDs).    

See it in action;
![dockerfile_bash_prompt.png](https://raw.githubusercontent.com/chadmayfield/chadmayfield.github.io/master/images/dockerfile_bash_prompt.png)

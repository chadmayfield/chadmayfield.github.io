---
layout:     post
title:      Enabling PAM debug logging
date:       2016-06-15 11:21:00
summary:    How to enable PAM debug logging on RHEL-like systems...
#categories: linux admin pam
---

[Pluggagable authentication module](https://en.wikipedia.org/wiki/Pluggable_authentication_module), or PAM for short, is now at the core of most modern Linux distributions.  In very simplistic terms, PAM is the authentication core that most Linux programs use to authenticate a user and/or program.  Want to login?  PAM reads /etc/shadow to verify the entered password.  Do you want to enforce a password policy to your users?  Yup, PAM does that.  Tuxradar has a great [overview of PAM](http://www.tuxradar.com/content/how-pam-works) (though some of it might be slightly outdated) and [this page](http://pig.made-it.com/pam.html) also has great information.

While PAM is easy to configure, it is quite easy to misconfigure and could completely open your system to the type of authentication attack that you are trying to prevent by using PAM.  Much like [CoreOS recently discovered](https://coreos.com/blog/security-brief-coreos-linux-alpha-remote-ssh-issue.html).  As you start to get deeper into PAM, always remember to backup your sane config to be able to revert back!

Recently I had to debug some PAM security misconfigurations on RHEL7 and needed a way to find the issues quickly.  So I enabled debug logging, which saved a lot of time!

To enable debug logging you'll need to be root.  Then, as always, backup your config;

<pre>cp -p /etc/syslog.conf /etc/syslog.conf.ORIG</pre>

On Red Hat based systems we'll need to have rsyslog send debug output to the debug log. (On Ubuntu/Debian based systems I believe they are sent to /var/log/auth.log.  So we add the following to the rsyslog.conf (rsyslog is the default system logger in RHEL6+);

<pre>*.debug        /var/log/debug.log</pre>

And touch the debug log file (if it doesn't exist);

<pre>test ! -f /var/log/debug.log && touch /var/log/debug.log</pre>

Then restart syslog;

<pre>systemctl restart rsyslog</pre>

And now all PAM debug output will be logged to /var/log/debug.log.  In this example I was attempting to change my user password after having purposefully messed up the selinux security context for /etc/shadow;

<pre>Jun 15 16:26:36 localhost passwd: pam_unix(passwd:chauthtok): authentication failure; logname= uid=1001 euid=0 tty=tty1 ruser= rhost=  user=chad</pre>

Reminder: Authentication failures will still be logged to /var/log/secure as well as the new debug log, for example;

<pre>Jun 15 16:07:00 localhost login: pam_unix(login:auth): check pass; user unknown
Jun 15 16:07:00 localhost login: pam_unix(login:auth): authentication failure; logname=LOGIN uid=0 euid=0 tty=tty1 ruser= rhost=
Jun 15 16:07:03 localhost login: FAILED LOGIN 1 FROM tty1 FOR (unknown), User not known to the underlying authentication module</pre>

An alternative method to debug PAM is to use the [pam_debug.so](http://www.linux-pam.org/Linux-PAM-html/sag-pam_debug.html) module to debug specific modules (not covered here... yet).

I hope this works for you as it did for me in debugging my specific issue.  As always, let me know if I have errors or can help in anyway.

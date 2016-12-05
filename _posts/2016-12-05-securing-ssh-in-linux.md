---
layout:     post
title:      Securing SSH in Linux
date:       2016-12-05 11:00:00
summary:    Securing SSH for servers and workstations
#categories: linux admin ssh security
---

### Introduction
Secure Shell is a powerful network tool in an administrator's (or user's) toolbox for logging into and interacting with any Linux, BSD, MacOS, or even Windows (under Cygwin) system.  Most users will use SSH to securely log into a system and use services on that system. I think [Wikipedia](https://en.wikipedia.org/wiki/OpenSSH) sums it up best;

> "Secure Shell (SSH) is a cryptographic network protocol for operating network services securely over an unsecured network. The best known example application is for remote login to computer systems by users. SSH provides a secure channel over an unsecured network in a client-server architecture, connecting an SSH client application with an SSH server."

In this article I will attempt to describe the steps that I take to secure a [default installation](https://linux.die.net/man/5/sshd_config) of SSHd, specifically OpenSSH (from now on I will refer to the SSH *daemon* as simply SSH). Some options are important, others could be controversial, but they work for me and I hope others get something out of this.  If you have questions or comments, please leave them below.

### Disclaimer
WARNING: I do not know everything, obviously. I am not a Unix wizard, I learn everyday.  However, I have used most of these steps to secure my SSH servers for years.  All that said, please use caution with them, do not use them unless you have an understanding of how they actually function. I have tried to include links to more in depth tutorials, man pages, etc., to help where possible. If there are errors or mispellings in this article, please let me know and I'll fix them as soon as possible.

### Table of Contents

1. [Why Secure SSH](#why-secure-ssh)
1. [Problems with default setups](#problems-with-default-setups)
1. [Step 1: Secure Your Users](#step-1-secure-your-users)
1. [Step 2: Disable unneeded user accounts](#step-2-disable-unneeded-user-accounts)
1. [Step 3: Keep system up to date](#step3-keep-system-up-to-date)
1. [Step 4: Use a firewall and rate limit connections](#step-4-use-a-firewall-and-rate-limit-connections)
1. [Step 5: Monitor the logs](#step-5-monitor-the-logs)
1. [Step 6: Configure key-based authentication and disable keyboard interactive logins](#step-6-configure-key-based-authentication-and-disable-keyboard-interactive-logins)
1. [Step 7: Disable root logins](#step-7-disable-root-logins)
1. [Step 8: Set maximum failed login attempts](#step-8-set-maximum-failed-login-attempts)
1. [Step 9: Reduce Max Startups](#step-9-reduce-max-startups)
1. [Step 10: Reduce the login grace time](#step-10-reduce-the-login-grace-time)
1. [Step 11: Limit remote users or groups](#step-11-limit-remote-users-or-groups)
1. [Step 12: Use privilege separation](#step-12-use-privilege-separation)
1. [Step 13: Enable strict mode](#step-13-enable-strict-mode)
1. [Step 14: Bind to specific interface](#step-14-bind-to-specific-interface)
1. [Step 15: Change the listening port](#step-15-change-the-listening-port)
1. [Step 16: Check other defaults](#step-16-check-other-defaults)
1. [Conclusion](#conclusion)

### Why Secure SSH
Some, in my opinion not many, may ask, why secure SSH?  Well, because it's secure! All administrators (and regular users) should know what SSH is and how to use it. As said in the introduction it enables secure interaction with a system.  So the question may arise isn't SSH secure by default?

### Problems with default setups
The answer to that simple question is loaded, yes most of the time it is secure by default, but it can be "more secure".  In the realm of information security there is a concept of [defense in depth](https://en.wikipedia.org/wiki/Defense_in_depth_(computing)), I always see it as conceptric rings of security.  If an attacker can bypass one layer (or ring) of security based on a misconfiguration or even a 0-day exploit then there are more defenses in place to stop the attack.  Securing SSH can reduce the attack surface giving an attacker fewer and fewer methods to exploit a system.  Some problems with some default installs (especially in the [IoT](https://en.wikipedia.org/wiki/Internet_of_things) world) can include using SSH v1, having root access enabled (especially with a weak password or even no password at all), and even having a low key strength (lower key strengths could be compromised).

### Step 1: Secure Your Users
A system is only as secure as it's *weakest* user. Let's imagine we have a user with a weak password, perhaps `Password1`. It is very possible to have an automated attack bot online scanning a large IP space for SSH servers, when it finds one it uses common account names with [easy to guess](https://xato.net/is-123456-really-the-most-common-password-51cd4259927d#.9viiyafap) [passwords](https://dazzlepod.com/uniqpass/). If one works, great!, the attacker is in and the bot reports back to the attacker.  It is only a matter of time until the attacker tries a [privilege escalation](https://en.wikipedia.org/wiki/Privilege_escalation) attack to gain root privileges on the system and then game over. It can be that fast.  So why make it easy?  Here are some things to remember when dealing with users;

* Force your users to choose **good** passwords.  (See [xkcd #936, password strength](https://xkcd.com/936/) or [xkcd #792, password reuse](https://xkcd.com/792/)).
* If possible use one time passwords via the [PAM Google Authenticator](http://www.howtogeek.com/121650/how-to-secure-ssh-with-google-authenticators-two-factor-authentication/) library or, though it's old, the [PAM opie library](https://www.rho.cc/?p=126).
* Install and configure [libpam_cracklib](https://www.cyberciti.biz/faq/securing-passwords-libpam-cracklib-on-debian-ubuntu-linux/) or [libpam_pwquality](http://www.deer-run.com/~hal/linux_passwords_pam.html) **and** [pam_tally2](http://www.linux-pam.org/Linux-PAM-html/sag-pam_tally2.html).

Now obviously, as always, there's a trade-off between usablitiy and security. Training your users to use good passwords and not to reuse passwords is paramount to a secure system. Enforcing good passwords habits will help users be more secure.. Password requirements are site and situation specific, though under 12 characters has been proven to be [too weak](https://en.wikipedia.org/wiki/Password_strength).

Perosnally, I have decided in the future to use some personally adapted form of the guidence provided by the United States National Institute for Standards and Technology or [NIST](https://www.nist.gov/).  In a draft publication called [Special Publication 800-63-3: Digital Authentication Guidelines](https://pages.nist.gov/800-63-3/), we can see **NEW** ways NIST is recommending the way we handle password policies;

* Minimum password length is 8 characters.
* Maximum password length is 64 characters.
* **All** characters should be allowed, ASCII and spaces, as well as Unicode characters.
* Should not be known-bad dictionary words.
* There's more! (Here's a [nice summary](http://www.slideshare.net/jim_fenton/toward-better-password-requirements), from [Jim Fenton's](https://twitter.com/jimfenton) 2016 PasswordsCon talk.)

It seems the days of a one letter, one number, one upper, one lower, and once special character are gone. These are good to increase the key-space to make it more difficult on the attacker to crack a password. According to NIST, their new stance is to **put the burden of authentication on the verifyer, not the user**.

### Step 2: Disable uneeded user accounts
This is pretty self-explanitory, disable any unneeded or unused user accounts.  In a multi-user enviroment users can leave a company and accounts stay active.  These accounts need to be disabled or removed, especially if the account uses a weak password as discussed above.  If the account is a system account and cannot be removed but it is not needed, then disable interactive logins or change the default shell to `/bin/false` or `/bin/nologin`.  This will disable the daemon's (or user's) login shell to prevent them from logging in.

### Step 3: Keep system up to date
Securing a system that you never plan on updating is an execise in futility. [Patching](https://en.wikipedia.org/wiki/Patch_(computing)) or updating keeps software "safer" (not invulnerable) from new software vulnerabilities and 0-day exploits.  Explaining how to update or discussing patching schedules is outside of the scope of this article.  However for Linux based distributions there are a couple of very good ways to **automatically** update a system, at the very least from the security repos.  If you are using Red Hat, CentOS, or Fedora based systems you can use [yum-cron](https://linuxaria.com/howto/enabling-automatic-updates-in-centos-7-and-rhel-7) and if you are using a Debian/Ubuntu distro you can use [unattended-upgrades](https://help.ubuntu.com/lts/serverguide/automatic-updates.html).  Both of these software packages are highly configurable and can be setup to email an administrator when updating.

Remember: **Update, Update Update!**

### Step 4: Use a firewall and rate limit connections
Every system should use a firewall regardless of system type (server or workstation), it is yet another line of defense in our "defense in depth" approach.  There are several possible ways that you can use a firewall to secure SSH;

##### A. Whitelist your IP address
Generally speaking whitelisting is more secure that blacklisting and it's much easier to maintain. Using [IPtables](https://www.netfilter.org/) you can whitelist your own personal IP address and block all other IP addresses from loging into SSH on the system.  The following example will accept all incoming connections to port 22 (the port on which SSH is listening) from IP address 1.2.3.4 and only that address. (For additional information on IPtables configurations in Ubuntu [look here](https://help.ubuntu.com/community/IptablesHowTo).)

```
# Accept all new connections from address 1.2.3.4 to SSH
iptables -A INPUT -p tcp -m state --state NEW --source 1.2.3.4 --dport 22 -j ACCEPT
# Drop all other connection attempts to SSH
iptables -A INPUT -p tcp --dport 22 -j DROP
```

##### B. Rate limit SSH connection attempts
Many automated SSH scanners will connect several times in quick succession and attempt to login with default usernames and passwords. This can go on over and over until success (because of a weak or default password... hence the steps above to [Secure your users](#secure-your-users)).  We can mitigate with threat by rate limiting how many times an IP address can connect to a system.  When combined with the step below, to [Reduce Max Startups](#reduce-max-startups), we will create a powerful defense against automated scanners. [Look here](https://debian-administration.org/article/187/Using_iptables_to_rate-limit_incoming_connections) for more in depth discussion on rate limiting.

```
# Add IP of connecting host to the "recent" list of IP addresses
iptables -I INPUT -p tcp --dport 22 -i eth0 -m state --state NEW -m recent --set
# Update IP to "recent" list and drop for 10 min if more than 4 attempts have been made
iptables -I INPUT -p tcp --dport 22 -i eth0 -m state --state NEW -m recent --update --seconds 600 --hitcount 4 -j DROP
```

##### C. Use external programs such as fail2ban or DenyHosts
Use external programs such as fail2ban and DenyHosts that are blacklist agents that can accomplish similar things, albeit a more permanent solution that the one above. These systems will permanently ban the offending IP, even if the attacking system is only compromised for a few days, and then restored to working order.  It will take administrator intervention to remove an the once bad, now good IP from the blacklist.

* [Fail2ban](http://fail2ban.sourceforge.net) scans log files and bans IPs that exhibit malicious behavior, such as many successive password authentication failures, port scans, etc..  Fail2ban can also be used in conjunction with other system services such as Apache and is very easy to setup.
* [DenyHosts](http://denyhosts.sourceforge.net/faq.html) is python script that analyzes the SSH server log and bans potential harmful IPs that are abusing the system. DenyHosts can be setup to email the administrator reports of banned IPs.
* [Daemon Shield](https://sourceforge.net/projects/daemonshield/) is yet another program similar to fail2ban and DenyHosts however I have never used it, though I left it here as refence.

##### D. When a firewall is not available use TCPwrappers to whitelist
When a firewall is not available make the best of a bad situation and use TCPwrappers, it's not my favorite solution, but I need to list it here nonetheless.  Many IoT devices have no firewall, yet they use xinetd and tcpwrappers and it can be leveraged to provide some security.  [TCPwrappers](https://en.wikipedia.org/wiki/TCP_Wrapper) is a host-based network ACL system that can whitelist and blacklist hosts that can connect to a system.

There are two files that control the tcpwrappers config, they are /etc/hosts.deny and /etc/hosts.allow.  The syntax used in these files is; `daemon_list : client_list [ : shell_command ]`. (`daemon_list` is a system daemon name and `client_list` is one or more hostnames.)  More information can be [found here](https://www.centos.org/docs/5/html/Deployment_Guide-en-US/ch-tcpwrappers.html).

To use tcpwrappers it's easy.  First let's deny all hosts to all processes;

```
echo "ALL:ALL" >> /etc/hosts.deny
```
Now let's allow our single IP;
```
echo "sshd:1.2.3.4" >> /etc/hosts.allow
```

That's it! You now have system in which only you will be able to login via SSH.

### Step 5: Monitor the logs
[System log files](https://en.wikipedia.org/wiki/Logfile) are the life blood of a system. They can tell an administrator about problems with hardware and software, whether attacks are occurring or if there are application errors, as well as many other things. If the log files are not being checked regularly a system can be breached, crash, or software can malfunction with no one knowing about it, which can prolong downtime. In a world of 99.9999999% [SLA's](https://en.wikipedia.org/wiki/Service-level_agreement) that's bad! Just like a car a computer system needs proper care and feeding. Reviewing log files can get overwhelming and especially boring, many can suffer from [alert fatigue](https://en.wikipedia.org/wiki/Alarm_fatigue).  The good news is, that there are many utlities out there to assist.  Two such open source utilties are;

* [logrotate](https://github.com/logrotate/logrotate), *"allows for the automatic rotation compression, removal and mailing of log files. Logrotate can be set to handle a log file daily, weekly, monthly or when the log file gets to a certain size."*
* [logcheck](http://logcheck.org/), which *"is a simple utility which is designed to allow a system administrator to view the logfiles which are produced upon hosts under their control. It does this by mailing summaries of the logfiles to them, after first filtering out "normal" entries."*

Personally part of my morning routine is to review my logrotate emails for any abnormalities. This allows me to focus on other things and manage my time better, detecting errors quickly.

### Step 6: Configure key-based authentication and disable keyboard interactive logins

##### A. Configure key-based authentication

Using key-based authentication is [preferred](https://security.stackexchange.com/questions/69407/why-is-using-an-ssh-key-more-secure-than-using-passwords) over passwords as they are cryptographically secure when authenticating the client.  There are [many](https://www.digitalocean.com/community/tutorials/how-to-configure-ssh-key-based-authentication-on-a-linux-server) different [tutorials](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/6/html/Deployment_Guide/s2-ssh-configuration-keypairs.html) that discuss in depth configuration options.  The three main concerns when creating keys are;

   1. At a minimum [use a strong key strength](http://security.stackexchange.com/a/115296) or even [encrypt the key](http://www.tedunangst.com/flak/post/new-openssh-key-format-and-bcrypt-pbkdf).
   2. Always, always, always, use a [STRONG passphrase](http://world.std.com/~reinhold/diceware.html) for the key!
   3. Keep the private key **[private](https://blog.codinghorror.com/keeping-private-keys-private/)**.

To create keys;

```
# if it doesn't exist create .ssh key dir
[ ! -d ~/.ssh ] && mkdir ~/.ssh
# create key with a 2048-bit+ key size and identifiable comment
##### enter a strong passphrase when prompted #####
ssh-keygen -t rsa -b 4096 -C "key identifier such as your_email@address.com"
# transfer the public key to system configuring for key-based logins
scp ~/.ssh/id_rsa.pub <username>@hostname:~/.ssh/authorized_keys
# fix permissions on keys if needed
chown -R <username>:<username> ~/.ssh
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
```
The SSH public/private key pair are now created and the public key has been transfered to the system as the `authorized_keys` file. Now enable SSH on the system to use key-based authentication;

```
sed -i 's/^#*PubkeyAuthentication \(yes\|no\)/PubkeyAuthentication yes/g' /etc/ssh/sshd_config /etc/ssh/sshd_config
```

##### B. Disable keyboard interactive logins
Used in conjunction with SSH key-based authentication is the disabling of keyboard-interactive logins with a password. On my systems, as I am the only user, I disable passwords immediately.  However this can be a tricky setting for administrators with many users as this will force **all** users to use keys.  We can enabled the setting;

```
sed -i 's/^PasswordAuthentication \(yes\|no\)/PasswordAuthentication no/g' /etc/ssh/sshd_config
sed -i 's/^#*ChallengeResponseAuthentication \(yes\|no\)/ChallengeResponseAuthentication no/g' /etc/ssh/sshd_config
```

OPTIONAL STEP: As mentioned this can be tricky for some administrators with systems that have many users.  Rather than disabling it for the entire system, you may want to disable interactive logins for *specific users*, in which case the `Match user <yourusername>` directive needs to be used.  The directive is added directly above the `PasswordAuthentication no` in the `sshd_config`;
```
sed -i 's/^PasswordAuthentication \(yes\|no\)/PasswordAuthentication no/g' /etc/ssh/sshd_config
sed -i '/^PasswordAuthetication no/i Match user yourusername' /etc/ssh/sshd_config
```

Now that the config is place, you can restart the daemon;

```
systemctl restart sshd (or kill -HUP "cat /var/run/sshd.pid")
```

### Step 7: Disable root logins
Within `sshd_config` there is an option `PermitRootLogin` which controls whether root can login via SSH. I always recommend to disable root logins via SSH.  However, there are three values for the  `PermitRootLogin` option;

* **PermitRootLogin no**; this is the most desirable option. Root should never be allowed to login via SSH to further reduce the attack surface.
* **PermitRootLogin without-password**; in situations where root is required to login (for say backups) this option can be used to disable password authentication and to force the use of key-based authentication. (NOTE: PubkeyAuthentication must be enabled too).
* **PermitRootLogin forced-commands-only**; this option is the same as the previous option but it limits root to only run specific commands listed in the `sshd_config` file.

As I said above, it is my recommendation that root logins are always **disabled**.

As a test a few years ago I deployed a VPS with a default SSH config and a weak password with root logins permitted.  In less that 48 hours an automated scanner had exploited the weak password and a malicious user had setup the VPS as a spam relay (which I immediately destroyed).  The moral of the story, **disable root logins**.

### Step 8: Set maximum failed login attempts
From the [man page](https://linux.die.net/man/5/sshd_config); *Specifies the maximum number of authentication attempts permitted per connection. Once the number of failures reaches half this value, additional failures are logged. The default is 6.*

Many may feel this is highly restrictive, but I always set `MaxAuthTries 3`. This is just a hold out from my governemnt compliance days.  If you can't login in three attempts with a ssh key, then perhaps you've forgotten your key passphrase. 

To enable the option;
```
sed -i 's/^#*MaxAuthTries [[:digit:]]/MaxAuthTries 3/g' /etc/ssh/sshd_config
```

### Step 9: Reduce Max Startups
Properly restricting the MaxStartups can protect against unauthenticated connection attempts to the SSH daemon. In other word an attacker doesn't have to wait to try a connection, they can try simultaneous connections to authenticate.  We have modified `MaxAuthTries` and rate limited the firewall but we can also enable and modify this option to give us additional coverage. The smaller the values the more difficult it is to make parallel connections.

So we set it;

```
sed -i 's/^#*MaxStartups.*$/MaxStartups 3:50:10/g' /etc/ssh/sshd_config
```

The three colon-seperated values stand for `start:rate:full`.  As explained [elsewhere](http://serverfault.com/a/275671) this means that we allow 3 users to login at a time and to randomly and linearly increase the dropped  connections to the max of 10 at a rate of 50%, (rate/100).  For a larger multi-user system these number need to be tuned so users are not denied access. See the [man page](https://linux.die.net/man/5/sshd_config) for additional details and a more succinct description.

### Step 10: Reduce the login grace time
Used in conjunction with the option of `MaxStartups` above, change the time to something more sane.  The default is 2 minutes which is more than enough time to enter a password.  I usually keep mine at 30 seconds.  After a user has not successfully logged in this amount of time the connection will be dropped.

We can change it;

```
sed -i 's/^#*LoginGraceTime [[:digit:]]m/LoginGraceTime 30/g' /etc/ssh/sshd_config
```

### Step 11: Limit remote users or groups
By default **all** valid users are able to log into a system using the default SSH config. Whitelisting, [as mentioned previously](#use-a-firewall-and-rate-limit-connections), is a wonderful way to add additional security to any system. Using the `AllowUsers`/`AllowGroups` and `DenyUsers`/`DenyGroups` directives we can whitelist and blacklist users on the system.

Anyone of the following to enable/restrict users or groups;
* `AllowGroups`: list of *allowed* user **groups** seperated by spaces. For example; `AllowGroups admins dbas`
* `AllowUsers`:  list of *allowed* **users** seperated by spaces. For example; `AllowUsers alice bob mary`
* `DenyGroups`: list of *denied* user **groups** seperated by spaces. For example; `DenyGroups sales webmasters`
* `DenyUsers`: list of *denied* **users** seperated by spaces. For example; `DenyUsers mysql www`

So if I wanted to only allow myself or anyone in the wheel group I would add the following to my `/etc/ssh/ssdh_config`;

```
AllowUsers chad
AllowGroups wheel
```

That way only me or anyone I add to the wheel group is explicitly allowed to use log into a system via SSH.  That's whitelisting and it's pretty restrictive.  However, to be less restrictive, blacklisting can be used.  In this example, the user 'mary' is on probation and the group 'sales' should never be allowed to login (because, hey they're sales!), however everyone else should be able to login freely.  To blacklist we use;

```
DenyUsers mary
DenyGroups sales
```

OPTIONAL: Keeping my configs clean is important, it decreases the time it takes to make changes, it helps with readability, and satisfies my "admin-OCD".  Usually I will create user groups, much like the `wheel` group and add users that are allowed SSH access to that group. That way in my config I only have one directive, `AllowGroups wheel`, to successfully whitelist the `wheel` group.  To add a user to an existing group;
```
usermod -a -G wheel <username>
```

### Step 12: Use prvilege seperation
When enabled, SSH will seperate prviliges by creating an unprivileged child process to handle the network traffic coming from your SSH sesstion. Very little in the SSH daemon will be run as root to protect it from any possible exploits to the SSH daemon itself. After a successful authentication an additional process will be created that is privileged as the authenticated user.  This setting should [always be set to yes](https://stigviewer.com/stig/red_hat_enterprise_linux_5/2013-04-10/finding/V-22486).  Luckily, the default is yes, though I check to make sure and modify it if needed;
```
sed -i 's/^#*UsePrivilegeSeparation.*/UsePrivilegeSeparation yes/g' /etc/ssh/sshd_config
```

### Step 13:  Enable strict mode
Enabling strict mode force SSH to check the user's home directory and ssh files for the proper mode and ownership (that they aren't world-writable). The default is set to yes, but I like to force it just to make sure;

```
sed -i 's/^#*StrictModes.*/StrictModes yes/g' /etc/ssh/sshd_config
```

### Step 14: Bind to specific interface
By default SSH listens on *all* interfaces, though a specific `ListenAddress` can be used to explicitly declare an interface address.  This can be used when multiple interfaces exist, escpecially if SSH should only be listening to internal network address but also has public interfaces. The format foraAdding the listening directive is in any one of the following;
```
ListenAddress host|IPv4_addr|IPv6_addr
ListenAddress host|IPv4_addr:port
ListenAddress [host|IPv6_addr]:port
```
Note: Multiple `ListenAddress` options are permitted. Also if the port option is not specific SSH will listen to any port options already defined, which must preceed this option. Which brings us to a "controversial" option...

### Step 15: Change the listening port
Like I said previously, this option for [most](https://major.io/2013/05/14/changing-your-ssh-servers-port-from-the-default-is-it-worth-it/) draws much critisim. The reason is most people say that it brings, [ **security through obscurity**](https://en.wikipedia.org/wiki/Security_through_obscurity).  For me it is not security issue, rather it is just a simple way to keep my logs clean from unwanted automated scanner noise.  I don't really care if I get scanned, it's a fact of online life.  What I do care about it having to sort through logs.  Running SSH on a non-standard port, just helps me filter out that noise.

By default, SSH will [always](https://www.iana.org/assignments/service-names-port-numbers/service-names-port-numbers.xhtml) run on port 22. To change it we simply modify the port;

```
sed -i 's/^#*Port [[:digit:]]/Port 22222/g' /etc/ssh/sshd_config
```

### Step 16: Check other defaults
There are other settings that SSH has to offer and they are secure by default, however you may want to check them to make sure. Some of them include; 

   * `UsePam yes`: Enable the use of [Linux PAM](https://chadmayfield.com/2016/06/15/enable-pam-debug-logging/).
   * `IgnoreRhosts yes`: Disables the use of .rhosts and .shosts
   * `RhostsRSAAuthentication no`: Applied only to SSHv1 and rhosts/host.equiv
   * `HostbasedAuthentication no`: Applies only to SSHv2 using rhosts for host based authentication
   * `PermitEmptyPasswords no`: Disables the use of empty passwords for login

### Conclusion
While this may feel like an exhaustive list, it isn't. There are still a few other options that can be used to further secure SSH, which I might tackle at a later date. There are other [guides](https://calomel.org/openssh.html) out there that have a ton of good information. Some have great [scripts](https://debian-administration.org/article/87/Keeping_SSH_access_secure#comment_11) that can monitor your system for logins.  I recommend looking at those seriously to further reduce your attack surface and increase your security, with a little effort you can make SSH very secure.

After all of these options has been applied don't forget to restart SSH for them to take effect.  You can then check your running SSH config to verify all settings are properly used;
```
/usr/sbin/sshd -T
```

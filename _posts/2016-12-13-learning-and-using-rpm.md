---
layout:     post
title:      Learning and Using RPM
date:       2016-12-13 20:06:00
summary:    Learning and using the RPM Package Manager is necessary for any adminstrator or power user to successfully use a Red Hat (CentOS) based system.  Learn about the history, advantages, and especially ways to successfully use RPM.
#categories: linux admin rpm redhat rhel package-management
---

> The RPM Package Manager (RPM) is a powerful command line driven package management system capable of installing, uninstalling, verifying, querying, and updating computer software packages. [rpm.org](http://rpm.org/about.html)

## Why use RPM?
[Red Hat](https://www.redhat.com/en) initially released the Red Hat Package Manager, or RPM, to the public in [1997](https://en.wikipedia.org/wiki/RPM_Package_Manager#History).  Historically, a Linux user had to manually build software from source and then install the built binaries, libraries, configuration files, and man pages (documentation) by hand (or script).  RPM solved that issue by simplifying the complexity of installing software on a Linux system.  RPM provides a mechanism to bundle several files into a single package to distribute software which in turn is easy to maintain. It also provides easy installation, upgradability, package verification, and even has dependency evaluation.  RPM packages can even be cryptographically verified using GPG for source verification.  Of note is that RPM is the [standard](https://en.wikipedia.org/wiki/Linux_Standard_Base#Choice_of_the_RPM_package_format) package manager for [Linux Standard Base](https://wiki.linuxfoundation.org/lsb/start) (LSB) which provides consistency to installed Linux systems across distributions.  And finally in my career, RPM has been one of the components that has made  [government compliance ](https://www.redhat.com/en/about/blog/red-Hats-decade-of-collaboration-with-government-and-the-open-source-community) easier, since it has consistency when dealing compliance [standards](https://www.redhat.com/en/technologies/industries/government/standards) and solid source verification since it comes directly from Red Hat.  In other words it's made my job easier.

## A closer look at RPM
Regardless of your Linux ideology (Debian vs Red Hat vs Gentoo vs ???) RPM can be a great tool.  Personally it's not my first choice since I use Debian/Ubuntu at home (and Red Hat at work), but it can be very powerful nonetheless.  As with any technology the key is to learn it and use it.  So let's take a little deeper look at RPM.

On the backend RPM uses Berkeley DB as a 'Packages' database to store all of the metadata of installed RPMs.  This database is stored in /var/lib/rpm/.  There are many front ends to RPM, including; [yum](http://yum.baseurl.org), [DNF](http://dnf.baseurl.org), and [Zypper](https://en.opensuse.org/Portal:Libzypp), to name a few.  This article will not cover front ends, however, they will be covered in a future installment.  RPMs are binary files that are *generally* delivered with a specific name format, which is;

`name-version-release.architecture.rpm`

Generally RPMs have an architecture format that is commonly i386, i686,  x86_64, ARM, or PPC, among others.  Some packages contain a `noarch` extension which means it is architecture independent.  RPMs can also be distributed with it's source which usually has a `.src.` in the file name.  Libraries can be distributed as two RPMs, one named `-devel` which contain development files such as headers and the other one has production ready binary files.  RPM knowns about dependencies and will refuse to install until they are met.  RPM will not automatically [solve dependency](https://en.m.wikipedia.org/wiki/Dependency_hell) issues like YUM.

RPMs include a *package label* which usually includes a software name, version (from the upstream source), package release (number of times packaged) or distribution (e.g.  fc22), [group](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/System_Administrators_Guide/sec-Working_with_Package_Groups.html), size, summary, description, and of course the architecture.  To view the package label of an RPM use; `rpm -qi filename.rpm`.

Most of those fields in the package label are derived from the [SPEC file](http://rpm.org/max-rpm-snapshot/s1-rpm-build-creating-spec-file.html), which is a major component in  [building](http://rpm.org/max-rpm-snapshot/ch-rpm-build.html) RPMs, it's basically the recipe.   [rpmbuild] (https://fedoraproject.org/wiki/How_to_create_an_RPM_package) is the command line utility used to create a RPM.   After it's built each binary RPM contains [four parts](http://rpm.org/max-rpm-snapshot/s1-rpm-file-format-rpm-file-format.html); the lead (file identification as RPM), signature (for verification of authenticity/integrity), header (metadata), and payload which is usually the file archive in [cpio](https://en.m.wikipedia.org/wiki/Cpio) format.  Most current versions of RPM allows the use of lzma, bzip2, or xz [compression](http://stackoverflow.com/questions/9292243/rpmbuild-change-compression-format).

## Using RPM
Most days when interacting with RPM I use a set of commands that I've committed to memory.  It is those harder problems with commands that aren't used often that get me, I usually have to hunt for those.  The following is a list of RPM commands that I use frequently enough that they are nice to have in one page.  

##### Read the RPM documentation
One of the most important things that you do is read the documentation. With these commands you can find the names of all of the man pages and a description of them as well as read any `man page`.

```
apropos rpm
man 8 rpm
```

##### Install a RPM
*NOTE:* You can also add the option `--test` to this command to make no changes to the system and simulate an install.

```
# rpm -ivh filename.rpm
```

##### Upgrade a RPM
*NOTE:* You can also add the option `--test` to this command to make no changes to the system and simulate an upgrade.

```
# rpm -Uvh filename.rpm
```

##### Remove a Package
*NOTE:* You can also add the option `--test` to this command to make no changes to the system and simulate a removal. The first option will verbosely erase a package, the second command will **not** verify pacakge dependencies during removal.

```
# rpm -eva packagename
# rpm -ev --nodeps packagename
```

##### Check RPM signature
Check all the digests and signatures in a package to verify integrity or origin.

```
# rpm --checksig filename.rpm
```

##### Verify all packages or a single package
Verify one or all packages are still configured properly or if changes have been made by another user or RPM.

```
# rpm -Va
# rpm -Vp filename.rpm
```

[See table below](#table-1)

##### Verify package owning a file

```
# rpm -Vf /path/to/file
```

[See table below](#table-1)

##### Find dependencies of a RPM or package
View a RPM or package's dependencies.

```
# rpm -qRp filename.rpm
# rpm -qR packagename
# rpm -q --whatrequires filename.rpm
```

##### List all installed RPMs
Listing RPMs can be refined and match using standard Linux/Unix tools such as command pipelining and grep.

```
# rpm -qa
```

##### List recently installed RPMs
List the recently installed RPMs in reverse chronological order.

```
# rpm -qa --last
```

##### Display installed info with short description

```
# rpm -qi packagename
# rpm -qip filename.rpm
```

##### Find out what package a file belongs
Find out what RPM provides a certain file.

```
# rpm -qf /path/to/file
--or--
# rpm -q --whatprovides /path/to/file
```

##### Find documentation of installed file
```
# rpm -qdf /path/to/file
```

##### List all files of an installed by a  package 
List all files installed by a package or RPM

```
# rpm -ql packagename
# rpm -qlp filename.rpm
# less filename.rpm
```

##### Display scripts used in installation
View the specific scripts that are used as part of the installation and uninstallation process.

```
# rpm -qp --scripts filename.rpm
```

##### Query RPM database with formatting
Query the RPM database with defined 'pretty' formats.

```
# rpm -qa --queryformat '%{name}-%{version}-%{release} %{size}\n'
```

##### Display config file for RPM
View the config files for a package or RPM.

```
# rpm -qc packagename
# rpm -qcp filename.rpm
```

##### Display config file for command
Display the config file for a command.

```
# rpm -qcf /path/to/file
```

##### Find state of files in packages
Display the state of all files in an RPM or package.

```
# rpm -qs packagename
# rpm -qsp filename.rpm
```

##### Import RPM GPG key
Import thrid-party or custom GPG keys for package integrity verification.

```
# rpm --import /path/to/public/keyfile
```

##### List all imported GPG keys
List the current imported GPG keys.

```
# rpm -qa gpg-pubkey*
```

##### View groups
A package group is a set of packages that accomplish a specific purpose, for example, Development Tools.  A unique list of groups installed can be found by querying RPM;

```
# rpm -qa --qf '%{group}\n' | sort -u
```

###### Table 1
 Attribute | Description
--------  | -------- 
S  | File size differs
M | Mode differs (includes permissions and file type)
5 | MD5 sum differs
D | Device major/minor number mismatch
L | readlink(2) path mismatch
U | User ownership differs
G | Group ownership differs
T | Modification time (mtime) differs

##### Fix corrupt RPM database
Sometimes errors can occur with the RPM database that will cause issues when interacting with RPM.  In this case the corrupt database need to be fixed.  A very quick and easy fix is;

```
rm /var/lib/rpm/__db*
rpm --rebuilddb
rpmdb_verify Packages
```

## Conclusion
RPM is a mature and powerful tool that should be understood before it is dismissed.  It provides an administrator easy and quick package manipulation and can help to decrease maintence costs over time.  For addtional, more in-depth, material see below. 

### Additional reading
https://en.m.wikipedia.org/wiki/RPM_Package_Manager
http://rpm5.org/docs/rpm-guide.html
http://rpm.org/documentation.html
http://tldp.org/LDP/lame/LAME/linux-admin-made-easy/using-rpm.html
https://www.ibm.com/developerworks/library/l-lpic1-102-5/
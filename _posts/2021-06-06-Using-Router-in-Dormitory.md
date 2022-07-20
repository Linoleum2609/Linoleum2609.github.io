---
layout: post
title:  "Using Router in Dormitory - Cross Compiling MiniEAP for Asuswrt-Merlin"
date:   2021-06-06 
tags: router asuswrt-merlin
---

Well, actually I don't want too many people in my university know this, so I decided writing this blog in English and not to mention the name of my school clearly here. You can find the name in the blog, if it isn't your school, hopefully, the part of cross compiling can help you.

---

I have two routers in my dormitory. During the last year, the software, called *[MiniEAP](https://github.com/updateing/minieap)*, ran on my Redmi router, and another router, manufactured by Netgear, allowed me to use 5GHz WLAN in 64 Channel, which is disabled in my Redmi router. It's pretty annoying, not only because of those messy cables, but also because when I want to connect to my Netgear router via ssh, I have to plug in another network cable to connect. That's why I finally decided to spend my Saturday to finish it. 	~~(COVID-19 sucks. I want to play maimai in game centre)~~

I decided to cross compile MiniEAP for my Netgear R6900, which is running Asuswrt-Merlin. To get started, you need Linux. I am using Debian sid, but you can use any distro you like. Anyway, it is still a better choice to check the [wiki](https://github.com/RMerl/asuswrt-merlin.ng/wiki)...

I found a docker on the Internet, and someone used it on macOS, but I haven't tried yet, so I will use Debian sid to introduce.

## Preparation  

### Dependencies

To build the environment, some packages need to be installed at first. In fact, you can find those packages everywhere on the Internet. Here is what I used:

```
sudo apt --no-install-recommends install autoconf automake bash bison bzip2 diffutils file flex g++ gawk gcc-multilib gettext gperf groff-base libncurses-dev libexpat1-dev libslang2 libssl-dev libtool libxml-parser-perl make patch perl pkg-config python sed shtool tar texinfo unzip zlib1g zlib1g-dev lib32z1-dev lib32stdc++6 automake1.11
```

Also add i386 arch to package dependencies (I guess no one use 32 bits or x86 here):

```
sudo dpkg --add-architecture i386
sudo apt-get update
sudo apt-get install libelf-dev:i386 libelf1:i386
```

### Source Code 

Maybe you have noticed that the repository had transferred to [asuswrt-merlin.ng](https://github.com/RMerl/asuswrt-merlin.ng), and [the old one](https://github.com/RMerl/asuswrt-merlin) had archived. However, it seems that it doesn't include toolchains in [asuswrt-merlin.ng](https://github.com/RMerl/asuswrt-merlin.ng). The toolchains is in another repository:

```
git clone https://github.com/RMerl/am-toolchains.git
```

And don't forget MiniEAP:

```
git clone https://github.com/updateing/minieap.git
```

In my school, the patches are necessary. These patches are used by GZHU originally, but they are also available in my school:

```
git clone https://github.com/ysc3839/openwrt-minieap.git package/minieap
```

### Applying Patches 

In `HOME` we have two folders, `minieap/` and `openwrt-minieap/`. Now `cd` to `minieap/`, then try these commands:

```
patch -p1 < ~/openwrt-minieap/patches/001-enable-gbconv.patch
patch -p1 < ~/openwrt-minieap/patches/002-remove-date-in-log-message.patch
patch -p1 < ~/openwrt-minieap/patches/003-disable-hdd-serial-query-and-show-warning.patch
patch -p1 < ~/openwrt-minieap/patches/004-fix-logging-buffer.patch
patch -p1 < ~/openwrt-minieap/patches/005-remove-pid-check-warn.patch
```

![wtf](/pics/2021-06-06/wtf.jpg)

Actually I want to use an asterisk to get them done at once, but it threw me this... wtf

### Environment 

After patching, we need to create some symlinks. For my router, R6900, I just need to focus on `BCM-SDK` :

```
sudo ln -s ~/am-toolchains/brcm-arm-sdk/hndtools-arm-linux-2.6.36-uclibc-4.5.3 /opt/brcm-arm
echo "PATH=\$PATH:/opt/brcm-arm/bin" >> ~/.profile
```

With those commands, while compiling, probably you won't success. Maybe you will got this after the `ldd` command:

```
$ ldd /opt/brcm-arm/libexec/gcc/arm-brcm-linux-uclibcgnueabi/4.5.3/cc1
      linux-gate.so.1 =>  (0xf77bf000)
      libmpc.so.2 => not found
      libmpfr.so.4 => not found
      libgmp.so.10 => not found
      libdl.so.2 => /lib/i386-linux-gnu/libdl.so.2 (0xf77aa000)
      libelf.so.1 => /usr/lib/i386-linux-gnu/libelf.so.1 (0xf7792000)
      libc.so.6 => /lib/i386-linux-gnu/libc.so.6 (0xf75e3000)
      /lib/ld-linux.so.2 (0xf77c0000)
```

Some libraries are missing. So you need:

```
export LD_LIBRARY_PATH=${LD_LIBRARY_PATH}:/opt/brcm-arm/lib:/usr/local/lib:/usr/lib
```

~~That's all.~~

```bash
$ ldd /opt/brcm-arm/libexec/gcc/arm-brcm-linux-uclibcgnueabi/4.5.3/cc1
      linux-gate.so.1 (0xf7f98000)
      libmpc.so.2 => /opt/brcm-arm/lib/libmpc.so.2 (0xf7f7f000)
      libmpfr.so.4 => /opt/brcm-arm/lib/libmpfr.so.4 (0xf7f30000)
      libgmp.so.10 => /opt/brcm-arm/lib/libgmp.so.10 (0xf7ed3000)
      libdl.so.2 => /lib/i386-linux-gnu/libdl.so.2 (0xf7ead000)
      libelf.so.1 => /lib/i386-linux-gnu/libelf.so.1 (0xf7e8f000)
      libc.so.6 => /lib/i386-linux-gnu/libc.so.6 (0xf7ca6000)
      libm.so.6 => /lib/i386-linux-gnu/libm.so.6 (0xf7ba2000)
      /lib/ld-linux.so.2 (0xf7f9a000)
      libz.so.1 => /lib/i386-linux-gnu/libz.so.1 (0xf7b85000)
```



## Cross Compiling 

I would like to deal with it one file by one file.

### minieap/config.mk

`config.mk` is used for compiling just like its filename. I would like a simple ~~and violent~~ method, just directly add this on the last of the file:

```
CC := ~/am-toolchains/brcm-arm-sdk/hndtools-arm-linux-2.6.36-uclibc-4.5.3/bin/arm-uclibc-linux-2.6.36-gcc
```

As I failed for many times, even I know I shouldn't do like this, but I am a little bit annoyed at last. **Plz adjust the paths for your own environment if necessary**.

Also remove the "#" if front of `PLUGIN_MODULES += ifaddrs`, as we need to use it later.

After all, the `config.mk` without comments looks like this:

```
PLUGIN_MODULES := \
  packet_plugin_printer \
  packet_plugin_rjv3

PLUGIN_MODULES += if_impl_sockraw

ENABLE_DEBUG  := false
ENABLE_ICONV  := false
ENABLE_GBCONV := true
STATIC_BUILD  := false

LIBICONV_STANDALONE := false

CUSTOM_CFLAGS :=
CUSTOM_LDFLAGS :=
CUSTOM_LIBS :=

CC := ~/am-toolchains/brcm-arm-sdk/hndtools-arm-linux-2.6.36-uclibc-4.5.3/bin/arm-uclibc-linux-2.6.36-gcc
PLUGIN_MODULES += ifaddrs
```

### minieap/Makefile

If you run `make` directly, you will get an error about `-Wpedantic`, which isn't supported by the version of gcc. Just replace it with `-std=gnu99`:

```
4c4
< COMMON_CFLAGS := $(CUSTOM_CFLAGS) $(CFLAGS) -Wall -Wpedantic -D_GNU_SOURCE
---
> COMMON_CFLAGS := $(CUSTOM_CFLAGS) $(CFLAGS) -Wall -std=gnu99 -D_GNU_SOURCE
```

(You can find the tip `use option -std=c99 or -std=gnu99 to compile your code` if you just remove `-Wpedantic`, but it seems that using `-std=c99` will cause some syntax errors.)

### ifaddrs

Then if you start compiling, you will get this sh*t:

```
util/net_util.o: In function `obtain_iface_mac':
net_util.c:(.text+0xa8): undefined reference to `getifaddrs'
net_util.c:(.text+0x180): undefined reference to `freeifaddrs'
util/net_util.o: In function `obtain_iface_ip_mask':
net_util.c:(.text+0x1b0): undefined reference to `getifaddrs'
net_util.c:(.text+0x36c): undefined reference to `freeifaddrs'
collect2: ld returned 1 exit status
```

It doesn't find those functions that the platform doesn't provide, so we need to add those files ourselves. That's why I add `PLUGIN_MODULES += ifaddrs` in `config.mk` before, as it enables us to use those files.

Don't worry, we still have mighty GitHub. Many file can be found in GitHub. I used this [ifaddrs.c](https://github.com/SWRT-dev/bluecave-asuswrt/blob/master/release/src/router/smartdns/src/lib/ifaddrs.c) and this [ifaddrs.h](https://github.com/lattera/glibc/blob/master/inet/ifaddrs.h). I know little about C or C++, so I can't give too much advice. Add `ifaddrs.h` in `includes/` and add `ifaddrs.c` in `util/ifaddrs/`. Don't forget to edit `config.mk` if you didn't do it before. 

After all things are done, enter `make` and start compiling. Compiling MiniEAP is pretty fast. You can type `file minieap` to check if everything is fine.

![file](/pics/2021-06-06/file.png)

## RUN!

Using `scp` or any other method to upload the file to your router. Type `./minieap` you will get a message asking you to login. With `-w` , you can save the configuration to `/etc/minieap.conf`. Don't forget to move your files **(one is your `minieap` program and another is the `/etc/minieap.conf`)** to `/jffs`, or the files will disappear after rebooting.

There're some problems:

### Auto Reboot

To use `crontab` in `Arsuswrt-Merlin` is a complex thing, as everything not in `/jffs` will be delete. However, we have [this](https://github.com/RMerl/asuswrt-merlin.ng/wiki/Scheduled-Reboot) in wiki. Just refer it.

### Auto Reauth

According to this [issue](https://github.com/updateing/minieap/issues/43), you need to delete `no-auto-reauth` to enable auto reauth. It is simple but I still can't reauth, but it is worth trying.

My solution is ping. Here is my script:

```bash
#!/bin/sh
while true
do
	ping -q -c 3 cn.bing.com >> /dev/null
	if [ $? -eq 0 ]
	then
		echo "[`date`]Everything is fine."
	else
		echo "[`date`]Failed. Restarting minieap." >> /var/log/watchdog.log
		/jffs/minieap/minieap
	fi
done
```

It means if the router cannot ping `cn.bing.com` successfully, it will write a log in `/var/log/watchdog.log` and restart MiniEAP. In fact I don't want any outputs, as my MiniEAP disconnects every 20 seconds...



![log](/pics/2021-06-06/log.png)

In User scripts, we have `wan-script`, add this three lines in the script:

```
cp /jffs/minieap/minieap.conf /etc/
/jffs/minieap/minieap
/jffs/minieap/watchdog.sh
```

**Adjust the paths for your own environment if necessary**.

`wan-script` contains the scripts that will run after WAN interface came up, in new version of `Asuswrt-Merlin`, it have been replaced by `wan-start`. The first line means copy new `minieap.conf` to `/etc`, as everything has gone after rebooting. Then it restarts `MiniEAP`. Finally, the `watchdog.sh` will run to keep auth.

Now enjoy your Campus Internet. Everything is fine.


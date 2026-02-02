# Setting up BSD-like systems

* [Supported versions](#supported-versions)
* [Features of BSD systems](#features-bsd-systems)
    * [No nfqueue](#no-nfqueue)
    * [Firewall types](#firewall types)
    * [Build](#build)
    * [Divert sockets](#divert-sockets)
    * [Lookup Tables](#lookup-tables)
    * [Loading ip tables from a file](#loading-ip-tables-from-file)
    * [No splice](#no-splice)
    * [mdig and ip2net](#mdig-and-ip2net)
* [FreeBSD](#freebsd)
    * [Uploading ipdivert](#uploading-ipdivert)
    * [Auto-restore ipfw rules and work in the background](#auto-restore-ipfw-rules-and-work-in-the-background)
    * [tpws in transparent mode](#tpws-in-transparent-mode)
    * [Launch dvtws](#launch-dvtws)
    * [PF в FreeBSD](#pf-в-freebsd)
    * [pfsense](#pfsense)
* [OpenBSD](#openbsd)
    * [tpws bind on ipv4](#tpws-bind-on-ipv4)
    * [tpws for passing traffic (old systems)](#tpws-for-passing-traffic-old-scheme-does-not-work-in-new-versions))
    * [tpws for passing traffic (new systems)](#tpws-for-passing-traffic-new-systems))
    * [Launch dvtws](#launch-dvtws)
    * [Problems with badsum](#problems-with-badsum)
    * [Feature of sending fake packets](#feature of sending-fake-packets)
    * [Reload PF tables](#reload-pf-tables)
* [MacOS](#macos)
    * [Introduction](#introduction)
    * [dvtws is useless](#dvtws is useless)
    * [tpws](#tpws)
    * [Problem link-local address](#problem-link-local-address)
    * [Build](#build)
    * [Easy installation](#easy-installation)
    * [Option Custom](#option-custom)


## Supported Versions
**FreeBSD** 11.x+ , **OpenBSD** 6.x+, partially **MacOS Sierra** +

> [!CAUTION]
> On older ones it may or may not work, it may or may not work.
> work. On **FreeBSD** 10, `dvtws` builds and works. With `tpws` there is
> problems due to the clang compiler version being too old. There probably will be
> work if you update the compiler. Can be added to the latest versions
> pfsense without a web interface in manual mode via the console.


## Features of BSD systems

### Missing nfqueue
There is no `nfqueue` in **BSD**. A similar mechanism is divert sockets. From the catalog
[`nfq/`](../nfq/) under **BSD** is compiled into `dvtws` instead of `nfqws`. He shares with
`nfqws` for most of the code and is almost identical in terms of command line options.

### Firewall Types
**FreeBSD** contains 3 firewalls: **IPFilter**, **ipfw** and **Packet Filter (PF
in the future)**. **OpenBSD** contains only **PF**.

### Assembly
Under **FreeBSD** `tpws` and `dvtws` are built using `make`.

Under **OpenBSD**:
```sh
make bsd
```

Under **MacOS**:
```sh
make mac
```

**FreeBSD** make recognizes BSDmakefiles, **OpenBSD** and **MacOS** do not. That's why
it uses a separate target in the Makefile. Build all sources:
```sh
make -C /opt/zapret
```

### Divert sockets
The Divert socket is an internal **BSD** kernel socket type. He is not attached to anyone
what network address, does not participate in data exchange through the network and
identified by port number `1..65535`. Queue number analogy
`NFQUEUE`. Traffic is turned on divert sockets using ipfw rules or
PF. If there is a divert rule in the firewall, but no one is listening on the divert port,
then the packets are dropped. This behavior is similar to the `NFQUEUE` rules without a parameter
`--queue-bypass`. On **FreeBSD** divert sockets can only be ipv4, although on
They accept both ipv4 and ipv6 frames. On **OpenBSD** divert sockets are created
separately for ipv4 and ipv6 and only work with one version of `ip` each. On
**MacOS** looks like divert sockets have been removed from the kernel. See the section about
**MacOS**. Sending to a divert socket works similarly to sending through a raw socket
on linux. The entire IP frame is transmitted, starting with the IP header. These features
are taken into account in `dvtws`.

### Lookup Tables
Scripts [`ipset/*.sh`](../ipset/) if ipfw is available, work with ipfw lookup
tables. This is a direct analogue of ipset. lookup tables are not separated into v4 and v6. They
can contain v4 and v6 addresses and subnets at the same time. If ipfw is missing,
then the action depends on the `LISTS_RELOAD` variable in config. If it is given, then
The command from `LISTS_RELOAD` is executed. Otherwise, nothing is done.
If `LISTS_RELOAD=-`, then table filling is disabled even if ipfw is present.

### Loading IP tables from a file
PF can load ip tables from a file. To use this opportunity
You should disable gzip compression for sheets via a config file option:
`GZIP_LISTS=0`.

### Lack of splice
**BSD** does not contain the splice system call. `tpws` works via forwarding
data in user mode both ways. It's slower, but not critical. Control
asynchronous sockets in `tpws` is based on the linux-specific epoll mechanism. IN
**BSD** to emulate it, epoll-shim is used - a layer for epoll emulation
based on kqueue.

### mdig and ip2net
mdig and ip2net are fully functional in **BSD**. There's nothing in them
system-dependent.


## FreeBSD

### Loading ipdivert
Divert sockets require a special kernel module, `ipdivert`.

- Place the following lines in `/boot/loader.conf` (create if missing):
```
ipdivert_load="YES"
net.inet.ip.fw.default_to_accept=1
```

`/etc/rc.conf`:
```
firewall_enable="YES"
firewall_script="/etc/rc.firewall.my"
```

`/etc/rc.firewall.my`:
```sh
$ ipfw -q -f flush
```

### Auto-recovery of ipfw rules and work in the background
In `/etc/rc.firewall.my` you can add ipfw rules so that they
restored after reboot. You can also run daemons from there.
zapret by adding `--daemon` to the parameters. For example like this:
```sh
$ pkill ^dvtws$
$ /opt/zapret/nfq/dvtws --port=989 --daemon --dpi-desync=multisplit --dpi-desync-split-pos=2
```

To restart the firewall and daemons, it will be enough to do:
```sh
$ /etc/rc.d/ipfw restart
```

### tpws in transparent mode
Brief instructions for running `tpws` in transparent mode.

> [!NOTE]  
> It is assumed that the LAN interface is called `em1`, WAN - `em0`.

#### All traffic
```sh
$ ipfw delete 100
$ ipfw add 100 fwd 127.0.0.1,988 tcp from me to any 80,443 proto ip4 xmit em0 not uid daemon
$ ipfw add 100 fwd ::1,988 tcp from me to any 80,443 proto ip6 xmit em0 not uid daemon
$ ipfw add 100 fwd 127.0.0.1,988 tcp from any to any 80,443 proto ip4 recv em1
$ ipfw add 100 fwd ::1,988 tcp from any to any 80,443 proto ip6 recv em1
$ /opt/zapret/tpws/tpws --port=988 --user=daemon --bind-addr=::1 --bind-addr=127.0.0.1
```

#### Traffic only to the zapret table, with the exception of the nozapret table

```sh
$ ipfw delete 100
$ ipfw add 100 allow tcp from me to table\(nozapret\) 80,443
$ ipfw add 100 fwd 127.0.0.1,988 tcp from me to table\(zapret\) 80,443 proto ip4 xmit em0 not uid daemon
$ ipfw add 100 fwd ::1,988 tcp from me to table\(zapret\) 80,443 proto ip6 xmit em0 not uid daemon
$ ipfw add 100 allow tcp from any to table\(nozapret\) 80,443 recv em1
$ ipfw add 100 fwd 127.0.0.1,988 tcp from any to any 80,443 proto ip4 recv em1
$ ipfw add 100 fwd ::1,988 tcp from any to any 80,443 proto ip6 recv em1
$ /opt/zapret/tpws/tpws --port=988 --user=daemon --bind-addr=::1 --bind-addr=127.0.0.1
```

> [!NOTE]  
> The zapret, nozapret, ipban tables are created by scripts from ipset, similar to
> Linux. Scripts can be updated in cron as root:
> ```sh
> $ crontab -e
> ```
>
> ```
> <...>
> 0 12 */2 * * /opt/zapret/ipset/get_config.sh
> ```

> [!CAUTION]  
> When using ipfw, `tpws` does not require elevated privileges to implement
> transparent mode. However, without root it is impossible to bind to ports `< 1024` and
> shift of UID/GID. Without shifts UID will recurses, the fortune of ipfw necessary
> create based on the UID under which `tpws` runs. Port Forwarding
> `>= 1024` may pose a threat to unprivileged traffic interception
> process, if suddenly `tpws` is not running.


### Launch dvtws

#### All traffic
```sh
$ ipfw delete 100
$ ipfw add 100 divert 989 tcp from any to any 80,443 out not diverted xmit em0
# required for autottl mode only
$ ipfw add 100 divert 989 tcp from any 80,443 to any tcpflags syn,ack in not diverted recv em0
$ /opt/zapret/nfq/dvtws --port=989 --dpi-desync=multisplit --dpi-desync-split-pos=2
```

#### Traffic only to the zapret table, with the exception of the nozapret table

```sh
$ ipfw delete 100
$ ipfw add 100 allow tcp from me to table\(nozapret\) 80,443
$ ipfw add 100 divert 989 tcp from any to table\(zapret\) 80,443 out not diverted not sockarg xmit em0
# required for autottl mode only
$ ipfw add 100 divert 989 tcp from table\(zapret\) 80,443 to any tcpflags syn,ack in not diverted not sockarg recv em0
$ /opt/zapret/nfq/dvtws --port=989 --dpi-desync=multisplit --dpi-desync-split-pos=2
```


### PF in FreeBSD
The setup is similar to **OpenBSD**, but there are important nuances.

- On **FreeBSD** PF support in `tpws` is disabled by default. To her
inclusive, need to utilize the parameter of `-enable-pf`.
- You cannot make ipv6 rdr to `::1`. You need to set the incoming address to link-local
interface. Look through `ifconfig` for the address `fe80:...` and add it to the rule.
- The syntax for `pf.conf` is slightly different. Newer version of PF.
- The limit on the number of table elements is set as follows:
  ```sh
  $ sysctl net.pf.request_maxcount=2000000
  ```
- Broken divert-to. It works, but the prevention mechanism does not work
looping. Someone already wrote a patch, but the problem is still in `14-RELEASE`
There is. Therefore, running `dvtws` through PF is not possible at this time.

  `/etc/pf.conf`:
  ```
  rdr pass on em1 inet6 proto tcp to port {80,443} -> fe80::31c:29ff:dee2:1c4d port 988
  rdr pass on em1 inet  proto tcp to port {80,443} -> 127.0.0.1 port 988
  ```

  ```sh
  $ /opt/zapret/tpws/tpws --port=988 --enable-pf --bind-addr=127.0.0.1 --bind-iface6=em1 --bind-linklocal=force
  ```

> [!NOTE]  
 > In PF it is not possible to do rdr-to from the same system where the proxy is running.
 > The route-to option does not save meta information. The destination address is lost.
 > Therefore, this option is suitable for squid, which takes the address from the application layer protocol, but not suitable for tpws, which relies on OS metadata.
 > rdr-to support is implemented via `/dev/pf`, so transparent mode **requires root**.


### pfsense

#### Description
pfsense is based on **FreeBSD** and uses the PF firewall, which has problems with
divert. Fortunately, the ipfw and ipdivert modules are included in the delivery of the latter
pfsense versions. They can be loaded via `kldload`.

Some older versions of pfsense require the firewall order to be reordered
via `sysctl`, making ipfw first. In newer ones these parameters are `sysctl`
are missing, but the system works as it should without them. In some cases
The PF firewall may limit the capabilities of `dvtws`, particularly in the area
IP fragmentation.

There are scrub rules by default for reassembling fragments.

Binaries from [`binaries/freebsd-x64`](../binaries/freebsd-x64) are compiled under
**FreeBSD 11**. They should work on subsequent versions of **FreeBSD**,
including pfsense. You can use `install_bin.sh`.

#### Autorun
An example of an autorun script is in [`init.d/pfsense`](../init.d/pfsense). His
should be placed in `/usr/local/etc/rc.d` and edited for rules
ipfw and launching daemons. There is a built-in editor `edit` as more acceptable
alternative to `vi'.

> [!NOTE]  
> Since there is no `git`, the most convenient way to copy files is through `ssh`.
> `curl` is present by default. You can copy zip with zapret and
> unpack to `/opt`, as is done on other systems. Then `dvtws`
> нужно запускать как `/opt/zapret/nfq/dvtws`. Либо скопировать только `dvtws`
> to `/usr/local/sbin`. Whatever you prefer.

> [!NOTE]  
> The ipset scripts work, there is cron. You can auto-update sheets.

> [!NOTE]  
> If you are bothered by the poverty of the existing repository, you can enable
> repository from **FreeBSD**, which is disabled by default.
>
> Change `no` to `yes` in `/usr/local/etc/pkg/repos/FreeBSD.conf`
>
> You can install all the usual software, including `git`, to download directly
> close with github.

`/usr/local/etc/rc.d/zapret.sh`  (chmod `755`):
```sh
#!/bin/sh

kldload ipfw
kldload ipdivert

# for older pfsense versions. newer do not have these sysctls
sysctl net.inet.ip.pfil.outbound=ipfw,pf
sysctl net.inet.ip.pfil.inbound=ipfw,pf
sysctl net.inet6.ip6.pfil.outbound=ipfw,pf
sysctl net.inet6.ip6.pfil.inbound=ipfw,pf

ipfw delete 100
ipfw add 100 divert 989 tcp from any to any 80,443 out not diverted xmit em0
pkill ^dvtws$
dvtws --daemon --port 989 --dpi-desync=multisplit --dpi-desync-split-pos=2

# required for newer pfsense versions (2.6.0 tested) to return ipfw to functional state
pfctl -d ; pfctl -e
```

#### tpws problems
As for `tpws`, there seems to be some conflict between the two firewalls, and
fwd rules in ipfw do not work. Redirection using PF works like this:
described in the section on **FreeBSD**. In PF you can only change rules as a whole
blocks - anchors. You can't just add or remove something. But
In order for an anchor to be processed, there must be a link to it from the main one
set of rules. You cannot touch it, otherwise the entire firewall will collapse. That's why
you will have to edit the pfsense script code.

1. Edit `/etc/inc/filter.inc` as follows:
```
	<...>
	/* MOD */
	$natrules .= "# ZAPRET redirection\n";
	$natrules .= "rdr-anchor \"zapret\"\n";

	$natrules .= "# TFTP proxy\n";
	$natrules .= "rdr-anchor \"tftp-proxy/*\"\n";
	<...>
```

2. Write a file with the contents of the anchor (for example, `/etc/zapret.anchor`):
```
rdr pass on em1 inet  proto tcp to port {80,443} -> 127.0.0.1 port 988
rdr pass on em1 inet6 proto tcp to port {80,443} -> fe80::20c:29ff:5ae3:4821 port 988
```

`fe80::20c:29ff:5ae3:4821` replace with your link local address of the LAN interface,
or remove the line if ipv6 is not needed.

3. Add to autorun `/usr/local/etc/rc.d/zapret.sh`:
```sh
$ pfctl -a zapret -f /etc/zapret.anchor
$ pkill ^tpws$
$ tpws --daemon --port=988 --enable-pf --bind-addr=127.0.0.1 --bind-iface6=em1 --bind-linklocal=force --split-pos=2
```

4. After reboot, check that the rules have been created:
```sh
$ pfctl -s nat
no nat proto carp all
nat-anchor "natearly/*" all
nat-anchor "natrules/*" all
<...>
no rdr proto carp all
rdr-anchor "zapret" all
rdr-anchor "tftp-proxy/*" all
rdr-anchor "miniupnpd" all

$ pfctl -s nat -a zapret
rdr pass on em1 inet proto tcp from any to any port = http -> 127.0.0.1 port 988
rdr pass on em1 inet proto tcp from any to any port = https -> 127.0.0.1 port 988
rdr pass on em1 inet6 proto tcp from any to any port = http -> fe80::20c:29ff:5ae3:4821 port 988
rdr pass on em1 inet6 proto tcp from any to any port = https -> fe80::20c:29ff:5ae3:4821 port 988
```

> [!NOTE]  
> There is also a more elegant way to run `tpws` via @reboot in cron and
> redirection rule in UI. This will avoid editing the pfsense code.


## OpenBSD

### tpws bind on ipv4
In `tpws` bind by default only on ipv6. For bind on ipv4 you need to specify `--bind-addr=0.0.0.0`.
Use `--bind-addr=0.0.0.0 --bind-addr=::` to achieve the same result as other OS defaults.
But it’s better not to do this, but to land on certain internal addresses or interfaces.


### tpws for passing traffic (the old scheme does not work in new versions)

In this variant, tpws explicitly contacts the pf redirector and tries to obtain the original destination address from it.
As practice shows, this does not work on new versions of OpenBSD. An ioctl error is returned.
The latest tested version where this works is 6.8. Between 6.8 and 7.4 the developers broke this mechanism.

`/etc/pf.conf`:
```
pass in quick on em1 inet  proto tcp to port {80,443} rdr-to 127.0.0.1 port 988
pass in quick on em1 inet6 proto tcp to port {80,443} rdr-to ::1 port 988
```

```sh
$ pfctl -f /etc/pf.conf
$ tpws --port=988 --user=daemon --bind-addr=::1 --bind-addr=127.0.0.1 --enable-pf
```

> [!NOTE]
> In PF it is not possible to do rdr-to from the same system where the proxy is running.
> The route-to option does not save meta information. The destination address is lost.
> Therefore, this option is suitable for squid, which takes the address from the application layer protocol, but not suitable for tpws, which relies on OS metadata.
> rdr-to support is implemented via `/dev/pf`, so transparent mode **requires root**.

### tpws for passing traffic (new systems)

Newer versions suggest using divert-to instead of rdr-to.
The minimum tested version where this works is 7.4. May or may not work on older ones - no study done.

`/etc/pf.conf`:
```
pass on em1 inet proto tcp to port {80,443} divert-to 127.0.0.1 port 989
pass on em1 inet6 proto tcp to port {80,443} divert-to ::1 port 989
```

tpws must have a bind to exactly the address specified in the pf rules. `0.0.0.0` or `::` does not work.

```sh
$ pfctl -f /etc/pf.conf
$ tpws --port=988 --user=daemon --bind-addr=::1 --bind-addr=127.0.0.1
```

> [!NOTE]
> It’s also not clear how to do divert from the system itself where tpws works.

### Launch dvtws

#### All traffic
`/etc/pf.conf`:
```
pass in  quick on em0 proto tcp from port {80,443} flags SA/SA divert-packet port 989 no state
pass in  quick on em0 proto tcp from port {80,443} no state
pass out quick on em0 proto tcp to   port {80,443} divert-packet port 989 no state
```

```sh
$ pfctl -f /etc/pf.conf
$ ./dvtws --port=989 --dpi-desync=multisplit --dpi-desync-split-pos=2
```

#### Traffic only to the zapret table, with the exception of the nozapret table

`/etc/pf.conf`:
```
set limit table-entries 2000000
table <zapret> file "/opt/zapret/ipset/zapret-ip.txt"
table <zapret-user> file "/opt/zapret/ipset/zapret-ip-user.txt"
table <nozapret> file "/opt/zapret/ipset/zapret-ip-exclude.txt"
pass out quick on em0 inet  proto tcp to   <nozapret> port {80,443}
pass in  quick on em0 inet  proto tcp from <zapret>  port {80,443} flags SA/SA divert-packet port 989 no state
pass in  quick on em0 inet  proto tcp from <zapret>  port {80,443} no state
pass out quick on em0 inet  proto tcp to   <zapret>  port {80,443} divert-packet port 989 no state
pass in  quick on em0 inet  proto tcp from <zapret-user>  port {80,443} flags SA/SA divert-packet port 989 no state
pass in  quick on em0 inet  proto tcp from <zapret-user>  port {80,443} no state
pass out quick on em0 inet  proto tcp to   <zapret-user>  port {80,443} divert-packet port 989 no state
table <zapret6> file "/opt/zapret/ipset/zapret-ip6.txt"
table <zapret6-user> file "/opt/zapret/ipset/zapret-ip-user6.txt"
table <nozapret6> file "/opt/zapret/ipset/zapret-ip-exclude6.txt"
pass out quick on em0 inet6 proto tcp to   <nozapret6> port {80,443}
pass in  quick on em0 inet6 proto tcp from <zapret6> port {80,443} flags SA/SA divert-packet port 989 no state
pass in  quick on em0 inet6 proto tcp from <zapret6> port {80,443} no state
pass out quick on em0 inet6 proto tcp to   <zapret6> port {80,443} divert-packet port 989 no state
pass in  quick on em0 inet6 proto tcp from <zapret6-user>  port {80,443} flags SA/SA divert-packet port 989 no state
pass in  quick on em0 inet6 proto tcp from <zapret6-user>  port {80,443} no state
pass out quick on em0 inet6 proto tcp to   <zapret6-user> port {80,443} divert-packet port 989 no state
```

```sh
$ pfctl -f /etc/pf.conf
$ ./dvtws --port=989 --dpi-desync=multisplit --dpi-desync-split-pos=2
```


### Problems with badsum
**OpenBSD** forces tcp checksum to be recalculated after divert, so
most likely `dpi-desync-fooling=badsum` will not work for you. When using
This parameter `dvtws` will warn about a possible problem.


### Feature of sending fake packets
In **OpenBSD** `dvtws` sends all fakes via divert socket, since this
the option via raw sockets is blocked. Apparently PF is automatic
prevents diverted frames from looping again, so looping problems
No.

divert-packet automatically implements the opposite rule for redirection. Trick with
no state and in rule allows you to bypass this problem, so as not to drive in vain
massive traffic through `dvtws`.


### Reloading PF tables
Scripts from ipset do not reload the tables to PF by default.

Чтобы они это делали, добавьте параметр в `/opt/zapret/config`:
```
LISTS_RELOAD="pfctl -f /etc/pf.conf"
```

Newer versions of `pfctl` understand the command to reload tables only. But this does not apply to **OpenBSD**. The new **FreeBSD** does.
```sh
$ pfctl -Tl -f /etc/pf.conf
```

> [!IMPORTANT]  
> Don't forget to turn off gzip compression: `GZIP_LISTS=0`

> [!IMPORTANT]  
> If there is no sheet file in your configuration, then you need it
> excluded from the PF rules. If suddenly there is no sheet, and it is specified in pf.conf, there will be
> firewall restart error.

> [!NOTE]
> Once configured, sheet updates can be placed in cron:
> ```sh
> $ crontab -e
> ```
>
> ```
> <...>
> 0 12 */2 * * /opt/zapret/ipset/get_config.sh
> ```


## MacOS

### Introduction
Initially, the kernel of this OS "darwin" was based on **BSD**, because it has a lot
similar to other versions of **BSD**. However, as in other mass commercial
projects, priorities shift away from the original. Yabloko people want what they want
and create.


### dvtws is useless
There used to be ipfw, then it was removed and replaced with PF. There are doubts that divert
the sockets in the kernel remain. Trying to create a divert socket does not give any errors, but
the resulting socket behaves exactly like raw, with all its inherited
blunts + also apple-specific ones. Divert-packet does not work in PF. Simple
grep of the `pfctl` binary shows that there is no word "divert" there, but in others
**BSD** versions have it. `dvtws` is built, but completely useless.


### tpws
`tpws` was adapted and is functional. Obtaining the destination address for
transparent proxy in PF (`DIOCNATLOOK`) was removed from the headers in the new SDK,
making it virtually undocumented.

Some definitions from older versions of the apple SDK have been moved to `tpws`.
We managed to establish a transparent regime with them. However, what will happen in the next versions
difficult to guess. There are no guarantees. Another feature of PF in **MacOS** is
check for root at the time of access to `/dev/pf`, which is not the case in other **BSD**.
`tpws` resets root privileges by default. Must be explicitly specified
parameter `--user=root`. Otherwise, PF behaves similarly to **FreeBSD**.
The syntax of `pf.conf` is the same.

> [!IMPORTANT]  
> On **MacOS** the redirect works both from passing traffic and from local
> systems via route-to. Since `tpws` is forced to run as root, for
> To avoid recursion, you have to allow traffic coming from root directly.
> Hence we have a drawback - **DPI bypass for root will NOT work**.

#### Work in transparent mode only for outgoing requests
`/etc/pf.conf`:
```
rdr pass on lo0 inet  proto tcp from !127.0.0.0/8 to any port {80,443} -> 127.0.0.1 port 988
rdr pass on lo0 inet6 proto tcp from !::1 to any port {80,443} -> fe80::1 port 988
pass out route-to (lo0 127.0.0.1) inet proto tcp from any to any port {80,443} user { >root }
pass out route-to (lo0 fe80::1) inet6 proto tcp from any to any port {80,443} user { >root }
```

```sh
$ pfctl -ef /etc/pf.conf
$ /opt/zapret/tpws/tpws --user=root --port=988 --bind-addr=127.0.0.1 --bind-iface6=lo0 --bind-linklocal=force
```

#### Working in transparent mode
> [!NOTE]
> It is assumed that the LAN interface name is `en1`

```sh
$ ifconfig en1 | grep fe80
        inet6 fe80::bbbb:bbbb:bbbb:bbbb%en1 prefixlen 64 scopeid 0x8
```

`/etc/pf.conf`:
```
rdr pass on en1 inet  proto tcp from any to any port {80,443} -> 127.0.0.1 port 988
rdr pass on en1 inet6 proto tcp from any to any port {80,443} -> fe80::bbbb:bbbb:bbbb:bbbb port 988
rdr pass on lo0 inet  proto tcp from !127.0.0.0/8 to any port {80,443} -> 127.0.0.1 port 988
rdr pass on lo0 inet6 proto tcp from !::1 to any port {80,443} -> fe80::1 port 988
pass out route-to (lo0 127.0.0.1) inet proto tcp from any to any port {80,443} user { >root }
pass out route-to (lo0 fe80::1) inet6 proto tcp from any to any port {80,443} user { >root }
```

```sh
$ pfctl -ef /etc/pf.conf
$ /opt/zapret/tpws/tpws --user=root --port=988 --bind-addr=127.0.0.1 --bind-iface6=lo0 --bind-linklocal=force --bind-iface6=en1 --bind-linklocal=force
```


### Link-local address problem
If you use **MacOS** as an ipv6 router, you will need
solve the issue with regularly changing link-local address. From some versions
**MacOS** uses permanent "secured" ipv6 addresses by default instead
generated based on MAC addresses.

Everything is great, but there is one problem. Only global remains constant
scope addresses. Link locals change periodically. The change is tied to the system
time. Reboots do not change the address, But if you move the time forward a day and
reboot - link local will become different (at least in vmware this is so).
There is very little information on the issue, but it looks like a bug. Should not change link
local. There is no point in hiding link local, but dynamic link local is not allowed
use as gateway address. The easiest way is to abandon "secured"
addresses. To do this, place the line `net.inet6.send.opmode=0` in
`/etc/sysctl.conf` and reboot the system.

All the same, temporary addresses will be used for outgoing connections, just like
in other systems. Or you don’t like the idea, you can register an additional
static ipv6 from the mask range `fc00::/7` - select any with length
prefix `128`. This can be done in system settings by creating an additional
adapter based on the same network interface, disable ipv4 in it and enter
static ipv6. It will be added to the automatically configured ones.


### Assembly
```sh
$ make -C /opt/zapret mac
```


### Easy installation
**MacOS** supports `install_easy.sh`

The kit includes binaries compiled for 64-bit with the option
`-mmacosx-version-min=10.8`. They should work on all supported versions
**MacOS**. If suddenly they don’t work, you can collect your own. Developer tools
are installed automatically when you run `make`.

> [!WARNING]  
> Internet sharing using the system **not supported**!
>
> Only a router configured on its own via PF is supported. If you
> suddenly turned on sharing and then turned it off, then access to sites may be lost
> at all.
>
> Treatment:
> ```sh
> $ pfctl -f /etc/pf.conf
> ```
>
> If you need Internet sharing, it is better to abandon the transparent mode and
> use socks proxy.

For autostart use launchd (`/Library/LaunchDaemons/zapret.plist`)
Управляющий скрипт : `/opt/zapret/init.d/macos/zapret`

The following commands work with `tpws` and the firewall at the same time (if
`INIT_APPLY_FW=1` в config)

```sh
$ /opt/zapret/init.d/macos/zapret start
$ /opt/zapret/init.d/macos/zapret stop
$ /opt/zapret/init.d/macos/zapret restart
```

Work only with tpws:
```sh
$ /opt/zapret/init.d/macos/zapret start-daemons
$ /opt/zapret/init.d/macos/zapret stop-daemons
$ /opt/zapret/init.d/macos/zapret restart-daemons
```

Working only with PF:
```sh
$ /opt/zapret/init.d/macos/zapret start-fw
$ /opt/zapret/init.d/macos/zapret stop-fw
$ /opt/zapret/init.d/macos/zapret restart-fw
```

Reloading all IP tables from files:
```sh
$ /opt/zapret/init.d/macos/zapret reload-fw-tables
```

> [!NOTE]  
> The installer configures `LISTS_RELOAD` in config, so the scripts
> [`ipset/*.sh`](../ipset/) automatically reload IP tables in PF.

> [!NOTE]  
> A cron job is automatically created on
> [`ipset/get_config.sh`](../ipset/get_config.sh), similar to openwrt.

When start-fw the script automatically modifies `/etc/pf.conf` by inserting there
anchors "banned". The modification is designed for `pf.conf`, which contains
default anchors from apple. If you have modified `pf.conf` and the modification is not
succeeded, there will be a warning about this. Don't ignore him. In this case you
you need to insert into your `pf.conf` (according to the order of the rule types):
```
rdr-anchor "zapret"
anchor "zapret"
```

> [!NOTE]  
> When uninstalling via `uninstall_easy.sh`, modifications to `pf.conf` are removed.

> [!NOTE]  
> start-fw creates 3 anchors files in `/etc/pf.anchors`: `zapret`, `zapret-v4`,
> `banned-v6`. The last 2 are connected from anchor "zapret".

> [!NOTE]  
> Tables `nozapret` and `nozapret6` belong to anchor "zapret".
>
> Таблиц `zaprā` `zapret-sear` в anch 'zap-v4'.
>
> Таблиц `zapret6` `zapret6-short` в anch 'zap-v6.
>
> If some version of the protocol is disabled, the corresponding anchor is empty and not
> mentioned in anchor "banned". Tables and rules are created only for those
> sheets that are actually in the ipset directory.


### Option Custom
Just like in other systems supported in the simple installer, you can
create your own custom scripts.

Расположение: `/opt/zapret/init.d/macos/custom`

`zapret_custom_daemons()` gets in `$1`: `0` or `1`. `0` = stop, `1` = start

custom firewall is different from the linux version. Instead of filling in `iptables` you
you need to generate rules for `zapret-v4` and `zapret-v6` anchors and issue them to
stdout. This is done in the `zapret_custom_firewall_v4()` and
`zapret_custom_firewall_v6()`. Table definitions are populated by the main script
\- you don't need to do this. You can reference the `zapret` and `zapret-user` tables
в v4, `zapret6` и `zapret6-shere`.

Cm. example [in file](../init.d/macos/custom.d.examples/50-extra-tpws).

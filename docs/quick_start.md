# Quick Linux/OpenWrt setup

> [!CAUTION]  
> Do not write in issue questions like “how to copy a file”, “how to download”, “how
> run", etc. That is, everything related to basic skills in handling the OS
> Linux. These questions will be closed immediately. If you have similar questions
> arise, I recommend not using this software or looking for help somewhere in
> another place. I can say the same thing to those who want to press 1 button to
> everything worked, and does not want to read and study at all. Alas, this was not delivered and
> they won't give you a ride. Look for other easier workarounds. This method **is not for
> ordinary user**.


## Introduction
Especially for those who want to get started quickly, but don't want to go too deep
into the sheet [readme.md](readme.md).

DPI bypass is a hacking technique. This word means a method
which is actively opposed and therefore is not automatically
Guaranteed performance in any conditions and on any resources, required
customization for specific conditions of your provider. Conditions may vary
over time, and the technique may begin or stop working, may
a re-analysis of the situation will be required. Individuals may be found
resources that are otherwise blocked and that do not work or have stopped
work. Some non-blocked resources may also break. Therefore very
It is advisable to have knowledge in the field of networks in order to be able to
analyze the technical situation. It wouldn't hurt to have bypass channels
proxying traffic in case DPI bypass does not help.

The option when you found a strategy somewhere on the Internet and are trying to adapt it to your case is obviously problematic.
There is no universal pill. Everywhere the situation is different. There are outright nonsense written by someone floating around the Internet, which is being replicated en masse by a public that does not understand anything.
Such options most often work unstable, only on some resources, only on some providers, do not work at all or break other resources. In the worst cases, they also flood the network.
Even if an option once worked well, tomorrow it may stop, and outdated information will remain on the network.

We will assume that you have a system based on traditional **linux** or
**openwrt**. If you have traditional Linux, the task is to bypass locks only on
this system, if openwrt - bypass blocking for connected devices. This
the most common case.


## Setup
> [!TIP]  
> For the installation procedure to work normally on openwrt, you need
> count on about 1-2 Mb of free space to install zapret itself and
> necessary additional packages. If there is little space and it is not possible
> increase using `extroot`, you may have to abandon the simple option
> installation and screw it on manually without existing launch scripts.
> You can use [lightweight `tpws` option](../init.d/openwrt-minimal),
> or try to put the required zapret additional packages into a compressed
> image `squashfs` using `image builder` and update the router with this option.

1. Скачайте последний [tar.gz релиз](https://github.com/bol-van/zapret/releases) в /tmp, распакуйте его, затем удалите архив.
For openwrt and firmware, use the `openwrt-embedded` option.
To save space in /tmp, you can download it via curl to stdout and immediately unpack it.
Example for openwrt for version zapret 71.4 (different for other URLs):
   ```
   $ curl -Lo - https://github.com/bol-van/zapret/releases/download/v71.4/zapret-v71.4-openwrt-embedded.tar.gz | tar -zx
   $ wget -O - https://github.com/bol-van/zapret/releases/download/v71.4/zapret-v71.4-openwrt-embedded.tar.gz | tar -zx
   ```
Example for traditional Linux for version zapret 71.4 (different for other URLs):
   ```
   $ curl -Lo - https://github.com/bol-van/zapret/releases/download/v71.4/zapret-v71.4.tar.gz | tar -zx
   $ wget -O - https://github.com/bol-van/zapret/releases/download/v71.4/zapret-v71.4.tar.gz | tar -zx
   ```
curl must be pre-installed. But in any case, it will be needed further.
   Вариант с wget будет работать только если установленный wget поддерживает https.

2. Make sure that you have disabled all means of bypassing blocking, including
himself is prohibited. The `uninstall_easy.sh` script is guaranteed to remove zapret.

3. If you are working in a virtual machine, you must use a connection to
network in bridge mode. NAT is **not** suitable.

4. Perform one-time steps to install the required packages in the OS and
setting up executable files of the correct architecture:
   ```sh
   $ install_bin.sh
   $ install_prereq.sh
   ```

   > You may be asked about the type of firewall (iptables/nftables) and use
   > ipv6. This is necessary to install the correct packages in the OS so that
   > install unnecessary things.

5. Run `blockcheck.sh`. The script first checks DNS. If they are displayed
messages about address spoofing, then you will need to solve the problem with DNS.
`blockcheck.sh` will switch to DoH in this case and will try to get and
use real IP addresses. But if you don't set up a solution to this
problems, the workaround will only work for those programs or OS that themselves
implement SecureDNS mechanisms. The bypass will not work for other programs.

   > Solving the DNS problem is beyond the scope of the project. Usually it is resolved either
   > replacing DNS servers from the provider with public ones (`1.1.1.1`, `8.8.8.8`),
   > or in case of interception by the provider of calls to third-party servers - through
   > special means of encrypting DNS requests, such as `dnscrypt`, `DoT`,
   > `DoH`.
   >
   > Another effective option is to use a resolver from yandex
   > (`77.88.8.88`) on non-standard port `1253`. Many providers do not
   > analyze DNS calls on non-standard ports.
   >
   > You can check if this option works like this:
   > ```sh
   > $ dig -p 53 @77.88.8.88 rutracker.org
   > $ dig -p 1253 @77.88.8.88 rutracker.org
   > ```
   >
   > If the DNS is indeed spoofed and the response to these 2 commands is different,
   > This means the method probably works.
   >
   > In openwrt DNS on a non-standard port can be specified in `/etc/config/dhcp`
   > in this way:
   >
   > ```
   > config dnsmasq
   > 	<...>
   > 	list server '77.88.8.88#1253'
   > ```
   >
   > If the IP and DNS settings are obtained automatically from the provider, in
   > `/etc/config/network` find the `wan` interface section and do this:
   >
   > ```
   > config interface 'wan'
   > 	<...>
   > 	option peerdns '0'
   > ```
   >
   > ```sh
   > $ /etc/init.d/network restart
   > $ /etc/init.d/dnsmasq restart
   > ```
   >
   > If this is not suitable, you can redirect calls to UDP and TCP ports
   > `53` your DNS server to `77.88.8.88:1253` means
   > `iptables`/`nftables`. In `/etc/resolv.conf` you cannot set DNS to
   > non-standard port.

6. `blockcheck.sh` allows you to identify a working strategy for bypassing blocking. By
the results of the script need to understand which option you will use: `nfqws`
or `tpws` and remember the found strategies for future use.

It should be understood that the script checks the availability of only a specific
domain you enter at the beginning with a specific curl program.
Different clients have their own fingerprint. Browsers have one, curl has another.
May or may not use multi-packet TLS with post-quantum cryptography (kyber).
The effectiveness of strategies may depend on this.
Usually other domains are blocked in a similar way, **but not a fact**.
There are special locks. Some parameters require tuning to suit the “common denominator”.
In most cases, you can combine several strategies into one universal one, and this is **extremely
desirable**, but it requires an understanding of how strategies work. prohibited **cannot
break through blocking by IP address**. To check multiple domains, enter
them separated by a space.

   > Nowadays, blockers are installed on main canals. In difficult cases
   > you may have several routes with different lengths along the HOPs, with DPI on
   > different hops. You have to overcome a whole zoo of DPIs, which also
   > are included in the work in a chaotic manner or in a manner dependent on
   > directions (server IP). the script may not always give you the results
   > the optimal strategy, which you simply need to rewrite in the settings. IN
   > in some cases you need to really think about what is happening when analyzing the result
   > on different strategies.
   >
   > Далее, имея понимание что работает на http, https, quic нужно
   > construct launch options `tpws` and/or `nfqws` using
   > multistrategy. How multi-strategies work is described in [readme.md](./readme.md#multiple-strategies).
   >
   > In short, parameters are usually constructed like this:
   > ```sh
   > "--filter-udp=443 'parameters for quic' <HOSTLIST_NOAUTO> --new
   > --filter-tcp=80,443 'объединенные параметры для http и https' <HOSTLIST>"
   > ```
   >
   > Or like this:
   > ```sh
   > "--filter-udp=443 'parameters for quic' <HOSTLIST_NOAUTO> --new
   > --filter-tcp=80 'параметры для http' <HOSTLIST> --new
   > --filter-tcp=443 'параметры для https' <HOSTLIST>"
   > ```
   >
   > `<HOSTLIST>` and `<HOSTLIST_NOAUTO>` are written like this. They are not needed for anything
   > replace. Launch scripts will do this if you have selected the filtering mode by
   > hostlists, and will be removed otherwise. If for some protocol it is necessary
   > fool around with everything without a standard hostlist - just remove `<HOSTLIST>` from there
   > and `<HOSTLIST_NOAUTO>`. You can write your own parameters `--hostlist` and
   > `--hostlist-exclude` for additional hostlists or in profiles
   > specializations for a specific resource. In the latter case, the standard
   > There is no need for a hostlist there. You should avoid specifying your own parameters
   > `--hostlist` to sheets from the ipset directory. This logic is included in
   > `<HOSTLIST>` and `<HOSTLIST_NOAUTO>`. The difference with `<HOSTLIST_NOAUTO>` is that
   > the standard autosheet for this profile is used as usual, that is
   > without automatic addition of domains. However, additions in other
   > profiles are automatically reflected in all others.
   >
   > If the strategies differ in IP protocol version and you cannot
   > combine, the filter is written like this:
   > ```sh
   > "--filter-l3=ipv4 --filter-udp=443 lparameters for quic ipv4' <HOSTLIST_NOAUTO> --new
   > --filter-l3=ipv4 --filter-tcp=80 'параметры для http ipv4' <HOSTLIST> --new
   > --filter-l3=ipv4 --filter-tcp=443 'параметры для https ipv4' <HOSTLIST> --new
   > --filter-l3=ipv6 --filter-udp=443 'parameters for quic ipv6' <HOSTLIST_NOAUTO> --new
   > --filter-l3=ipv6 --filter-tcp=80 'параметры для http ipv6' <HOSTLIST> --new
   > --filter-l3=ipv6 --filter-tcp=443 'параметры для https ipv6' <HOSTLIST>"
   > ```
   >
   > But this is a completely copy-paste option. The more you combine strategies and
   > reduce their total number, the better.
   >
   > If you don't need to fool individual protocols, it would be best to remove
   > extra ports from the traffic interception system through the `TPWS_PORTS` parameters,
   > `NFQWS_PORTS_TCP`, `NFQWS_PORTS_UDP` and remove the corresponding profiles
   > multistrategy.
   >
   > | Protocol | Port | Note |
   > |---|---|---|
   > | `tcp` | `80` | `http` соединение |
   > | `tcp` | `443` | `https` соединение |
   > | `udp` | `443` | `quic` connection |
   >
   > If phase zero desynchronization methods are used (`--mss`,
   > `--wssize`, `--dpi-desync=syndata`) and the `hostlist` filtering mode, then that’s it
   > parameters related to these methods should be placed in separate
   > multi-strategy profiles that will receive control before the name is determined
   > hosta It is necessary to understand the algorithm of how multi-strategies work. The most reliable
   > An option would be to duplicate these parameters into 2 profiles. Some
   > will work depending on the `MODE_FILTER` parameter.
   >
   > ```sh
   > "--filter-tcp=80 'параметры для http' <HOSTLIST> --new
   > --filter-tcp=443 'параметры для https' --wssize 1:6 <HOSTLIST> --new
   > --filter-tcp=443 --wssize 1:6"
   > ```
   >
   > In this example, `wssize` will always be applied to the tcp port `443` outside
   > depending on the `MODE_FILTER` parameter. The hostlist will be ignored,
   > если таковой имеется. К http применять `wssize` вредно и бессмысленно.
   >
   > Sometimes you need to add your own parameters to strategies.
   > For example, you need to change the number of repetitions of fakes or set your own fake.
   > This is done through the shell variables `PKTWS_EXTRA`, `TPWS_EXTRA`.
   >
   > ```PKTWS_EXTRA="--dpi-desync-repeats=10 --dpi-desync-fake-tls=/tmp/tls.bin" ./blockcheck.sh```
   >
   > Trying all the combinations can lead to waiting for weeks, so a reasonable one was chosen
   > the backbone of the check on which you can hang your customizations.

7. Run the easy installation script - `install_easy.sh` Select `nfqws`
and/or `tpws`, then agree to edit the parameters. Will open
editor, where you enter the strategy created at the previous stage.

8. Answer all other questions in `install_easy.sh` according to the output
annotations.

9. Remove the directory from /tmp from which the installation was performed.

## Complete removal

1. Прогоните `/opt/zapret/uninstall_easy.sh`.
2. Agree to remove dependencies in openwrt.
3. Удалите каталог `/opt/zapret`.

## Bottom line
This is the minimum instruction to quickly figure out where to start.
However, this is not a guaranteed solution and in some cases you will not be able to
without knowledge and basic "Talmud". Details and full technical description
are described in [README](readme.md).

If individual **not blocked** resources break, you should add them to
exceptions, or use a restrictive `ipset` or host list. Better
select strategies that cause minimal damage. There are strategies
are quite harmless, but there are very damaging ones that are only suitable for
targeted penetration of individual resources when there is nothing better. good
the strategy can be very disruptive due to poorly selected limiters for fakes
\- ttl, fooling.

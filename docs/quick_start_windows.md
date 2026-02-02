# Windows Quick Setup

Especially for those who want to get started quickly, but don’t want to go too deep into the sheet [readme.md](./readme.md).
> [!CAUTION]  
> As always, computer literacy falls entirely on you.
> You must be able to work with the Windows console and have minimal skills in handling `bat`, `cmd` command files.
> If there is no literacy and a lot of _"how?"_ arises on basic things, then this program is not for you.
> The developer will not answer questions from the computer literacy school series.
> If you still want to continue, ask questions in the github discussions or forums.
> Perhaps someone will help you. But there is no need to write an issue on github. They will close immediately.

## A little clarification

DPI bypass is a hacking technique. This word means a method that is actively opposed and therefore
performance is not automatically guaranteed under any conditions and on any resources,
requires configuration for specific conditions from your provider. Conditions may change over time,
and the technique may begin or stop working, a re-analysis of the situation may be required.
Individual resources that are otherwise blocked may be discovered that are not working or have stopped working.
Some unblocked resources may also break.
Therefore, it is very desirable to have knowledge in the field of networks in order to be able to analyze the technical situation.
It would be a good idea to have traffic proxy bypass channels in case DPI bypass does not help.

The option when you found a strategy somewhere on the Internet and are trying to adapt it to your case is obviously problematic.
There is no universal pill. Everywhere the situation is different. There are outright nonsense written by someone floating around the Internet, which is being replicated en masse by a public that does not understand anything.
Such options most often work unstable, only on some resources, only on some providers, do not work at all or break other resources. In the worst cases, they also flood the network.
Even if an option once worked well, tomorrow it may stop, and outdated information will remain on the network.

You need to be especially careful with third-party assemblies. There may be viruses in there. Not in every build, but scammers have already been noticed.
Video on YouTube on how to easily bypass the blocking, attached archive, in which some nonsense written in Python downloads the malware.

We will assume that you have Windows 7 or higher. The task is to bypass blockages from the system itself.

## I DID SOMETHING, IT DIDN’T WORK, HOW TO DELETE IT NOW

If you did not install zapret as a service or scheduled task (and this requires editing cmd files),
Just close the winws window and run windivert_delete.cmd.
An alternative is to restart your computer.
After which you can delete the folder with zapret. This completes the uninstallation.
If you installed zapret as a service, then you probably know how to remove it.

If suddenly among what you clicked on there are the words “general”, “alt”, “.bat”, “autorun”, or there are files that are missing
in the original repositories, then this is a build. You will not receive an answer on how to remove it from the developer zapret.
Ask the collectors themselves. The developer does not provide a simple solution, the assemblers do this, but they themselves are responsible for their products.

## Setup

1. Скачайте и распакуйте архив https://github.com/bol-van/zapret-win-bundle/archive/refs/heads/master.zip.

2. If you have Windows 7 x64, run `win7/install_win7.cmd` once. The batch file will replace the windivert files with a version compatible with Windows 7.

   > There is no ready-made complete option for 32-bit Windows systems.

   > On windows 11 arm64, run `arm64/install_arm64.cmd` as administrator and restart your computer.
   > Read [docs/windows.md](./windows.md)
   >
   > Please be aware that antivirus programs may not respond well to windivert.
   > cygwin also has an impressive list of incompatibilities with antiviruses, although modern antiviruses
   > more or less learned to be friends with him.
   > If the problem occurs, use exceptions. If it doesn't help, disable your antivirus completely.

3. Make sure that you have disabled all means of bypassing locks, including zapret itself.

4. If you are working in a virtual machine, you must use a network connection in bridge mode. nat doesn't work

5. Run `blockcheck\blockcheck.cmd`.  blockcheck checks **DNS** first.
If messages about address spoofing are displayed, then you will need to solve the problem with **DNS**.
blockcheck will switch to **DoH** _(DNS over HTTPS)_ in this case and will try to obtain and use real IP addresses.
But unless you configure a solution to this problem, the workaround will only work for those programs
which themselves implement SecureDNS mechanisms. The bypass will not work for other programs.

   > Solving the DNS problem is beyond the scope of the project. Usually it can be solved either by replacing DNS servers
   > from the provider to public (`1.1.1.1`, `8.8.8.8`), or in case of interception of requests by the provider
   > к сторонним серверам - через специальные средства шифрования DNS запросов, такие как [dnscrypt](https://www.dnscrypt.org/), **DoT** _(DNS over TLS)_, **DoH**.
   > In modern browsers, DoH is most often enabled by default, but curl will use the normal system DNS.
   > win11 supports system DoH out of the box. They are not configured by default.
   > There is an unofficial workaround to enable DoH in the latest win10 builds.
   > For other systems, you need a third-party solution that works on the DNS proxy principle.
   >
   > Тут все разжевано как и где это включается : https://hackware.ru/?p=13707

6. blockcheck allows you to identify a working strategy for bypassing blocking.
The script log will be saved in `blockcheck\blockcheck.log`.
Remember/rewrite the strategies you find.

It should be understood that the script checks the availability of only a specific
domain you enter at the beginning with a specific curl program.
Different clients have their own fingerprint. Browsers have one, curl has another.
May or may not use multi-packet TLS with post-quantum cryptography (kyber).
The effectiveness of strategies may depend on this.
Usually other domains are blocked in a similar way, **but not a fact**.
There are special locks. Some parameters require tuning to suit the “common denominator”.
In most cases, you can combine several strategies into one universal one, and this is **extremely
desirable**, but it requires an understanding of how strategies work. prohibited **cannot
break through blocking by IP address**. To check multiple domains, enter them separated by a space.

   > Nowadays, blockers are installed on main canals. In difficult cases
   > you may have several routes with different lengths along the HOPs, with DPI on
   > different hops. You have to overcome a whole zoo of DPIs, which also
   > are included in the work in a chaotic manner or in a manner dependent on
   > directions (server IP). the script may not always give you the results
   > the optimal strategy, which you simply need to rewrite in the settings. IN
   > in some cases you need to really think about what is happening when analyzing the result
   > on different strategies.
   >
   > Далее, имея понимание что работает на http, https, quic, нужно сконструировать параметры запуска winws
   > using multi-strategy. How multi-strategies work is described in [readme.md](./readme.md#multiple-strategies).
   >
   > First of all, you need to assemble a filter for intercepted traffic. This is done through parameters
   > `--wf-l3`, `--wf-tcp`, `--wf-udp`.
   > `--wf-l3` refers to the version of the IP protocol - ipv4 or ipv6.
   > `--wf-tcp` and `--wf-udp` contain a comma-separated list of ports or port ranges.
   > 
   > Пример стандартного фильтра для перехвата http, https, quic : `--wf-tcp=80,443` `--wf-udp=443`
   > 
   > The filter should be the minimum required. Intercepting unnecessary traffic will only waste CPU resources and slow down the Internet.
   >
   > Briefly about multi-strategy, the parameters are usually constructed like this:
   > ```
   > --filter-udp=443 'parameters for quic' --new
   > --filter-tcp=80,443 'обьединенные параметры для http и https'
   > ```
   >
   > Or like this:
   > ```
   > --filter-udp=443 'parameters for quic' --new
   > --filter-tcp=80 'параметры для http' --new
   > --filter-tcp=443 'параметры для https'
   > ```
   >
   > If the strategies differ in IP protocol version and you cannot combine them, the filter is written like this:
   > ```
   > --filter-l3=ipv4 --filter-udp=443 "parameters for quic ipv4" --new
   > --filter-l3=ipv4 --filter-tcp=80 'параметры для http ipv4' --new
   > --filter-l3=ipv4 --filter-tcp=443 'параметры для https ipv4' --new
   > --filter-l3=ipv6 --filter-udp=443 "parameters for quic ipv6" --new
   > --filter-l3=ipv6 --filter-tcp=80 "параметры для http ipv6" --new
   > --filter-l3=ipv6 --filter-tcp=443 "параметры для https ipv6"
   > ```
   >
   > But this is a completely copy-and-paste option.
   > The more strategies you combine and reduce their total number, the better it will be.
   >
   > If you do not need to fool individual protocols, it would be best to remove them from the traffic interception system via
   > parameters `--wf-*` and remove the corresponding multi-strategy profiles.
   > tcp 80 - http, tcp 443 - https, udp 443 - quic.
   >
   > If phase zero desynchronization methods are used (`--mss`, `--wssize`, `--dpi-desync=syndata`) and hostlist filtering,
   > then all parameters related to these methods should be placed in separate multi-strategy profiles, which will receive
   > control until the hostname is determined. It is necessary to understand the algorithm of how multi-strategies work.
   >
   > ```
   > --filter-tcp=80 'параметры для http' --new
   > --filter-tcp=443 'параметры для https' --hostlist=d:/users/user/temp/list.txt --new
   > --filter-tcp=443 --wssize 1:6
   > ```
   >
   > The autohostlist profile has priority, so wssize needs to be written there:
   >
   > ```
   > --filter-tcp=80 'параметры для http' --new
   > --filter-tcp=443 'параметры для https' --wssize 1:6 --hostlist-auto=d:/users/user/temp/autolist.txt
   > ```
   >
   > In these examples, wssize will always be applied to tcp port 443 and the hostlist will be ignored.
   > К http применять wssize вредно и бессмысленно.
   >
   > Sometimes you need to add your own parameters to strategies.
   > For example, you need to change the number of repetitions of fakes or set your own fake.
   > This is done through the shell variables `PKTWS_EXTRA`, `TPWS_EXTRA`. Use the `cygwin/cygwin-admin.cmd` shell.
   >
   > ```PKTWS_EXTRA="--dpi-desync-repeats=10 --dpi-desync-fake-tls=/tmp/tls.bin" blockcheck```
   >
   > Trying all the combinations can lead to waiting for weeks, so a reasonable one was chosen
   > the backbone of the check on which you can hang your customizations.

7. Test the found strategies on winws. Winws should be taken from zapret-winws.
To do this, open a windows command prompt as an administrator in the zapret-winws directory.
The easiest way to do this is through `_CMD_ADMIN.cmd`. He will raise the rights himself and go to the desired directory.

8. Provide convenient download bypass locks.

   > There are 2 options. Manual launch via a shortcut or automatic launch at system startup, outside the context of the current user.
   > The last option is divided into launching through the task scheduler and through windows services.
   >
   > If you want a manual launch, copy `preset_russia.cmd` to `preset_my.cmd` (`<your_name>.cmd`) and adapt it to your launch parameters.
   > Then you can create a shortcut on your desktop to `preset_my.cmd`. Don't forget that you need to run as administrator.
   >
   > But it would be better to do a non-interactive automatic launch along with the system.
   > zapret-winws has `task_*` batch files for managing scheduler tasks.
   > There you should change the contents of the `WINWS1` variable to your winws launch parameters. Paths with spaces must be escaped with quotes followed by a backslash: `\"`.
   > After creating tasks, run them. Check that the bypass occurs after restarting windows.
   >
   > Windows services are configured in the same way. See `service_*.cmd`

9. If individual unblocked resources break down, you need to use a restrictive
ipset or host sheet. Read the main Talmud [readme.md](./readme.md) for details.
But it would be even better to select strategies that break the minimum.
There are strategies that are quite harmless, and there are very damaging ones that are only suitable for targeted penetration of individual resources,
when there is nothing better. A good strategy can be very disruptive due to poorly chosen limiters for fakes - ttl, fooling.

> [!CAUTION]  
> This is the minimum instruction to help you figure out where to start. However, this is not a panacea.
> In some cases, you will not be able to do without knowledge and the basic “Talmud”.

Details and full technical description are described in [readme.md](./readme.md)

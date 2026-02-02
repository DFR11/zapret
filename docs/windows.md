# Windows

## tpws

Running tpws is only possible in the Linux version under **WSL** _(Windows Subsystem for Linux)_.
There is no native option for Windows, since it uses epoll, which does not exist for Windows.

tpws in socks mode can be run under more or less modern builds of Windows 10 and Windows server
with WSL installed. It is not at all necessary to install the Ubuntu distribution, as they will tell you in almost every
article about WSL, which you will find on the Internet. tpws is a static binary, it does not need a distribution.

Install WSL:
 `dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all`

From the release, copy `binaries/linux-x86_64/tpws_wsl.tgz` to the target system.

Run:
 `wsl --import tpws "%USERPROFILE%\tpws" tpws_wsl.tgz`

Run:
`wsl -d tpws --exec /tpws --uid=1 --no-resolve --socks --bind-addr=127.0.0.1 --port=1080 <fooling_options>`

Register socks `127.0.0.1:1080` in a browser or other program.

Uninstall : `wsl --unregister tpws`


> [!NOTE]
> Tested on windows 10 build 19041 (20.04) under WSL1. On WSL2 these commands may not work.
If you have WSL2, then you have a working Linux virtual machine.
If you know how to handle it, you can run tpws on it without any problems.


Possible problems:
- The `--oob` and `--mss` functions do not work due to WSL implementation limitations.
`--disorder` does not work due to the peculiarities of the windows tcp/ip stack.

- RST detection in autohostlist may not work.

- WSL can glitch with splice, causing the process to loop. `--nosplice` may be required.

- Tcp user timeout is not supported.
To get rid of error messages add:
 `--local-tcp-user-timeout=0 --remote-tcp-user-timeout=0`.
These messages are for informational purposes only and do not affect operation.

## winws

This is a variant of the nfqws packet filter for Windows, built on top of windivert.
All functions are functional, but the ipset functionality is missing in the kernel. It is implemented in user mode. Filters based on a large number of IP addresses are not possible.
Working with passing traffic, for example in the case of “sharing” the connection, is impossible.
Administrator rights are required to use windivert.
Unix-specific options such as `--uid`, `--user`, etc. are excluded. All other parameters are similar to nfqws and dvtws.

Working with a packet filter is based on two actions:
1) Selecting redirected traffic in kernel mode and passing it to the packet filter in user mode.
2) The actual processing of redirected packets in the packet filter.

Windows does not have built-in traffic redirection tools such as _iptables_, _nftables_, _pf_ or _ipfw_.
Therefore, a third-party windivert kernel driver is used. It works starting from windows 7. On systems with
secure boot there may be problems due to driver signature. In this case, disable `secureboot` or enable `testsigning` mode.
On windows 7 there will probably be problems loading windivert. Read the relevant section below.

The _iptables_ task in **winws** is solved internally through windivert filters.
windivert has its own filter language, similar to wireshark's.
[Документация по фильтрам windivert.](https://reqrypt.org/windivert-doc.html#filter_language)
To avoid writing complex filters manually, various simplified options for automatically constructing filters are provided.

```
 --wf-iface=<int>[.<int>]               ; числовые индексы интерфейса и суб-интерфейса
 --wf-l3=ipv4|ipv6                      ; фильтр L3 протоколов. по умолчанию включены ipv4 и ipv6.
 --wf-tcp=[~]port1[-port2]              ; фильтр портов для tcp. ~ означает отрицание
 --wf-udp=[~]port1[-port2]              ; фильтр портов для udp. ~ означает отрицание
 --wf-raw-part=<filter>|@<filename>     ; частичный windivert фильтр из параметра или из файла. имени файла предшествует символ @. может быть множество частей. сочетается с --wf-tcp,--wf-udp.
 --wf-filter-lan=0|1                    ; отфильтровывать адреса назначения, не являющиеся глобальными inet адресами ipv4 или ipv6. по умолчанию - 1.
 --wf-raw=<filter>|@<filename>          ; полный windivert фильтр из параметра или из файла. имени файла предшествует символ @. замещает --wf-raw-part,--wf-tcp,--wf-udp.
 --wf-save=<filename>                   ; сохранить сконструированный фильтр windivert в файл для последующей правки вручную
 --ssid-filter=ssid1[,ssid2,ssid3,...]  ; включать winws только когда подключена любая из указанных wifi сетей
 --nlm-filter=net1[,net2,net3,...]      ; включать winws только когда подключена любая из указанных сетей NLM
 --nlm-list[=all]                       ; вывести список сетей NLM. по умолчанию только подключенных, all - всех.
 ```

The parameters `--wf-l3`, `--wf-tcp`, `--wf-udp` can take multiple values ​​separated by commas.

You can find out the interface numbers like this: `netsh int ip show int`.
Some types of connections cannot be seen there. In this case, run **winws** with the `--debug` option and look at IfIdx there.
SubInterface is used by windivert, but almost always **0**, it can be omitted. It is probably needed in rare cases.

The standard filter designer automatically includes incoming tcp packets with tcp synack and tcp rst for the functions to work correctly
autottl и autohostlist. При включении autohostlist так же перенаправляются пакеты данных с http redirect с кодами 302 и 307.
Unless otherwise specified, a filter is added to exclude non-Internet ipv4 and ipv6 addresses.
Complex non-standard scenarios may require your own filters. The full --wf-raw filter overrides everything else.
Partial filters `--wf-raw-part` are compatible with each other and `--wf-tcp` and `--wf-udp`. They allow you to avoid writing
bulky full filters, focusing only on adding some special payload.
`--wf-save` allows you to write the resulting windivert filter to a file. The maximum filter size is **16 Kb**.

windivert filtering is done in the kernel. This is incomparably easier in terms of resources than redirecting packets to the user mode space,
for winws to make the decision. Therefore, make the most of windivert's capabilities.
For example, if you need to fool wireguard on all ports, you will have to forward all ports to winws. Or write a windivert filter that will cut off wireguard based on the contents of the packet.
The difference in processor load is colossal. In the first case - up to 100% of one core cpu depending on the volume of outgoing udp traffic (hello, torrent and uTP), in the second - close to 0.
In addition to the load on the processor, you can also reduce your speed, as one core will not cope with processing your gigabit Internet. And on older laptops you will also get an airplane-like whine of the cooling system, leading to its wear.

You can run multiple **winws** processes with different strategies. However, you should not make intersecting filters.

In `--ssid-filter` you can specify an unlimited number of wifi network names (**SSID**), separated by commas. If at least one network is specified,
then winws is enabled only if the specified **SSID** is connected. If the **SSID** disappears, winws is disabled. If **SSID** appears again,
winws is turned on. This is necessary so that you can apply separate fooling to each individual wifi network.
Network names must be written in the case in which the system sees them. The comparison is case sensitive!
At the same time, there are no checks of where the traffic actually goes. If, for example, ethernet is connected at the same time,
and the traffic goes there, then the foolishness is turned on and off simply by the presence of a wifi network, to which the traffic may not go.
And this can break the stupidity on ethernet. Therefore, it will also be useful to add the `--wf-iface` filter to the wifi adapter interface index,
so as not to affect other traffic.

`--nlm-filter` is similar to `--ssid-filter`, but works with Network List Manager (NLM) network names or GUIDs.
These are the networks you see in Control Panel under Network and Sharing Center.
The network does not mean a specific adapter, but rather the network environment of a specific connection.
Usually the mac address of the gateway is checked. You can connect to the network through any adapter, and it will remain the same.
If you connect, say, to different routers via cable, you will have different networks.
And if you connect to one router through 2 different network cards on the same computer, there will be one network.
NLM abstracts network adapter types. It works with both wifi and ethernet and any others.
Therefore, this is a more universal method than the **SSID** filter.
However, there is also an unpleasant side. In Windows 7, you could easily click on the network icon and select the type: private or public.
There you could look at the list of networks and combine them. So that, for example, you can connect via cable and wifi
to one router, and the system perceived these connections as one network.
In subsequent versions of Windows they greatly reduced these features. It seems there are no built-in tools to fully manage
network locations in win10/11. There is something in **powershell**.
You can dig deeper into the registry here:
`HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\NetworkList`
You need to change the ProfileGUID in `Signatures\Unmanaged`. Names can be changed in Profiles.
There are some third party utilities. There is something that allows you to view and delete network profiles, but not merge them.
The fact is that in MS they really messed it up. The network list engine is still the same, and it is capable of everything that was in win7.
You don’t have to deal with this problem, but simply indicate, separated by commas, the names of the networks or GUIDs that the system has selected.
Or if you only have wifi, then use `--ssid-filter`. At least there is a guarantee that the **SSID** corresponds to reality,
but the system didn’t call them anything differently.

If the paths contain national characters, then when calling winws from `cmd` or `bat` the encoding must be used **OEM**.
For the Russian language this is 866. Paths with spaces must be enclosed in quotes.
When using the @<config_file> option, the encoding in the file must be **UTF-8** without **BOM mark**.

There is a non-obvious point regarding launching **winws** from the cygwin shell. If in the directory where winws is located there is
a copy of `cygwin1.dll`, **winws** will not start.
If you need to run under cygwin, you should remove or move `cygwin1.dll` from `binaries/windows-x86_64`. This is necessary for blockcheck to work.
From the cygwin shell you can send winws signals via `kill` in the same way as in `*nix`.

How to get windows 7 and winws compatible cygwin:

`curl -O https://www.cygwin.com/setup-x86_64.exe`

`setup-x86_64.exe --allow-unsupported-windows --no-verify --site http://ctm.crouchingtigerhiddenfruitbat.org/pub/cygwin/circa/64bit/2024/01/30/231215`

> [!IMPORTANT]
> You should choose to install curl.

To build from source, _gcc-core_, _make_, _zlib-devel_ are required.
Build from the nfq directory with the command `make cygwin64` or `make cygwin32` for 64 and 32 bit versions, respectively.
**winws** requires `cygwin1.dll`, `windivert.dll`, `windivert64.sys` or `windivert32.sys`.
They can be taken from `binaries/win64` and `binaries/win32`.

For _arm64_ windows there is no signed windivert driver and no cygwin.
However, x64 Windows 11 emulation allows you to use everything except WinDivert64.sys without changes.
But in this case, you need to replace WinDivert64.sys with the unsigned _arm64_ version and set the testsigning mode.

## Windows 7 и windivert

Windows driver signing requirements have changed in 2021.
Official free Windows 7 updates ended in 2020.
After this, paid updates for the **ESU** program continued for several years.
It is in these **ESU** updates that the Windows 7 kernel update is located, allowing you to load the driver
_windivert 2.2.2-A_, which comes with zapret.
Therefore the options are as follows:

1) Взять `windivert64.sys` и `windivert.dll` версии _2.2.0-C_ или _2.2.0-D_ отсюда : https://reqrypt.org/download
and replace these 2 files.
В [zapret-win-bundle](https://github.com/bol-van/zapret-win-bundle) есть отдельных 2 места, где находится **winws** : [_zapret-winws_](https://github.com/bol-van/zapret-win-bundle/tree/master/zapret-winws) и [_blockcheck/zapret/nfq_](https://github.com/bol-van/zapret-win-bundle/tree/master/blockcheck).
It needs to be changed in both places.
An alternative option when using win bundle is to run `win7\install_win7.cmd`

> [!NOTE]
> This option has been tested and should work. However, the 10 year old patch that includes SHA256 signatures is still needed.

2) Hack **ESU** :
https://hackandpwn.com/windows-7-esu-patching/
http://www.bifido.net/tweaks-and-scripts/8-extended-security-updates-installer.html
and update the system

3) Использовать UpdatePack7R2 от simplix : https://blog.simplix.info
> [!WARNING]
> But there is a problem with this pack. The author is from Ukraine, he was very offended by the Russians.
> If the control panel is set to RU or BY, an unpleasant dialog appears.
> To get around this problem, you can temporarily set any other region, then return it.
> There is also no guarantee that the author did not put some malicious code there.
> Use at your own risk.

A safer option is to download the latest normal pre-war version: 22.2.10
https://nnmclub.to/forum/viewtopic.php?t=1530323
It is enough for _windivert 2.2.2-A_ to work on Windows 7.

## blockcheck

`blockcheck.sh` is written in the _posix shell_ and requires some standard _posix_ utilities. In Windows, of course, this is not the case.
Therefore, it is impossible to just run `blockcheck.sh`.
To do this, you need to download and install _cygwin_ as described in the previous section.
Should be run as administrator _cygwin shell_ via `cygwin.bat`.
In it you need to go to the directory with zapret.
Windows path backslashes need to be doubled, replaced with forward slashes, or mapped to unix path.
Correct options:
- `cd C:\\Users\\vasya`
- `cd C:/Users/vasya`
- `cd /cygdrive/c/Users/vasya`

There is a non-obvious point regarding launching **winws** from the _cygwin_ shell. If there is a copy of `cygwin1.dll` in the directory where **winws** is located, **winws** will not start. You need to rename the `cygwin1.dll` file.
Then everything is as in _*nix_: 1 time `./install_bin.sh`, then `./blockcheck.sh`.
You can't use WSL, it's not the same.

_cygwin_ is not needed for normal work. **winws** is not needed.

Однако, хотя такой способ и работает, использование **winws** сильно облегчает [zapret-win-bundle](https://github.com/bol-van/zapret-win-bundle).
There is no problem with `cygwin.dll`.

## Zapret-win-bundle

Можно не возиться с _cygwin_, а взять готовый пакет, включающий в себя _cygwin_ и _blockcheck_ : https://github.com/bol-van/zapret-win-bundle
There, maximum convenience has been made for focusing on the zapret itself, excluding fuss with installing _cygwin_,
entries into directories, launches under administrator and other purely technical issues in which there may be
mistakes and misunderstandings, and a beginner without a basis of knowledge can become completely confused.

`/zapret-winws` - ​​here is everything you need to run winws in everyday working mode. the rest is not needed.\
`/zapret-winws/_CMD_ADMIN.cmd` - get the cmd command line in this directory as an administrator for testing **winws**
with parameters entered manually\
`/blockcheck/blockcheck.cmd` - just click on it to go to _blockcheck_ with a log entry in `blockcheck/blockcheck.log`\
`/cygwin/cygwin.cmd` - launch the _cygwin bash_ environment under the current user\
`/cygwin/cygwin-admin.cmd` - launch the _cygwin bash_ environment as an administrator

In the _cygwin_ environment, aliases are already configured for winws, blockcheck, ip2net, mdig. No need to mess with paths!

> [!TIP]
> From cygwin you can not only test winws, but also send signals.
> Available commands:
>-  `pidof`
>-  `kill`
>-  `killall`
>-  `pgrep`
>-  `pkill`

But it is important to understand that in this way it will not be possible to send signals to **winws** launched from _zapret-winws_,
because it has its own `cygwin1.dll`, and they do not share the common unix process space.
_zapret-winws_ is a standalone kit for everyday use, not requiring anything else, but also not associated with the _cygwin_ environment.
Killall.exe is present in _zapret-winws_ specifically for sending winws signals.

The cygwin environment can be used to write to the winws debug log file. To do this, use the tee command.
`winws --debug --wf-tcp=80,443 | tee winws.log`
`winws.log` will be in `cygwin/home/<username>`
If you have Windows 7, then Notepad will not understand unix-style line breaks. Use the command
`unix2dos winws.log`

> [!CAUTION]
> Since 32-bit windows are in little demand, _zapret-win-bundle_ exists only in the version for windows _x64/arm64_.

## Autostart winws

To run **winws** along with windows there are 2 options. Task scheduler or windows services.

You can create and manage tasks through the schtasks console program.
The `task_*.cmd` files are prepared in the `binaries/windows-x86_64/winws` directory.
They implement the creation, deletion, start and stop of one copy of the winws process with parameters from the `%WINWS1%` variable.
Correct the parameters to the strategy you need. If a different strategy is used for different filters, multiply the code
for tasks _winws1_, _winws2_, _winws3_,_..._

The launch option via windows services is configured in the same way. See `service_*.cmd`.

All batch files must be run as administrator.

You can also manage tasks from the graphical scheduler management program `taskschd.msc`

## Windows Server Features

winws is linked with wlanapi.dll, which is not installed in windows server by default.
To solve this problem, run the power shell as an administrator and run the command `Install-WindowsFeature -Name Wireless-Networking`.
Then reboot the system.

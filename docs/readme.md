# zapret v72.3

# ATTENTION, beware of scammers

zapret is free and open source.
Anyone who forces you to download zapret only from its resource, demands to remove links, videos, files, justifying these demands with copyright, is violating the [license](./LICENSE.txt).
However, this does not exclude [voluntary donations](#support-developer).

# Multilanguage README

[![en](https://img.shields.io/badge/lang-en-red.svg)](./readme.en.md)
[![ru](https://img.shields.io/badge/lang-ru-green.svg)](./readme.md)

***

  - [Why is this necessary](#why is this necessary)
  - [Quick start](#quick start)
  - [How it works](#how-it-works)
  - [What's happening in Russia now](#what-is-happening-in-Russia now)
  - [How to implement this in practice on a linux system](#how-to-implement this in practice on a linux system)
  - [When it won't work](#when-it-won't-work)
  - [nfqws](#nfqws)
    - [DPI DESYNCHRONIZATION ATTACK](#dpi-desynchronization-attack)
    - [FAKES](#fake)
    - [MODIFICATION OF FAKES](#modification-fake)
    - [TCP SEGMENTATION](#tcp-segmentation)
    - [OVERLAP SEQUENCE NUMBERS](#overlap-sequence-numbers)
    - [IP_ID DESTINATION](#destination-ip_id)
    - [IPV6 SPECIFIC MODES](#ipv6-specific-modes)
    - [MODIFICATION OF ORIGINAL](#modification-of-original)
    - [DUPLICATES](#duplicates)
    - [COMBINING DESYNCHRONIZATION METHODS](#combining-desynchronization-methods)
    - [CACHE IP](#cache-ip)
    - [DPI RESPONSE TO SERVER RESPONSE](#dpi-reaction-to-server-response)
    - [SYNACK MODE](#synack mode)
    - [SYNDATA MODE](#mode-syndata)
    - [VIRTUAL MACHINES](#virtual-machines)
    - [CONNTRACK](#conntrack)
    - [REASSEMBLING](#reassembling)
    - [UDP SUPPORT](#support-udp)
    - [IP FRAGMENTATION](#ip-fragmentation)
    - [MULTIPLE STRATEGIES](#multiple-strategies)
    - [FILTERING BY WIFI](#filtering-by-wifi)
    - [IPTABLES FOR NFQWS](#iptables-for-nfqws)
    - [NFTABLES FOR NFQWS](#nftables-for-nfqws)
    - [FLOW OFFLOADING](#flow-offloading)
    - [FEATURES OF GARDENS](#features of hardware)
    - [FOOLING ON THE SERVER'S SIDE](#fooling-on-the-server's side)
  - [tpws](#tpws)
    - [TCP SEGMENTATION IN TPWS](#tcp-segmentation-in-tpws)
    - [TLSREC](#tlsrec)
    - [MSS](#mss)
    - [OTHER DUMPING PARAMETERS](#other-fooling-parameters)
    - [MULTIPLE STRATEGIES](#multiple-strategies-1)
    - [SERVICE PARAMETERS](#service-parameters)
    - [IPTABLES FOR TPWS](#iptables-for-tpws)
    - [NFTABLES FOR TPWS](#nftables-for-tpws)
  - [ip2net](#ip2net)
  - [mdig](#mdig)
  - [Ways to get a list of blocked IPs](#ways-to-get-a-list-of-blocked-ips)
  - [Filtering by domain names](#filtering-by-domain-names)
  - [autohostlist filtering mode](#filtering-mode-autohostlist)
  - [Provider check](#provider check)
  - [Select parameters](#select-parameters)
  - [Screwing to the firewall management system or your launch system](#screwing-to-the firewall management system or your launch system)
  - [option custom](#option-custom)
  - [Easy installation](#easy-installation)
  - [Installation under systemd](#installation-under-systemd)
  - [Simple openwrt installation](#simple-openwrt-installation)
  - [Installation on openwrt in severely low disk space mode](#installation-on-openwrt in severely low-disk-space mode)
  - [Android](#android)
  - [Mobile modems and routers huawei](#mobile-modems-and-routers-huawei)
  - [FreeBSD, OpenBSD, MacOS](#freebsd-openbsd-macos)
  - [Windows](#windows)
  - [Other firmware](#other-firmware)
  - [Bypassing blocking through a third-party host](#bypassing-blocking-through-third-party-host)
  - [Why is it worth investing in a VPS](#why is it worth investing in a VPS)
  - [Support the developer](#support-developer)
***

## Why is this necessary?

A standalone DPI countermeasure that does not require the connection of any third party servers. May help
bypass blocking or slowdown of HTTP(S) sites, signature analysis of TCP and UDP protocols, for example, for the purpose of blocking
VPN.

The project is aimed primarily at low-power embedded devices - routers running OpenWrt. Supported
traditional Linux systems, FreeBSD, OpenBSD, partially macOS. In some cases, self-screwing is possible
solutions for various firmwares.

Most of the functionality works on Windows.

## Quick start

- [Linux/openWrt](./quick_start.md)
- [Windows](./quick_start_windows.md)

## How does this work

In the simplest case, you are dealing with passive DPI. Passive DPI can read traffic from the stream, can inject
its own packets, but cannot block passing packets. If the request is "bad", passive DPI will inject an RST packet,
optionally supplementing it with an HTTP redirect package. If the fake packet is injected only for the client, in this case you can
use iptables commands to drop RST and/or redirect to a stub according to certain conditions that need to be selected
for each provider individually. This way we bypass the consequences of the ban trigger. If passive DPI
sends the RST packet to the server as well, then you cannot do anything about it. Your task is to prevent
activation of the prohibition trigger. Iptables alone are no longer enough. This project is aimed specifically at preventing
triggering the ban, rather than eliminating its consequences.

Active DPI is placed in the wire section and can drop packets according to any criteria, including recognizing TCP streams and
block any packets belonging to the thread.

How to prevent the ban trigger from being triggered? Send something that DPI does not count on and that breaks its algorithm
recognition of requests and their blocking.

Some DPIs cannot recognize an HTTP request if it is divided into TCP segments. For example, request
vida `GET / HTTP/1.1\r\nHost: kinozal.tv......`
we send in two parts: first comes `GET`, then `/ HTTP/1.1\r\nHost: kinozal.tv.....`. Other DPIs stumble when
the `Host:` header is written in a different case: for example, `host:`. Adding an extra space works in some places
after the method: `GET /` → `GET /`
or adding a dot at the end of the host name: `Host: kinozal.tv.`

There is also more advanced magic aimed at overcoming DPI at the batch level.

More details about DPI:\
https://habr.com/ru/post/335436 или https://web.archive.org/web/20230331233644/https://habr.com/ru/post/335436/ \
https://geneva.cs.umd.edu/papers/geneva_ccs19.pdf

## What's happening in Russia now

Previously, before the introduction of widespread TSPU systems, a zoo of different DPIs from providers was used. There were some
active, some passive. Now the time for simple iptables is completely gone. Active DPI TSPU is active everywhere, but in some places
Additional old DPIs from the zoo may remain unchecked. In this case, you have to go around several at once
DPI. There are more and more off-registry blockings, which you only learn about when something is unavailable, in
there are no lists of this. Some ranges of IP addresses are blocked (offline bypass is not possible)
and protocols (VPN). On some IP bands, a more stringent filter is used to detect attempts to deceive through
segmentation. This must be due to some services that try to cheat DPI in this way.

## How to implement this in practice on a Linux system

In short, the options can be classified according to the following scheme:

1) Passive DPI that does not send RST to the server. iptables commands individually customized for the provider will help. On
rutracker in the section "bypassing blocking - other methods" there is a separate topic on this issue. In this project
not considered. If you don't allow the banning trigger to go off, then you won't have to fight it.
consequences.
2) Modification of TCP connection at the flow level. Implemented via proxy or transparent proxy.
3) Modification of TCP connection at the packet level. Implemented through the NFQUEUE queue handler and raw sockets.

For options 2 and 3, the tpws and nfqws programs are implemented, respectively. For them to work, you need to start them with
the necessary parameters and redirect certain traffic to them using iptables or nftables.

## When it won't work

* If DNS is changed. This problem is easy to deal with.
* If blocking is carried out by IP.
* If the connection passes through a filter that can reconstruct the TCP connection and that follows all standards.
For example, we are being turned into squid. The connection goes through a full tcpip stack of the operating system.
The project is aimed at deceiving DPI, which, due to limited resources and high traffic, is forced to interpret it only in a limited way.
It will not be possible to deceive a full-fledged OS stack and full-fledged server applications.

## nfqws

This program is a packet modifier and NFQUEUE queue handler. For BSD systems there is an adapted version -
dvtws, compiled from the same sources (see [BSD documentation](./bsd.md)).

```
@<config_file>|$<config_file>                             ; читать конфигурацию из файла. опция должна быть первой. остальные опции игнорируются.

--debug=0|1                                               ; 1=выводить отладочные сообщения
--dry-run                                                 ; проверить опции командной строки и выйти. код 0 - успешная проверка.
--version                                                 ; вывести версию и выйти
--comment                                                 ; любой текст (игнорируется)
--daemon                                                  ; демонизировать прогу
--pidfile=<file>                                          ; сохранить PID в файл
--user=<username>                                         ; менять uid процесса
--uid=uid[:gid]                                           ; менять uid процесса
--qnum=N                                                  ; номер очереди N
--bind-fix4                                               ; пытаться решить проблему неверного выбора исходящего интерфейса для сгенерированных ipv4 пакетов
--bind-fix6                                               ; пытаться решить проблему неверного выбора исходящего интерфейса для сгенерированных ipv6 пакетов
--ctrack-timeouts=S:E:F[:U]                               ; таймауты внутреннего conntrack в состояниях SYN, ESTABLISHED, FIN, таймаут udp. по умолчанию 60:300:60:60
--ctrack-disable=[0|1]                                    ; 1 или остутствие аргумента отключает conntrack
--ipcache-lifetime=<int>                                  ; время жизни записей кэша IP в секундах. 0 - без ограничений.
--ipcache-hostname=[0|1]                                  ; 1 или отсутствие аргумента включают кэширование имен хостов для применения в стратегиях нулевой фазы
--wsize=<winsize>[:<scale_factor>]                        ; менять tcp window size на указанный размер в SYN,ACK. если не задан scale_factor, то он не меняется (устарело !)
--wssize=<winsize>[:<scale_factor>]                       ; менять tcp window size на указанный размер в исходящих пакетах. scale_factor по умолчанию 0. (см. conntrack !)
--wssize-cutoff=[n|d|s]N                                  ; изменять server window size в исходящих пакетах (n), пакетах данных (d), относительных sequence (s) по номеру меньше N
--wssize-forced-cutoff=0|1                                ; 1(default)=автоматически отключать wssize в случае обнаружения известного протокола
--synack-split=[syn|synack|acksyn]                        ; выполнить tcp split handshake. вместо SYN,ACK отсылать только SYN, SYN+ACK или ACK+SYN
--orig-ttl=<int>                                          ; модифицировать TTL оригинального пакета
--orig-ttl6=<int>                                         ; модифицировать ipv6 hop limit оригинальных пакетов.  если не указано, используется значение --orig-ttl
--orig-autottl=[<delta>[:<min>[-<max>]]|-]                ; режим auto ttl для ipv4 и ipv6. по умолчанию: +5:3-64. "0:0-0" или "-" отключает функцию
--orig-autottl6=[<delta>[:<min>[-<max>]]|-]               ; переопределение предыдущего параметра для ipv6
--orig-tcp-flags-set=<int|0xHEX|flaglist>                 ; устанавливать указанные tcp флаги (flags |= value). число , либо список через запятую : FIN,SYN,RST,PSH,ACK,URG,ECE,CWR,AE,R1,R2,R3
--orig-tcp-flags-unset=<int|0xHEX|flaglist>               ; удалять указанные tcp флаги (flags &= ~value)
--orig-mod-start=[n|d|s]N                                 ; применять orig-mod только в исходящих пакетах (n), пакетах данных (d), относительных sequence (s) по номеру больше или равно N
--orig-mod-cutoff=[n|d|s]N                                ; применять orig-mod только в исходящих пакетах (n), пакетах данных (d), относительных sequence (s) по номеру меньше N
--dup=<int>                                               ; высылать N дубликатов до оригинала
--dup-replace=[0|1]                                       ; 1 или отсутствие аргумента блокирует отправку оригинала. отправляются только дубликаты.
--dup-ttl=<int>                                           ; модифицировать TTL дубликатов
--dup-ttl6=<int>                                          ; модифицировать ipv6 hop limit дубликатов. если не указано, используется значение --dup-ttl
--dup-autottl=[<delta>[:<min>[-<max>]]|-]                 ; режим auto ttl для ipv4 и ipv6. по умолчанию: +1:3-64. "0:0-0" или "-" отключает функцию
--dup-autottl6=[<delta>[:<min>[-<max>]]|-]                ; переопределение предыдущего параметра для ipv6
--dup-tcp-flags-set=<int|0xHEX|flaglist>                  ; устанавливать указанные tcp флаги (flags |= value). число , либо список через запятую : FIN,SYN,RST,PSH,ACK,URG,ECE,CWR,AE,R1,R2,R3
--dup-tcp-flags-unset=<int|0xHEX|flaglist>                ; удалять указанные tcp флаги (flags &= ~value)
--dup-fooling=<fooling>                                   ; дополнительные методики как сделать, чтобы дубликат не дошел до сервера. none md5sig badseq badsum datanoack ts hopbyhop hopbyhop2
--dup-ts-increment=<int|0xHEX>                            ; инкремент TSval для ts. по умолчанию -600000
--dup-badseq-increment=<int|0xHEX>                        ; инкремент sequence number для badseq. по умолчанию -10000
--dup-badack-increment=<int|0xHEX>                        ; инкремент ack sequence number для badseq. по умолчанию -66000
--dup-ip-id=same|zero|seq|rnd                             ; режим назначения ip_id для пакетов dup
--dup-start=[n|d|s]N                                      ; применять dup только в исходящих пакетах (n), пакетах данных (d), относительных sequence (s) по номеру больше или равно N
--dup-cutoff=[n|d|s]N                                     ; применять dup только в исходящих пакетах (n), пакетах данных (d), относительных sequence (s) по номеру меньше N
--hostcase                                                ; менять регистр заголовка "Host:" по умолчанию на "host:".
--hostnospace                                             ; убрать пробел после "Host:" и переместить его в конец значения "User-Agent:" для сохранения длины пакета
--methodeol                                               ; добавить перевод строки в unix стиле ('\n') перед методом и убрать пробел из Host: : "GET / ... Host: domain.com" => "\nGET  / ... Host:domain.com"
--hostspell=HoST                                          ; точное написание заголовка Host (можно "HOST" или "HoSt"). автоматом включает --hostcase
--domcase                                                 ; домен после Host: сделать таким : TeSt.cOm
--ip-id=seq|seqgroup|rnd|zero                             ; режим назначения ip_id для генерированных пакетов
--dpi-desync=[<mode0>,]<mode>[,<mode2]                    ; атака по десинхронизации DPI. mode : synack syndata fake fakeknown rst rstack hopbyhop destopt ipfrag1 multisplit multidisorder fakedsplit hostfakesplit fakeddisorder ipfrag2 udplen tamper
--dpi-desync-fwmark=<int|0xHEX>                           ; бит fwmark для пометки десинхронизирующих пакетов, чтобы они повторно не падали в очередь. default = 0x40000000
--dpi-desync-ttl=<int>                                    ; установить ttl для десинхронизирующих пакетов
--dpi-desync-ttl6=<int>                                   ; установить ipv6 hop limit для десинхронизирующих пакетов. если не указано, используется значение --dpi-desync-ttl
--dpi-desync-autottl=[<delta>[:<min>[-<max>]]|-]          ; режим auto ttl для ipv4 и ipv6. по умолчанию: 1:3-20. "0:0-0" или "-" отключает функцию
--dpi-desync-autottl6=[<delta>[:<min>[-<max>]]|-]         ; переопределение предыдущего параметра для ipv6
--dpi-desync-tcp-flags-set=<int|0xHEX|flaglist>           ; устанавливать указанные tcp флаги (flags |= value). число , либо список через запятую : FIN,SYN,RST,PSH,ACK,URG,ECE,CWR,AE,R1,R2,R3
--dpi-desync-tcp-flags-unset=<int|0xHEX|flaglist>         ; удалять указанные tcp флаги (flags &= ~value)
--dpi-desync-fooling=<fooling>                            ; дополнительные методики как сделать, чтобы фейковый пакет не дошел до сервера. none md5sig badseq badsum datanoack ts hopbyhop hopbyhop2
--dpi-desync-repeats=<N>                                  ; посылать каждый генерируемый в nfqws пакет N раз (не влияет на остальные пакеты)
--dpi-desync-skip-nosni=0|1                               ; 1(default)=не применять dpi desync для запросов без hostname в SNI, в частности для ESNI
--dpi-desync-split-pos=N|-N|marker+N|marker-N             ; список через запятую маркеров для tcp сегментации в режимах split и disorder
--dpi-desync-split-seqovl=N|-N|marker+N|marker-N          ; единичный маркер, определяющий величину перекрытия sequence в режимах split и disorder. для split поддерживается только положительное число.
--dpi-desync-split-seqovl-pattern=[+ofs]@<filename>|0xHEX ; чем заполнять фейковую часть overlap
--dpi-desync-fakedsplit-pattern=[+ofs]@<filename>|0xHEX   ; чем заполнять фейки в fakedsplit/fakeddisorder
--dpi-desync-fakedsplit-mod=mod[,mod]                     ; может быть none, altorder=0|1|2|3 + 0|8|16
--dpi-desync-hostfakesplit-midhost=marker+N|marker-N      ; маркер дополнительного разреза сегмента с оригинальным хостом. должен попадать в пределы хоста.
--dpi-desync-hostfakesplit-mod=mod[,mod]                  ; может быть none, host=<hostname>, altorder=0|1
--dpi-desync-ts-increment=<int|0xHEX>                     ; инкремент TSval для ts. по умолчанию -600000
--dpi-desync-badseq-increment=<int|0xHEX>                 ; инкремент sequence number для badseq. по умолчанию -10000
--dpi-desync-badack-increment=<int|0xHEX>                 ; инкремент ack sequence number для badseq. по умолчанию -66000
--dpi-desync-any-protocol=0|1                             ; 0(default)=работать только по http request и tls clienthello  1=по всем непустым пакетам данных
--dpi-desync-fake-tcp-mod=mod[,mod]                       ; список через запятую режимов runtime модификации tcp фейков (любых) : none, seq
--dpi-desync-fake-http=[+ofs]@<filename>|0xHEX	          ; файл, содержащий фейковый http запрос для dpi-desync=fake, на замену стандартному www.iana.org
--dpi-desync-fake-tls=[+ofs]@<filename>|0xHEX|![+offset]  ; файл, содержащий фейковый tls clienthello для dpi-desync=fake, на замену стандартному. '!' = стандартный фейк
--dpi-desync-fake-tls-mod=mod[,mod]                       ; список через запятую режимов runtime модификации фейков : none,rnd,rndsni,sni=<sni>,dupsid,padencap
--dpi-desync-fake-unknown=[+ofs]@<filename>|0xHEX         ; файл, содержащий фейковый пейлоад неизвестного протокола для dpi-desync=fake, на замену стандартным нулям 256 байт
--dpi-desync-fake-syndata=[+ofs]@<filename>|0xHEX         ; файл, содержащий фейковый пейлоад пакета SYN для режима десинхронизации syndata
--dpi-desync-fake-quic=[+ofs]@<filename>|0xHEX            ; файл, содержащий фейковый QUIC Initial
--dpi-desync-fake-wireguard=[+ofs]@<filename>|0xHEX       ; файл, содержащий фейковый wireguard handshake initiation
--dpi-desync-fake-dht=[+ofs]@<filename>|0xHEX             ; файл, содержащий фейковый пейлоад DHT протокола для dpi-desync=fake, на замену стандартным нулям 64 байт
--dpi-desync-fake-discord=[+ofs]@<filename>|0xHEX         ; файл, содержащий фейковый пейлоад Discord протокола нахождения IP адреса для голосовых чатов для dpi-desync=fake, на замену стандартным нулям 64 байт
--dpi-desync-fake-stun=[+ofs]@<filename>|0xHEX            ; файл, содержащий фейковый пейлоад STUN протокола для dpi-desync=fake, на замену стандартным нулям 64 байт
--dpi-desync-fake-unknown-udp=[+ofs]@<filename>|0xHEX     ; файл, содержащий фейковый пейлоад неизвестного udp протокола для dpi-desync=fake, на замену стандартным нулям 64 байт
--dpi-desync-udplen-increment=<int>                       ; на сколько увеличивать длину udp пейлоада в режиме udplen
--dpi-desync-udplen-pattern=[+ofs]@<filename>|0xHEX       ; чем добивать udp пакет в режиме udplen. по умолчанию - нули
--dpi-desync-start=[n|d|s]N                               ; применять dpi desync только в исходящих пакетах (n), пакетах данных (d), относительных sequence (s) по номеру больше или равно N
--dpi-desync-cutoff=[n|d|s]N                              ; применять dpi desync только в исходящих пакетах (n), пакетах данных (d), относительных sequence (s) по номеру меньше N
--hostlist=<filename>                                     ; действовать только над доменами, входящими в список из filename. поддомены автоматически учитываются, если хост не начинается с '^'.
                                                          ; в файле должен быть хост на каждой строке.
                                                          ; список читается при старте и хранится в памяти в виде иерархической структуры для быстрого поиска.
                                                          ; при изменении времени модификации файла он перечитывается автоматически по необходимости
                                                          ; список может быть запакован в gzip. формат автоматически распознается и разжимается
                                                          ; списков может быть множество. пустой общий лист = его отсутствие
                                                          ; хосты извлекаются из Host: хедера обычных http запросов и из SNI в TLS ClientHello.
--hostlist-domains=<domain_list>                          ; фиксированный список доменов через зяпятую. можно использовать # в начале для комментирования отдельных доменов.
--hostlist-exclude=<filename>                             ; не применять дурение к доменам из листа. может быть множество листов. схема аналогична include листам.
--hostlist-exclude-domains=<domain_list>                  ; фиксированный список доменов через зяпятую. можно использовать # в начале для комментирования отдельных доменов.
--hostlist-auto=<filename>                                ; обнаруживать автоматически блокировки и заполнять автоматический hostlist (требует перенаправления входящего трафика)
--hostlist-auto-fail-threshold=<int>                      ; сколько раз нужно обнаружить ситуацию, похожую на блокировку, чтобы добавить хост в лист (по умолчанию: 3)
--hostlist-auto-fail-time=<int>                           ; все эти ситуации должны быть в пределах указанного количества секунд (по умолчанию: 60)
--hostlist-auto-retrans-threshold=<int>                   ; сколько ретрансмиссий запроса считать блокировкой (по умолчанию: 3)
--hostlist-auto-debug=<logfile>                           ; лог положительных решений по autohostlist. позволяет разобраться почему там появляются хосты.
--new                                                     ; начало новой стратегии (новый профиль)
--skip                                                    ; не использовать этот профиль . полезно для временной деактивации профиля без удаления параметров.
--filter-l3=ipv4|ipv6                                     ; фильтр версии ip для текущей стратегии
--filter-tcp=[~]port1[-port2]|*                           ; фильтр портов tcp для текущей стратегии. ~ означает инверсию. установка фильтра tcp и неустановка фильтра udp запрещает udp. поддерживается список через запятую.
--filter-udp=[~]port1[-port2]|*                           ; фильтр портов udp для текущей стратегии. ~ означает инверсию. установка фильтра udp и неустановка фильтра tcp запрещает tcp. поддерживается список через запятую.
--filter-l7=<proto>                                       ; фильтр протокола L6-L7. поддерживается несколько значений через запятую. proto : http tls quic wireguard dht discord stun unknown
--filter-ssid=ssid1[,ssid2,ssid3,...]                     ; фильтр по имени wifi сети (только для linux)
--ipset=<filename>                                        ; включающий ip list. на каждой строчке ip или cidr ipv4 или ipv6. поддерживается множество листов и gzip. перечитка автоматическая.
--ipset-ip=<ip_list>                                      ; фиксированный список подсетей через запятую. можно использовать # в начале для комментирования отдельных подсетей.
--ipset-exclude=<filename>                                ; исключающий ip list. на каждой строчке ip или cidr ipv4 или ipv6. поддерживается множество листов и gzip. перечитка автоматическая.
--ipset-exclude-ip=<ip_list>                              ; фиксированный список подсетей через запятую. можно использовать # в начале для комментирования отдельных подсетей.
```

`--debug` allows you to output a detailed log of actions to the console, syslog or file. The order may be important.
options. `--debug` is best specified at the very beginning. Options are analyzed sequentially. If the error occurs when
checking the option, but it has not yet come to the analysis of `--debug`, then the messages will not be output to the file or syslog. At
When logging to a file, the process does not keep the file open. For each entry, the file is opened and then closed. So
the file can be deleted at any time, and it will be created again at the first message to the log. But keep in mind that if you
If you run the process as root, the UID will be changed to non-root. At the beginning, the owner is changed to the log file, otherwise the entry will be
impossible. If you later delete a file and the process does not have rights to create a file in its directory, the log will no longer be
will be conducted. Instead of deleting, it is better to use truncate. In the shell this can be done using the command ": >filename"

Many options that load binary data from files support loading from a hex string or from a file.
hex string starts with "0x". You can write the file name as is or use the "@" prefix.
If "+<number>" is specified before the "@" prefix, then this means an offset of the payload within the file.
The file can be loaded entirely from position zero, modifications can be applied to it, requiring the complete file (TLS),
but the transmission will go from the offset position. offset must be less than the file length. If a mod is applied to a data block,
which reduces the data size, and offset is not less than the new data length, there will be an error.

### DPI DESYNCHRONIZATION ATTACK

Its essence is as follows. The original request is taken, modified, and fake information is added (fakes)
in such a way that the server OS transmits the original request unchanged to the server process, and DPI sees something else.
Something he won't block. The server sees one thing, DPI - another. DPI does not understand that a prohibited request is being sent and does not block it.

There are an arsenal of possibilities to achieve such a result.
This could be the transmission of fake packets so that they reach the DPI, but do not reach the server. Fragmentation can be used at the TCP level (segmentation) or at the IP level.
There are attacks based on playing with tcp sequence numbers or mixing up the order of tcp segments.
The methods can be combined in various ways.

### FAKES

Fakes are individual packets generated by nfqws that carry false information for DPI.
They either should not reach the server, or they can reach it, but must be discarded by it.
Otherwise, the TCP connection is broken or the integrity of the transmitted stream is violated, which is guaranteed to lead to a breakdown of the resource.
There are a number of methods to solve this problem.

* `md5sig` adds the TCP option **MD5 signature**. Does not work on all servers. Packages with md5 are usually only dropped by linux.
A significant increase in tcp packet length is required to accommodate the tcp option. When processing multi-hop requests (TLS Kyber)
the first packet is full under MTU. With fakedsplit/fakeddisorder at small positions, individual TCP segments are large enough for injection
md5 tcp option caused an MTU overflow and a "message too long" sending error. `nfqws` does not know how to redistribute data between tcp segments,
Therefore, we must either abandon kyber, or increase the split position, or abandon fakedsplit/fakeddisorder.
* `badsum` corrupts the TCP checksum. It will not work if your device is behind a NAT that does not allow packets with an invalid amount to pass through. Most
The common NAT router configuration in Linux does not allow them to pass through. Most home routers are built on Linux.
Non-passing is ensured like this: default sysctl kernel setting
`net.netfilter.nf_conntrack_checksum=1` causes conntrack to check tcp and udp checksums of incoming packets and
set state INVALID for packages with an invalid amount. Usually in iptables rules a rule for drop is inserted
packets with the INVALID state in the FORWARD chain. The combined combination of these factors leads to failure of badsum
through such a router. In OpenWrt out of the box `net.netfilter.nf_conntrack_checksum=0`, in other routers there is often no, and not
this can always be changed. In order for nfqws to work through a router, you need to set the specified sysctl value to 0.
nfqws on the router itself will work without this setting, because the checksum of locally created packets is not
never checked. If the router is behind another NAT, for example the provider's, and it does not allow invalid packets, you will do nothing
you can't do anything about it. But usually providers still skip badsum. On some adapters/switches/drivers
rx-checksum offload is forcibly enabled, badsum packets are cut off before they are received in the OS. In this case, if something
can only be done by modifying the driver, which seems to be an extremely non-trivial task. It has been established that it is so
Some routers based on mediatek behave. badsum packets leave the client OS, but are not seen by the router in br-lan
via tcpdump. However, if nfqws is executed on the router itself, the bypass can work. badsum leave normally
external interface.
* `badseq` increases the TCP sequence number by a certain value, thereby displaying it from the TCP window.
Such packets will most likely be discarded by the receiving node, but so will DPI if it is sequence oriented
numbers. By default, the seq offset is set to -10000. Practice has shown that some DPIs do not pass seq outside
specific window. However, such a small offset can cause problems with significant streaming and
packet loss. If you use `--dpi-desync-any-protocol` you may need to set badseq increment
0x80000000. This will provide a reliable guarantee that a fake packet will not wedge itself into the tcp window on the server. It was the same
  замечено, что badseq ломает логику некоторых DPI при анализе http, вызывая зависание соединения. Причем на тех же DPI
TLS with badseq works fine.
* `TTL` would seem to be the best option, but it requires individual configuration for each provider. If the DPI is
further than the provider’s local sites, then you can cut off your access to them. The situation is aggravated by the presence of TSPU on
highways, which forces the TTL to be quite high, increasing the risk of a fake reaching the server. IP required
exclude list, filled in manually. Together with ttl you can use md5sig. It won't spoil anything, but it will give a good
chance of operation of sites to which a “bad” packet will reach via TTL. If you can't find an automatic solution,
use the file `zapret-hosts-user-exclude.txt`. Some stock router firmware fixes the outgoing TTL,
Without disabling this option it will not work through them. WHAT SHOULD YOU CHOOSE TTL: find the minimum value when
which bypass still works. This will be your DPI hop number.
* `hopbyhop` only applies to ipv6. Added ipv6 extension header `hop-by-hop options`. In the `hopbyhop2` variant
2 headers are added, which is a violation of the standard and is guaranteed to be discarded by the protocol stack in all operating systems.
One hop-by-hop header is accepted by all operating systems, however, on some channels/providers such packets may be filtered and
don't reach. The calculation is that DPI will analyze the hop-by-hop packet, but it will either not reach the recipient due to
provider filters, or will be discarded by the server, because there are two headers.
* `datanoack` sends fakes with the tcp ACK flag removed. Servers don’t accept this, but DPI can. This technique
can break NAT and does not always work with iptables if masquerade is used, even from the local system (almost always
on ipv4 routers). On systems with iptables without masquerade and on nftables it works without restrictions. Experimentally
It turned out that many provider NATs do not discard these packets, so it even works with the provider’s internal IP.
But it will not pass Linux NAT, so this technique most likely will not work behind a home router, but it may work from it.
It can also work through a router if the connection is via a wire and hardware acceleration is enabled on the router.
* Manipulate tcp flags using `--dpi-desync-tcp-flags-set` and `--dpi-desync-tcp-flags-unset`. You can make it disabled
a combination of flags that the server will not accept, but DPI will accept. For example, set SYN in fakes. But this may not work on all servers.
`datanoack` can be replaced by `--dpi-desync-tcp-flags-unset=ACK`.
Packets with invalid flags may be discarded while traversing NAT.
* `ts` adds the value ts increment (default -600000) to the TSval value of the tcp timestamp. Servers are dropping packets
with TSval within certain limits. According to practical tests, the increment should be somewhere from -100 to -0x80000000.
timestamps are generated by the client OS. In Linux, timestamps are enabled by default, in Windows they are disabled by default.
Can be enabled via the command `netsh interface tcp set global timestamps=enabled`.
ts fooling requires that timestamps be enabled, otherwise it will not work. It must be enabled on each client device.
TSecr is left unchanged. It is also required that the server understands timestamps, but this is true in most cases.
* `autottl`. The essence of the mode is to automatically determine the TTL so that the packet almost certainly passes the DPI and does not reach the
server (`--dpi-desync-autottl`). Or vice versa - the TTL was barely enough for it to reach the server (see `--dup-autottl`, `--orig-autottl`).
The base TTL values ​​of 64,128,255 are taken and the incoming packet is looked at (yes, you need to send the first incoming packet to nfqws!).
The path length is calculated and `delta` is added. delta can be positive or negative.
To specify a positive delta, you need to specify the unary sign **+** in front of the number.
If it is absent or if the unary sign **-** is present, the delta is considered negative.
If the TTL is outside the min,max range, then the min,max values ​​are taken to fit within
range. If the delta is negative and the received TTL is greater than the path length, or the delta is positive and the received TTL is less than the path length,
then the automation did not work and fixed values ​​are taken: `--dpi-desync-ttl`, `--orig-ttl`, `--dup-ttl`.
The technique allows us to solve the problem when the entire network is blocked by barriers (DPI, TSPU) everywhere
possible, including highways. But it can potentially fail. For example, if there is an asymmetry between the incoming and outgoing channels
to a specific server. Some servers issue a non-standard TTL (google), so they turn out to be complete nonsense.
If you do not take into account such exceptions, then this technique will work well on some providers, but on others it will cause more problems,
than good. Somewhere you may need to tune parameters. It is better to use with an additional limiter.

Fooling modes can be combined in any combination. `--dpi-desync-fooling` takes multiple values ​​separated by commas.

It is possible to specify multiple fakes by repeating the `--dpi-desync-fake-???` parameters, except for `--dpi-desync-fake-syndata`.
Fakes will be sent in the order specified. `--dpi-desync-repeats` repeats every fake sent.
The final order will be like this: `fake1 fake1 fake1 fake2 fake2 fake2 fake3 fake3 fake3 .....`

### MODIFICATION OF FAKES

Any tcp fakes are sent with the original sequence by default, even if there are several of them.
If you set `--dpi-desync-fake-tcp-mod=seq`, then several fakes will be sent with increasing sequence number in this way,
as if they were tcp segments of one fake.

nfqws has a basic fake version for TLS built into it. It can be overridden with the `--dpi-desync-fake-tls` option.
Overriding fakes makes it possible to use any data as a fake for TLS.
You can use a fake Client Hello with any fingerprint and any SNI.

Some modifications can be made at runtime using `--dpi-desync-fake-tls-mod`.
Some of them work when processing each TLS Client Hello and can be adjusted to the data being sent.
Modifications require a fully valid TLS Client Hello as a fake, they do not work with arbitrary data.

 * `none`. Do not apply any modifications.
 * `rnd`. Randomize the `random` and `session id` fields. Executed for every request.
 * `dupsid`. Copy the `session ID` from the transmitted TLS Client Hello. Takes precedence over `rnd`. Executed for every request.
 * `rndsni`. Randomize SNI. If SNI >=7 characters, a random level 2 domain with a known TLD is used, otherwise it is filled with random characters without a dot. Executes once at startup.
 * `sni=<sni>`. Replace sni with the specified value. Max SNI length is 63 bytes. The total length of the TLS fake and the lengths in the TLS Client Hello structure change. Executes once at startup. If combined with `rndsni`, executed before it.
 * `padencap`. The padding extension is extended to the size of the transmitted TLS Client Hello (including the multi-packet version with kyber). If padding is missing, it is added to the end. If present, padding is required to come last extension. All lengths are corrected to create the appearance of including the transmitted TLS Client Hello in the padding extension. The size of the fake does not change. The calculation is based on DPI, which does not analyze sequence numbers properly. Executed for every request.

By default, if your own fake for TLS is not specified, the `rnd,rndsni,dupsid` modifications are used. If fake is specified, `none` is used.
This matches the behavior of older versions of the program with the addition of the `dupsid` function.

If a modification mode is specified and there are many TLS fakes, the latest modification mode is applied to each of them.
If the modification mode is set after the fake, then it replaces the previous mode.
This way you can use different modification modes for different fakes.
If it is impossible to modify the fake at the launch stage, the program ends with an error.

If a TLS fake comes first, the one-time modification mode is set for it, then a non-TLS fake comes, then there will be an error.
You need to use `--dpi-desync-fake-tls-mod=none'.

Пример : `--dpi-desync-fake-tls=iana_org.bin --dpi-desync-fake-tls-mod=rndsni --dpi-desync-fake-tls=0xaabbccdd --dpi-desync-fake-tls-mod=none'

### TCP SEGMENTATION

 * `multisplit`. we cut the request at the positions specified in `--dpi-desync-split-pos`.
 * `multidisorder`. we cut the request at the positions specified in `--dpi-desync-split-pos` and send it in reverse order.
 * `fakedsplit`. various options for mixing fakes and originals in direct order
 * `fakeddisorder`. various options for mixing fakes and originals in reverse order
 * `hostfakesplit` (altorder=0). faking part of the request with the host: original before the host, fake host, original host (+ optional cutting with a midhost marker), fake host, original after the host
 * `hostfakesplit` (altorder=1). faking part of the request with the host: original before the host, fake host, original after the host, original host (+ optional cutting with a midhost marker)
 * `fakeddisorder`. similar to `fakedsplit`, only in reverse order: fake 2nd part, 2nd part, fake 2nd part, fake 1st part, 1st part, fake 1st part.

For `fakedsplit` and `fakeddisorder` there are variations in the order of the segments.
The `--dpi-desync-fakedsplit-mod=altorder=N` parameter specifies a number that affects the presence of individual fakes:

Altorder modes for `fakedsplit` for the part of a multi-batch request where there is a split position:
 * `altorder=0`. fake part 1, part 1, fake part 1, fake part 2, part 2, fake part 2
 * `altorder=1`. Part 1, part 1 fake, part 2 fake, part 2, part 2 fake
 * `altorder=2`. Part 1, fake part 2, part 2, fake part 2
 * `altorder=3`. Part 1, fake part 2, part 2

Altorder modes for `fakeddisorder` for part of a multi-batch request where there is a split position:
 * `altorder=0`. fake part 2, part 2, fake part 2, fake part 1, part 1, fake part 1
 * `altorder=1`. Part 2, fake part 2, fake part 1, part 1, fake part 1
 * `altorder=2`. Part 2, fake part 1, part 1, fake part 1
 * `altorder=3`. Part 2, fake part 1, part 1

Altorder modes for `fakedsplit` and `fakeddisorder` for the part of a multi-batch request where there is no split position:
 * `altorder=0`. fake, original, fake
 * `altorder=8`. original, fake
 * `altorder=16`. original

The final number `altorder=N` is calculated as the sum of the numbers from these two groups. Default is `altorder=0`.

The content of fakes in `fakedsplit`/`fakeddisorder` is determined by the parameter `--dpi-desync-fakedsplit-pattern` (default 0x00).
Fake data is taken from the pattern with an offset corresponding to the offset of the sent parts, taking into account packet offsets in multi-packet requests.
The dimensions of the fakes correspond to the lengths of the parts being sent.
The purpose of these modes is to make it as difficult as possible to identify original data among fakes.

Using `fakedsplit` or `fakeddisorder` on TLS kyber with md5sig fooling may result in "message too long" errors if the split position is small,
because there will be MTU overshoot due to md5 tcp option.

The 'hostfakesplit' mode has the task of minimal interference by a fake - precisely on that part of the request, on the basis of which DPI makes a decision to block. Specifically, the host name.
By default, a fake host is generated randomly each time from the set `[a-z0-9]`. If the length is more than 7 characters, a period is added 3 characters before the end, simulating a TLD, and the last 3 characters are filled with one of several well-known TLDs.

You can override the generation template using `--dpi-desync-hostfakesplit-mod=host=<hostname>`. In the latter case, the specified hostname will always be on the right.
On the left it will be padded to the size of the original host as a subdomain with random characters. Example: "www.networksolutions.com" -> "h8xmdba4tv7a8.google.com".
If the size of the original host is smaller than the template, the template will be cut: "habr.com" -> "ogle.com".
If the size of the original host is larger than the template by 1, you will get an invalid empty subdomain: "www.xxx.com" => ".google.com".
Therefore, it is worth using the shortest possible hosts from those allowed: “ya.ru”, “vk.com”.

`--dpi-desync-hostfakesplit-mod=altorder=1` allows you to change the order of parts to an alternative option.
`altorder=1` sends fragments in such an order that when sequentially assembling segments on DPI, it receives a fully assembled original request with a spoofed host.
The real host comes in a separate segment after. That is, in this version the disorder variety is used. The server accepts fragments with a broken sequence order.

Optionally, you can cut the original host. For example, `--dpi-desync-hostfakesplit-midhost=midsld`. The cutting position must fall inside the host.
Multi-packet requests are only supported if the initial packet slicing does not include hostname positions. In the latter case, the foolishness is canceled.

The `fakedsplit` option has several alternative slicing orders - from 0 to 3. The mode is set in the `--dpi-desync-fakedsplit-mod=altorder=N` parameter.
Each subsequent altorder removes some of the fakes.

Markers are used to determine cutting positions.

* **Absolute positive marker** - numerical offset within a packet or group of packets from the beginning.
* **Absolute negative marker** is a numeric offset within a packet or group of packets from the next byte. -1 indicates the last byte.
* **Relative marker** - a positive or negative offset relative to a logical position within a packet or group of packets.

Relative positions:

* **method** - start of the HTTP method ('GET', 'POST', 'HEAD', ...). The method is usually always at position 0, but may move due to `--methodeol`. Then the position can become 1 or 2.
* **host** - начало имени хоста в известном протоколе (http, TLS)
* **endhost** - the byte following the last byte of the host name
* **sld** - the beginning of the 2nd level domain in the host name
* **endsld** - the byte following the last byte of the 2nd level domain in the host name
* **midsld** - middle of the 2nd level domain in the host name
* **sniext** - the beginning of the SNI extension data field in TLS. Any extension consists of 2-byte fields type and length, followed by a data field.

Example of a list of markers: `100,midsld,sniext+1,endhost-2,-10`.

When a packet is split, the first step is to resolve the markers - finding all the specified relative positions and applying offsets.
If a relative position is not present in the current protocol, such positions are not applied and are discarded.
Next, positions are normalized relative to the offset of the current packet in a group of packets (multi-packet TLS requests with kyber, for example).
All positions beyond the current package are discarded. The remaining ones are sorted in ascending order and duplicates are removed.
In the `multisplit` and `multidisorder` options, if there are no positions left, no splitting occurs.

The `fakedsplit` and `fakeddisorder` options apply only one split position. Its search among the list `--dpi-desync-split-pos` is carried out in a special way.
First, all relative markers are checked. If a suitable one is found among them, it is applied. Otherwise, all absolute markers are checked.
If nothing is found among them, position 1 is applied.

Например, можно написать `--dpi-desync-split-pos=method+2,midsld,5`. Если протокол http, разбиение будет на позиции `method+2`.
If the TLS protocol is at the `midsld` position. If the protocol is unknown and `--dpi-desync-any-protocol` is enabled, the split will be at position 5.
To make everything more clear, you can use different profiles for different protocols and indicate only one position that is definitely in this protocol.

### OVERLAPPING SEQUENCE NUMBERS

`seqovl` adds a byte to the beginning of one of the TCP segments `seqovl` with the sequence number shifted negatively by the amount `seqovl`.
For `split` - to the beginning of the first segment, for `disorder` - to the beginning of the penultimate segment sent (the second in the original order).

In the case of `split`, the calculation is based on the fact that the previous send, if there was one, has already entered the server application socket, so the new incoming part is only partially in
within the current window (in-window). At the front, the fake part is discarded, and the remaining part contains the original and
starts at the beginning of window, so it ends up on the socket. The server application receives everything that the client actually sends,
discarding the fake out-of-window part. But DPI cannot understand this, so it experiences sequence desynchronization.
It is mandatory that the first segment together with `seqovl` does not exceed the MTU length. This situation is recognized automatically on Linux and `seqovl` is canceled.
In other systems the situation will not be recognized and this will result in a broken connection. Therefore, choose the first split position and `seqovl` such that the MTU is not exceeded in any case.
Otherwise, the foolishness may not work or work chaotically.

For `disorder` overlap goes to the penultimate part of the packet sent.
For simplicity, we will assume that the split is into 2 parts; they are sent in the order “2 1” with the original order “1 2”.
It is imperative that `seqovl` be less than the position of the first split, otherwise everything sent will be transferred to the socket immediately, including the fake, breaking the application-level protocol.
This situation is easily detected by the program, and `seqovl` is canceled. Increasing the package size is impossible in principle.
If the condition is met, the 2nd part of the package is completely in-window, so the server OS accepts it in its entirety, including the fake.
But since the initial part of the data from packet 1 has not yet been received, the fake and real data remain in the kernel memory without being sent to the server application.
As soon as the 1st part of the packet arrives, it overwrites the fake part in the kernel memory.
The kernel receives data from parts 1 and 2, so the next step is sending to the application socket.
This is the behavior of all unix OSes, except solaris - leaving the last received data.
Windows leaves old data, so disorder with seqovl will lead to connection freezes
when working with Windows servers. Solaris is practically dead, there are very few Windows servers.
You can use sheets if necessary.
The method allows you to do without fooling and TTL. Fakes are mixed with real data.
`fakedsplit/fakeddisorder` still adds additional individual fakes.

`seqovl` in the `split` variant can only be an absolute positive value, since it is only applied in the first packet.
In the `disorder` option, all marker options are allowed.
They are automatically normalized to the current packet in the series. You can split on `midsld` and do seqovl on `midsld-1`.

### ASSIGNMENT IP_ID

Some DPIs strip the ipv4 field of the ip_id header. The protection consists of recognizing the ip_id assignment order, which is uncharacteristic for different OSes,
but typical for some anti-DPI software. Typically, OS increments ip_id for each subsequent packet.
For example, on TSPU, repetition of non-zero fake and non-fake ip_ids causes a block trigger on the `googlevideo.com` IP ranges.

If fakes or additional tcp segments are sent, then in any case the sequence will be broken, since the OS will not know anything about the fakes inserted
and will not increase its ip_id counter by the number of fakes or additional tcp segments.
To maintain consistency, it would be necessary to intercept the entire connection until the end, which is very resource-intensive.
Therefore, after processing a series of generated packets, ip_id returns to the value that the OS knows about.

The `ip-id` parameter refers to the profile and specifies the ip_id assignment mode when sending packets generated in nfqws.

 * `seq` (default) : take the last ip_id of the real packet. subsequent generated packets receive ip_id increased by 1, except in the case of `multidisorder`.
for `multidisorder` within segments where there are split positions, the ip_id value is increased by the number of parts, then decreased by 1 with each part sent.
 * `seqgroup` : same as `seq`, but fakes are the same size as the original segments, masquerading as the original and given the same ip_id.
 * `rnd` : assign a random ip_id to all generated packets
 * `zero` : assign ip_id=0 to all generated packets. in this case Linux and BSD will send 0, Windows will assign consecutive ip_ids to all packets (thereby automatically solving the packet counter glitch problem).

There is no ip_id field in ipv6 headers; the parameter is ignored for ipv6.

### IPV6 SPECIFIC MODES

The desynchronization modes `hopbyhop`, `destopt` and `ipfrag1` (not to be confused with fooling!) apply only to ipv6 and are
in adding a `hop-by-hop options`, `destination options` or `fragment` header to all packages subject to desynchronization.
Here you must understand that adding a header increases the size of the package, and therefore cannot be used
to maximum size packages. This occurs when transmitting large messages.
If it is impossible to send the package, the scam will be canceled and the package will be sent in the original.
The calculation is that DPI will see 0 in the next header field of the main `ipv6` header and will not jump across
extension headers in search of a transport header. Thus, it will not understand that it is tcp or udp, and will miss the packet
without analysis. Perhaps some DPIs will buy into this.
Can be combined with any 2nd phase modes, except for the `ipfrag1+ipfrag2` option.
For example, `hopbyhop,multisplit` means to split the TCP packet into several segments and add a hop-by-hop to each of them.
With `hopbyhop,ipfrag2` the sequence of headers will be: `ipv6,hop-by-hop`,`fragment`,`tcp/udp`.
The `ipfrag1` mode may not always work without special preparation. See `IP fragmentation` section.

### MODIFICATION OF THE ORIGINAL

The `--orig-ttl` and `--orig-ttl6` options allow you to change the TTL of the original packets.
If further manipulations are related to the original, for example, TCP segmentation is in progress, then the original
the data are modified original packages. That is, in this example, TCP segments will have a changed TTL.

The `--orig-autottl` and `--orig-autottl6` options work similarly to `dpi-desync-autottl`, but using the original packages.
The delta should be specified as positive with a unary `+` sign, otherwise the original will not reach the server, and you will not receive anything at all.
Example: `--orig-autottl=+5:3-64`.

`--orig-mod-start` and `--orig-mod-cutoff` set a limiter on the beginning and end of modification of the original.
Scheme analogous to `--dpi-desync-start` and `--dpi-desync-cutoff`.

The function can be useful when DPI hunts for fakes and blocks the connection if there are suspicious signs,
in particular, the changed TTL of the fake relative to the original.

### DUPLICATES

Duplicates are copies of the original packages sent before them. Enabled by the parameter `--dup=N`, where N is the number of takes,
not including the original. `--dup-replace` disables sending the original.

Sending duplicates occurs only in cases where the original is also sent without reconstruction.
For example, if TCP segmentation occurs, then the original is actually dropped and replaced with artificially constructed segments.
No duplicates will be sent. The same applies to changing the composition of ipv6 headers, tamper mode for DHT and others.

All variants of stupidity can be used, as for desync: `--dup-ttl`. `--dup-ttl6`, `--dup-fooling`. Whether these packets need to reach the server and in what form is up to you to decide according to your intended strategy.

The `--dup-autottl` and `--dup-autottl6` options work similarly to `dpi-desync-autottl`, but for duplicates.
The delta can be specified as positive with the unary sign `+`, or it can be negative. Depends on your idea.
Example: `--dup-autottl=-2:3-64`.

`--dup-start` and `--dup-cutoff` specify the start and end limit for applying the duplicate strategy.
Scheme analogous to `--dpi-desync-start` and `--dpi-desync-cutoff`.

The function can help when DPI shows the difference in the characteristics of fakes and the original.
With duplicates you can try to force DPI to accept that the entire session is anomalous.
For example, we have a TCP session with MD5 immediately from the first SYN packet. This means that subsequent MD5s will be perceived normally.

### COMBINING DESYNCHRONIZATION METHODS

In the dpi-desync parameter, you can specify up to 3 modes separated by commas.

* Phase 0 - involves work at the connection establishment stage: `synack`, `syndata`, `--wsize`, `--wssize`. This phase is not affected by filters based on [hostlist](#multiple-strategies), except for the case described [further](#cache-ip).
* Phase 1 - sending something before the original data packet: `fake`, `rst`, `rstack`.
* Phase 2 - sending a modified form of the original data packet (for example, `fakedsplit` or `ipfrag2`).

Modes require indication in ascending order of phase numbers.

### CACHE IP

ipcache is a structure in process memory that allows you to remember some information based on the IP address key and interface name,
which can later be extracted and used as missing data. Currently this applies in the following situations:

1. IP,interface => hop count . The number of hops to the server is cached for subsequent use in autottl directly from the first packet, when there has been no response yet. While there is no entry in the cache, autottl will not be applied immediately. If the request is repeated before the autottl entry lifetime expires, the autottl entry will be applied immediately.

2. IP => hostname. The host name is cached, outside of interface binding, for later use in phase zero strategies. The mode is disabled by default and is enabled through the `ipcache-hostname` parameters.
This technique is experimental. Its problem is that, as such, there is no one-to-one correspondence between the domain and the IP. Multiple domains can refer to the same IP address.
In the event of a collision, the host name is replaced with the latest option.
A domain can jump across different IPs on a CDN. One address now, another in an hour. This problem is solved through the lifetime of cache entries: `--ipcache-lifetime`. Default is 2 hours.
However, it may also happen that in your case the use of technology brings more benefits than problems. Be prepared for seemingly incomprehensible behavior that can only be investigated via `--debug` log.

When the SIGUSR2 signal is issued, the process prints the contents of ipcache to the console.

### DPI RESPONSE TO SERVER RESPONSE

There are DPIs that analyze responses from the server, in particular the certificate from ServerHello, where domains are registered.
Confirmation of delivery of ClientHello is an ACK packet from the server with an ACK sequence number corresponding to the length of ClientHello+1.
In the disorder variant, a partial acknowledgment (SACK) usually comes first, then a full ACK.
If instead of an ACK or SACK there is an RST packet with minimal delay, then DPI will cut you off at the stage of your request.
If the RST comes after a full ACK after a delay equal to approximately a ping to the server,
then probably DPI reacts to the server response.
DPI may lag behind the flow if the ClientHello satisfied it and does not check the ServerHello.
Then you're in luck. The fake option might work.
If it doesn’t lag behind and persistently checks ServerHello, then you can try to force the server to send ServerHello in parts
via `--wssize` option (see conntrack).
If this does not help, then it is hardly possible to do anything about it without help from the server.
The best solution is to enable TLS 1.3 support on the server. In it, the server certificate is transmitted in encrypted form.
This is a recommendation to all administrators of blocked sites. Enable TLS 1.3. This will give you more opportunities to overcome DPI.

### SYNACK MODE

In the geneva documentation this is called "TCB turnaround". An attempt to mislead DPI regarding
client and server roles.

Since the mode breaks NAT, the technique can only work if between the attacking device
and DPI no NAT. The attack will not work through a NAT router, but can work from one.
To implement an attack on passing traffic, nftables and the [POSTNAT](#nftables-for-nfqws) scheme are required.

### SYNDATA MODE

Everything is simple here. Data is added to the SYN packet. All operating systems ignore them unless TCP fast open (TFO) is used,
and DPI can be perceived without understanding whether there is TFO there or not.
The original connections to the TFO are not touched as this will definitely break them.
Without the qualifying parameter, 16 zero bytes are added.

### VIRTUAL MACHINES

Many nfqws packet magic techniques do not work from inside a VM from virtualbox and vmware in NAT mode.
TTL is forcibly replaced, fake packets do not pass through. It is necessary to configure the network in bridge mode.

### CONNTRACK

nfqws is equipped with a limited implementation of tracking the state of tcp connections (conntrack).
It is included to implement some DPI countermeasures.
conntrack is able to monitor the connection phase: SYN,ESTABLISHED,FIN, the number of packets in each direction,
sequence numbers. conntrack is able to “feed” packets in both directions or only in one direction.
A connection is included in the table when packets with the SYN or SYN,ACK flags are detected.
Therefore, if conntrack is needed, in the iptables redirection rules the connection should go to nfqws from the very first
packet, although it may then be terminated by the connbytes filter.
For UDP, the initiator of getting into the table is the first UDP packet. It also determines the direction of flow.
The first UDP packet is considered to come from the client to the server. Next, all packages with matching
`src_ip,src_port,dst_ip,dst_port` are considered to belong to this thread until the inactivity time expires.
conntrack is simple, it was not written taking into account all kinds of attacks on the connection, it does not check
packages for the validity of sequence numbers or checksum. His task is only to serve the needs of nfqws, he usually
It feeds only on outgoing traffic, therefore it is insensitive to substitutions from the external network.
A connection is removed from the table as soon as it is no longer needed to be monitored or after an inactivity timeout.
There are separate timeouts for each connection phase. These can be changed with the `--ctrack-timeouts` option.

`--wssize` allows you to change the size of the tcp window for the server from the client so that it sends the following responses in parts.
For this to work on all server OSes, it is necessary to change the window size in each packet outgoing from the client before sending the message,
the response to which must be split (for example, TLS ClientHello). This is why conntrack is needed to
know when to stop. If you don't stop and set low wssize all the time, the speed will drop catastrophically.
On Linux this can be stopped using connbytes, but on BSD systems there is no such option.
В случае http(s) останавливаемся сразу после отсылки первого http запроса или TLS ClientHello.
Если вы имеете дело с не http(s), то вам потребуется параметр `--wssize-cutoff`. Он устанавливает предел, с которого действие
wssize is terminated. The prefix d before the number means to consider only packets with data payload, the prefix s - relative sequence number,
In other words, the number of bytes transferred by the client + 1.
Если проскочит пакет с http request или TLS ClientHello, действие wssize прекращается сразу же, не дожидаясь wssize-cutoff,
if the `--wssize-forced-cutoff=0` option is not specified.
If your protocol is prone to long periods of inactivity, you should increase the ESTABLISHED phase timeout via the `--ctrack-timeouts` option.
The default timeout is low - only 5 minutes.
Don't forget that nfqws feeds on packages coming to it. If you have restricted packets coming through connbytes,
then the table may remain hanging connections in the ESTABLISHED phase, which will fall off only after a timeout.
To diagnose the conntrack state, send the SIGUSR1 signal to the nfqws process: `killall -SIGUSR1 nfqws`.
The current table will be printed to stdout by nfqws.

Usually, in the SYN packet, the client also sends, in addition to window size, a TCP extension `scaling factor`.
**scaling factor** is a power of two by which window size is multiplied: 0=>1, 1=>2, 2=>4, ..., 8=>256, ...
In the wssize parameter, the scaling factor is indicated separated by a colon.
The scaling factor can only decrease; increases are blocked to prevent the server from exceeding the window size.
To force the server to fragment the ServerHello to avoid leaking the server name from the server certificate to the DPI,
It's best to use `--wssize=1:6`. The basic rule is to make `scale_factor` as large as possible so that after recovery
window size the resulting window size has become the maximum possible. If you do 64:0 it will be very slow.
On the other hand, we cannot allow the server response to become large enough for DPI to find what it is looking for.

`--wssize` does not work in profiles with hostlists, because it is effective from the very beginning of the connection, when it is not yet possible
make a decision about getting on the list. However, a profile with auto hostlist may contain --wssize.
`--wssize` can slow down the speed and/or increase the response time of sites, so if there are other ways that work
bypass DPI, it is better to use them.

`--dpi-desync-cutoff` allows you to set a limit at which dpi-desync will stop being applied.
The available prefixes are n,d,s, similar to `--wssize-cutoff`.
Useful in conjunction with `--dpi-desync-any-protocol=1`.
On connections that are prone to inactivity, you should change the conntrack timeouts.
If the connection has dropped out of conntrack and the `--dpi-desync-cutoff` option is specified, `dpi desync` will not be applied.

### REASSEMBLING

nfqws supports reassembly of some types of queries.
Currently these are TLS and QUIC ClientHello. They can be long if you enable post-quantum in chrome
tls-kyber cryptography, and usually take 2 or 3 packages. kyber is enabled by default starting with chromium 124.
chrome randomizes the TLS fingerprint. SNI can be either at the beginning or at the end, that is
get into any package. stateful DPI usually reassembles the entire request, and only then
makes a decision to block.
If a TLS or QUIC packet is received with a partial ClientHello, the build process begins and the packages
are delayed and not sent until its end. Upon completion of the build, the packages go through desynchronization
based on the fully assembled ClientHello.
If there is any error in the build process, the delayed packets are immediately sent to the network and desynchronization is aborted.

There is special support for all tcp split options for multi-segment TLS.
If you specify a split position greater than the length of the first packet, then the split does not necessarily occur in the first packet, but in the
which accounted for the final position.
If, say, the client sent a TLS ClientHello of length 2000, the SNI starts at 1700,
and the options `fake,multisplit` are specified, then the first packet is preceded by fake, then the first packet in the original,
and the last packet is split into 2 segments. As a result, we have a fake at the beginning and 3 real segments.

### UDP SUPPORT

Attacks on udp are more limited in scope. udp cannot be fragmented other than at the ip level.

For UDP, only desynchronization modes `fake`, `fakeknown`, `hopbyhop`, `destopt`, `ipfrag1`, `ipfrag2`, `udplen`, `tamper` are valid.
The first phase modes are `fake`, `fakeknown`, `hopbyhop`, `destopt`, `ipfrag1`. The second phases are `ipfrag2`, `udplen`, `tamper`.
As usual, a combination of the first and second phase modes is possible, but not two modes of the same phase.

`udplen` increases the size of the udp packet by the number of bytes specified in `--dpi-desync-udplen-increment`.
Padding is filled with zeros by default, but you can set your own pattern.
Designed to cheat DPI based on packet sizes.
It may work if the user protocol is not strictly tied to the size of the udp payload.
Tamper mode means modifying packets of known protocols in a protocol-specific manner.
Currently only works with DHT.
Supports detection of QUIC Initial packets with content and hostname decryption, i.e. parameter
`--hostlist` will work.
The wireguard handshake initiation, DHT (starts with 'd1', ends with 'e'), STUN and
[Discord Voice IP Discovery](https://discord.com/developers/docs/topics/voice-connections#ip-discovery).
To desynchronize other protocols, be sure to specify `--dpi-desync-any-protocol`.
Implemented conntrack for udp. You can use --dpi-desync-cutoff. conntrack timeout for udp
can be changed by the 4th parameter in `--ctrack-timeouts`.
The fake attack is only useful for stateful DPI, it is useless for analysis at the individual packet level.
By default, fake content is 64 zeros. You can specify the file in `--dpi-desync-fake-unknown-udp`.

### IP FRAGMENTATION

The modern network practically does not allow fragmented tcp at the ip level to pass through.
This is better with udp, since some udp protocols can rely on this mechanism (IKE of older versions).
However, in some places it happens that fragmented udp is also cut.
Linux-based routers can spontaneously assemble or re-fragment packets.
The fragmentation position is set separately for tcp and udp. The default is 24 and 8 respectively, must be a multiple of 8.
The offset is calculated from the transport header.

There are a number of issues around working with fragments on Linux, without understanding which nothing may work.

ipv4: Linux allows sending ipv4 fragments, but the standard iptables settings in the OUTPUT chain may cause sending errors.

ipv6: There is no way for an application to be guaranteed to send fragments without defragmenting in conntrack.
It works differently on different systems. Somewhere they leave normally, somewhere the packets are defragmented.
For kernels <4.16, it looks like there is no other way to solve this problem other than unloading the `nf_conntrack` module.
which pulls up the `nf_defrag_ipv6` dependency. It just performs defragmentation.
For kernels 4.16+ the situation is slightly better. Packets in the NOTRACK state are excluded from defragmentation.
To avoid cluttering the description, see an example of a solution to this problem in `blockcheck.sh`.

Sometimes it is necessary to load the `ip6table_raw` module with the `raw_before_defrag=1` parameter.
In OpenWrt, module parameters are specified with a space after their names in the `/etc/modules.d` files.
On traditional systems, see if `iptables-legacy` or `iptables-nft` is used. If legacy, then you need to create a file
`/etc/modprobe.d/ip6table_raw.conf` with contents:
```
options ip6table_raw raw_before_defrag=1
```
On some traditional distributions you can change the current ip6tables via: update-alternatives --config ip6tables
If you want to stay on iptables-nft, you will have to rebuild the patched version. The patch is quite small.
In `nft.c` find the snippet:
```
        {
            .name	= "PREROUTING",
            .type	= "filter",
            .prio	= -300,	/* NF_IP_PRI_RAW */
            .hook	= NF_INET_PRE_ROUTING,
        },
        {
            .name	= "OUTPUT",
            .type	= "filter",
            .prio	= -300,	/* NF_IP_PRI_RAW */
            .hook	= NF_INET_LOCAL_OUT,
        },
```
and replace -300 with -450 everywhere.

This must be done manually; there is no automation in `blockcheck.sh`.

Or you can get rid of this problem once and for all by using `nftables`. There you can create a `netfilter hook`
with any priority. Use priority -401 and lower.

When using iptables and NAT, there seems to be no way to attach a queue handler after the NAT.
The packet enters nfqws with the source address of the internal network, then is fragmented and is no longer processed by NAT.
So it goes to the external network with src ip 192.168.x.x. Therefore, the method does not work.
Apparently the only working method is to abandon iptables and use nftables.
The hook must be priority 101 or higher.

### MULTIPLE STRATEGIES

**nfqws** is capable of reacting differently to different requests and using different strategies for fooling around.
This is implemented by supporting multiple fooling profiles.
Profiles are separated on the command line with the `--new` parameter. The first profile is created automatically.
It doesn't need `--new`. Each profile has a filter. By default it is empty, that is, the profile satisfies
any conditions.
The filter can contain hard parameters: IP protocol version, ipset and tcp/udp ports.
They are always uniquely identified even at the zero desynchronization phase, when the host and L7 are still unknown.
Host lists and the application layer protocol (l7) can act as a soft filter.
The L7 protocol usually becomes known after the first data packet.
When a request is received, profiles are checked in order from first to last to
achieving the first match with the filter.
Hard filter parameters are checked first. If there is a mismatch, there is an immediate transition to the next profile.
If a profile satisfies the hard filter and the L7 filter and contains an auto-hostlist, it is selected immediately.
If a profile satisfies the hard filter and L7 filter, it has a hostlist and we don't have a hostname yet,
there is a transition to the next profile. Otherwise, the hostlists of this profile are checked.
If the hostname matches the sheets, that profile is selected. Otherwise, it moves on to the next one.
It may happen that before receiving the host name or recognizing the L7 protocol, the connection goes through one profile,
and when figuring out these parameters, the profile changes on the fly. This can even happen twice - when figuring out L7
and hostname. Most often, this identification is combined into one action, since one packet, as a rule, identifies both L7 and the host.
Therefore, if you have zero-phase fooling parameters, carefully consider what can happen when switching strategies.
See the debug log to better understand what nfqws does.
Profiles are numbered from 1 to N. An empty profile with number 0 is created last in the chain.
It is used when no filter conditions match.

> [!IMPORTANT]
> Multiple strategies were created only for cases where it is impossible to combine
> available strategies for different resources. Copying strategies from blockcheck for different sites
> into many profiles without understanding how they work will lead to a clutter of parameters that don’t matter
> will not cover all possible blocked resources. You will only get stuck in this mess.

> [!IMPORTANT]
> The user-mode implementation of ipset was not created as a convenient replacement for the *nix version implemented in the kernel.
> The kernel version works much more efficiently. This was designed for systems without ipset support in the kernel.
> Specifically, Windows and Linux kernels compiled without nftables and ipset kernel modules. For example, android does not have ipset.

### FILTERING BY WIFI

The wifi network name has nothing to do with the network interface of the wifi adapter.
There is only one interface, you can connect to any network. Different networks have different strategies.
The strategy from network A does not work or breaks network B. What to do?

You can manually launch and remove nfqws instances. But you can do it differently.
In the windows version of winws there is a global filter `--ssid-filter`.
It enables or disables the winws instance depending on the connection of any adapter to a specific wifi network.
This does not take routing into account. This approach is possible because several winws instances on intersecting filters can be attached to windivert.
When changing wifi networks, some will turn on, others will turn off.

For Linux, a different solution is used. The `--filter-ssid` filter refers to a specific profile.
It is not possible to hang multiple nfqws instances on the same queue or route the same traffic to multiple queues.
Connecting and disconnecting different instances from the queue is associated with difficulties in synchronizing between them.
Therefore, one instance must process traffic, and it must be able to work with different wifi networks.
This is implemented in the `--filter-ssid` parameter. It takes a list of wifi network names (SSIDs) separated by commas, similar to `--ssid-filter` for winws.
When choosing a profile, it matters where the specific package being processed goes. Which interface? Or from which interface the packet came, if it is considered incoming.
Therefore, even if part of your traffic goes to one network, part to another, and part does not go over wifi at all, then all this can be configured.

Information about connected networks is obtained in the manner used by the `iw dev <ifname> info` command (nl80211).
Unfortunately, this method is broken on kernels from 5.19 to the latest (6.14 does not work).
In this case, the iwgetid (wireless extensions) method is used.
Wireless extensions are considered deprecated and are implemented on new cores as a compatibility layer.
Some kernels can be built without wireless extensions.
Before using `--filter-ssid`, make sure that any of these commands return the SSID.

All wifi interfaces are scanned and a list of interface->SSIDs is compiled. It is updated as it becomes available.
packets, but not more than 1 time per second.

### IPTABLES FOR NFQWS

> [!CAUTION]
> Starting with Linux kernels 6.17, there is a kernel configuration parameter CONFIG_NETFILTER_XTABLES_LEGACY, which may be "not set" by default in the distribution. Missing this setting disables iptables-legacy. This is part of the iptables deprecation process. However, iptables-nft will work because they use backend nftables.

iptables to launch an attack on the first data packets in a tcp connection:

```
iptables -t mangle -I POSTROUTING -o <внешний_интерфейс> -p tcp -m multiport --dports 80,443 -m connbytes --connbytes-dir=original --connbytes-mode=packets --connbytes 1:6 -m mark ! --mark 0x40000000/0x40000000 -j NFQUEUE --queue-num 200 --queue-bypass
```

Этот вариант применяем, когда DPI не следит за всеми запросами http внутри keep-alive сессии.
Если следит, направляем только первый пакет от https и все пакеты от http :

```
iptables -t mangle -I POSTROUTING -o <внешний_интерфейс> -p tcp --dport 443 -m connbytes --connbytes-dir=original --connbytes-mode=packets --connbytes 1:6 -m mark ! --mark 0x40000000/0x40000000 -j NFQUEUE --queue-num 200 --queue-bypass
iptables -t mangle -I POSTROUTING -o <внешний_интерфейс> -p tcp --dport 80 -m mark ! --mark 0x40000000/0x40000000 -j NFQUEUE --queue-num 200 --queue-bypass
```

mark is needed so that the generated fake package does not come back to us for processing. nfqws sets fwmark when sending it.
Although nfqws is able to independently distinguish marked packets, the mark filter in iptables is needed when using connbytes,
to prevent the packets from being reordered. Queue processing is a deferred process.
If the kernel has packets to be sent out of turn, it sends them immediately.
Changing the correct order of packets during desynchronization breaks the whole idea.
Deadlocks were also noticed when the number of packets sent from nfqws was large enough and there was no mark filter.
The process may freeze. Therefore, the presence of a mark filter in ip/nf tables can be considered mandatory.

Why `--connbytes 1:6` :
* 1 - for the operation of desynchronization methods of the 0th phase and correct operation of conntrack
* 2 - sometimes the data comes in the 3rd packet 3-way handshake
* 3 - standard situation of receiving one request packet
* 4-6 - in case of retransmission or a request several packets long (TLSClientHello with kyber, for example)

Autottl mode requires redirection of the incoming `SYN,ACK` packet or the first packet of the connection (which is usually the same thing).
Для режима autohostlist необходимы входящие RST и http redirect.
Можно построить фильтр на tcp flags для выделения `SYN,ACK` и модуле u32 для поиска характерных паттернов http redirect,
but it's easier to use connbytes to isolate a few initial incoming packets.

`
iptables -t mangle -I PREROUTING -i <внешний интерфейс> -p tcp -m multiport --sports 80,443 -m connbytes --connbytes-dir=reply --connbytes-mode=packets --connbytes 1:3 -m mark ! --mark 0x40000000/0x40000000 -j NFQUEUE --queue-num 200 --queue-bypass
`

For quick:

```
iptables -t mangle -I POSTROUTING -o <внешний_интерфейс> -p udp --dport 443 -m connbytes --connbytes-dir=original --connbytes-mode=packets --connbytes 1:6 -m mark ! --mark 0x40000000/0x40000000 -j NFQUEUE --queue-num 200 --queue-bypass
```

6 packets are taken to cover cases of possible retransmissions of quic initial in case of a bad connection or if the server is not feeling well, and the application insists on quic, without switching to tcp.
And also for autohostlist to work using quic. However, autohostlist for quic is not recommended.

### NFTABLES FOR NFQWS

You can start with a basic configuration.

```
IFACE_WAN=wan

nft create table inet ztest

nft add chain inet ztest post "{type filter hook postrouting priority mangle;}"
nft add rule inet ztest post oifname $IFACE_WAN meta mark and 0x40000000 == 0 tcp dport "{80,443}" ct original packets 1-6 queue num 200 bypass
nft add rule inet ztest post oifname $IFACE_WAN meta mark and 0x40000000 == 0 udp dport 443 ct original packets 1-6 queue num 200 bypass

# auto hostlist with avoiding wrong ACK numbers in RST,ACK packets sent by russian DPI
sysctl net.netfilter.nf_conntrack_tcp_be_liberal=1 
nft add chain inet ztest pre "{type filter hook prerouting priority filter;}"
nft add rule inet ztest pre iifname $IFACE_WAN tcp sport "{80,443}" ct reply packets 1-3 queue num 200 bypass
```

To enable IP fragmentation and `datanoack` on passing packets, a special chain configuration is required that redirects packets after NAT.
In zapret scripts this scheme is called `POSTNAT`, and it is only possible on nftables.
Packets generated by nfqws must be marked as **nottrack** early on so that they are not corrupted by NAT.

```
IFACE_WAN=wan

nft create table inet ztest

nft add chain inet ztest postnat "{type filter hook postrouting priority srcnat+1;}"
nft add rule inet ztest postnat oifname $IFACE_WAN meta mark and 0x40000000 == 0 tcp dport "{80,443}" ct original packets 1-6 queue num 200 bypass
nft add rule inet ztest postnat oifname $IFACE_WAN meta mark and 0x40000000 == 0 udp dport 443 ct original packets 1-6 queue num 200 bypass

nft add chain inet ztest predefrag "{type filter hook output priority -401;}"
nft add rule inet ztest predefrag "mark & 0x40000000 != 0x00000000 notrack"
```

Removing a test table:

```
nft delete table inet ztest
```

### FLOW OFFLOADING

If your device supports hardware acceleration (flow offloading, hardware nat, hardware acceleration), then
iptables may not work. When offloading is enabled, the packet does not go through the normal netfilter path. Necessary or its
disable, or selectively manage it.

The new kernels feature software flow offloading (SFO).
Packets passing through SFO also bypass most of the iptables mechanisms. When enabled SFO works
DNAT/REDIRECT (tpws). These connections are excluded from offloading. However, other connections go through SFO, because
NFQUEUE will only fire before the connection is placed in the flowtable. In practice, this means that almost all nfqws functionality will not work.
Offload is enabled via a special target in iptables `FLOWOFFLOAD` or via flowtable in nftables.

It is not necessary to pass all traffic through offload.
tpws already bypasses offload "by design", and to work with nfqws, the first few packets in a tcp connection or udp session are enough.
While the session is not directed to offload, it is processed in the usual way through a full-fledged netfilter.
As soon as the offload rule is triggered on any incoming or outgoing packet, the entire session finally leaves netfilter and goes offload.
Therefore, zapret scripts take the rules for the NFQUEUE that they created, and from them create exemption rules that prevent the session from going offload ahead of time, and then “release” it.
In this case, incoming packets are not allowed to start offloading; only outgoing packets are triggered.
This scheme provides almost zero negative impact on speed, while covering the needs of nfqws and simplifying the table rules.

OpenWrt does not provide offload selective control, so zapret scripts support their own selective control system.

iptables target `FLOWOFFLOAD` is a proprietary OpenWrt invention.
The offload control in nftables is implemented in the base Linux kernel without patches.
nftables is the only way to enable offload on classic Linux.

### FEATURES OF GLANDS

2 problems have been noticed on mediatek devices.

The mediatek ethernet driver discards tcp and udp packets with an incorrect checksum at the hardware level, this is not disabled.
As a result, fooling badsum will not work through the router, but will work from it.

Another mediatek issue affecting both ethernet and wireless appears on udp when offload rx-gro-list is enabled.
As long as nfqueue is missing, everything is fine. As soon as nfqueue appears, some packets drop out.
This is especially noticeable in the fooling of QUIC with kyber.

<details>
  <summary><b>shell код лечения</b></summary>

```
append_separator_list()
{
    # $1 - var name to receive result
    # $2 - separator
    # $3 - quoter
    # $4,$5,... - elements
    local _var="$1" sep="$2" quo="$3" i

    eval i="\$$_var"
    shift; shift; shift
    while [ -n "$1" ]; do
        if [ -n "$i" ] ; then
            i="$i$sep$quo$1$quo"
        else
            i="$quo$1$quo"
        fi
        shift
    done
    eval $_var="\$i"
}
resolve_lower_devices()
{
    # $1 - bridge interface name
    [ -d "/sys/class/net/$1" ] && {
        find "/sys/class/net/$1" -follow -maxdepth 1 -name "lower_*" |
        {
            local l lower lowers
            while read lower; do
                lower="$(basename "$lower")"
                l="${lower#lower_*}"
                [  "$l" != "$lower" ] && append_separator_list lowers ' ' '' "$l"
            done
            printf "$lowers"
        }
    }
}

# it breaks nfqueue
lans=$(resolve_lower_devices br-lan)
for int in $lans; do
    ethtool -K $int rx-gro-list off
done
```
</details>

This code must be called after the LAN interface is up, when all bridge members are already included in the bridge.
You can use the hook in `/etc/hotplug.d/iface`. `ethtool` must be installed.

mediatek issues have been confirmed on MT7621 (TP-Link Archer C6U v1) and MT7981 (Xiaomi AX3000T).
Other chipsets may or may not be similarly affected. There are no broader statistics.


### FOOLING FROM THE SERVER

This is also possible.
nfqws is designed to withstand client-side attacks, so it recognizes forward and reverse traffic based on its role in establishing a TCP connection.
If SYN passes, then the source IP is the client. If SYN,ACK passes, then the source IP is the server.
For UDP, the client is considered to be the source IP of the first packet passed through two ip-port links.
On the server, client traffic will be considered received traffic, and server traffic will be outgoing traffic.

`--wsize` works either way, it can be used on both the client and the server.
The remaining techniques only work if nfqws considers the traffic to be client traffic.
Therefore, to use them on traffic outgoing from the server, conntrack must be disabled with the `--ctrack-disable` parameter.
If the package is not found in conntrack, it is processed as if it were a client package.

Most protocols will not be recognized, because their recognition system is designed to contain packets from the client.
To enable techniques like `fake` or `multisplit` you need to use `--dpi-desync-any-protocol` with a connbytes limiter or
with a delimiter based on the contents of the packet or its headers.
start/cutoff are not available because they are tied to conntrack.

The `synack-split` technique allows you to split the tcp segment SYN,ACK into separate parts with SYN and ACK.
In response to this, the client sends SYN,ACK, which usually characterizes the server.
For some DPIs, this may break the algorithm, and they stop blocking prohibited content.
Здесь [подробное описание](https://nmap.org/misc/split-handshake.pdf) что есть split handshake.

Traffic is usually redirected according to the source port number and the original direction.
original is outgoing traffic from the system, reply is incoming traffic.


## tpws

tpws - это transparent proxy.
```
@<config_file>|$<config_file>                     ; читать конфигурацию из файла. опция должна быть первой. остальные опции игнорируются.

--debug=0|1|2|syslog|@<filename>                  ; 0,1,2 = логирование на косоль : 0=тихо, 1(default)=подробно, 2=отладка.
--debug-level=0|1|2                               ; указать уровень логирования для syslog и @<filename>
--dry-run                                         ; проверить опции командной строки и выйти. код 0 - успешная проверка.
--version                                         ; вывести версию и выйти

--daemon                                          ; демонизировать прогу
--pidfile=<file>                                  ; сохранить PID в файл
--user=<username>                                 ; менять uid процесса
--uid=uid[:gid]                                   ; менять uid процесса
--bind-addr                                       ; на каком адресе слушать. может быть ipv4 или ipv6 адрес
                                                  ; если указан ipv6 link local, то требуется указать с какого он интерфейса : fe80::1%br-lan
--bind-linklocal=no|unwanted|prefer|force         ; no : биндаться только на global ipv6
                                                  ; unwanted (default) : предпочтительно global, если нет - LL
                                                  ; prefer : предпочтительно LL, если нет - global
                                                  ; force : биндаться только на LL
--bind-iface4=<iface>                             ; слушать на первом ipv4 интерфейса iface
--bind-iface6=<iface>                             ; слушать на первом ipv6 интерфейса iface
--bind-wait-ifup=<sec>                            ; ждать до N секунд появления и поднятия интерфейса
--bind-wait-ip=<sec>                              ; ждать до N секунд получения IP адреса (если задан --bind-wait-ifup - время идет после поднятия интерфейса)
--bind-wait-ip-linklocal=<sec>
                                                  ; имеет смысл только при задании --bind-wait-ip
                                                  ; --bind-linklocal=unwanted	: согласиться на LL после N секунд
                                                  ; --bind-linklocal=prefer	: согласиться на global address после N секунд
--bind-wait-only                                  ; подождать все бинды и выйти. результат 0 в случае успеха, иначе не 0.
--connect-bind-addr                               ; с какого адреса подключаться во внешнюю сеть. может быть ipv4 или ipv6 адрес
                                                  ; если указан ipv6 link local, то требуется указать с какого он интерфейса : fe80::1%br-lan
                                                  ; опция может повторяться для v4 и v6 адресов
                                                  ; опция не отменяет правил маршрутизации ! выбор интерфейса определяется лишь правилами маршрутизации, кроме случая v6 link local.
--socks                                           ; вместо прозрачного прокси реализовать socks4/5 proxy
--no-resolve                                      ; запретить ресолвинг имен через socks5
--resolve-threads                                 ; количество потоков ресолвера
--port=<port>                                     ; на каком порту слушать
--maxconn=<max_connections>                       ; максимальное количество соединений от клиентов к прокси
--maxfiles=<max_open_files>                       ; макс количество файловых дескрипторов (setrlimit). мин требование (X*connections+16), где X=6 в tcp proxy mode, X=4 в режиме тамперинга.
                                                  ; стоит сделать запас с коэффициентом как минимум 1.5. по умолчанию maxfiles (X*connections)*1.5+16
--max-orphan-time=<sec>                           ; если вы запускаете через tpws торрент-клиент с множеством раздач, он пытается установить очень много исходящих соединений,
                                                  ; большая часть из которых отваливается по таймауту (юзера сидят за NAT, firewall, ...)
                                                  ; установление соединения в linux может длиться очень долго. локальный конец отвалился, перед этим послав блок данных,
                                                  ; tpws ждет подключения удаленного конца, чтобы отослать ему этот блок, и зависает надолго.
                                                  ; настройка позволяет сбрасывать такие подключения через N секунд, теряя блок данных. по умолчанию 5 сек. 0 означает отключить функцию
                                                  ; эта функция не действует на успешно подключенные ранее соединения

--local-rcvbuf=<bytes>                            ; SO_RCVBUF для соединений client-proxy
--local-sndbuf=<bytes>                            ; SO_SNDBUF для соединений client-proxy
--remote-rcvbuf=<bytes>                           ; SO_RCVBUF для соединений proxy-target
--remote-sndbuf=<bytes>                           ; SO_SNDBUF для соединений proxy-target
--nosplice                                        ; не использовать splice на linux системах
--skip-nodelay                                    ; не устанавливать в исходящих соединения TCP_NODELAY. несовместимо со split.
--local-tcp-user-timeout=<seconds>                ; таймаут соединений client-proxy (по умолчанию : 10 сек, 0 = оставить системное значение)
--remote-tcp-user-timeout=<seconds>               ; таймаут соединений proxy-target (по умолчанию : 20 сек, 0 = оставить системное значение)
--fix-seg=<int>                                   ; исправлять неудачи tcp сегментации ценой задержек для всех клиентов и замедления. ждать до N мс. по умолчанию 30 мс.
--ipcache-lifetime=<int>                          ; время жизни записей кэша IP в секундах. 0 - без ограничений.
--ipcache-hostname=[0|1]                          ; 1 или отсутствие аргумента включают кэширование имен хостов для применения в стратегиях нулевой фазы

--split-pos=N|-N|marker+N|marker-N                ; список через запятую маркеров для tcp сегментации
--split-any-protocol                              ; применять сегментацию к любым пакетам. по умолчанию - только к известным протоколам (http, TLS)
--disorder[=http|tls]                             ; путем манипуляций с сокетом вынуждает отправлять первым второй сегмент разделенного запроса
--oob[=http|tls]                                  ; отправить байт out-of-band data (OOB) в конце первой части сплита
--oob-data=<char>|0xHEX                           ; переопределить байт OOB. по умолчанию 0x00.
--hostcase                                        ; менять регистр заголовка "Host:". по умолчанию на "host:".
--hostspell=HoST                                  ; точное написание заголовка Host (можно "HOST" или "HoSt"). автоматом включает --hostcase
--hostdot                                         ; добавление точки после имени хоста : "Host: kinozal.tv."
--hosttab                                         ; добавление табуляции после имени хоста : "Host: kinozal.tv\t"
--hostnospace                                     ; убрать пробел после "Host:"
--hostpad=<bytes>                                 ; добавить паддинг-хедеров общей длиной <bytes> перед Host:
--domcase                                         ; домен после Host: сделать таким : TeSt.cOm
--methodspace                                     ; добавить пробел после метода : "GET /" => "GET  /"
--methodeol                                       ; добавить перевод строки перед методом  : "GET /" => "\r\nGET  /"
--unixeol                                         ; конвертировать 0D0A в 0A и использовать везде 0A
--tlsrec=N|-N|marker+N|marker-N                   ; разбивка TLS ClientHello на 2 TLS records на указанной позиции. Минимальное смещение - 6.
--mss=<int>                                       ; установить MSS для клиента. может заставить сервер разбивать ответы, но существенно снижает скорость
--tamper-start=[n]<pos>                           ; начинать дурение только с указанной байтовой позиции или номера блока исходяшего потока (считается позиция начала принятого блока)
--tamper-cutoff=[n]<pos>                          ; закончить дурение на указанной байтовой позиции или номере блока исходящего потока (считается позиция начала принятого блока)
--hostlist=<filename>                             ; действовать только над доменами, входящими в список из filename. поддомены автоматически учитываются, если хост не начинается с '^'.
                                                  ; в файле должен быть хост на каждой строке.
                                                  ; список читается при старте и хранится в памяти в виде иерархической структуры для быстрого поиска.
                                                  ; при изменении времени модификации файла он перечитывается автоматически по необходимости
                                                  ; список может быть запакован в gzip. формат автоматически распознается и разжимается
                                                  ; списков может быть множество. пустой общий лист = его отсутствие
                                                  ; хосты извлекаются из Host: хедера обычных http запросов и из SNI в TLS ClientHello.
--hostlist-domains=<domain_list>                  ; фиксированный список доменов через зяпятую. можно использовать # в начале для комментирования отдельных доменов.
--hostlist-exclude=<filename>                     ; не применять дурение к доменам из листа. может быть множество листов. схема аналогична include листам.
--hostlist-exclude-domains=<domain_list>          ; фиксированный список доменов через зяпятую. можно использовать # в начале для комментирования отдельных доменов.
--hostlist-auto=<filename>                        ; обнаруживать автоматически блокировки и заполнять автоматический hostlist (требует перенаправления входящего трафика)
--hostlist-auto-fail-threshold=<int>              ; сколько раз нужно обнаружить ситуацию, похожую на блокировку, чтобы добавить хост в лист (по умолчанию: 3)
--hostlist-auto-fail-time=<int>                   ; все эти ситуации должны быть в пределах указанного количества секунд (по умолчанию: 60)
--hostlist-auto-debug=<logfile>                   ; лог положительных решений по autohostlist. позволяет разобраться почему там появляются хосты.
--new                                             ; начало новой стратегии (новый профиль)
--skip                                            ; не использовать этот профиль . полезно для временной деактивации профиля без удаления параметров.
--filter-l3=ipv4|ipv6                             ; фильтр версии ip для текущей стратегии
--filter-tcp=[~]port1[-port2]|*                   ; фильтр портов tcp для текущей стратегии. ~ означает инверсию. поддерживается список через запятую.
--filter-l7=[http|tls|quic|wireguard|dht|unknown] ; фильтр протокола L6-L7. поддерживается несколько значений через запятую.
--ipset=<filename>                                ; включающий ip list. на каждой строчке ip или cidr ipv4 или ipv6. поддерживается множество листов и gzip. перечитка автоматическая.
--ipset-ip=<ip_list>                              ; фиксированный список подсетей через запятую. можно использовать # в начале для комментирования отдельных подсетей.
--ipset-exclude=<filename>                        ; исключающий ip list. на каждой строчке ip или cidr ipv4 или ipv6. поддерживается множество листов и gzip. перечитка автоматическая.
--ipset-exclude-ip=<ip_list>                      ; фиксированный список подсетей через запятую. можно использовать # в начале для комментирования отдельных подсетей.
```

### TCP SEGMENTATION IN TPWS

tpws, like nfqws, supports multiple query segmentation. Split positions are specified in `--split-pos`.
Markers are indicated separated by commas. For a description of the markers, see the [nfqws](#tcp-segmentation) section.

At the application level there is generally no guaranteed way to force the kernel to spit out
a block of data cut in a specific place. The OS keeps a send buffer (SNDBUF) on each socket.
If the socket has the TCP_NODELAY option enabled and the buffer is empty, then each send results in sending
a separate IP packet or a group of packets, if the block does not fit into one IP packet.
However, if at the time of send there is already an unsent buffer, then the OS will append the data to it,
There will be no sending in a separate package. But in this case there is no guarantee anyway,
that some message block will go at the beginning of the packet, which is what DPI is actually designed for.
The partitioning will be done according to the MSS, which depends on the MTU of the outgoing interface.
Thus, DPIs looking at the beginning of the TCP packet data field will be broken in any case.
Протокол http относится к запрос-ответным протоколам. Новое сообщение посылается только тогда,
when the server received the request and fully returned the response. This means that the request was actually not only sent,
but also accepted by the other side, and therefore the sending buffer is empty, and the next 2 sends will result
to sending data segments in different IP packets.

So tpws only provides splitting by making separate send calls, and this usually works reliably,
if you break it down not into too many parts and not into too small successive parts.
In the latter case, Linux may still merge some parts, which will lead to a mismatch of the actual segmentation
specified split positions. Other operating systems behave more predictably in this matter. No spontaneous unification was observed.
Therefore, you should not abuse splits and especially small neighboring packages.

As practice shows, problems can begin if the number of splits is more than one.
On some systems, stable results were observed up to 8 splits, on others, problems already began after 2 splits.
One split works reliably unless it is part of a massive streaming.
If segmentation fails, the message `WARNING! segmentation failed.
If you see it, this is a reason to reduce the number of split positions.
If this is not an option, there is a `--fix-seg` option for Linux kernels >=4.6. It allows you to wait for the sending to complete before sending the next part.
But this option breaks the asynchronous event processing model. While waiting, all other connections are not processed
and freeze for a short time. In practice, this may be a very small wait - less than 10 ms.
It is executed only if a split occurs and there is a real need for waiting.
This option is not recommended for highly loaded systems. But it may be suitable for home use, and you won’t even notice these delays.

If you are trying to split a massive transfer with `--split-any-protocol` when the information arrives faster than the send,
then without `--fix-seg` segmentation errors will pour in in a continuous stream.
Working on a massive stream without the `--tamper-start` and `--tamper-cutoff` limiters is usually meaningless.

tpws works at the socket level, so it receives a long request that does not fit into 1 packet (TLS with kyber) as a whole block.
For each split part, it makes a separate `send()` call. But the OS will not be able to send data in one packet if the size exceeds the MTU.
If the segment is too large, the OS will additionally cut it into smaller ones. The result should be similar to nfqws.

`--disorder` causes every 2nd packet to be sent with TTL=1, starting from the first.
All even-numbered packets arrive to the server at once. It retransmits the rest of the OS, and they come later.
This in itself creates additional latency (200ms on linux for the first retransmission).
There is no other way to create disorder in the socket version.
The final order for 6 segments is `2 4 6 1 3 5`.

`--oob` sends 1 byte of out-of-band data after the first split segment. `oob` in each segment of the split proved to be unreliable.
The server receives oob on the socket.

The combination of `oob` and `disorder` is only possible on Linux. Other operating systems don't know how to handle this. The URG flag is lost during retransmissions.
The server receives oob on the socket. The combination of these parameters in operating systems other than Linux causes an error during the startup phase.

### TLSREC

`--tlsrec` allows you to cut the TLS ClientHello into 2 TLS records within one tcp segment. Can use standard
marker mechanism for specifying relative positions.

`--tlsrec` ломает значительное количество сайтов. Криптобиблиотеки (openssl, ...) на оконечных http серверах
TLS separated segments are accepted without problems, but middleboxes do not always. Middleboxes include CDN
or DDoS protection systems. Therefore, using `--tlsrec` without delimiters is hardly advisable.
In the Russian Federation, `--tlsrec` usually does not work with TLS 1.2, because the censor parses the server certificate from ServerHello.
Works only with TLS 1.3, since this information is encrypted there.
However, now there are few sites left that do not support TLS 1.3.

### MSS

`--mss` sets the TCP_MAXSEG socket option. The client issues this value in the tcp options of the SYN packet.
The server responds with its MSS in SYN,ACK. In practice, servers usually reduce the size of the packets they send, but they
still do not fit into the low MSS specified by the client. Typically, the more the client specified, the more
sends the server. On TLS 1.2, if the server splits the cast so that the domain from the certificate does not end up in the first packet,
this can fool the DPI, secant response of the server.
The scheme can significantly reduce speed and does not work on all sites.

It is compatible with the hostlist filter only in [some cases](#multiple-strategies-1), when it is possible to find out the hostname at the time of the foolishness.

By applying this option to TLS1.3 sites, if the browser also supports TLS1.3, then you are only making things worse.
But there is no way to automatically know when to use it and when not to, since MSS only goes into
3-way handshake even before data exchange, and the TLS version can only be determined by the server response, which
may cause DPI reaction.
Use only when there is nothing better or for individual resources.
Для http использовать смысла нет, поэтому заводите отдельный desync profile с фильтром по порту 443.
Works only on Linux, does not work on BSD and MacOS.

### OTHER DAMING PARAMETERS

The `--hostpad=<bytes>` parameter adds padding headers before `Host:` by the specified number of bytes.
If the `<bytes>` size is too large, then it is split into different 2K headers.
Общий буфер приема http запроса - 64K, больший паддинг не поддерживается, да и http сервера
this is no longer accepted.
Useful against DPIs performing TCP reassembly with a limited buffer.
Если техника работает, то после некоторого количества bytes http запрос начнет проходить до сайта.
If the critical padding size is around MTU, then most likely DPI does not perform packet reassembly, and it would be better to use regular TCP segmentation options.
If reassembly is still performed, then the critical size will be around the size of the DPI buffer. It can be 4K or 8K, other values ​​are possible.

### MULTIPLE STRATEGIES

They work similarly to **nfqws**, except for some points.
There is no `--filter-udp` option because **tpws** does not support udp.
Zero phase methods (`--mss`) can work on a hostlist only in two cases:
if the socks mode and remote resolving of hosts through a proxy are used, or the [IP cache] system (#cache-ip) is used to remember the IP->hostname correspondence.
The functionality of your setup in the same mode may depend on
whether the client uses remote resolving. This may not be obvious. It works in one program, but not in another.

If you are using a profile with a hostlist, and you always need mss, specify mss in the profile with a hostlist,
create another profile without a hostlist, if you don’t already have one, and specify mss in it again.
Then, in any case, mss will be executed.

If you need mss by hostlist, specify `--mss` only in the profile with the hostlist and make sure that any of the necessary conditions for working in this mode are present.

Use `curl --socks5` and `curl --socks5-hostname` to test your strategy.
See the output of `--debug` to ensure the settings are correct.

### SERVICE PARAMETERS

`--debug` allows you to output a detailed log of actions to the console, syslog or file.
The order of the options may be important. `--debug` is best specified at the very beginning.
Options are analyzed sequentially. If there is an error when checking an option, but the analysis of `--debug` has not yet come,
then messages will not be output to a file or syslog.
`--debug=0|1|2` allows you to immediately enable logging to the console and specify the level in one parameter.
Retained for compatibility with older versions. To select the level in syslog or file mode use
separate parameter `--debug-level`. If in these modes `--debug` you do not specify the level via `--debug-level`, then
Level 1 is automatically assigned.
When logging to a file, the process does not keep the file open. For each entry, the file is opened and then closed.
So the file can be deleted at any time, and it will be created again at the first message to the log.
But keep in mind that if you run the process as root, the UID will be changed to non-root.
At the beginning, the owner is changed to the log file, otherwise recording will be impossible. If you later delete the file,
and the process will no longer have rights to create a file in its directory, the log will no longer be kept.
Instead of deleting, it is better to use truncate.
In the shell this can be done using the command ": >filename"

tpws can be bound to many interfaces and IP addresses (up to 32 pcs).
There is always only one port.
The `--bind-iface*` and `--bind-addr` options create a new bind.
The remaining parameters `--bind-*` refer to the last bind.
To bind to all ipv4, specify `--bind-addr "0.0.0.0"`, for all ipv6 - `"::"`. `--bind-addr=""` - bind to all ipv4 and ipv6.
Selecting the mode of using link local ipv6 addresses (`fe80::/8`):
```
--bind-iface6 --bind-linklocal=no : сначала приватный адрес fc00::/7, затем глобальный адрес
--bind-iface6 --bind-linklocal=unwanted : сначала приватный адрес fc00::/7, затем глобальный адрес, затем link local.
--bind-iface6 --bind-linklocal=prefer : сначала link local, затем приватный адрес fc00::/7, затем глобальный адрес.
--bind-iface6 --bind-linklocal=force : только link local
```
If no bind is specified, then a default bind is created for all addresses of all interfaces.
To bind to a specific link-local address, do this: `--bind-iface6=fe80::aaaa:bbbb:cccc:dddd%iface-name`
The `--bind-wait*` parameters can help in situations where you need to take an IP from an interface, but it is not there yet, it is not up
or not configured.
Different systems catch ifup events differently and do not guarantee that the interface has already received an IP address of a certain type.
In general, there is no single mechanism to respond to an event like “a link local address has appeared on interface X.”
To bind to a known ip, when the interface is not yet configured, you need to do this: `--bind-addr=192.168.5.3 --bind-wait-ip=20`
In transparent mode, a bind is possible to any non-existent address, in socks mode - only to an existing one.

The rcvbuf and sndbuf parameters allow you to set setsockopt SO_RCVBUF SO_SNDBUF for local and remote connections.

`--skip-nodelay` can be useful when tpws is being used without fooling around, to match the MTU to the MTU of the system on which tpws is running.
This can be useful for hiding the fact that you are using a VPN. Reduced MTU - 1 of the detection methods
suspicious connection. With tcp proxy, your connections are indistinguishable from those the gateway itself would make.

`--local-tcp-user-timeout` and `--remote-tcp-user-timeout` set the timeout value in seconds
for client-proxy and proxy-server connections. This timeout corresponds to the linux socket option
TCP_USER_TIMEOUT. Timeout refers to the time during which the buffered data
not transmitted or an acknowledgment (ACK) from the other party was not received for the transmitted data.
This timeout has nothing to do with the time of absence of any transmission through the socket only because
that there is no data to transfer. Useful for reducing the time it takes to close hanging connections.
Supported on Linux and MacOS only.

The `--socks` mode does not require elevated privileges (except for the bind to privileged ports 1..1023).
Socks 4 and 5 versions are supported without authorization. The protocol version is recognized automatically.
Connections to the IP of the same device on which tpws is running, including localhost, are prohibited.
socks5 allows you to remotely resolve hosts (curl : --socks5-hostname firefox : socks_remote_dns=true).
tpws supports this feature asynchronously, without blocking the processing of other connections, using
multi-threaded pool of resolvers. The number of threads is determined automatically depending on `--maxconn`,
but you can prepare and the elm turn parameter `--resolver-threads`.
The request to socks is paused until the domain is converted to an IP address in one of the threads
resolver. The wait may be longer if all threads are busy.
If the `--no-resolve` option is specified, connections by hostname are denied and a resolver pool is not created.
This saves resources.

### IPTABLES FOR TPWS

To redirect a tcp connection to a transparent proxy, use the following commands:

```
iptables -t nat -I OUTPUT -o <внешний_интерфейс> -p tcp --dport 80 -m owner ! --uid-owner tpws -j DNAT --to 127.0.0.127:988
iptables -t nat -I PREROUTING -i <внутренний_интерфейс> -p tcp --dport 80 -j DNAT --to 127.0.0.127:988
```

The first command is for connections from the system itself, the second is for connections passing through the router.

DNAT on localhost works in the OUTPUT chain, but does not work in the PREROUTING chain without enabling the parameter
route_localnet :

`sysctl -w net.ipv4.conf.<internal_interface>.route_localnet=1`

You can use `-j REDIRECT --to-port 988` instead of DNAT, however in this case the transparent proxy process must
listen on the IP address of the incoming interface or on all addresses. Listening to everyone is not good from the point of view
security. You can listen on one (local) one, but in the case of an automated script you will have to recognize it, then
dynamically fit into the command. In any case, additional effort is required. Using route_localnet also has
potential security problems. You make available everything that hangs on `127.0.0.0/8` for the local subnet <
internal_interface>. Services are usually bound to `127.0.0.1`, so you can use iptables to block incoming
to `127.0.0.1` not from the lo interface, or hang tpws on any other IP from `127.0.0.0/8`, for example on `127.0.0.127`,
and allow incoming people not from lo only to this IP.

```
iptables -A INPUT ! -i lo -d 127.0.0.127 -j ACCEPT
iptables -A INPUT ! -i lo -d 127.0.0.0/8 -j DROP
```

The filter by owner is necessary to exclude recursive redirection of connections from tpws itself. tpws runs under
by the user **tpws**, an exclusion rule is set for it.

ip6tables work almost exactly the same as ipv4, but there are a number of important nuances. In DNAT you should take the --to address in
square brackets. For example :

`ip6tables -t nat -I OUTPUT -o <external_interface> -p tcp --dport 80 -m owner ! --uid-owner tpws -j DNAT --to [::1]:988`

The route_localnet parameter does not exist for ipv6. DNAT on localhost (::1) is only possible in the OUTPUT chain. In a chain
PREROUTING DNAT is possible on any global address or link local address of the same interface from which the packet came.
NFQUEUE works without changes.

### NFTABLES FOR TPWS

Basic configuration:

```
IFACE_WAN=wan
IFACE_LAN=br-lan

sysctl -w net.ipv4.conf.$IFACE_LAN.route_localnet=1

nft create table inet ztest

nft create chain inet ztest localnet_protect
nft add rule inet ztest localnet_protect ip daddr 127.0.0.127 return
nft add rule inet ztest localnet_protect ip daddr 127.0.0.0/8 drop
nft create chain inet ztest input "{type filter hook input priority filter - 1;}"
nft add rule inet ztest input iif != "lo" jump localnet_protect

nft create chain inet ztest dnat_output "{type nat hook output priority dstnat;}"
nft add rule inet ztest dnat_output meta skuid != tpws oifname $IFACE_WAN tcp dport { 80, 443 } dnat ip to 127.0.0.127:988
nft create chain inet ztest dnat_pre "{type nat hook prerouting priority dstnat;}"
nft add rule inet ztest dnat_pre meta iifname $IFACE_LAN tcp dport { 80, 443 } dnat ip to 127.0.0.127:988
```

Removing a table:
```
nft delete table inet ztest
```

## ip2net

The ip2net utility is designed to convert an ipv4 or ipv6 ip list into a list of subnets
in order to reduce the size of the list. Input is taken from stdin, output is given in `stdout`.

```
-4                             ; лист - ipv4 (по умолчанию)
-6                             ; лист - ipv6
--prefix-length=min[-max]      ; диапазон рассматриваемых длин префиксов. например : 22-30 (ipv4), 56-64 (ipv6)
--v4-threshold=mul/div         ; ipv4 : включать подсети, в которых заполнено по крайней мере mul/div адресов. например : 3/4
--v6-threshold=N               ; ipv6 : минимальное количество ip для создания подсети
```
The list may contain entries of the form ip/prefix and ip1-ip2. Such records are thrown to stdout without changes.
They are accepted by the ipset command. ipset can create optimal ip/prefix coverage for hash:net sheets from ip1-ip2.
FreeBSD's ipfw understands ip/prefix, but does not understand ip1-ip2.
ip2net filters input data, throwing out incorrect IP addresses.

A subnet is selected that contains the specified minimum of addresses.
For ipv4, the minimum is set as a percentage of the subnet size (mul/div. for example, 3/4), for ipv6 the minimum is set directly.

The subnet size is selected using the following algorithm:
First, in the specified range of prefix lengths, subnets are searched for in which the number of addresses is maximum.
If several such networks are found, the smallest network is taken (the prefix is ​​larger).
For example, the parameters v6_threshold=2 prefix_length=32-64 are set, the following ipv6 are available:
```
1234:5678:aaaa::5
1234:5678:aaaa::6
1234:5678:aaac::5
Результат будет :
1234:5678:aaa8::/45
```
These addresses are also included in the /32 subnet. However, there is no point in going through carpet bombing,
when the same addresses fit into /45 and there are exactly the same number of them.
If you change v6_threshold=4, the result will be:
```
1234:5678:aaaa::5
1234:5678:aaaa::6
1234:5678:aaac::5
```
That is, the IPs will not be combined into a subnet because there are too few of them.
If you change `prefix_length=56-64` the result will be:
```
1234:5678:aaaa::/64
1234:5678:aaac::5
```

The required CPU time for calculations depends heavily on the width of the range of prefix lengths, the size of the subnets being searched, and the length of the tile.
If ip2net is taking too long, don't use too large subnets and reduce the range of prefix lengths.
Please note that mul/div arithmetic is integer. If the bit grid exceeds 32 bit, the result is unpredictable.
No need to do this: 5000000/10000000. 1/2 is much better.

## mdig

The program is designed for multi-threaded resolving of large sheets via system DNS.
It takes a list of domains from stdin and outputs the result of the resolving to stdout. Errors are output to stderr.

```
--threads=<threads_number>	; количество потоков. по умолчанию 1.
--family=<4|6|46>              ; выбор семейства IP адресов : ipv4, ipv6, ipv4+ipv6
--verbose                      ; дебаг-лог на консоль
--stats=N                      ; выводить статистику каждые N доменов
--log-resolved=<file>          ; сохранять успешно отресолвленные домены в файл
--log-failed=<file>            ; сохранять неудачно отресолвленные домены в файл
--dns-make-query=<domain>      ; вывести в stdout бинарный DNS запрос по домену. если --family=6, запрос будет AAAA, иначе A.
--dns-parse-query              ; распарсить бинарный DNS ответ и выдать все ivp4 и ipv6 адреса из него в stdout
```

The `--dns-make-query` and `--dns-parse-query` parameters allow resolving one domain through an arbitrary channel.
For example, this is how you can perform a DoH request using only mdig and curl :
```
mdig --family=6 --dns-make-query=rutracker.org | curl --data-binary @- -H "Content-Type: application/dns-message" https://cloudflare-dns.com/dns-query | mdig --dns-parse-query
```

## Ways to get a list of blocked IPs

!!! nftables cannot work with ipsets. Its own similar mechanism requires a huge amount of RAM
!!! for loading large sheets.  For example, even 256 Mb is not enough to store 100K records in nfset.
!!! If you need large sheets on home routers, roll back to iptables+ipset.

1) Add blocked domains to `ipset/zapret-hosts-user.txt` and run `ipset/get_user.sh`
The output will be `ipset/zapret-ip-user.txt` with IP addresses.

Scripts called get_reestr_* operate with a dump of the registry of blocked sites:

2) `ipset/get_reestr_resolve.sh` receives a list of domains from rublacklist and then resolves them into IP addresses
to the file ipset/zapret-ip.txt.gz. There are ready-made IP addresses in this list, but apparently they are there exactly in the form
which is entered into the register by RosKomPozor. Addresses can change, it’s a shame they don’t have time to update them, and providers rarely
   банят по IP : вместо этого они банят http запросы с "нехорошим" заголовком "Host:" вне зависимости
from IP address. Therefore, the script resolves everything itself, although it takes a lot of time.
The multi-threaded resolver mdig (our own development) is used.

3) `ipset/get_reestr_preresolved.sh`. the same as 2), only the already resolved list is taken
from a third party resource.

4) `ipset/get_reestr_preresolved_smart.sh`. same as 3), with the addition of the full range of some
autonomous systems (jumping IP addresses from cloudflare, facebook, ...) and some subdomains of blocked sites

Scripts called `get_antifilter_*` operate with lists of addresses and subnet masks from the antifilter.network and antifilter.download sites:

5) `ipset/get_antifilter_ip.sh`. получает лист https://antifilter.download/list/ip.lst.

6) `ipset/get_antifilter_ipsmart.sh`. получает лист https://antifilter.network/download/ipsmart.lst.
smart summarization of individual addresses from ip.lst using masks from /32 to /22

7) `ipset/get_antifilter_ipsum.sh`. получает лист https://antifilter.download/list/ipsum.lst.
summarization of individual addresses from ip.lst by mask /24

8) `ipset/get_antifilter_ipresolve.sh`. получает лист https://antifilter.download/list/ipresolve.lst.
pre-resolved list, similar to that obtained with get_reestr_resolve. only ipv4.

9) `ipset/get_antifilter_allyouneed.sh`. получает лист https://antifilter.download/list/allyouneed.lst.
A summary list of prefixes created from ipsum.lst and subnet.lst.

10) `ipset/get_refilter_ipsum.sh`.
    Список берется отсюда : https://github.com/1andrevich/Re-filter-lists

All variants of the considered scripts automatically create and fill in the ipset.
Options 2-10 additionally call option 1.

11) `ipset/get_config.sh`. this script calls what is written in the GETLIST variable from the config file
If the variable is not defined, then only the sheets for ipset nozapret/nozapret6 are resolved.

RKN sheets change all the time. New trends are emerging. RAM requirements may vary.
Therefore, an infrequent, but still regular audit of what is going on on your router is necessary.
Or you may only find out about the problem when your wifi starts to disappear constantly, and you have to
reboot it every 2 hours (sledgehammer method).

The most RAM-friendly options are `get_antifilter_allyouneed.sh`, `get_antifilter_ipsum.sh`, `get_refilter_*.sh`.

The `zapret-ip.txt` and `zapret-ipban.txt` sheets are saved in compressed form into .gz files.
This allows you to reduce their size many times and save space on the router.
You can disable sheet compression using the config parameter GZIP_LISTS=0.

On routers, it is not recommended to call these scripts more than once every 2 days, since saving is in progress
either to the internal flash memory of the router, or in the case of extroot - to a flash drive.
In both cases, writing too frequently can kill the flash drive, but if this happens to the internal
flash memory, then you will simply kill the router.

Forced updating of `ipset` is performed by the script `ipset/create_ipset.sh`.
If the `no-update` parameter is passed, the script does not update the `ipset`, but only creates it if it is missing and fills it.
This is useful when several consecutive script calls may occur. There is no point in refilling several times
`ipset` is a time-consuming operation on large sheets. Sheets can be updated once every few days, and only then
call `create_ipset` without the `no-update` parameter. In all other cases, you should use `no-update`.

The RKN list has already reached an impressive size of hundreds of thousands of IP addresses. Therefore, to optimize `ipset`
The `ip2net` utility is used. It takes a list of individual IP addresses and tries to intelligently create subnets from it to reduce
number of addresses. `ip2net` prunes incorrect entries in sheets, ensuring that there are no errors when loading them.
`ip2net` is written in C because the operation is resource intensive. The router may not support other methods.

You can list domains in `ipset/zapret-hosts-user-ipban.txt`. Their IP addresses will be placed
into a separate ipset `ipban`. It can be used to force wrap all
connections to a transparent proxy `redsocks` or to a VPN.

**IPV6** : If ipv6 is enabled, then additional sheets with the same name are created, but with a "6" at the end before the extension.
`zapret-ip.txt` => `zapret-ip6.txt`
The zapret6 and ipban6 ipsets are created.
Antifilter sheets do not contain a list of ipv6 addresses.

**IP EXCLUSION SYSTEM**. All scripts resolve the `zapret-hosts-user-exclude.txt` file, creating `zapret-ip-exclude.txt` and `zapret-ip-exclude6.txt`.
They are driven into the nozapret and nozapret6 ipsets. All rules created by init scripts are created taking into account these ipsets.
The IPs placed in them do not participate in the process.
`zapret-hosts-user-exclude.txt` can contain domains, ipv4 and ipv6 addresses or subnets.

**FreeBSD**. The ipset/*.sh scripts work the same on FreeBSD. Instead of ipset, they create lookup tables ipfw with similar names.
ipfw tables, unlike ipset, can contain both ipv4 and ipv6 addresses and subnets in one table, so there is no separation.

The LISTS_RELOAD config parameter specifies an arbitrary command to reload sheets.
This is especially useful on BSD systems with PF.
LISTS_RELOAD=- disables sheet reloading.

## Filtering by domain names

An alternative to ipset is to use tpws or nfqws with a list of domains.
Both daemons accept an unlimited number of include (`--hostlist`) and exclude (`--hostlist-exclude`) sheets.
The exclude sheets are checked first. When entering them, there is a renunciation of foolishness.
Next, if there are include sheets, the domain is checked to see if it is included in them. If you are not on the list, stop fooling around.
If all include sheets are empty, this is equivalent to the absence of include sheets. The restriction stops working.
In other cases, foolishness occurs.
There is not a single list - it’s always foolishness.
There is only an exclude list - fooling everyone except.
There is only an include list - fooling only them.
There are both - foolishness is only include, except exclude.

In the launch system this is played out as follows.
There are 2 include lists:
`ipset/zapret-hosts-user.txt.gz` or `ipset/zapret-hosts-user.txt`,
`ipset/zapret-hosts.txt.gz` or `ipset/zapret-hosts.txt`
and 1 exclude list
`ipset/zapret-hosts-user-exclude.txt.gz` или `ipset/zapret-hosts-user-exclude.txt`

In the `MODE_FILTER=hostlist` or `MODE_FILTER=autohostlist` filtering modes, the launch system passes **nfqws** or **tpws** all sheets whose files are present.
The transfer occurs by replacing the `<HOSTLIST>` and `<HOSTLIST_NOAUTO>` markers with real parameters `--hostlist`, `--hostlist-exclude`, `--hostlist-auto`.
If suddenly the include sheets are present, but they are all empty, then the work is similar to the absence of an include sheet.
The file is there, but despite this everything is fooled except exclude.
If you need exactly this mode, it is not necessary to delete `zapret-hosts-users.txt`. It is enough to make it empty.

Subdomains are taken into account automatically. For example, the line "ru" adds "\*.ru" to the list. The line "\*.ru" in the list will not work.
You can use the `^` character at the beginning of the host to override automatic subdomain counting.

The list of RKN domains can be obtained by scripts
```
ipset/get_reestr_hostlist.sh
ipset/get_antizapret_domains.sh
ipset/get_reestr_resolvable_domains.sh
ipset/get_refilter_domains.sh
```
It is placed in `ipset/zapret-hosts.txt.gz`.

When the modification time or file size changes, the lists are re-read automatically.
After non-atomic change operations, you can send a HUP signal to tpws/nfqws to force a reread of all sheets.

When filtering by domain names, the daemon must be launched without filtering by ipset.
tpws и nfqws решают нужно ли применять дурение в зависимости от хоста, полученного из протокола прикладного уровня (http, tls, quic).
When using large lists, including the RKN list, estimate the amount of RAM on the router!
If after starting the daemon the RAM is full or ooms occur, then you need to abandon such large lists.

## autohostlist filtering mode

This mode allows you to analyze both requests from the client and responses from the server.
If the host is not yet in any leaves and a blocking-like situation is detected,
the host is automatically added to the `autohostlist` list both in memory and in the file.
**nfqws** or **tpws** maintain this file themselves.
To prevent any host from getting caught in `autohostlist`, use `hostlist-exclude`.
If it does get there, delete the entry from the file manually. The processes will automatically re-read the file.
**tpws**/**nfqws** themselves assign the owner of the file to the user under which they work after resetting the privileges,
to be able to update the sheet.

In the case of **nfqws**, this mode requires redirection of incoming traffic as well.
It is highly recommended to use the `connbytes` limiter to prevent **nfqws** from handling gigabytes.
For the same reason, using this mode on BSD systems is not recommended. There is no `connbytes` filter.

On Linux systems, when using nfqws and the connbytes filter, you may need:
`sysctl net.netfilter.nf_conntrack_tcp_be_liberal=1`
It has been observed that some DPIs in Russia return RST with an incorrect ACK. This is accepted by the tcp/ip stack
linux, but after a while it acquires the INVALID status in conntrack. That's why rules with `connbytes` work
through time, not overdone RST package **nfqws*.

How can DPIs generally behave when they receive a “bad request” and decide to block:

1) Freeze: it simply freezes, blocking the passage of packets over the TCP channel.
2) RST: sends RST to client and/or server
3) Редирект: (только для http) отправляет редирект на сайт-заглушку
4) Подмена сертификата: (только для https) полный перехват TLS сеанса с попыткой всунуть что-то
your client. It is used infrequently, because browsers complain about it.

**nfqws** and **tpws** can cut options 1-3, they do not recognize 4.
Due to the specifics of working with individual packets or with a TCP channel, tpws and nfqws recognize these situations
differently.
What is considered a blocking-like situation:
1) **nfqws** Several retransmissions of the first request in a TCP session in which there is a host.
2) **nfqws,tpws** RST, which came in response to the first request with the host.
3) **nfqws,tpws** HTTP redirect, which came in response to the first request with the host, to the global address
with a 2nd level domain that does not match the 2nd level domain of the original request.
4) **tpws** closing the connection by the client after sending the first request to the host, if there was no one to it
response from the server. This usually happens due to a timeout when there is no response (a "hang" case).

To reduce the likelihood of false positives, there is a counter for situations similar to blocking.
If more than a certain number of them occur within a certain time, the host is considered blocked
and is entered into `autohostlist`. The strategy to bypass the blocking immediately begins to work.
If during the counting process the website responds without signs of blocking, the counter is reset.
This was probably a temporary site glitch.

In practice, working with this mode looks like this.
The first time the user visits the site and receives a stub, the connection is reset or the browser freezes,
crashing out due to a timeout with a message about the impossibility of loading the page.
You need to hit F5, forcing the browser to try again. After some try the site
starts working, and then it will always work.

With this mode, you can use bypass techniques that break a significant number of sites.
If the site does not behave as if it is blocked, then the bypass will not be applied.
Otherwise, there is nothing to lose anyway.
However, there may be temporary server failures leading to a situation similar to blocking.
False positives may occur. If this happens, the strategy may begin to break down
unblocked site. Unfortunately, you will have to control this situation manually.
Add such domains to `ipset/zapret-hosts-user-exclude.txt` to avoid repetition.
To later figure out why the domain was added to the list, you can enable `autohostlist debug log`.
It is useful because it works without constantly viewing the output of **nfqws** in debug mode.
Only the main events leading to the inclusion of a host in the list are recorded in the log.
From the log you can understand how to avoid false positives and whether this mode is suitable for you at all.

It is possible to use one `autohostlist` with multiple processes. All processes check the file modification time.
If the file was modified in another process, it is reread.
All processes must run under the same uid to have access rights to the file.

`zapret` scripts lead `autohostlist` to `ipset/zapret-hosts-auto.txt`.
`install_easy.sh` saves this file when upgrading `zapret`.
The `autohostlist` mode includes the `hostlist` mode.
Possible news `ipset/zapret-hosts-user.txt`, `ipset/zapret-hosts-user-exclude.txt`.

## Provider check

Before setting up, you need to do research on what kind of trouble your provider has given you.

You need to find out if it is replacing DNS and what DPI bypass method works.
The `blockcheck.sh` script will help you with this.

If the DNS is changed, but the provider does not intercept calls to third-party DNS, change the DNS to public.
For example: 8.8.8.8, 8.8.4.4, 1.1.1.1, 1.0.0.1, 9.9.9.9
If the DNS is spoofed and the provider intercepts calls to third-party DNS, configure `dnscrypt`.
Another effective option is to use the resolver from yandex 77.88.8.88 on a non-standard port 1253.
Many providers do not analyze DNS requests on non-standard ports.
`blockcheck` if it sees a DNS substitution, automatically switches to the DoH server.

You should run a `blockcheck` on several blocked sites and identify the general nature of the blocking.
Different sites can be blocked in different ways, you need to look for a technique that works on most.
To write the output of `blockcheck.sh` to a file, run: `./blockcheck.sh | tee /tmp/blockcheck.txt`.

Проанализируйте какие методы дурения DPI работают, в соответствии с ними настройте `/opt/zapret/config`.

Keep in mind that providers may have multiple DPIs or requests may go through different channels
using the load balancing method. Balancing may mean that different branches have different DPIs or
they are on different hops. This situation may result in unstable bypass operation.
They pulled the curl several times. Either it works, then connection reset or redirect. `blockcheck.sh` outputs
strange results. Then split works on the 2nd. hope, then on the 4th. The reliability of the result is questionable.
In this case, set up multiple repetitions of the same test. The test will only be considered successful if
if all attempts are successful.

When using `autottl` you should test as many different domains as possible. This technique
It may work stably on some providers, but on others you will need to find out under what parameters
it is stable, in the third there is complete chaos, and it is easier to give up.

`Blockcheck` has 3 scanning levels.
* `quick` - find at least something working as quickly as possible.
* `standard` makes it possible to conduct research into how and what DPI reacts to in terms of bypass methods.
* `force` gives maximum checks even in cases where the resource works without bypass or with simpler strategies.

There are a number of other parameters that will not be asked in the dialog, but which can be overridden via
variables.

```
CURL - замена программы curl
CURL_MAX_TIME - время таймаута curl в секундах
CURL_MAX_TIME_QUIC - время таймаута curl для quic. если не задано, используется значение CURL_MAX_TIME
CURL_MAX_TIME_DOH - время таймаута curl для DoH серверов
CURL_CMD=1 - показывать команды curl
CURL_OPT - дополнительные параметры curl. `-k` - игнор сертификатов. `-v` - подробный вывод протокола
CURL_HTTPS_GET=1 - использовать метод GET вместо HEAD для https
DOMAINS - список тестируемых доменов через пробел
IPVS=4|6|46 - тестируемые версии ip протокола
ENABLE_HTTP=0|1 - включить тест plain http
ENABLE_HTTPS_TLS12=0|1 - включить тест https TLS 1.2
ENABLE_HTTPS_TLS13=0|1 - включить тест https TLS 1.3
ENABLE_HTTP3=0|1 - включить тест QUIC
REPEATS - количество попыток тестирования
PARALLEL=0|1 - включить параллельные попытки. может обидеть сайт из-за долбежки и привести к неверному результату
SCANLEVEL=quick|standard|force - уровень сканирования
BATCH=1 - пакетный режим без вопросов и ожидания ввода в консоли
HTTP_PORT, HTTPS_PORT, QUIC_PORT - номера портов для соответствующих протоколов
SKIP_DNSCHECK=1 - отказ от проверки DNS
SKIP_IPBLOCK=1 - отказ от тестов блокировки по порту или IP
SKIP_TPWS=1 - отказ от тестов tpws
SKIP_PKTWS=1 - отказ от тестов nfqws/dvtws/winws
PKTWS_EXTRA, TPWS_EXTRA - дополнительные параметры nfqws/dvtws/winws и tpws, указываемые после основной стратегии
PKTWS_EXTRA_1 .. PKTWS_EXTRA_9, TPWS_EXTRA_1 .. TPWS_EXTRA_9 - отдельно дополнительные параметры, содержащие пробелы
PKTWS_EXTRA_PRE - дополнительные параметры для nfqws/dvtws/winws, указываемые перед основной стратегией
PKTWS_EXTRA_PRE_1 .. PKTWS_EXTRA_PRE_9 - отдельно дополнительные параметры, содержащие пробелы
SECURE_DNS=0|1 - принудительно выключить или включить DoH
DOH_SERVERS - список URL DoH через пробел для автоматического выбора работающего сервера
DOH_SERVER - конкретный DoH URL, отказ от поиска
UNBLOCKED_DOM - незаблокированный домен, который используется для тестов IP block
MIN_TTL,MAX_TTL - пределы тестов с TTL. MAX_TTL=0 отключает тесты.
MIN_AUTOTTL_DELTA,MAX_AUTOTTL_DELTA - пределы тестов с autottl по дельте. MAX_AUTOTTL_DELTA=0 отключает тесты.
SIMULATE=1 - включить режим симуляции для отладки логики скрипта. отключаются реальные запросы через curl, заменяются рандомным результатом.
SIM_SUCCESS_RATE=<percent> - вероятность успеха симуляции в процентах
```

Running example with variables:\
`SECURE_DNS=1 SKIP_TPWS=1 CURL_MAX_TIME=1 CURL=/tmp/curl ./blockcheck.sh`

**PORT SCAN**\
If a compatible `netcat` is present on the system (ncat from nmap or openbsd ncat. OpenWrt does not have one by default),
то выполняется сканирование портов http или https всех IP адресов домена.
If no IP responds, the result is obvious. You can stop scanning.
It will not stop automatically because netcats do not provide sufficient detail about the causes of the error.
If only part of the IP is available, then chaotic failures can be expected, because connection goes to a random address
from the list.

**CHECK FOR PARTIAL IP block**\
A partial block means a situation where there is a connection to ports, but through a specific transport
or the application protocol always receives a DPI response, regardless of the requested domain.
This check will also not give an automatic verdict/decision, because there can be a lot of variations.
Instead, the analysis of what is happening is entrusted to the user himself or those who will read the log.
The essence of this check is to try to pull an unblocked IP with a blocked domain and vice versa, analyzing
at the same time the DPI reaction. DPI response usually manifests itself in the form of a timeout (request hangup), connection reset
или http redirect на заглушку. Любой другой вариант скорее всего говорит об отсутствии реакции DPI.
В частности, любые http коды, кроме редиректа, ведущего именно на заглушку, а не куда-то еще.
On TLS - handshake errors without delays.
A certificate error can indicate either a DPI reaction with a MiTM attack (certificate substitution) or
that the receiving server of an unblocked domain still accepts your TLS `handshake` with a foreign domain,
trying to issue a certificate without the requested domain. Additional analysis is required.
If there is a reaction to a blocked domain on all IP addresses, then there is a domain block.
If there is a reaction to an unblocked domain on the IP addresses of a blocked domain, then there is an IP block.
Accordingly, if there is both, then there is both a block by IP and a block by domain.
An unblocked domain is first checked for availability at the original address.
If unavailable, the test is canceled because it will not be informative.

If it is determined that there is a partial IP block on DPI, then most likely all other tests will fail
regardless of traversal strategies. But there are some exceptions. For example, breaking through `ipv6
option headers`. Or make it so that it cannot recognize the application layer protocol.
Further tests may be worthwhile.

**EXAMPLES OF BLOCKING ONLY BY DOMAIN WITHOUT BLOCKING BY IP**

```
> testing iana.org on it's original
!!!!! AVAILABLE !!!!!
> testing rutracker.org on 192.0.43.8 (iana.org)
curl: (28) Operation timed out after 1002 milliseconds with 0 bytes received
> testing iana.org on 172.67.182.196 (rutracker.org)
HTTP/1.1 409 Conflict
> testing iana.org on 104.21.32.39 (rutracker.org)
HTTP/1.1 409 Conflict

> testing iana.org on it's original ip
!!!!! AVAILABLE !!!!!
> testing rutracker.org on 192.0.43.8 (iana.org)
curl: (28) Connection timed out after 1001 milliseconds
> testing iana.org on 172.67.182.196 (rutracker.org)
curl: (35) OpenSSL/3.2.1: error:0A000410:SSL routines::ssl/tls alert handshake failure
> testing iana.org on 104.21.32.39 (rutracker.org)
curl: (35) OpenSSL/3.2.1: error:0A000410:SSL routines::ssl/tls alert handshake failure

> testing iana.org on it's original ip
!!!!! AVAILABLE !!!!!
> testing rutracker.org on 192.0.43.8 (iana.org)
HTTP/1.1 307 Temporary Redirect
Location: https://www.gblnet.net/blocked.php
> testing iana.org on 172.67.182.196 (rutracker.org)
HTTP/1.1 409 Conflict
> testing iana.org on 104.21.32.39 (rutracker.org)
HTTP/1.1 409 Conflict

> testing iana.org on it's original ip
!!!!! AVAILABLE !!!!!
> testing rutracker.org on 192.0.43.8 (iana.org)
curl: (35) Recv failure: Connection reset by peer
> testing iana.org on 172.67.182.196 (rutracker.org)
curl: (35) OpenSSL/3.2.1: error:0A000410:SSL routines::ssl/tls alert handshake failure
> testing iana.org on 104.21.32.39 (rutracker.org)
curl: (35) OpenSSL/3.2.1: error:0A000410:SSL routines::ssl/tls alert handshake failure
```


**EXAMPLE OF A FULL IP BLOCK OR TCP PORT BLOCK IN THE ABSENCE OF A DOMAIN BLOCK**

```
* port block tests ipv4 startmail.com:80
  ncat -z -w 1 145.131.90.136 80
  145.131.90.136 does not connect. netcat code 1
  ncat -z -w 1 145.131.90.152 80
  145.131.90.152 does not connect. netcat code 1

* curl_test_http ipv4 startmail.com
- checking without DPI bypass
  curl: (28) Connection timed out after 2002 milliseconds
  UNAVAILABLE code=28

- IP block tests (requires manual interpretation)

> testing iana.org on it's original ip
!!!!! AVAILABLE !!!!!
> testing startmail.com on 192.0.43.8 (iana.org)
HTTP/1.1 302 Found
Location: https://www.iana.org/
> testing iana.org on 145.131.90.136 (startmail.com)
curl: (28) Connection timed out after 2002 milliseconds
> testing iana.org on 145.131.90.152 (startmail.com)
curl: (28) Connection timed out after 2002 milliseconds
```

## Selecting parameters

Файл `/opt/zapret/config` используется различными компонентами системы и содержит основные настройки.
It needs to be reviewed and edited if necessary.

On Linux systems you can choose to use `iptables` or `nftables`.
By default on traditional Linux, `nftables` is selected if nft is installed.
On OpenWrt, `nftables` is selected by default on new versions with firewall4.

`FWTYPE=iptables`

On `nftables` you can disable the standard scheme for intercepting traffic after NAT and switch to intercepting before NAT.
This will make it impossible to use some methods of fooling on passing traffic, as is the case with `iptables`.
nfqws will begin to receive packet addresses from the local network and display them in the logs.

`POSTNAT=0`

There are 3 standard launch options, configurable separately and independently: `tpws-socks`, **tpws**, **nfqws**.
They can be used either separately or together. For example, you need to make a combination
of the methods available only in **tpws** and only in **nfqws**. They can be used together.
**tpws** will transparently localize traffic on the system and apply its foolishness, **nfqws** will fool the traffic,
coming from the system itself after processing by **tpws**.
Or you can install socks proxy on the same system without parameters in order to gain access to bypassing blocks through a proxy.
Thus, all 3 modes may well be used together.
Also unconditionally and independently, in addition to the standard options, all custom scripts in `init.d/{sysv,openwrt,macos}/custom.d` are applied.

However, when combining tpws and nfqws with intersection via L3/L4 protocols, not everything is as simple as it might seem at first glance.
tpws always works first, followed by nfqws. nfqws receives already “stupid” traffic from tpws.
It turns out that the fool is fooling the fool, and the fool does not work because she was fooled.
This is such a fun moment. nfqws stops recognizing protocols and applying methods.
Some methods of fooling from tpws nfqws are able to recognize and work correctly, but most are not.
The solution is to use `--dpi-desync-any-protocol` in nfqws and work as with an unknown protocol.
Combining tpws and nfqws is an advanced option that requires a deep understanding of what is happening.
It is highly advisable to analyze nfqws actions using the `--debug` log. Is everything as you planned?

Simultaneous use of tpws and nfqws without crossing over L3/L4 (that is, nfqws - udp, tpws - tcp or nfqws - port 443, tpws - port 80 or nfqws - ipv4, tpws - ipv6) does not present any problems.

`tpws-socks` requires configuration of **tpws** parameters, but does not require traffic interception.
The remaining options require separate settings for traffic interception and options for the daemons themselves.
Each option involves launching one instance of the corresponding daemon. All the differences in fooling methods
для `http`, `https`, `quic` и т.д. должны быть отражены через схему мультистратегий.
In this sense, the setup is similar to the `winws` option on Windows, and transferring configs should not be too difficult.

The basic rule for setting up interception is to intercept only the minimum necessary.
Any interception of unnecessary things is a pointless load on your system.
The `--ipset` daemon options must be used wisely. You should not intercept all traffic so that later using the --ipset parameter
select only a handful of IPs. This will work, but is very inefficient in terms of system load.
Use kernel mode ipsets. If necessary, write and use `custom scripts`.
But if you are already working on all IPs, and you need to write a small IP specialization, then --ipset is quite appropriate.

For convenience, daemon settings can be written on multiple lines using double or single quotes.
To use the standard updatable hostlists from the `ipset` directory, use the <HOSTLIST> token.
It will be replaced with parameters corresponding to the MODE_FILTER mode, and real existing files will be substituted.
If MODE_FILTER does not assume a standard hostlist, <HOSTLIST> will be replaced with the empty string.
Standard hostlists should be inserted in the final strategies (default strategies) that close the chains by
group of filter parameters. There may be several such places.
There is no need to use <HOSTLIST> in narrow specializations and in those profiles that will definitely not be accessed
трафик с известными протоколами, откуда поддерживается извлечение имени хоста (`http`, `tls`, `quic`).
<HOSTLIST_NOAUTO> - это вариация, при которой стандартный автолист используется как обычный.
That is, blocked domains are not automatically added to this profile.
But if something is added on another profile, then this profile will accept the changes automatically.

***Change mark bit to prevent looping***\
`DESYNC_MARK=0x40000000`

***Changing the mark bit to mark packets passing through the POSTNAT scheme (nftables only)***\
`DESYNC_MARK_POSTNAT=0x20000000`

***If uncommented, marks packages that should be processed as zapret.***\
`#FILTER_MARK=0x10000000`

The bit must be set by your own rules.
* For iptables - in the mangle PREROUTING and mangle OUTPUT chains before the zapret rules (iptables -I _after_ applying the zapret rules).
* For nftables - in the output and prerouting hooks with priority -102 or lower.

Any marking criteria. For example, IP address or source interface. This is the answer to the question “how can I make sure that the TV does not go through the zapret or so that only my computer goes through it.”

***Enabling the standard tpws option in socks mode***\
`TPWS_SOCKS_ENABLE=0`

***What port will tpws socks listen on. only localhost and LAN***\ are listened to
`TPPORT_SOCKS=987`

***tpws parameters for socks mode***
```
TPWS_SOCKS_OPT="
--filter-tcp=80 --methodeol <HOSTLIST> --new
--filter-tcp=443 --split-pos=1,midsld --disorder <HOSTLIST>"
```

***Enabling the standard tpws option in transparent mode***\
`TPWS_ENABLE=0`

***Which tcp ports should be forwarded to tpws***\
`TPWS_PORTS=80,443`

***tpws parameters for transparent mode***
```
TPWS_OPT="
--filter-tcp=80 --methodeol <HOSTLIST> --new
--filter-tcp=443 --split-pos=1,midsld --disorder <HOSTLIST>"
```

***Enabling the standard nfqws option***\
`NFQWS_ENABLE=0`

***Which tcp and udp ports should be forwarded to nfqws using the connbytes limiter***

connbytes allows you to redirect only a specified number of initial packets from each connection in each direction - ingress and egress.
This is a more efficient kernel-mode replacement for the nfqws `--dpi-desync-cutoff=nX` option.
```
NFQWS_PORTS_TCP=80,443
NFQWS_PORTS_UDP=443
```

***How ​​many initial incoming and outgoing packets need to be forwarded to nfqws in each direction***
```
NFQWS_TCP_PKT_OUT=$((6+$AUTOHOSTLIST_RETRANS_THRESHOLD))
NFQWS_TCP_PKT_IN=3
NFQWS_UDP_PKT_OUT=$((6+$AUTOHOSTLIST_RETRANS_THRESHOLD))
NFQWS_UDP_PKT_IN=0
```

***Set ports to redirect to nfqws without connbytes limiter***\
There is traffic for which the entire outgoing session must be redirected without restrictions.
Типичное применение - поддержка http keepalives на stateless DPI.
This puts a significant load on the processor. Use only if you understand why. Most often this is not necessary.
Incoming traffic is limited by connbytes through the PKT_IN parameters.
If you specify any ports here, it is advisable to remove them from the version with the connbytes limiter
```
NFQWS_PORTS_TCP_KEEPALIVE=80
NFQWS_PORTS_UDP_KEEPALIVE=
```

***nfqws parameters***
```
NFQWS_OPT="
--filter-tcp=80 --dpi-desync=fake,multisplit --dpi-desync-split-pos=method+2 --dpi-desync-fooling=md5sig <HOSTLIST> --new
--filter-tcp=443 --dpi-desync=fake,multidisorder --dpi-desync-split-pos=1,midsld --dpi-desync-fooling=badseq,md5sig <HOSTLIST> --new
--filter-udp=443 --dpi-desync=fake --dpi-desync-repeats=6 <HOSTLIST_NOAUTO>
```

***Host filtering mode:***
```
none - применять дурение ко всем хостам
ipset - ограничить дурение ipset-ом zapret/zapret6
hostlist - ограничить дурение списком хостов из файла
autohostlist - режим hostlist + распознавание блокировок и ведение автоматического листа
```
`MODE_FILTER=none`

***Setting up a selective traffic offload management system (only if supported)***
```
donttouch: выборочное управление отключено, используется системная настройка, простой инсталлятор выключает системную настройку, если она не совместима с выбранным режимом
none: выборочное управление отключено, простой инсталлятор выключает системную настройку
software: выборочное управление включено в режиме software, простой инсталлятор выключает системную настройку
hardware: выборочное управление включено в режиме hardware, простой инсталлятор выключает системную настройку
```
`FLOWOFFLOAD=donttouch`

The `GETLIST` parameter tells the `install_easy.sh` installer which script to run
to update the list of blocked ips or hosts.
It is also called via `get_config.sh` from scheduled jobs (crontab or systemd timer).
Place here the name of the script that you will use to update the sheets.
If not needed, the parameter should be commented out.

You can individually disable ipv4 or ipv6. If the parameter is commented out or not equal to "1",
use of the protocol is permitted.
```
DISABLE_IPV4=1
DISABLE_IPV6=1
```

Number of threads for multi-threaded DNS resolver mdig (1..100).
The more there are, the faster, but won’t your DNS server be offended by the slugging?\
`MDIG_THREADS=30`

A place to store temporary files. When downloading huge registries, `/tmp` may not have enough space.
If the file system is on normal media (not the router’s built-in memory), then you can
indicate the location on the flash drive or disk.
`TMPDIR=/opt/zapret/tmp`

***Options for creating ipsets and nfsets***

```
SET_MAXELEM=262144
IPSET_OPT="hashsize 262144 maxelem 2097152"
```

A hook that allows you to enter IP addresses dynamically. $1 = table name\
Addresses are printed to stdout. In the case of nfset, the problem of possible intersection of intervals is automatically solved.\
`IPSET_HOOK="/etc/zapret.ipset.hook"`

***About swearing in dmesg due to lack of memory.***

It may happen that there is enough memory in the system, but when trying to fill a huge `ipset`
The kernel begins to swear loudly, `ipset` is not completely filled.\
The likely cause is that the `hashsize` specified when creating `ipset` (create_ipset.sh) is exceeded.
The list is re-allocated and no continuous memory fragments of the required length are found.
This can be treated by increasing `hashsize`. But the larger the `hashsize`, the more `ipset` takes up in memory.
Setting `hashsize` too large for lists that are not large enough is not practical.

***Options for calling ip2net. Separately for ipv4 and ipv6 sheets.***

```
IP2NET_OPT4="--prefix-length=22-30 --v4-threshold=3/4"
IP2NET_OPT6="--prefix-length=56-64 --v6-threshold=5"
```

***Set autohostlist mode.***

When increasing AUTOHOSTLIST_RETRANS_THRESHOLD and using nfqws, you should reconsider the parameter values
NFQWS_TCP_PKT_OUT and NFQWS_UDP_PKT_OUT. All retransmissions must be received by nfqws, otherwise the "request stuck" trigger will not fire.

```
AUTOHOSTLIST_RETRANS_THRESHOLD=3
AUTOHOSTLIST_FAIL_THRESHOLD=3
AUTOHOSTLIST_FAIL_TIME=60
AUTOHOSTLIST_DEBUG=0
```

***Enable or disable compression of large sheets in ipset/\*.sh scripts.***

`GZIP_LISTS=1`

***Command for reloading firewall IP tables.***

If not specified or empty, ipset or ipfw is automatically selected if available.
On BSD systems with PF there is no automatic loading. There you need to specify the command explicitly: `pfctl -f /etc/pf.conf`
On newer pfctls (available in new FreeBSDs, not in OpenBSD 6.8), you can give the command to load only tables: `pfctl -Tl -f /etc/pf.conf`
"-" means disabling sheet loading even if there is a supported backend.
```
LISTS_RELOAD="pfctl -f /etc/pf.conf"
LISTS_RELOAD=-
```

OpenWrt has a default network 'lan'. Only traffic from this network will be forwarded to tpws.
But it is possible to specify other networks or a list of networks:\
`OPENWRT_LAN="lan lan2 lan3"`

In OpenWrt, interfaces with a default route are taken as wan. Separately for ipv4 and ipv6.
This can be overridden:
```
OPENWRT_WAN4="wan4 vpn"
OPENWRT_WAN6="wan6 vpn6"
```

The `INIT_APPLY_FW=1` parameter allows the init script to independently apply iptables rules.\
With other values ​​or if the parameter is commented out, the rules will not be applied.\
This is useful if you have a firewall management system, in the settings of which you should add rules.\
Not applicable on OpenWrt when using firewall3+iptables.

`FILTER_TTL_EXPIRED_ICMP=1` enables mechanisms for blocking icmp time exceeded packets, sent by routers along the packet path in response to TTL/HL exhaustion.
In Linux, the connection is terminated by the system if the following icmp is received in response to the first packet (for tcp - SYN). A similar scheme exists in datagram sockets.
icmp blocking is carried out exclusively by iptables/nftables.
To avoid touching all traffic, PRENAT mode uses connmark to mark sessions that nfqws has worked on. This cannot be done in POSTNAT mode.
therefore, all sessions wrapped on nfqws are flagged.
It is better to disable the setting if you do not expect problems from icmp, as in this case there will be less unnecessary interference in traffic.

***The following settings are not relevant for openwrt:***

If your system works as a router, then you need to enter the names of the internal and external interfaces:
```
IFACE_LAN=eth0
IFACE_WAN=eth1
IFACE_WAN6="henet ipsec0"
```
Multiple interfaces can be entered separated by a space.
If IFACE_WAN6 is not specified, then the value of IFACE_WAN is taken.

> [!IMPORTANT]
> Setting up routing, masquerade, etc. is not included in the task of ban.
> Only modes that provide interception of transit traffic are enabled.
> It is possible to define multiple interfaces like this:

`IFACE_LAN="eth0 eth1 eth2"`

## Screwing to a firewall management system or your launch system

If you are using some kind of firewall management system, then it may conflict
with the existing launch script. If the rules were applied again, it could break the iptables settings from zapret.
In this case, the rules for iptables must be attached to your firewall separately from running tpws or nfqws.

_The following calls allow you to apply or remove iptables rules individually:_

```
/opt/zapret/init.d/sysv/zapret start_fw
/opt/zapret/init.d/sysv/zapret stop_fw
/opt/zapret/init.d/sysv/zapret restart_fw
```

_And this is how you can start or stop daemons separately from the firewall:_

```
/opt/zapret/init.d/sysv/zapret start_daemons
/opt/zapret/init.d/sysv/zapret stop_daemons
/opt/zapret/init.d/sysv/zapret restart_daemons
```

`nftables` virtually eliminate conflicts between different control systems, since they allow
use independent tables and hooks. A separate nf table "zapret" is used.
If your system doesn't touch it, most likely everything will be fine.

_For `nftables` there are several additional calls:_

View sets of interfaces related to lan, wan and wan6. Traffic is being turned off along them.
And also the flow table with the names of the ingress hook interfaces.\
`/opt/zapret/init.d/sysv/zapret list_ifsets`

Update interface sets related to lan, wan and wan6.
For traditional Linux, the list of interfaces is taken from the config variables IFACE_LAN, IFACE_WAN.
For OpenWrt it is determined automatically. The lanif set can be extended with the OPENWRT_LAN parameter.
All lan and wan interfaces are also added to the ingress hook from the flow table.\
`/opt/zapret/init.d/sysv/zapret reload_ifsets`

Viewing a table without set contents. Calls `nft -t list table inet zapret`\
`/opt/zapret/init.d/sysv/zapret list_table`

_It is also possible to attach your script to any stage of applying and removing the firewall from zapret scripts:_
```
INIT_FW_PRE_UP_HOOK="/etc/firewall.zapret.hook.pre_up"
INIT_FW_POST_UP_HOOK="/etc/firewall.zapret.hook.post_up"
INIT_FW_PRE_DOWN_HOOK="/etc/firewall.zapret.hook.pre_down"
INIT_FW_POST_DOWN_HOOK="/etc/firewall.zapret.hook.post_down"
```

These settings are available in config.
May be useful if you need to use nftables sets, for example `ipban`/`ipban6`.
nfsets belong to only one table, so you will have to write rules for the zapret table,
which means you need to synchronize with the application/removal of rules by zapret scripts.

## Option custom

custom scripts are small shell programs that control non-standard modes of using zapret
or special cases that cannot be integrated into the main part without cluttering and littering the code.
To use custom, you should place the files in the following directories depending on your system:
```
/opt/zapret/init.d/sysv/custom.d
/opt/zapret/init.d/openwrt/custom.d
/opt/zapret/init.d/macos/custom.d
```
The directory will be scanned in alphabetical order and each script will be applied.

In `init.d` there is `custom.d.examples.linux`, in `init.d/macos` there is `custom.d.examples`.
These are ready-made scripts that can be copied to `custom.d`. You can use them as a basis for writing your own.

***For Linux the code is written in a function***
```
zapret_custom_daemons
zapret_custom_firewall
zapret_custom_firewall_nft
zapret_custom_firewall_nft_flush
```

***For macos***
```
zapret_custom_daemons
zapret_custom_firewall_v4
zapret_custom_firewall_v6
```

zapret_custom_daemons raises **nfqws**/**tpws** daemons in the quantity you need and with the parameters you need.
The first parameter contains the operation code: 1 = start, 0 = stop.
The scheme for launching daemons in OpenWrt is different - procd is used.
Therefore, there is no stop logic as it is unnecessary; the stop is never called.

zapret_custom_firewall raises and removes `iptables` rules.
The first parameter contains the operation code: 1 = start, 0 = stop.

zapret_custom_firewall_nft raises nftables rules.
The stop logic is absent as unnecessary. Standard zapret chains are deleted automatically.
However, sets and rules from your own chains are not removed.
They need to be cleaned up in zapret_custom_firewall_nft_flush.
If you don’t have sets or your own chains, you don’t have to define the function or leave it empty.

If you don't need iptables or nftables, you don't have to write the corresponding function.

On linux you can use the local variables `FW_EXTRA_PRE` and `FW_EXTRA_POST`.\
`FW_EXTRA_PRE` adds code to the ip/nf tables rules before the code generated by the helper functions.\
`FW_EXTRA_POST` adds code after.

In Linux, helper functions add a rule to the beginning of chains, that is, before existing ones.
Therefore, specializations should come after more general options.
For macos the rule is the opposite. There the rules are added at the end.
For the same reason, the firewall in Linux is first applied in standard mode, then custom,
and in MacOS first custom, then standard mode.

In macos firewall functions do not enter anything themselves. Their task is only to output text to stdout,
containing rules for the pf anchor. The wrapper will do the rest.

Pay special attention to the daemon number in the functions `run_daemon`, `do_daemon`, `do_tpws`, `do_tpws_socks`, `do_nfqws`,
port numbers **tpws** and queues **nfqueue**.
They must be unique in all scripts. There will be an error when overlaying.
Therefore, use functions to dynamically obtain these values ​​from the pool.

`custom` scripts can use variables from `config`. You can put your own variables in `config`
and use them in scripts.
You can use helper functions. They are part of the general shell function space.
Useful functions can be taken from example scripts. Also see `common/*.sh`.
By using a helper function, you eliminate the need to consider all possible cases
type of presence/absence of ipv6, whether the system is a router, interface names, ...Helpers take this into account. You only need to focus on the `{ip,nf}tables` filters and daemon parameters.

## Easy installation

`install_easy.sh` automates manual versions of installation procedures.
It supports OpenWrt, Linux systems based on systemd or openrc and MacOS.

For more flexible configuration, before starting the installer, you should complete the “Selecting parameters” section.

If the launch system is supported, but a package manager that is not supported by the installer is used
or the names of the packages do not match those specified in the installer, the packages must be installed manually.
curl is always required. `ipset` - only for `iptables` mode, for `nftables` - not needed.

For completely cut-off distributions (alpine), you need to separately install `iptables` and `ip6tables`, or `nftables`.

The kit includes static binaries for most architectures. One of them will do
with a probability of 99%. But if you have an exotic system, the installer will try to build the binaries itself
via make. This requires gcc, make and the necessary **-dev** packages. You can force the mode
compilation with the following call:

`install_easy.sh make`

Under OpenWrt, everything is immediately ready to use the system as a router.
The names of the WAN and LAN interfaces are known from the system settings.
Under other systems, you configure the router yourself. The installer does not interfere with this.
Depending on the selected mode, the installer may ask for LAN and WAN interfaces.
You need to understand that passing traffic is turned over to **tpws** in transparent mode before routing is performed,
Therefore, filtering is possible over the LAN but not over the WAN.
The decision to divert local outgoing traffic to **tpws** is made after routing has been completed,
Therefore, the situation is the opposite: LAN does not make sense, filtering via WAN is possible.
Reversal to **nfqws** always occurs after routing, so only WAN filtering is applicable to it.
The ability for traffic to pass in one direction or another is configured by you during the router configuration process.

Деинсталляция выполняется через `uninstall_easy.sh`. После выполнения деинсталляции можно удалить каталог `/opt/zapret`.

## Installation under systemd

If you like systemd and want to get as close to it as possible, you can abandon the zapret startup scripts
and raise `tpws` and `nfqws` instances as separate systemd units. In this case, you will have to manually write the iptables/nftables rules
and somehow raise them. For example, write an additional systemd unit for this.
You also need to build the binaries in a special way using `make systemd`.

The zapret package includes templates `init.d/systemd/{nfqws@.service,tpws@.service}`.
A short list of commands for their use is given in the comments in these files.

## The simple installation of openwrt

It only works if you have enough space on your router.

Copy zapret to the router in `/tmp`.

Run the installer:\
`sh /tmp/zapret/install_easy.sh`

Он скопирует в `/opt/zapret` только необходимый минимум файлов.

After successful installation, you can remove zapret from tmp to free up RAM:\
`rm -r /tmp/zapret`

For more flexible configuration, before starting the installer, you should complete the “Selecting parameters” section.

The simple installation system is designed for any intentional or unintentional change in access rights to files.
Resistant to repack under Windows. After copying to `/opt` the rights will be forcibly restored.


## Installation on openwrt in severely low disk space mode

Requires about 120-200 kb on disk. I'll have to give up everything except **tpws**.

**Instructions for openwrt 22 and higher with nftables**

There are no dependencies to install.

***Installation:***

1) Copy everything from `init.d/openwrt-minimal/tpws/*` to the openwrt root.
2) Copy the **tpws** binary of the appropriate architecture to `/usr/bin/tpws`.
3) Set file permissions: `chmod 755 /etc/init.d/tpws /usr/bin/tpws`
4) Edit `/etc/config/tpws`
* If ipv6 is not needed, edit `/etc/nftables.d/90-tpws.nft` and comment out the lines with the ipv6 redirect.
5) `/etc/init.d/tpws enable`
6) `/etc/init.d/tpws start`
7) `fw4 restart`

***Complete removal:***

1) `/etc/init.d/tpws disable`
2) `/etc/init.d/tpws stop`
3) `rm -f /etc/nftables.d/90-tpws.nft /etc/firewall.user /etc/init.d/tpws /usr/bin/tpws`
4) `fw4 restart`

**Instructions for openwrt 21 and lower with iptables**

***Install dependencies:***
1) `opkg update`
2) `opkg install iptables-mod-extra`
* only for IPV6: `opkg install ip6tables-mod-nat`

Make sure there is nothing significant in `/etc/firewall.user`.
If there is, do not blindly follow the instructions. Combine the code or create your own `firewall include` in `/etc/config/firewall`.

***Installation:***

1) Copy everything from `init.d/openwrt-minimal/tpws/*` to the openwrt root.
2) Copy the **tpws** binary of the appropriate architecture to `/usr/bin/tpws`.
3) Set file permissions: `chmod 755 /etc/init.d/tpws /usr/bin/tpws`
4) Edit `/etc/config/tpws`
* If you don't need ipv6, edit /etc/firewall.user and set DISABLE_IPV6=1 there.
5) `/etc/init.d/tpws enable`
6) `/etc/init.d/tpws start`
7) `fw3 restart`

***Complete removal:***

1) `/etc/init.d/tpws disable`
2) `/etc/init.d/tpws stop`
3) `rm -f /etc/nftables.d/90-tpws.nft /etc/firewall.user /etc/init.d/tpws`
4) `touch /etc/firewall.user`
5) `fw3 restart`


## Android

Without root, forget about nfqws and tpws in transparent proxy mode. tpws will only work in `--socks` mode.

Android kernels have NFQUEUE support. nfqws works.

Stock kernels do not have ipset support. In general, the complexity of the task of raising ipset varies from
“not easy” to “almost impossible.” Unless you find a ready-made assembled kernel for your device.

tpws will work anyway, it doesn't require anything special.

Although Linux versions work for Android, it is recommended to use binaries specially compiled for bionic.
They will not have problems with DNS, local time and user and group names.\
It is recommended to use gid 3003 (AID_INET). Otherwise, you may get permission denied to create a socket.
For example: `--uid 1:3003`\
In iptables, specify: `! --uid-owner 1` instead of `! --uid-owner tpws`.\
Write a shell script with iptables and tpws, run it using your root manager.
Autorun scripts are here:\
magisk : /data/adb/service.d\
supersu: /system/su.d

**nfqws** may have such a glitch. When starting with the default uid (0x7FFFFFFF) and running on a cellular interface
and the external power cable is disconnected, the system may partially hang. The touchscreen and buttons stop working,
but the animation on the screen can continue. If the screen has been turned off, it is impossible to turn it on with the power button.
Changing the UID to a low one (--uid 1 will do) solves this problem.
The glitch was noticed on Android 8.1 on a device based on the mediatek platform.

The answer to the question of where to put tpws on Android without root, so that you can then launch it from applications.
Upload the file via adb shell to /data/local/tmp/, preferably to a subfolder.
```
mkdir /data/local/tmp/zapret
adb push tpws /data/local/tmp/zapret
chmod 755 /data/local/tmp/zapret /data/local/tmp/zapret/tpws
chcon u:object_r:system_file:s0 /data/local/tmp/zapret/tpws
```
How to find a strategy to bypass a cellular operator: the easiest way is to distribute the Internet to a computer.
Any supported OS will do this. Connect Android via USB cable to your computer and enable modem mode.
Run the standard blockcheck procedure. When transferring rules to a phone, reduce TTL by 1,
if rules with TTL are present in the strategy. If tested on Windows, remove the `--wf-*` parameters.

Blockcheck is not supported in the android shell, but if you have root, you can deploy the rootfs of some Linux distribution.
This is best done from a computer via adb shell.
If there is no computer, then chrooting is the only option, although it is inconvenient.
Something lightweight will do, like alpine or even OpenWrt.
If this is not an android emulator, then the universal architecture is arm (any option).
If you know for sure that your OS is 64-bit, then it is better to use arm64 instead of arm.
You can find out the architecture with the command `uname -a`.

```
mount --bind /dev /data/linux/dev
mount --bind /proc /data/linux/proc
mount --bind /sys /data/linux/sys
chroot /data/linux
```

The first thing you will need to do is configure DNS once. It won't start on its own.

`echo nameserver 1.1.1.1 >/etc/resolv.conf`

Next, you need to install iptables-legacy using the package manager. Required **NOT** iptables-nft,
which is usually present by default. There is no nftables in the android kernel.\
`ls -la $(which iptables)`\
The link must point to the legacy option.
If not, then install the necessary packages for your distribution, and make sure the links are correct.\
`iptables -S`\
This way you can check that your `iptables` saw what android put there. `iptables-nft` will throw an error.
Далее качаем zapret в `/opt/zapret`. Обычные действия с `install_prereq.sh`, `install_bin.sh`, `blockcheck.sh`.

Please note that the strategies for bypassing a cellular operator and home wifi will likely be different.
It is easy to select a cellular operator using the iptables parameter `-o <interface name>`. The name could be, for example, `ccmni0`.
It's easy to see via `ifconfig`.
Wifi network - usually `wlan0`.

You can switch blockcheck between the operator and wifi along with the entire Internet - by turning wifi on or off.
If you find a strategy for wifi and enter it into autostart, then when connecting to another wifi
it may not work or even break something, so think about whether it’s worth it.
It might be better to make scripts like “start home wifi bypass”, “remove home wifi bypass”,
and use them as needed from the terminal.
But it’s better to bypass home wifi on a router.


## Huawei mobile modems and routers

Devices such as E3372, E8372, E5770 share a common system ideology.
There are 2 computing cores. One kernel runs vxworks, the other runs linux.
4pda has modified firmware with telnet and adb. They should be used.

Further statements are tested on E8372. It may be similar or similar to others.
There are additional hardware units for offload network functions.
Not all traffic goes through Linux. Outgoing traffic from the modem itself passes
the OUTPUT chain is normal, on FORWARD =>wan some packets fall out of tcpdump.

tpws works as usual.

`nfqueue` поломан, можно собрать фиксящий модуль https://github.com/im-0/unfuck-nfqueue-on-e3372h,
using sources from Huawei open source. The sources contain the toolchain and semi-assembled,
irrelevant kernel. The config can be taken from the working modem from `/proc/config.gz`.
Using these sources, craftsmen can build the `unfuck_nfqueue.ko` module.
After using it, NFQUEUE and nfqws for arm work fine.

To avoid the problem with offload when using nfqws, you should combine tpws in tcp proxy and nfqws mode.
NFQUEUE rules are written for the OUTPUT chain.
connbytes will have to be omitted since there is no module in the kernel. But it's not fatal.

Autorun script - `/system/etc/autorun.sh`. Create your own zapret configuration script,
run from the end of autorun.sh via "&". The script should do sleep 5 at the beginning to wait
raising the network and iptables from Huawei.

> [!WARNING]
> This modem is experiencing chaotic resets of tcp connections for unknown reasons.
> It looks like this if you run curl from the modem itself:
```
curl www.ru
curl: (7) Failed to connect to www.ru port 80: Host is unreachable
```
A socket error EHOSTUNREACH (errno -113) occurs. The same can be seen in tpws.
The browser does not load parts of web pages, images, styles.
In tcpdump on the external interface eth_x, only a single and unresponsive SYN packet is visible, without ICMP messages.
The OS somehow finds out that it is impossible to establish a TCP connection and throws an error.
If you connect from a client, the SYNs disappear and the connection is not established.
The client OS performs retransmission, and after some time the connection is successful.
Therefore, without tcp proxying in this situation, sites are slow, but they load, but with proxying
the connection is made, but soon resets without any data, and browsers do not attempt to establish
him again. Therefore, the quality of browsing with tpws may be worse, but the problem is not with tpws.
The frequency of resets increases noticeably if the torrent client is running and there are many TCP connections.
However, the reason is not that the conntrack table is full. Increasing limits and clearing conntrack does not help.
Presumably this feature is related to the processing of connection reset packets in hardware offload.
I don't have an exact answer to the question. If you know, please share.
In order not to degrade the quality of browsing, you can filter the turn on tpws using an IP filter.
There is no ipset support. This means that all you can do is create individual rules
for a small number of hosts.

Some draft scripts are present in [files/huawei](./../files/huawei/). _Not a ready-made solution!_ Look, study, adapt.\
Здесь можно скачать готовые полезные статические бинарники для arm, включая curl : https://github.com/bol-van/bins


## FreeBSD, OpenBSD, MacOS

Described in [BSD documentation](./bsd.md)

## Windows

Described in [Windows documentation](./windows.md)


## Other firmware

For static binaries, it doesn’t matter what they are running on: PC, Android, set-top box, router, any other device.
Any firmware or Linux distribution will do. Static binaries will run on everything.
They only need a kernel with the necessary build options or modules.
But in addition to binaries, the project also uses scripts that involve some
standard programs.

The main reasons why you can’t just go and install this system on anything:
* lack of access to the device via shell
* no root
* lack of r/w partition for recording and non-volatile storage of files
* inability to set something to autorun
* no cron
* non-disabled flow offload or other proprietary stuff in netfilter
* lack of kernel modules or kernel build options
* lack of iptables modules (/usr/lib/iptables/lib*.so)
* lack of standard programs (such as ipset, curl) or their castration (easy replacement)
* castrated or custom shell sh

If your firmware has everything you need, then you can adapt zapret to your device to one degree or another.
You may not be able to lift all parts of the system, but you can at least try
raise tpws and send all traffic to port 80 to it via -j REDIRECT.
If you have somewhere to write tpws, it is possible to execute commands at startup, then at least
you can do this. Most likely there is support for REDIRECT in the kernel. It is definitely available on any router,
on other devices it is questionable. NFQUEUE, ipset are missing on most firmwares due to their uselessness.

Rebuilding the kernel or modules for it will most likely be quite difficult.
To do this, you will need to at least obtain the sources of your firmware.
User mode components can be introduced relatively painlessly if there is a place to write them down.
Especially for devices with an r/w area, there is an entware project.
Some firmware even have the ability to easily install it via a web interface.
entware contains a repository of user-mode components that are installed in /opt.
With their help, you can compensate for the lack of software in the main firmware, with the exception of the kernel.

You can try to use the sysv init script in the same way as described in the section
"Screwing it to the firewall control system or your launch system."
If you complain about the lack of some basic programs, they should be replenished through entware.
Before running the script, the path to additional programs must be placed in PATH.

_A detailed description of settings for other firmware is beyond the scope of this project._

OpenWrt is one of the few relatively full-fledged Linux systems for embedded devices.
It is characterized by the following things, which served as the basis for choosing this particular firmware:
* full root access to the device via shell. On factory firmware it is most often absent, on many alternative ones there is
* root r/w. this is almost a unique feature of OpenWrt. factory and most alternative firmware
built on top of squashfs root (r/o), and the configuration is stored in a specially formatted area
built-in memory called nvram.  systems without r/w roots are strongly castrated. they don't have
Possibility of additional installation of software from the repository without any special tricks and is mainly sharpened
for a slightly more advanced user than usual and control of existing functionality via a web interface,
but the functionality is fixedly limited. alternative firmwares can usually mount r/w partition
in some area of ​​the file system, factory ones can usually mount only flash drives connected to USB,
and it’s not a fact that there is support for unix file systems. May only support fat and ntfs.
* the ability to move the root file system to external media (extroot) or create an overlay on it (overlay)
* availability of opkg package manager and software repository
* flow offload is predictable, standard and selectively controlled, as well as disabled
* The repository contains all the kernel modules; they can be additionally installed via opkg. There is no need to rebuild the kernel.
* The repository contains all iptables modules, they can be additionally installed via opkg
* the repository has a huge number of standard programs and additional software
* the presence of an SDK that allows you to collect the missing


## Bypassing blocking through a third-party host

If offline bypass does not work, you have to redirect traffic through a third-party host.
It is proposed to use a transparent redirect via socks5 using `iptables+redsocks`, or `iptables+iproute+vpn`.
Setting up the redsocks version of OpenWrt described in [redsocks.txt](./redsocks.txt).
Setting the variant with `iproute+wireguard` - in [wireguard_iproute_openwrt.txt](./wireguard_iproute_openwrt.txt).


## Why is it worth investing in a VPS?

VPS is a virtual server. There are a huge number of data centers offering this service.
Any task can be performed on a VPS. From a simple website to a sophisticated proprietary system.
You can also use VPS to create your own VPN or proxy.
The sheer breadth of possible uses and the prevalence of the service minimize the possibilities
regulators to ban services of this type. Yes, if white lists are introduced, the solution will be abandoned, but it will be a different story
a reality in which other solutions will have to be invented.
Until this is done, no one will ban hosting companies simply because they provide hosting services.
You, as an individual, most likely are not needed by anyone. Think about how you differ from a well-known VPN provider.
The VPN provider provides a _simple_ and _affordable_ block bypass service for the masses.
This fact makes it a prime target for blocking. RKN will send a notification after refusal to cooperate
will block the VPN. The prepaid amount will be lost.
Regulators do not and never will have the resources to conduct a complete inspection of every server on the network.
A Chinese scenario is possible, in which DPI detects VPN protocols and dynamically bans IP servers,
providing unlicensed VPN. But with knowledge and head, you can always obfuscate
VPN traffic or use other types of VPN that are more resistant to DPI analysis, or simply less well-known,
and therefore less likely to be detected by the regulator.
You have the freedom to do whatever you want on your VPS, adapting to new conditions.
Yes, this will require knowledge. You choose to learn and keep the situation under control when nothing is forbidden to you
cannot, or submit to the system.

VPS can be purchased from many places. There are portals specialized in searching for VPS offers.\
Например, [вот этот](https://vps.today).
For a personal VPN server, the most minimal configuration is usually sufficient, but with unlimited traffic or
with a large traffic limit (terabytes). The type of VPS is also important. OpenVZ is suitable for OpenVPN, but
you won’t be able to install WireGuard or IPsec on it, that is, everything that requires kernel mode.
Kernel mode requires a type of virtualization that involves running a full-fledged instance of the Linux OS
along with the core. KVM, Xen, Hyper-V, VMware are suitable.

In terms of price, you can find offers that will be cheaper than a ready-made VPN service, but at the same time you are the boss of your own shop
and you don’t risk getting banned by the regulator, unless “at the same time” - carpet bombing with a ban on millions of IPs.
In addition, if everything seems completely complicated to you, what you read causes stupor and you know for sure that nothing
If you can’t do anything described above, you can at least use dynamic SSH port forwarding
to receive an encrypted SOCKS proxy and register it in the browser. Linux knowledge is not needed at all.
This option is the least stressful for dummies, although not the most convenient to use.

## Support the developer

USDT `0x3d52Ce15B7Be734c53fc9526ECbAB8267b63d66E`

BTC  `bc1qhqew3mrvp47uk2vevt5sctp7p2x9m7m5kkchve`

ETH  `0x3d52Ce15B7Be734c53fc9526ECbAB8267b63d66E`

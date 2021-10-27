Предварительно был сгенерирован статический ключ OpenVPN сервера
```
openvpn --genkey --secret static.key
```
## 1. Между двумя виртуалками поднять vpn в режимах
- tun
- tap

### 1.1 tun

#### 1.1.1 server конфиг
<pre>
dev tun
ifconfig 10.0.0.1 10.0.0.2
secret static.key
</pre>
<details>
<summary>server статус</summary>

    [vagrant@tunServer ~]$ systemctl status openvpn-server@server.service
    ● openvpn-server@server.service - OpenVPN service for server
      Loaded: loaded (/usr/lib/systemd/system/openvpn-server@.service; enabled; vendor preset: disabled)
      Active: active (running) since Wed 2021-10-20 14:12:13 UTC; 20h ago
        Docs: man:openvpn(8)
              https://community.openvpn.net/openvpn/wiki/Openvpn24ManPage
              https://community.openvpn.net/openvpn/wiki/HOWTO
    Main PID: 4082 (openvpn)
      Status: "Initialization Sequence Completed"
      CGroup: /system.slice/system-openvpn\x2dserver.slice/openvpn-server@server.service
              └─4082 /usr/sbin/openvpn --status /run/openvpn-server/status-server.log --status-version 2 --suppress-timestamps --config server.conf
    [vagrant@tunServer ~]$ ip a
    1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
        link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
        inet 127.0.0.1/8 scope host lo
          valid_lft forever preferred_lft forever
        inet6 ::1/128 scope host
          valid_lft forever preferred_lft forever
    2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
        link/ether 52:54:00:4d:77:d3 brd ff:ff:ff:ff:ff:ff
        inet 10.0.2.15/24 brd 10.0.2.255 scope global noprefixroute dynamic eth0
          valid_lft 83610sec preferred_lft 83610sec
        inet6 fe80::5054:ff:fe4d:77d3/64 scope link
          valid_lft forever preferred_lft forever
    3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
        link/ether 08:00:27:74:12:46 brd ff:ff:ff:ff:ff:ff
        inet 192.168.10.1/30 brd 192.168.10.3 scope global noprefixroute eth1
          valid_lft forever preferred_lft forever
        inet6 fe80::a00:27ff:fe74:1246/64 scope link
          valid_lft forever preferred_lft forever
    4: tun0: <POINTOPOINT,MULTICAST,NOARP,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UNKNOWN group default qlen 100
        link/none
        inet 10.0.0.1 peer 10.0.0.2/32 scope global tun0
          valid_lft forever preferred_lft forever
        inet6 fe80::a413:7d2:70be:e1c5/64 scope link flags 800
          valid_lft forever preferred_lft forever
</details>

#### 1.1.2 client конфиг
<pre>
dev tun
ifconfig 10.0.0.1 10.0.0.2
secret static.key
</pre>

<details>
<summary>client статус</summary>

    [vagrant@tunClient ~]$ systemctl status openvpn-client@client.service
    ● openvpn-client@client.service - OpenVPN tunnel for client
      Loaded: loaded (/usr/lib/systemd/system/openvpn-client@.service; enabled; vendor preset: disabled)
      Active: active (running) since Wed 2021-10-20 14:15:02 UTC; 20h ago
        Docs: man:openvpn(8)
              https://community.openvpn.net/openvpn/wiki/Openvpn24ManPage
              https://community.openvpn.net/openvpn/wiki/HOWTO
    Main PID: 4433 (openvpn)
      Status: "Initialization Sequence Completed"
      CGroup: /system.slice/system-openvpn\x2dclient.slice/openvpn-client@client.service
              └─4433 /usr/sbin/openvpn --suppress-timestamps --nobind --config client.conf
    [vagrant@tunClient ~]$ ip a
    1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
        link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
        inet 127.0.0.1/8 scope host lo
          valid_lft forever preferred_lft forever
        inet6 ::1/128 scope host
          valid_lft forever preferred_lft forever
    2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
        link/ether 52:54:00:4d:77:d3 brd ff:ff:ff:ff:ff:ff
        inet 10.0.2.15/24 brd 10.0.2.255 scope global noprefixroute dynamic eth0
          valid_lft 84758sec preferred_lft 84758sec
        inet6 fe80::5054:ff:fe4d:77d3/64 scope link
          valid_lft forever preferred_lft forever
    3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
        link/ether 08:00:27:f2:66:f2 brd ff:ff:ff:ff:ff:ff
        inet 192.168.10.2/30 brd 192.168.10.3 scope global noprefixroute eth1
          valid_lft forever preferred_lft forever
        inet6 fe80::a00:27ff:fef2:66f2/64 scope link
          valid_lft forever preferred_lft forever
    4: tun0: <POINTOPOINT,MULTICAST,NOARP,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UNKNOWN group default qlen 100
        link/none
        inet 10.0.0.2 peer 10.0.0.1/32 scope global tun0
          valid_lft forever preferred_lft forever
        inet6 fe80::6d6b:6283:efb7:4233/64 scope link flags 800
          valid_lft forever preferred_lft forever
</details>

### 1.2 tap
#### 1.2.1 server конфиг
<pre>
dev tap
ifconfig 10.0.0.1 255.255.255.0
topology subnet
secret static.key
status /var/run/openvpn-status.log
log /var/log/openvpn.log
verb 3
</pre>

<details>
<summary>server статус</summary>

    [root@tapServer ~]# systemctl status openvpn-server@server.service
    ● openvpn-server@server.service - OpenVPN service for server
      Loaded: loaded (/usr/lib/systemd/system/openvpn-server@.service; enabled; vendor preset: disabled)
      Active: active (running) since Wed 2021-10-20 14:24:07 UTC; 25min ago
        Docs: man:openvpn(8)
              https://community.openvpn.net/openvpn/wiki/Openvpn24ManPage
              https://community.openvpn.net/openvpn/wiki/HOWTO
    Main PID: 4765 (openvpn)
      Status: "Initialization Sequence Completed"
      CGroup: /system.slice/system-openvpn\x2dserver.slice/openvpn-server@server.service
              └─4765 /usr/sbin/openvpn --status /run/openvpn-server/status-server.log --status-version 2 --suppress-timestamps --config server.conf

    Oct 20 14:24:07 tapServer systemd[1]: Stopped OpenVPN service for server.
    Oct 20 14:24:07 tapServer systemd[1]: Starting OpenVPN service for server...
    Oct 20 14:24:07 tapServer systemd[1]: Started OpenVPN service for server.
    [root@tapServer ~]# ip a
    1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
        link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
        inet 127.0.0.1/8 scope host lo
          valid_lft forever preferred_lft forever
        inet6 ::1/128 scope host
          valid_lft forever preferred_lft forever
    2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
        link/ether 52:54:00:4d:77:d3 brd ff:ff:ff:ff:ff:ff
        inet 10.0.2.15/24 brd 10.0.2.255 scope global noprefixroute dynamic eth0
          valid_lft 84728sec preferred_lft 84728sec
        inet6 fe80::5054:ff:fe4d:77d3/64 scope link
          valid_lft forever preferred_lft forever
    3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
        link/ether 08:00:27:a5:2b:58 brd ff:ff:ff:ff:ff:ff
        inet 192.168.10.1/30 brd 192.168.10.3 scope global noprefixroute eth1
          valid_lft forever preferred_lft forever
        inet6 fe80::a00:27ff:fea5:2b58/64 scope link
          valid_lft forever preferred_lft forever
    5: tap0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UNKNOWN group default qlen 100
        link/ether a6:63:96:88:53:9e brd ff:ff:ff:ff:ff:ff
        inet 10.0.0.1/24 brd 10.0.0.255 scope global tap0
          valid_lft forever preferred_lft forever
        inet6 fe80::a463:96ff:fe88:539e/64 scope link
          valid_lft forever preferred_lft forever
</details>

#### 1.2.2. client конфиг
<pre>
dev tap
remote 192.168.10.1
ifconfig 10.0.0.2 255.255.255.0
topology subnet
secret static.key
status /var/run/openvpn-status.log
log /var/log/openvpn.log
verb 3
</pre>

<details>
<summary>tap client статус</summary>

    [vagrant@tapClient ~]$ systemctl status openvpn-client@client.service
    ● openvpn-client@client.service - OpenVPN tunnel for client
      Loaded: loaded (/usr/lib/systemd/system/openvpn-client@.service; enabled; vendor preset: disabled)
      Active: active (running) since Wed 2021-10-20 14:24:15 UTC; 20h ago
        Docs: man:openvpn(8)
              https://community.openvpn.net/openvpn/wiki/Openvpn24ManPage
              https://community.openvpn.net/openvpn/wiki/HOWTO
    Main PID: 4765 (openvpn)
      Status: "Initialization Sequence Completed"
      CGroup: /system.slice/system-openvpn\x2dclient.slice/openvpn-client@client.service
              └─4765 /usr/sbin/openvpn --suppress-timestamps --nobind --config client.conf
    [vagrant@tapClient ~]$ ip a
    1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
        link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
        inet 127.0.0.1/8 scope host lo
          valid_lft forever preferred_lft forever
        inet6 ::1/128 scope host
          valid_lft forever preferred_lft forever
    2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
        link/ether 52:54:00:4d:77:d3 brd ff:ff:ff:ff:ff:ff
        inet 10.0.2.15/24 brd 10.0.2.255 scope global noprefixroute dynamic eth0
          valid_lft 55819sec preferred_lft 55819sec
        inet6 fe80::5054:ff:fe4d:77d3/64 scope link
          valid_lft forever preferred_lft forever
    3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
        link/ether 08:00:27:36:6d:d0 brd ff:ff:ff:ff:ff:ff
        inet 192.168.10.2/30 brd 192.168.10.3 scope global noprefixroute eth1
          valid_lft forever preferred_lft forever
        inet6 fe80::a00:27ff:fe36:6dd0/64 scope link
          valid_lft forever preferred_lft forever
    5: tap0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UNKNOWN group default qlen 100
        link/ether b6:6b:fb:64:ae:01 brd ff:ff:ff:ff:ff:ff
        inet 10.0.0.2/24 brd 10.0.0.255 scope global tap0
          valid_lft forever preferred_lft forever
        inet6 fe80::b46b:fbff:fe64:ae01/64 scope link
          valid_lft forever preferred_lft forever
</details>


### 1.3 Прочуствовать разницу.

Виртуальные интерфейсы ``tun`` и ``tap`` работают на разных уровнях: tun - L3, tap - L2. У пакета tap туннеля оверхед больше, чем у tun, а значит скорость с tun интерфейсом должна быть выше. Проверить это можно с помощью утилиты ``iperf``. Тест запускался в 3 потомка на 60 секунд: ``iperf -c 10.0.0.1 -P 3 -t 60``

<details>
<summary>tun интерфейс</summary>

    [vagrant@tunClient ~]$ iperf -c 10.0.0.1 -P 3 -t 60
    ------------------------------------------------------------
    Client connecting to 10.0.0.1, TCP port 5001
    TCP window size: 85.5 KByte (default)
    ------------------------------------------------------------
    [  5] local 10.0.0.2 port 51002 connected with 10.0.0.1 port 5001
    [  4] local 10.0.0.2 port 51000 connected with 10.0.0.1 port 5001
    [  3] local 10.0.0.2 port 50998 connected with 10.0.0.1 port 5001
    [ ID] Interval       Transfer     Bandwidth
    [  4]  0.0-60.0 sec   367 MBytes  51.4 Mbits/sec
    [  3]  0.0-60.0 sec   370 MBytes  51.7 Mbits/sec
    [  5]  0.0-60.1 sec   331 MBytes  46.2 Mbits/sec
    [SUM]  0.0-60.1 sec  1.04 GBytes   149 Mbits/sec
</details>

<details>
<summary>tap интерфейс</summary>

    [vagrant@tapClient ~]$ iperf -c 10.0.0.1 -P 3 -t 60
    ------------------------------------------------------------
    Client connecting to 10.0.0.1, TCP port 5001
    TCP window size: 85.5 KByte (default)
    ------------------------------------------------------------
    [  5] local 10.0.0.2 port 51002 connected with 10.0.0.1 port 5001
    [  4] local 10.0.0.2 port 51000 connected with 10.0.0.1 port 5001
    [  3] local 10.0.0.2 port 50998 connected with 10.0.0.1 port 5001
    [ ID] Interval       Transfer     Bandwidth
    [  4]  0.0-60.2 sec   310 MBytes  43.1 Mbits/sec
    [  5]  0.0-60.3 sec   344 MBytes  47.9 Mbits/sec
    [  6]  0.0-60.3 sec   362 MBytes  50.3 Mbits/sec
    [SUM]  0.0-60.3 sec  1016 MBytes   141 Mbits/sec
</details>

tun, как и ожидалось, оказался немного быстрее.

## 2. Поднять RAS на базе OpenVPN с клиентскими сертификатами, подключиться с локальной машины на виртуалку.

Настройка сервера проводилась +- по [статье](https://habr.com/ru/post/233971/) на хабре:

* инициализируем хранилище ключей easyrsa init-pki
* создаем корневой сертификат easyrsa build-ca nopass
* генерируем ключ Диффи-Хелмана easyrsa gen-dh
* создаем сертификат и ключ сервера easyrsa build-server-full server nopass
* создаем сертификат и ключ клиента easyrsa build-client-full client nopass

```
- name: Create openvpn server keys
  shell: |
    easyrsa init-pki
    easyrsa build-ca nopass
    easyrsa gen-dh
    easyrsa build-server-full server nopass
    easyrsa build-client-full client nopass
  args:
    chdir: /etc/openvpn/server
```

Кофиги сервера и клиента генерируются Ansible из шаблонов.

Конфиг клиента скачивается на хост машину в папку provision.

```
- name: Fetch client.conf
  fetch:
    become: yes
    src: /vagrant/client.conf
    dest: "{{ playbook_dir }}/client.conf"
    flat: yes
```

Проверяем статус сервера после завершения provision:
<details>
<summary>Service status</summary>

    [root@rasServer ~]# systemctl status openvpn-server@server.service
    ● openvpn-server@server.service - OpenVPN service for server
      Loaded: loaded (/usr/lib/systemd/system/openvpn-server@.service; enabled; vendor preset: disabled)
      Active: active (running) since Wed 2021-10-27 09:47:29 UTC; 2h 34min ago
        Docs: man:openvpn(8)
              https://community.openvpn.net/openvpn/wiki/Openvpn24ManPage
              https://community.openvpn.net/openvpn/wiki/HOWTO
    Main PID: 6722 (openvpn)
      Status: "Initialization Sequence Completed"
      CGroup: /system.slice/system-openvpn\x2dserver.slice/openvpn-server@server.service
              └─6722 /usr/sbin/openvpn --status /run/openvpn-server/status-server.log --status-version 2 --suppress-timestamps --config server.conf

    Oct 27 09:47:29 rasServer systemd[1]: Stopped OpenVPN service for server.
    Oct 27 09:47:29 rasServer systemd[1]: Starting OpenVPN service for server...
    Oct 27 09:47:29 rasServer systemd[1]: Started OpenVPN service for server.
</details>

<details>
<summary>ip a</summary>

    [root@rasServer ~]# ip a
    1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
        link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
        inet 127.0.0.1/8 scope host lo
          valid_lft forever preferred_lft forever
        inet6 ::1/128 scope host
          valid_lft forever preferred_lft forever
    2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
        link/ether 52:54:00:4d:77:d3 brd ff:ff:ff:ff:ff:ff
        inet 10.0.2.15/24 brd 10.0.2.255 scope global noprefixroute dynamic eth0
          valid_lft 76729sec preferred_lft 76729sec
        inet6 fe80::5054:ff:fe4d:77d3/64 scope link
          valid_lft forever preferred_lft forever
    3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
        link/ether 08:00:27:76:49:41 brd ff:ff:ff:ff:ff:ff
        inet 10.0.1.35/24 brd 10.0.1.255 scope global noprefixroute dynamic eth1
          valid_lft 554sec preferred_lft 554sec
        inet6 fe80::a00:27ff:fe76:4941/64 scope link
          valid_lft forever preferred_lft forever
    4: eth2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
        link/ether 08:00:27:c6:ba:8e brd ff:ff:ff:ff:ff:ff
        inet 192.168.10.3/27 brd 192.168.10.31 scope global noprefixroute eth2
          valid_lft forever preferred_lft forever
        inet6 fe80::a00:27ff:fec6:ba8e/64 scope link
          valid_lft forever preferred_lft forever
    6: tun0: <POINTOPOINT,MULTICAST,NOARP,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UNKNOWN group default qlen 100
        link/none
        inet 192.168.20.1/27 brd 192.168.20.31 scope global tun0
          valid_lft forever preferred_lft forever
        inet6 fe80::cba9:6e08:e19d:31aa/64 scope link flags 800
          valid_lft forever preferred_lft forever
</details>

Пробуем подключиться к серверу с хост машины

<details>
<summary>openvpn --config /etc/openvpn/client/client.conf</summary>

    [root@kepler ~]# openvpn --config /etc/openvpn/client/client.conf
    Wed Oct 27 16:27:43 2021 Warning: Error redirecting stdout/stderr to --log file: /var/log/openvpn/openvpn.log: No such file or directory (errno=2)
    Wed Oct 27 16:27:43 2021 OpenVPN 2.4.11 x86_64-redhat-linux-gnu [SSL (OpenSSL)] [LZO] [LZ4] [EPOLL] [PKCS11] [MH/PKTINFO] [AEAD] built on Apr 21 2021
    Wed Oct 27 16:27:43 2021 library versions: OpenSSL 1.1.1g FIPS  21 Apr 2020, LZO 2.08
    Wed Oct 27 16:27:43 2021 WARNING: No server certificate verification method has been enabled.  See http://openvpn.net/howto.html#mitm for more info.
    Wed Oct 27 16:27:43 2021 Outgoing Control Channel Authentication: Using 160 bit message hash 'SHA1' for HMAC authentication
    Wed Oct 27 16:27:43 2021 Incoming Control Channel Authentication: Using 160 bit message hash 'SHA1' for HMAC authentication
    Wed Oct 27 16:27:43 2021 TCP/UDP: Preserving recently used remote address: [AF_INET]10.0.1.35:1194
    Wed Oct 27 16:27:43 2021 Socket Buffers: R=[212992->212992] S=[212992->212992]
    Wed Oct 27 16:27:43 2021 UDP link local (bound): [AF_INET][undef]:1194
    Wed Oct 27 16:27:43 2021 UDP link remote: [AF_INET]10.0.1.35:1194
    Wed Oct 27 16:27:43 2021 NOTE: UID/GID downgrade will be delayed because of --client, --pull, or --up-delay
    Wed Oct 27 16:27:43 2021 TLS: Initial packet from [AF_INET]10.0.1.35:1194, sid=617516e9 3b10d9b3
    Wed Oct 27 16:27:43 2021 VERIFY OK: depth=1, CN=OHW
    Wed Oct 27 16:27:43 2021 VERIFY OK: depth=0, CN=server
    Wed Oct 27 16:27:43 2021 Control Channel: TLSv1.2, cipher TLSv1.2 ECDHE-RSA-AES256-GCM-SHA384, 2048 bit RSA
    Wed Oct 27 16:27:43 2021 [server] Peer Connection Initiated with [AF_INET]10.0.1.35:1194
    Wed Oct 27 16:27:45 2021 SENT CONTROL [server]: 'PUSH_REQUEST' (status=1)
    Wed Oct 27 16:27:45 2021 PUSH: Received control message: 'PUSH_REPLY,topology subnet,route-gateway 192.168.20.1,route 192.168.10.0 255.255.255.224 192.168.20.1,ping 10,ping-restart 120,ifconfig 192.168.20.10 255.255.255.224,peer-id 0,cipher AES-256-GCM'
    Wed Oct 27 16:27:45 2021 OPTIONS IMPORT: timers and/or timeouts modified
    Wed Oct 27 16:27:45 2021 OPTIONS IMPORT: --ifconfig/up options modified
    Wed Oct 27 16:27:45 2021 OPTIONS IMPORT: route options modified
    Wed Oct 27 16:27:45 2021 OPTIONS IMPORT: route-related options modified
    Wed Oct 27 16:27:45 2021 OPTIONS IMPORT: peer-id set
    Wed Oct 27 16:27:45 2021 OPTIONS IMPORT: adjusting link_mtu to 1625
    Wed Oct 27 16:27:45 2021 OPTIONS IMPORT: data channel crypto options modified
    Wed Oct 27 16:27:45 2021 Data Channel: using negotiated cipher 'AES-256-GCM'
    Wed Oct 27 16:27:45 2021 Outgoing Data Channel: Cipher 'AES-256-GCM' initialized with 256 bit key
    Wed Oct 27 16:27:45 2021 Incoming Data Channel: Cipher 'AES-256-GCM' initialized with 256 bit key
    Wed Oct 27 16:27:45 2021 ROUTE_GATEWAY 10.0.1.1/255.255.255.0 IFACE=enp4s0 HWADDR=00:e0:4c:ed:65:9b
    Wed Oct 27 16:27:45 2021 TUN/TAP device tun0 opened
    Wed Oct 27 16:27:45 2021 TUN/TAP TX queue length set to 100
    Wed Oct 27 16:27:45 2021 /sbin/ip link set dev tun0 up mtu 1500
    Wed Oct 27 16:27:45 2021 /sbin/ip addr add dev tun0 192.168.20.10/27 broadcast 192.168.20.31
    Wed Oct 27 16:27:45 2021 /sbin/ip route add 192.168.10.0/27 via 192.168.20.1
    Wed Oct 27 16:27:45 2021 GID set to nobody
    Wed Oct 27 16:27:45 2021 UID set to nobody
    Wed Oct 27 16:27:45 2021 WARNING: this configuration may cache passwords in memory -- use the auth-nocache option to prevent this
    Wed Oct 27 16:27:45 2021 Initialization Sequence Completed
</details>

<details>
<summary>ip a</summary>

    [root@kepler ~]# ip a
    1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
        link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
        inet 127.0.0.1/8 scope host lo
          valid_lft forever preferred_lft forever
        inet6 ::1/128 scope host
          valid_lft forever preferred_lft forever
    2: enp4s0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
        link/ether 00:e0:4c:ed:65:9b brd ff:ff:ff:ff:ff:ff
        inet 10.0.1.18/24 brd 10.0.1.255 scope global noprefixroute enp4s0
          valid_lft forever preferred_lft forever
        inet 10.0.1.19/24 brd 10.0.1.255 scope global secondary noprefixroute enp4s0:0
          valid_lft forever preferred_lft forever
        inet 10.0.1.20/24 brd 10.0.1.255 scope global secondary noprefixroute enp4s0:1
          valid_lft forever preferred_lft forever
        inet6 fe80::2e0:4cff:feed:659b/64 scope link
          valid_lft forever preferred_lft forever
    3: vboxnet0: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN group default qlen 1000
        link/ether 0a:00:27:00:00:00 brd ff:ff:ff:ff:ff:ff
    4: vboxnet3: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN group default qlen 1000
        link/ether 0a:00:27:00:00:03 brd ff:ff:ff:ff:ff:ff
    5: vboxnet4: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc fq_codel state DOWN group default qlen 1000
        link/ether 0a:00:27:00:00:04 brd ff:ff:ff:ff:ff:ff
        inet 192.168.99.1/24 brd 192.168.99.255 scope global vboxnet4
          valid_lft forever preferred_lft forever
        inet6 fe80::800:27ff:fe00:4/64 scope link
          valid_lft forever preferred_lft forever
    6: vboxnet1: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN group default qlen 1000
        link/ether 0a:00:27:00:00:01 brd ff:ff:ff:ff:ff:ff
    7: vboxnet2: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN group default qlen 1000
        link/ether 0a:00:27:00:00:02 brd ff:ff:ff:ff:ff:ff
    8: vboxnet5: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc fq_codel state DOWN group default qlen 1000
        link/ether 0a:00:27:00:00:05 brd ff:ff:ff:ff:ff:ff
        inet 172.28.128.1/24 brd 172.28.128.255 scope global vboxnet5
          valid_lft forever preferred_lft forever
        inet6 fe80::800:27ff:fe00:5/64 scope link
          valid_lft forever preferred_lft forever
    13: tun0: <POINTOPOINT,MULTICAST,NOARP,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UNKNOWN group default qlen 100
        link/none
        inet 192.168.20.10/27 brd 192.168.20.31 scope global tun0
          valid_lft forever preferred_lft forever
        inet6 fe80::24c4:283f:7446:2f9b/64 scope link stable-privacy
          valid_lft forever preferred_lft forever
</details>

Пробуем пинговать сервер
<details>
<summary>ping</summary>

    [root@kepler ~]# ping 192.168.10.3
    PING 192.168.10.3 (192.168.10.3) 56(84) bytes of data.
    64 bytes from 192.168.10.3: icmp_seq=1 ttl=64 time=0.829 ms
    64 bytes from 192.168.10.3: icmp_seq=2 ttl=64 time=0.867 ms
    64 bytes from 192.168.10.3: icmp_seq=3 ttl=64 time=0.882 ms
    ^C
    --- 192.168.10.3 ping statistics ---
    3 packets transmitted, 3 received, 0% packet loss, time 2026ms
    rtt min/avg/max/mdev = 0.829/0.859/0.882/0.032 ms
</details>

Все работает ;)
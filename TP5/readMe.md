###TP5 - Premier pas dans le monde Cisco
##I. Préparation du lab
1. Préparation VMs

Fait

2. Préparation Routeurs Cisco
On télécharge l'image disque Cisco
Le réseau utilisé est 10.5.3.0/30,les IP seront 10.5.12.1 et 10.5.12.2 pour les deux routeurs.
Récapitulation des IPs:
Machines
net1
net2
net3
client1.tp5.b1
X
10.5.2.10
X
client2.tp5.b1
X
10.5.2.11
X
router1.tp5.b1
10.5.1.254
X
10.5.12.1
router2.tp5.b1
X
10.5.2.254
10.5.12.2
server1.tp5.b1
10.5.1.10
X
X
##II. Lancement et configuration du lab
1. Checklist IP VMs
Désactivation de SELinux, la carte NAT déjà désactivée dans le patron.
Puis on défini ip, domaine et ssh.

2. Checklist IP Routeurs
Depuis GNS3(Terminal)
Pour le router1:
`# ip address 10.5.1.254 255.255.255.0`
`# hostname router1.tp5.b1`

3. Checklist routes
 router1:
 `# ip route 10.5.2.0 255.255.255.0 10.5.12.2`
 server1:
Routes
```
[root@server1 ~]# nano /etc/sysconfig/network-scripts/route-eth0

10.2.0.0/24 via 10.2.0.254 dev eth0 
```

Hosts
```
[root@server1 ~]# nano /etc/hosts

10.5.2.10 client1 client1.tp5.b1
10.5.2.11 client2 client2.tp5.b1
```


Depuis client2:

```
[root@client2 ~]# ping client1.tp5.b1 -c 2 && echo &&  ping server1.tp5.b1 -c 2
PING client1 (10.5.2.10) 56(84) bytes of data.
64 bytes from client1 (10.5.2.10): icmp_seq=1 ttl=64 time=6.56 ms
64 bytes from client1 (10.5.2.10): icmp_seq=2 ttl=64 time=4.80 ms

--- client1 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1001ms
rtt min/avg/max/mdev = 4.830/5.683/6.537/0.856 ms

PING server1 (10.5.1.10) 56(84) bytes of data.
64 bytes from server1 (10.5.1.10): icmp_seq=1 ttl=62 time=27.8 ms
64 bytes from server1 (10.5.1.10): icmp_seq=2 ttl=62 time=33.8 ms

--- server1 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1002ms
rtt min/avg/max/mdev = 27.911/30.842/33.774/2.936 ms

```

#III. DHCP
1. Mise en place du serveur DHCP
* Renommer la machine
On renomme la machine de manière permanente
[root@client2 ~]# nano /etc/hostname
hostname dhcp-net2.tp5.b1
* Installer le serveur DHCP
On active la carte NAT `ifup eth2`
```
[root@client2 ~]# ip a show dev eth2
4: eth2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 00:50:00:00:05:02 brd ff:ff:ff:ff:ff:ff
    inet 192.168.20.17/24 brd 192.168.20.255 scope global noprefixroute dynamic eth2
       valid_lft 7143sec preferred_lft 7143sec
    inet6 fe80::250:ff:fe00:502/64 scope link 
       valid_lft forever preferred_lft forever
```
On installe le package DHCP:
`[root@dhcp-net2 ~]# yum install -y dhcp`
`[root@dhcp-net2 ~]# ifup eth0 && ifdown eth2`

* Démarrer le serveur DHCP
On modifie le fichier de configuration,
puis on le lance
`[root@dhcp-net2 ~]# systemctl start dhcpd`
Afin que ça se lance quand on lance la machine :
`[root@dhcp-net2 ~]# systemctl enable dhcpd`
Puis on vérifie que tout est bon
```
[root@dhcp-net2 ~]# systemctl status dhcpd -l
● dhcpd.service - DHCPv4 Server Daemon
   Loaded: loaded (/usr/lib/systemd/system/dhcpd.service; enabled; vendor preset: disabled)
   Active: active (running) since mer. 2019-02-20 17:27:20 CET; 2min 48s ago
     Docs: man:dhcpd(8)
           man:dhcpd.conf(5)
 Main PID: 3881 (dhcpd)
   Status: "Dispatching packets..."
   CGroup: /system.slice/dhcpd.service
           └─3881 /usr/sbin/dhcpd -f -cf /etc/dhcp/dhcpd.conf -user dhcpd -group dhcpd --no-pid

févr. 20 17:27:20 dhcp-net2.tp5.b1 dhcpd[3881]: Listening on LPF/eth0/00:50:00:00:05:00/10.5.2.0/24
févr. 20 17:27:20 dhcp-net2.tp5.b1 dhcpd[3881]: Sending on   LPF/eth0/00:50:00:00:05:00/10.5.2.0/24
févr. 20 17:27:20 dhcp-net2.tp5.b1 dhcpd[3881]: Sending on   Socket/fallback/fallback-net
févr. 20 17:27:20 dhcp-net2.tp5.b1 systemd[1]: Started DHCPv4 Server Daemon.
```
* Faire un test
En utilisant la VM client1.tp5.b1:
-   Modification du fichier /etc/sysconfig/network-scripts/ifcfg-eth0 pour une modification permanente
-   dhclient -v pour demander une adresse ip.
```
[root@client1 ~]# dhclient -v eth0
Internet Systems Consortium DHCP Client 4.2.5
Copyright 2004-2013 Internet Systems Consortium.
All rights reserved.
For info, please visit https://www.isc.org/software/dhcp/

Listening on LPF/eth0/00:50:00:00:04:00
Sending on   LPF/eth0/00:50:00:00:04:00
Sending on   Socket/fallback
DHCPDISCOVER on eth0 to 255.255.255.255 port 67 interval 3 (xid=0x6ad2d7)
DHCPREQUEST on eth0 to 255.255.255.255 port 67 (xid=0x6 ad2d7)
DHCPOFFER from 10.5.2.11
DHCPACK from 10.5.2.11 (xid=0x6ad2d7)
bound to 10.5.2.50 -- renewal in 232 seconds.
```

# TP 3 - Routage, ARP, Spéléologie réseau :computer:

## Préparation de l'environnement

### 2 - Mise en place du lab

On désactive la carte NAT avec `sudo ifdown enp0s3`.
Ensuite, dans le fichier `/etc/sysconfig/network-scripts/ifcfg-enp0s3` on change `ONBOOT='yes'` 
en `ONBOOT='no'` puis on vérifie la modification avec la commande`ip a` :

```bash
[sghouthi@client1 ~]$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 08:00:27:98:38:c5 brd ff:ff:ff:ff:ff:ff
3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 08:00:27:92:14:91 brd ff:ff:ff:ff:ff:ff
    inet 10.3.1.11/24 brd 10.3.1.255 scope global noprefixroute enp0s8
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fe92:1491/64 scope link 
       valid_lft forever preferred_lft forever
```

Avec la commande `ss -tulnp`, on s'assure que le port ssh est en écoute sur le port `7777`:

```bash
[sghouthi@client1 ~]$ ss -tulnp                                                                        
Netid State      Recv-Q Send-Q    Local Address:Port                   Peer Address:Port              
tcp   LISTEN     0      100           127.0.0.1:25                                *:*                  
tcp   LISTEN     0      128                   *:7777                              *:*                  
tcp   LISTEN     0      100               [::1]:25                             [::]:*                  
tcp   LISTEN     0      128                [::]:7777                           [::]:*
```

On s'assure ensuite que le firewall est bien activé et qu'il autorise le port 7777/tcp en tapant la commande `sudo firewall-cmd --list-all` : 

```bash
[sghouthi@client1 ~]$ sudo firewall-cmd --list-all
[sudo] password for agrorec: 
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: enp0s8
  sources: 
  services: dhcpv6-client ssh
  ports: 7777/tcp
  protocols: 
  masquerade: no
  forward-ports: 
  source-ports:
  icmp-blocks: 
  rich rules:
```

On vérifie le hostname avec la commande `hostname`

```bash
[sghouthi@client1 ~]$ hostname
client1.net1.tp3
```

On modifie le fichier `/etc/hosts`, puis on vérifie avec un `cat` :

```bash
[sghouthi@client1 ~]$ cat /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
10.3.1.11   tp3.b1 client1.net1.tp3
```

On fait un `ip a` sur la vm :

```bash
[sghouthi@client1 ~]$ cat /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
10.3.1.11   tp3.b1 client1.net1.tp3
[agrorec@client1 ~]$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 08:00:27:98:38:c5 brd ff:ff:ff:ff:ff:ff
3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 08:00:27:92:14:91 brd ff:ff:ff:ff:ff:ff
    inet 10.3.1.11/24 brd 10.3.1.255 scope global noprefixroute enp0s8
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fe92:1491/64 scope link 
       valid_lft forever preferred_lft forever
```

On `ping` 10.3.1.11 avec la machine host vers la VM : 

```bash
sghouthi-pc :: ~ » ping 10.3.1.11 -c 3
PING 10.3.1.11 (10.3.1.11) 56(84) bytes of data.
64 bytes from 10.3.1.11: icmp_seq=1 ttl=64 time=0.777 ms
64 bytes from 10.3.1.11: icmp_seq=2 ttl=64 time=0.746 ms
64 bytes from 10.3.1.11: icmp_seq=3 ttl=64 time=0.671 ms

--- 10.3.1.11 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2020ms
rtt min/avg/max/mdev = 0.671/0.731/0.777/0.044 ms
```

---

#### Maintenant on peut vérifier qu'au sein de notre lab le client1 peut ping rooter :

```bash
[sghouthi@client1 ~]$ ping 10.3.1.254 -c 3
PING 10.3.1.254 (10.3.1.254) 56(84) bytes of data.
64 bytes from 10.3.1.254: icmp_seq=1 ttl=64 time=1.33 ms
64 bytes from 10.3.1.254: icmp_seq=2 ttl=64 time=2.67 ms
64 bytes from 10.3.1.254: icmp_seq=3 ttl=64 time=1.25 ms

--- 10.3.1.254 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2005ms
rtt min/avg/max/mdev = 1.253/1.753/2.678/0.656 ms
```

Et inversement rooter peut ping client1 :

```bash
[sghouthi@router ~]$ ping 10.3.1.11 -c 3
PING 10.3.1.11 (10.3.1.11) 56(84) bytes of data.
64 bytes from 10.3.1.11: icmp_seq=1 ttl=64 time=1.23 ms
64 bytes from 10.3.1.11: icmp_seq=2 ttl=64 time=1.36 ms
64 bytes from 10.3.1.11: icmp_seq=3 ttl=64 time=1.33 ms

--- 10.3.1.11 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2012ms
rtt min/avg/max/mdev = 1.232/1.309/1.365/0.056 ms
```

On de ping le rooter depuis le server1 : 

```bash
[sghouthi@server1 ~]$ ping 10.3.2.254 -c 3
PING 10.3.2.254 (10.3.2.254) 56(84) bytes of data.
64 bytes from 10.3.2.254: icmp_seq=1 ttl=64 time=1.31 ms
64 bytes from 10.3.2.254: icmp_seq=2 ttl=64 time=2.46 ms
64 bytes from 10.3.2.254: icmp_seq=3 ttl=64 time=1.32 ms

--- 10.3.2.254 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2009ms
rtt min/avg/max/mdev = 1.314/1.701/2.466/0.541 ms
```

Et inversement : 

```bash
[sghouthi@router ~]$ ping 10.3.2.11 -c 3
PING 10.3.2.11 (10.3.2.11) 56(84) bytes of data.
64 bytes from 10.3.2.11: icmp_seq=1 ttl=64 time=1.44 ms
64 bytes from 10.3.2.11: icmp_seq=2 ttl=64 time=1.66 ms
64 bytes from 10.3.2.11: icmp_seq=3 ttl=64 time=1.50 ms

--- 10.3.2.11 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2007ms
rtt min/avg/max/mdev = 1.441/1.537/1.667/0.095 ms
```
---

## I. Mise en place du routage
### 1. Configuration du routage sur `rooter`

Tout d'abord on vérifie si l'IP Forwarding est activée. Pour cela, on tape la commande `cat /proc/sys/net/ipv4/ip_forward` : 

```bash
[sghouthi@router ~]$ cat /proc/sys/net/ipv4/ip_forward
0
``` 

Le 0 nous indique que l'IP Forwarding n'est pas activé on tape donc la commande :

```bash
[sghouthi@router ~]$ sudo sysctl -w net.ipv4.ip_forward=1
net.ipv4.ip_forward = 1
```

La machine est donc capable maintenant de "rooter" des paquets

### 2. Ajouter les routes statiques

On tape la commande `sudo ip route add 10.3.2.0/24 via 10.3.1.254 dev enp0s8` pour joindre le réseau net2, on vérifie avec la commande `ip r s` : 

```bash
[sghouthi@client1 ~]$ ip r s                                                 
10.3.1.0/24 dev enp0s8 proto kernel scope link src 10.3.1.11 metric 101 
10.3.2.0/24 via 10.3.1.254 dev enp0s8
```
On le voit apparaître sur la deuxième ligne.

On refait la même étape pour `server1` pour qu'il joigne `net1` en tapant la commande `sudo ip route add 10.3.1.0/24 via 10.3.2.254 dev enp0s8` : 

```bash
[sghouthi@server1 ~]$ ip r s
10.3.1.0/24 via 10.3.2.254 dev enp0s8 
10.3.2.0/24 dev enp0s8 proto kernel scope link src 10.3.2.11 metric 101
```

Cette fois-ci on peut le vérifier sur la première ligne.

&nbsp;

On vérifie que ça fonctionne bien avec un ping des deux côtés :

```bash
[sghouthi@client1 ~]$ ping 10.3.2.254 -c 3
PING 10.3.2.254 (10.3.2.254) 56(84) bytes of data.
64 bytes from 10.3.2.254: icmp_seq=1 ttl=64 time=1.23 ms
64 bytes from 10.3.2.254: icmp_seq=2 ttl=64 time=1.30 ms
64 bytes from 10.3.2.254: icmp_seq=3 ttl=64 time=1.35 ms

--- 10.3.2.254 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2004ms
rtt min/avg/max/mdev = 1.233/1.296/1.352/0.064 ms
```

Puis avec `traceroute` : 

```bash
[sghouthi@client1 ~]$ traceroute 10.3.2.11
traceroute to 10.3.2.11 (10.3.2.11), 30 hops max, 60 byte packets
 1  gateway (10.3.1.254)  2.140 ms  1.850 ms  1.745 ms
 2  gateway (10.3.1.254)  1.361 ms !X  0.863 ms !X  2.109 ms !X
```

> client1 vers net2

Puis : 
```bash
[sghouthi@server1 ~]$ ping 10.3.1.254 -c 3
PING 10.3.1.254 (10.3.1.254) 56(84) bytes of data.
64 bytes from 10.3.1.254: icmp_seq=1 ttl=64 time=2.93 ms
64 bytes from 10.3.1.254: icmp_seq=2 ttl=64 time=1.17 ms
64 bytes from 10.3.1.254: icmp_seq=3 ttl=64 time=1.46 ms

--- 10.3.1.254 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2005ms
rtt min/avg/max/mdev = 1.171/1.854/2.930/0.771 ms
```

Et avec `traceroute` :

```bash
[sghouthi@server1 ~]$ traceroute 10.3.1.11
traceroute to 10.3.1.11 (10.3.1.11), 30 hops max, 60 byte packets
 1  gateway (10.3.2.254)  1.122 ms  0.488 ms  0.823 ms
 2  gateway (10.3.2.254)  0.626 ms !X  0.980 ms !X  1.009 ms !X
```

> server1 vers net1

---

### 3. Comprendre le routage

|             | MAC src       | MAC dst       | IP src       | IP dst       |
| ----------- | ------------- | ------------- | ------------ | ------------ |
| Dans `net1` (trame qui entre dans `router`) | 08:00:27:92:14:91 | 08:00:27:76:e0:78 | 10.3.1.11 | 10.3.2.11 |
| Dans `net2` (trame qui sort de `router`) | 08:00:27:16:98:9d | 08:00:27:33:cd:4e | 10.3.1.11 | 10.3.2.11 |

Demander confirmation au grand manitou alias it4like

---

## II. ARP
### 1. Tables ARP
#### Client1 : 

```bash
[sghouthi@client1 ~]$ ip neigh show
10.3.1.254 dev enp0s8 lladdr 08:00:27:76:e0:78 STALE
10.3.1.1 dev enp0s8 lladdr 0a:00:27:00:00:00 REACHABLE
```

`10.3.1.254` : adresse du rooter
&nbsp;

`dev enp0s8` : en passant par la carte nommée **enp0s8**
&nbsp;

`lladdr` : Acronyme pour l'adresse MAC
&nbsp; 

`08:00:27:76:e0:78` : MAC de la carte **enp0s8** du rooter
&nbsp;

`0a:00:27:00:00:00` : MAC de la carte host-only 1
&nbsp;

`STALE` : Entrée valable mais suspecte
&nbsp; 

`REACHABLE` : Entrée valable jusqu'à expiration du délai d'accéssibilité

&nbsp;

#### Server1

```bash
[sghouthi@server1 ~]$ ip neigh show 
10.3.2.254 dev enp0s8 lladdr 08:00:27:16:98:9d STALE
10.3.2.1 dev enp0s8 lladdr 0a:00:27:00:00:01 REACHABLE
```


`10.3.2.254` : adresse du router
&nbsp;

`dev enp0s8` : en passant par la carte nommée **enp0s8**
&nbsp;

`lladdr` : Acronyme pour l'adresse MAC
&nbsp;

`08:00:27:16:98:9d` : MAC de la carte **enp0s9** du router
&nbsp;

`0a:00:27:00:00:01` : MAC de la carte host-only 2
&nbsp; 

`STALE` : Entrée valable mais suspecte
&nbsp; 

`REACHABLE` : Entrée valable jusqu'à expiration du délai d'accéssibilité

&nbsp;

#### Rooter

```bash
[sghouthi@router ~]$ ip neigh show  
10.3.1.11 dev enp0s8 lladdr 08:00:27:92:14:91 STALE
10.3.2.11 dev enp0s9 lladdr 08:00:27:33:cd:4e STALE
10.3.1.1 dev enp0s8 lladdr 0a:00:27:00:00:00 REACHABLE
```

`10.3.1.11` : adresse du client1
&nbsp;

`10.3.2.11` : adresse du server1
&nbsp;

`dev enp0s8` : en passant par la carte nommée **enp0s8**
&nbsp;

`dev enp0s9` : en passant par la carte nommée **enp0s9**
&nbsp;

`lladdr` : Acronyme pour l'adresse MAC
&nbsp; 

`08:00:27:92:14:91` : MAC de la carte enp0s8 de client1
&nbsp;

`08:00:27:33:cd:4e` : MAC de la carte enp0s8 de server1
&nbsp;

`0a:00:27:00:00:00` : MAC de la carte host-only 1
&nbsp; 

`STALE` : Entrée valable mais suspecte
&nbsp;
 
`REACHABLE` : Entrée valable jusqu'à expiration du délai d'accéssibilité

---

### 2. Requêtes ARP
#### A. Table ARP 1

On observe d'abord la table ARP vidé de client1 : 

```bash
[sghouthi@client1 ~]$ sudo ip neigh flush all
[sghouthi@client1 ~]$ ip neigh show          
10.3.1.1 dev enp0s8 lladdr 0a:00:27:00:00:00 REACHABLE
```
Seul le router est joignable, puis on fait un `ping 10.3.2.11 -c 3` pour envoyer 3 paquets a `server1`, on regarde de nouveau la table ARP : 

```bash
[sghouthi@client1 ~]$ ip neigh show
10.3.1.254 dev enp0s8 lladdr 08:00:27:76:e0:78 REACHABLE
10.3.1.1 dev enp0s8 lladdr 0a:00:27:00:00:00 REACHABLE
```

On observe que l'entrée vers `server1` est possible car on lit **REACHABLE**


#### B. Table ARP 2

On vide les tables ARP comme au dessus, puis on tape la commande `ping 10.3.2.11 -c 3` et on obtient une nouvelle ligne dans la table ARP du server1 :

```bash
[sghouthi@server1 ~]$ ip neigh show
10.3.2.254 dev enp0s8 lladdr 08:00:27:16:98:9d REACHABLE
10.3.2.1 dev enp0s8 lladdr 0a:00:27:00:00:01 REACHABLE
```

La première ligne est celle qui dit que l'entrée du rooter est valable. Et la deuxième ligne vient de s'ajouter suite au `ping` et elle dit que l'entrée du client1 est valable.

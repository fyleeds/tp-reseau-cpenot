# TP4 : TCP, UDP et services réseau

Dans ce TP on va explorer un peu les protocoles TCP et UDP. 

**La première partie est détente**, vous explorez TCP et UDP un peu, en vous servant de votre PC.

La seconde partie se déroule en environnement virtuel, avec des VMs. Les VMs vont nous permettre en place des services réseau, qui reposent sur TCP et UDP.  
**Le but est donc de commencer à mettre les mains de plus en plus du côté administration, et pas simple client.**

Dans cette seconde partie, vous étudierez donc :

- le protocole SSH (contrôle de machine à distance)
- le protocole DNS (résolution de noms)
  - essentiel au fonctionnement des réseaux modernes

![TCP UDP](./pics/tcp_udp.jpg)

# Sommaire

- [TP4 : TCP, UDP et services réseau](#tp4--tcp-udp-et-services-réseau)
- [Sommaire](#sommaire)
- [0. Prérequis](#0-prérequis)
- [I. First steps](#i-first-steps)
- [II. Mise en place](#ii-mise-en-place)
  - [1. SSH](#1-ssh)
  - [2. Routage](#2-routage)
- [III. DNS](#iii-dns)
  - [1. Présentation](#1-présentation)
  - [2. Setup](#2-setup)
  - [3. Test](#3-test)

# 0. Prérequis

➜ Pour ce TP, on va se servir de VMs Rocky Linux. On va en faire plusieurs, n'hésitez pas à diminuer la RAM (512Mo ou 1Go devraient suffire). Vous pouvez redescendre la mémoire vidéo aussi.  

➜ Si vous voyez un 🦈 c'est qu'il y a un PCAP à produire et à mettre dans votre dépôt git de rendu

➜ **L'emoji 🖥️ indique une VM à créer**. Pour chaque VM, vous déroulerez la checklist suivante :

- [x] Créer la machine (avec une carte host-only)
- [ ] Définir une IP statique à la VM
- [ ] Donner un hostname à la machine
- [ ] Vérifier que l'accès SSH fonctionnel
- [ ] Vérifier que le firewall est actif
- [ ] Remplir votre fichier `hosts`, celui de votre PC, pour accéder au VM avec un nom
- [ ] Dès que le routeur est en place, n'oubliez pas d'ajouter une route par défaut aux autres VM pour qu'elles aient internet


# I. First steps


🌞 **Déterminez, pour ces 5 applications, si c'est du TCP ou de l'UDP**

- avec Wireshark, on va faire les chirurgiens réseau
- déterminez, pour chaque application :
  - IP et port du serveur auquel vous vous connectez
  - le port local que vous ouvrez pour vous connecter
1. Discord:
- [ ] 162.159.136.232
- [ ]  443
- [ ] 54250

2. Steam:
- [ ]  185.25.182.77
- [ ]  443
- [ ] 53677

3. Teams:
- [ ]  52.113.199.174
- [ ]  https
- [ ] 53485

4. Razer Central Service:
- [ ]  185.25.182.76
- [ ]  27017
- [ ] 51351

5. Epic Games:
- [ ]  54.175.230.103
- [ ]  443
- [ ] 53620

PS C:\WINDOWS\system32> netstat -n -b -p tcp

Connexions actives

  Proto  Adresse locale         Adresse distante       État

[Discord.exe]
  TCP    192.168.1.16:59844     162.159.136.234:https  ESTABLISHED

[Steam.exe]
  TCP    192.168.1.16:50955     185.25.182.77:27022    ESTABLISHED

[Teams.exe]
  TCP    192.168.1.16:51074     52.113.199.174:https   ESTABLISHED

[RazerCentralService.exe]
  TCP    192.168.1.16:51351     185.25.182.76:27017    TIME_WAIT

 [EpicGamesLauncher.exe]
  TCP    192.168.1.16:50955     54.175.230.103:53620    ESTABLISHED
  


🌞 **Demandez l'avis à votre OS**

- votre OS est responsable de l'ouverture des ports, et de placer un programme en "écoute" sur un port
- il est aussi responsable de l'ouverture d'un port quand une application demande à se connecter à distance vers un serveur
- bref il voit tout quoi
- utilisez la commande adaptée à votre OS pour repérer, dans la liste de toutes les connexions réseau établies, la connexion que vous voyez dans Wireshark, pour chacune des 5 applications


🦈🦈🦈🦈🦈 **Bah ouais, captures Wireshark à l'appui évidemment.** Une capture pour chaque application, qui met bien en évidence le trafic en question.

# II. Mise en place

## 1. SSH

🖥️ **Machine `node1.tp4.b1`**

- n'oubliez pas de dérouler la checklist (voir [les prérequis du TP](#0-prérequis))
- donnez lui l'adresse IP `10.4.1.11/24`

Connectez-vous en SSH à votre VM.

🌞 **Examinez le trafic dans Wireshark**

- **déterminez si SSH utilise TCP ou UDP**
- SSH utilise TCP car on effectue une connexion et UDP fontctionne en datagramme "sans connexion".
  - pareil réfléchissez-y deux minutes, logique qu'on utilise pas UDP non ?
- **repérez le *3-Way Handshake* à l'établissement de la connexion**
  - c'est le `SYN` `SYNACK` `ACK`fire
- **repérez du trafic SSH**
- **repérez le FIN ACK à la fin d'une connexion**
- entre le *3-way handshake* et l'échange `FIN`, c'est juste une bouillie de caca chiffré, dans un tunnel TCP

> **SUR WINDOWS, pour cette étape uniquement**, utilisez Git Bash et PAS Powershell. Avec Powershell il sera très difficile d'observer le FIN ACK.

🌞 **Demandez aux OS**

- repérez, avec une commande adaptée (`netstat` ou `ss`), la connexion SSH depuis votre machine

TCP    10.4.1.1:60967         10.4.1.11:22           ESTABLISHED
 [ssh.exe]

- ET repérez la connexion SSH depuis votre VM

tcp      ESTAB    0         52                              10.4.1.11:22                10.4.1.1:60967


🦈 **Je veux une capture clean avec le 3-way handshake, un peu de trafic au milieu et une fin de connexion**

## 2. Routage

Ouais, un peu de répétition, ça fait jamais de mal. On va créer une machine qui sera notre routeur, et **permettra à toutes les autres machines du réseau d'avoir Internet.**

🖥️ **Machine `router.tp4.b1`**

- n'oubliez pas de dérouler la checklist (voir [les prérequis du TP](#0-prérequis))
- donnez lui l'adresse IP `10.4.1.254/24` sur sa carte host-only
- ajoutez-lui une carte NAT, qui permettra de donner Internet aux autres machines du réseau
- référez-vous au TP précédent

> Rien à remettre dans le compte-rendu pour cette partie.

# III. DNS

## 2. Setup


🌞 **Dans le rendu, je veux**

- un `cat` des fichiers de conf
- un `systemctl status named` qui prouve que le service tourne bien
- [clementpenot@dns-server ~]$ systemctl status named
● named.service - Berkeley Internet Name Domain (DNS)
     Loaded: loaded (/usr/lib/systemd/system/named.service; disabled; vendor preset: disabled)
     Active: active (running) since Sat 2022-11-05 23:47:10 CET; 4s ago
    Process: 41956 ExecStartPre=/bin/bash -c if [ ! "$DISABLE_ZONE_CHECKING" == "yes" ]; then /usr>
    Process: 41960 ExecStart=/usr/sbin/named -u named -c ${NAMEDCONF} $OPTIONS (code=exited, statu>
   Main PID: 41961 (named)
      Tasks: 4 (limit: 5906)
     Memory: 14.6M
        CPU: 67ms
     CGroup: /system.slice/named.service
             └─41961 /usr/sbin/named -u named -c /etc/named.conf

Nov 05 23:47:10 dns-server named[41961]: configuring command channel from '/etc/rndc.key'
Nov 05 23:47:10 dns-server named[41961]: command channel listening on 127.0.0.1#953
Nov 05 23:47:10 dns-server named[41961]: configuring command channel from '/etc/rndc.key'
Nov 05 23:47:10 dns-server named[41961]: command channel listening on ::1#953
Nov 05 23:47:10 dns-server named[41961]: managed-keys-zone: loaded serial 0
Nov 05 23:47:10 dns-server named[41961]: zone 1.4.10.in-addr.arpa/IN: loaded serial 2019061800
Nov 05 23:47:10 dns-server named[41961]: zone tp4.b1/IN: loaded serial 2019061800
Nov 05 23:47:10 dns-server named[41961]: all zones loaded
Nov 05 23:47:10 dns-server systemd[1]: Started Berkeley Internet Name Domain (DNS).
Nov 05 23:47:10 dns-server named[41961]: running
lines 1-22/22 (END)
- une commande `ss` qui prouve que le service écoute bien sur un port
- ss
- tcp    ESTAB  0       52                         10.4.1.201:ssh           10.4.1.1:60873

🌞 **Ouvrez le bon port dans le firewall**

- grâce à la commande `ss` vous devrez avoir repéré sur quel port tourne le service
  - vous l'avez écrit dans la conf aussi toute façon :)
- ouvrez ce port dans le firewall de la machine `dns-server.tp4.b1` (voir le mémo réseau Rocky)

## 3. Test

🌞 **Sur la machine `node1.tp4.b1`**

- configurez la machine pour qu'elle utilise votre serveur DNS quand elle a besoin de résoudre des noms
- assurez vous que vous pouvez :
  - résoudre des noms comme `node1.tp4.b1` et `dns-server.tp4.b1`
  - mais aussi des noms comme `www.google.com`

🌞 **Sur votre PC**

- utilisez une commande pour résoudre le nom `node1.tp4.b1` en utilisant `10.4.1.201` comme serveur DNS

*  Get-DnsClientServerAddress

>InterfaceAlias               Interface Address ServerAddresses
                             Index     Family
--------------               --------- ------- ---------------
> Ethernet 2                           3 IPv4    {}
Ethernet 2                           3 IPv6    {fec0:0:0:ffff::1, fec0:0:0:ffff::2, fec...
Ethernet 3                          19 IPv4    {}
Ethernet 3                          19 IPv6    {fec0:0:0:ffff::1, fec0:0:0:ffff::2, fec...
Ethernet 4                          15 IPv4    {}
Ethernet 4                          15 IPv6    {fec0:0:0:ffff::1, fec0:0:0:ffff::2, fec...
Connexion au réseau local* 1         5 IPv4    {}
Connexion au réseau local* 1         5 IPv6    {fec0:0:0:ffff::1, fec0:0:0:ffff::2, fec...
Connexion au réseau local* 2        24 IPv4    {}
Connexion au réseau local* 2        24 IPv6    {fec0:0:0:ffff::1, fec0:0:0:ffff::2, fec...
Wi-Fi                                8 IPv4    {192.168.1.1}
Wi-Fi                                8 IPv6    {2a01:cb19:a50:9f00:b2bb:e5ff:feb7:637c}
Connexion réseau Bluetooth          26 IPv4    {}
Connexion réseau Bluetooth          26 IPv6    {fec0:0:0:ffff::1, fec0:0:0:ffff::2, fec...
Loopback Pseudo-Interface 1          1 IPv4    {}
Loopback Pseudo-Interface 1          1 IPv6    {fec0:0:0:ffff::1, fec0:0:0:ffff::2, fec...

*  Set-DNSClientServerAddress 
"Ethernet 4" -ServerAddresses ("10.4.1.201")
* nslookup node1.tp4.b1
* ipconfig /flushdns
🦈 **Capture d'une requête DNS vers le nom `node1.tp4.b1` ainsi que la réponse**

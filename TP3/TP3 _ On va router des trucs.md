
# TP3 : On va router des trucs


## Sommaire

- [TP3 : On va router des trucs](#tp3--on-va-router-des-trucs)
  - [Sommaire](#sommaire)
  - [I. ARP](#i-arp)
    - [1. Echange ARP](#1-echange-arp)
    - [2. Analyse de trames](#2-analyse-de-trames)
  - [II. Routage](#ii-routage)
    - [1. Mise en place du routage](#1-mise-en-place-du-routage)
    - [2. Analyse de trames](#2-analyse-de-trames-1)
    - [3. Accès internet](#3-accès-internet)
  - [III. DHCP](#iii-dhcp)
    - [1. Mise en place du serveur DHCP](#1-mise-en-place-du-serveur-dhcp)
    - [2. Analyse de trames](#2-analyse-de-trames-2)


## I. ARP

Première partie simple, on va avoir besoin de 2 VMs.

| Machine  | `10.3.1.0/24` |
|----------|---------------|
| `john`   | `10.3.1.11`   |
| `marcel` | `10.3.1.12`   |

```schema
   john               marcel
  ┌─────┐             ┌─────┐
  │     │    ┌───┐    │     │
  │     ├────┤ho1├────┤     │
  └─────┘    └───┘    └─────┘
```

> Référez-vous au [mémo Réseau Rocky](../../cours/memo/rocky_network.md) pour connaître les commandes nécessaire à la réalisation de cette partie.

### 1. Echange ARP

🌞**Générer des requêtes ARP**

- effectuer un `ping` d'une machine à l'autre
![](https://i.imgur.com/sMxTpwv.jpg)
- [ ] ping vm2>vm1
- observer les tables ARP des deux machines
- repérer l'adresse MAC de `john` dans la table ARP de `marcel` et vice-versa

- [ ] -ip neighbour 
 de chaque coté pour afficher les adresses MAC des vm et passerelle.
- [ ] MAC(john)= 08:00:27:84:72:bb
- [ ] MAC(marcel)=08:00:27:12:c8:b1

- prouvez que l'info est correcte (que l'adresse MAC que vous voyez dans la table est bien celle de la machine correspondante)

- [ ] On compare les adresses MAC obtenus par les tables arp avec la commande ip-a en locale qui affiche aussi l'adresse mac de la carte réseau. On trouve alors la même adresse.

  - une commande pour voir la MAC de `marcel` dans la table ARP de `john`
  ip neigh show 10.3.1.12
  - et une commande pour afficher la MAC de `marcel`, depuis `marcel`
   on ajoute un ligne permanente a la table arp de marcel:
   sudo ip neigh add 10.3.1.12 lladdr de:08:00:27:12:c8:b1 dev enp0s8 nud permanent

### 2. Analyse de trames

🌞**Analyse de trames**

- utilisez la commande `tcpdump` pour réaliser une capture de trame
- videz vos tables ARP, sur les deux machines, puis effectuez un `ping`

- [ ] Pour vider la table ARP: sudo ip neigh flush all

🦈 **Capture réseau `tp3_arp.pcapng`** qui contient un ARP request et un ARP reply

> **Si vous ne savez pas comment récupérer votre fichier `.pcapng`** sur votre hôte afin de l'ouvrir dans Wireshark, et me le livrer en rendu, demandez-moi.

## II. Routage

Vous aurez besoin de 3 VMs pour cette partie. **Réutilisez les deux VMs précédentes.**

| Machine  | `10.3.1.0/24` | `10.3.2.0/24` |
|----------|---------------|---------------|
| `router` | `10.3.1.254`  | `10.3.2.254`  |
| `john`   | `10.3.1.11`   | no            |
| `marcel` | no            | `10.3.2.12`   |

> Je les appelés `marcel` et `john` PASKON EN A MAR des noms nuls en réseau 🌻

```schema
   john                router              marcel
  ┌─────┐             ┌─────┐             ┌─────┐
  │     │    ┌───┐    │     │    ┌───┐    │     │
  │     ├────┤ho1├────┤     ├────┤ho2├────┤     │
  └─────┘    └───┘    └─────┘    └───┘    └─────┘
```

### 1. Mise en place du routage

🌞**Activer le routage sur le noeud `router`**

> Cette étape est nécessaire car Rocky Linux c'est pas un OS dédié au routage par défaut. Ce n'est bien évidemment une opération qui n'est pas nécessaire sur un équipement routeur dédié comme du matériel Cisco.

🌞**Ajouter les routes statiques nécessaires pour que `john` et `marcel` puissent se `ping`**

- il faut taper une commande `ip route add` pour cela, voir mémo
- [ ] temporaire: ip route add 10.3.1.0 via 10.3.2.254 dev enp0s3 (marcel)
- [ ] permanent: sudo nano /etc/sysconfig/network-scripts/route-enp0s3 
y inscrire: 10.3.1.0 via 10.3.2.254 dev enp0s3 
(marcel)
- il faut ajouter une seule route des deux côtés

- [ ] ip r s (marcel):
10.3.1.0/24 via 10.3.2.254 dev enp0s3 proto static metric 100                                                        10.3.2.0/24 dev enp0s3 proto kernel scope link src 10.3.2.12 metric 100  

- une fois les routes en place, vérifiez avec un `ping` que les deux machines peuvent se joindre
PING 10.3.1.11 (10.3.1.11) 56(84) bytes of data.                                                                     64 bytes from 10.3.1.11: icmp_seq=1 ttl=63 time=1.22 ms  
![THE SIZE](./pics/thesize.png)

### 2. Analyse de trames

🌞**Analyse des échanges ARP**

- videz les tables ARP des trois noeuds
- effectuez un `ping` de `john` vers `marcel`
- regardez les tables ARP des trois noeuds
- john:
ip neigh show                                                                                  10.3.1.254 dev enp0s3 lladdr 08:00:27:65:d9:1e REACHABLE                                                             10.3.1.1 dev enp0s3 lladdr 0a:00:27:00:00:03 REACHABLE 

-marcel:
10.3.2.254 dev enp0s3 lladdr 08:00:27:8c:ee:8f STALE                                                                 10.3.2.1 dev enp0s3 lladdr 0a:00:27:00:00:07 DELAY 

-router:
10.3.1.11 dev enp0s3 lladdr 08:00:27:84:72:bb
10.3.2.12 dev enp0s3 lladdr 08:00:27:12:c8:b1

- essayez de déduire un peu les échanges ARP qui ont eu lieu

-les trame contiennent l'ip
-Les MAC servent a transmettre les trames dans un LAN.

-Quand on passe d'un réseau à un autre, il faut garder les ip pour connaitre la source et destination réseau initial mais on modifie les MAC pour permettre un echange au sein du LAN.
- répétez l'opération précédente (vider les tables, puis `ping`), en lançant `tcpdump` sur `marcel`

- **écrivez, dans l'ordre, les échanges ARP qui ont eu lieu, puis le ping et le pong, je veux TOUTES les trames** utiles pour l'échange

Par exemple (copiez-collez ce tableau ce sera le plus simple) :

|
| ordre | type trame  | IP source | MAC source              | IP destination | MAC destination            |
|-------|-------------|-----------|-------------------------|----------------|----------------------------|
| 1     | Requête ARP |10.3.2.12          | `marcel` `08:00:27:8c:ee:8f` |    10.3.2.0          | `Broadcast`, `FF:FF:FF:FF:FF` |
| 2     | Réponse ARP | 10.3.2.254         |`router` `08:00:27:8c:ee:8f`                    | 10.3.2.12              | `marcel` `08:00:27:8c:ee:8f`    |
| 3     | Ping        | 10.3.2.254          | `router` `08:00:27:8c:ee:8f`                       | 10.3.2.12              | `marcel` `08:00:27:12:c8:b1`
| 4     | Pong        | 10.3.2.12              | `marcel` `08:00:27:12:c8:b1`  | 10.3.2.254          | `router` `08:00:27:8c:ee:8f`
| 5   | Requête ARP         | 10.3.2.12       | `marcel` `08:00:27:12:c8:b1`                     |   10.3.2.254             |        `router`     `08:00:27:8c:ee:8f`                ||
| 6  | Réponse ARP         | 10.3.2.254       | `router` `08:00:27:8c:ee:8f`                     |    10.3.2.12            |       `marcel` `08:00:27:12:c8:b1`                      |
| 7     | Ping        | 10.3.2.254          | `router` `08:00:27:8c:ee:8f`                       | 10.3.2.12              | `marcel` `08:00:27:12:c8:b1`
| 8     | Pong        | 10.3.2.12              | `marcel` `08:00:27:12:c8:b1`  | 10.3.2.254          | `router` `08:00:27:8c:ee:8f`
|

> Vous pourriez, par curiosité, lancer la capture sur `john` aussi, pour voir l'échange qu'il a effectué de son côté.

🦈 **Capture réseau `tp3_routage_marcel.pcapng`**

### 3. Accès internet

🌞**Donnez un accès internet à vos machines**

- ajoutez une carte NAT en 3ème inteface sur le `router` pour qu'il ait un accès internet
ajout dans Vbox>>>> Tools>>>NetworkManager>>>Add Nat
et selectionner dans les parametres de router>>>>Network>>>>Carte NAT
- ajoutez une route par défaut à `john` et `marcel`
 ip route add default via 10.3.2.254 et 10.3.1.254
  - vérifiez que vous avez accès internet avec un `ping`
  - le `ping` doit être vers une IP, PAS un nom de domaine
- donnez leur aussi l'adresse d'un serveur DNS qu'ils peuvent utiliser
  - vérifiez que vous avez une résolution de noms qui fonctionne avec `dig`
  - puis avec un `ping` vers un nom de domaine


# Serveur VPN


L'idée globale est la suivante :

- s'assurer qu'on peut joindre la machine à distance
- s'y connecter en SSH
- mettre en place quelques bonnes pratiques de sécurité
  - gestion d'utilisateurs
  - serveur SSH
- installer le serveur VPN sur la machine à distance
- s'y connecter avec un client adapté

# I. Setup machine distante


## 1. Serveur SSH

### A. Connexion par clé

➜ **Génération d'une paire de clé** SUR LE CLIENT

```bash
# étape à réaliser sur VOTRE PC
$ ssh-keygen -t rsa -b 4096
```

# II. Serveur VPN
add rights to user:
in root mode
>sudo su
>usermod -aG wheel rocky

## Step 1 — Installing WireGuard and Generating a Key Pair
extra software to join the wiregard repositories:
>sudo dnf install elrepo-release epel-release

Install wiregard tool : 
>sudo dnf install kmod-wireguard wireguard-tools

Create wg genkey and wg pubkey the keys, and add the private key to WireGuard’s configuration file.:
>wg genkey | sudo tee /etc/wireguard/private.key

Change permissions:
>sudo chmod go= /etc/wireguard/private.key

create the corresponding public key:
>sudo cat /etc/wireguard/private.key | wg pubkey | sudo tee /etc/wireguard/public.key
## Step 2/3 — Choosing IPv4 and Creating a WireGuard Server Configuration
Change Ipv4:
>[rocky@vps-5c2e15b5 ~]$ sudo nano wg0.conf

>[rocky@vps-5c2e15b5 ~]$ sudo cat wg0.conf
[Interface]
PrivateKey = ELszDVrs46F+oSUZjBtctNtTcv1EnvaR/bdFv2UzoWM=
Address = 10.8.0.1/24
ListenPort = 51820
SaveConfig = true

## Step 4 — Adjusting the WireGuard Server’s Network Configuration
Access internet through peer in the wireguard server:

>sudo nano /etc/sysctl.conf
net.ipv4.ip_forward=1

confirm:
>sudo sysctl -p
net.ipv4.ip_forward = 1

## Step 5 — Configuring the WireGuard Server’s Firewall
Add Firewall UDP Port :
>sudo firewall-cmd --zone=public --add-port=51820/udp --permanent

Autorize to connect other devices in the network:
>sudo firewall-cmd --zone=internal --add-interface=wg0 --permanent

Autorize to rewrite traffic in the internal interface:
>sudo firewall-cmd --zone=public --add-rich-rule='rule family=ipv4 source address=10.8.0.0/24 masquerade' --permanent

Firewall reload:
>sudo firewall-cmd --reload

Verify Firewall Settings:
>[rocky@vps-5c2e15b5 etc]$ sudo firewall-cmd --zone=public --list-all
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: eth0
  sources:
  services: cockpit dhcpv6-client ssh
  ports: 51820/udp
  protocols:
  forward: no
  masquerade: no
  forward-ports:
  source-ports:
  icmp-blocks:
  rich rules:
        rule family="ipv4" source address="10.8.0.0/24" masquerade

Verify Firewall Authorized Interface:
>[rocky@vps-5c2e15b5 etc]$ sudo firewall-cmd --zone=internal --list-interfaces
wg0

## Step 6 — Starting the WireGuard Server

Enable Connection to VPN as long as the server:

>sudo systemctl enable wg-quick@wg0.service

verify VPN Status: sudo systemctl status wg-quick@wg0.service
> wg-quick@wg0.service - WireGuard via wg-quick(8) for wg0
   Loaded: loaded (/usr/lib/systemd/system/wg-quick@.service; enabled; vendor preset: disabled)
   Active: active (exited) since Sun 2022-11-20 18:48:44 UTC; 10min ago
     Docs: man:wg-quick(8)
           man:wg(8)
           https://www.wireguard.com/
           https://www.wireguard.com/quickstart/
           https://git.zx2c4.com/wireguard-tools/about/src/man/wg-quick.8
           https://git.zx2c4.com/wireguard-tools/about/src/man/wg.8
  Process: 1026 ExecStart=/usr/bin/wg-quick up wg0 (code=exited, status=0/SUCCESS)
 Main PID: 1026 (code=exited, status=0/SUCCESS)

>Nov 20 18:48:43 vps-5c2e15b5.vps.ovh.net systemd[1]: Starting WireGuard via wg-quick(8) for wg0...

>Nov 20 18:48:44 vps-5c2e15b5.vps.ovh.net wg-quick[1026]: [#] ip link add wg0 type wireguard

>Nov 20 18:48:44 vps-5c2e15b5.vps.ovh.net wg-quick[1026]: [#] wg setconf wg0 /dev/fd/63

>Nov 20 18:48:44 vps-5c2e15b5.vps.ovh.net wg-quick[1026]: [#] ip -4 address add 10.8.0.1/24 dev wg0

>Nov 20 18:48:44 vps-5c2e15b5.vps.ovh.net wg-quick[1026]: [#] ip link set mtu 1420 up dev wg0

>Nov 20 18:48:44 vps-5c2e15b5.vps.ovh.net systemd[1]: Started WireGuard via wg-quick(8) for wg0.

## Step 7 — Configuring a WireGuard Peer
run Wireguard.exe on windows10 and let generate automatically a pair of keys wg

Config wg0 on the Peer Machine:
>[Interface]
PrivateKey = QHMxCmG8TkI6s3nNZsBURcoJ8MK+L2kNCxQh4SCI6Gs=
Address = 10.8.0.2/24

>[Peer]
PublicKey = UC0G2O71ZoO4OJTRZUJkyNb0d7hxt64jMws/WIuvSwA=
AllowedIPs = 10.8.0.0/24
Endpoint = 51.195.136.139:51820

## Step 8 — Adding the Peer’s Public Key to the WireGuard Server
Adding the Peer pub key: 
>sudo wg set wg0 peer 7dcWBG7w2/jwhepvpCH0a8rKUcv7uCCco1S81cDtCiI= allowed-ips 10.8.0.2

Check the status of the tunnel:
>sudo wg

>Output
interface: wg0
 public key: U9uE2kb/nrrzsEU58GD3pKFU3TLYDMCbetIsnV8eeFE=
 private key: (hidden)
 listening port: 51820

>peer: 7dcWBG7w2/jwhepvpCH0a8rKUcv7uCCco1S81cDtCiI=
 allowed ips: 10.8.0.2/32

## Step 9 — Connecting the WireGuard Peer to the Tunnel

Activate peer tunnel on windows using "activate" button

Ping request to the vpn server:
>PS C:\Users\clemc> ping -c 1 10.8.0.1

>Envoi d’une requête 'Ping'  10.8.0.1 avec 32 octets de données :
Réponse de 10.8.0.1 : octets=32 temps=55 ms TTL=64

>PS C:\Users\clemc> ping -c 1 1.1.1.1

>Envoi d’une requête 'Ping'  1.1.1.1 avec 32 octets de données :
Réponse de 1.1.1.1 : octets=32 temps=3 ms TTL=64

Check Tunnel Status on Server:
>[rocky@vps-5c2e15b5 ~]$ sudo wg

>interface: wg0
  public key: UC0G2O71ZoO4OJTRZUJkyNb0d7hxt64jMws/WIuvSwA=
  private key: (hidden)
  listening port: 51820

>peer: 7dcWBG7w2/jwhepvpCH0a8rKUcv7uCCco1S81cDtCiI=
  endpoint: 86.215.117.28:55001
  allowed ips: 10.8.0.2/32
  latest handshake: 4 minutes, 54 seconds ago
  transfer: 4.43 KiB received, 664 B sent


# III. Rendu attendu

- un fichier que je peux utiliser pour me connecter à votre VPN: 
->wg0.conf dans le repo a importer dans les parametres vpn
---
title: Configurer WireGuard sur Fedora CoreOS
author: Damien
description: Protéger son accès à internet, accéder à son réseau local depuis l'extérieur, filtrer les pubs... plusieurs bonnes choses qu'on peut faire via WireGuard
tags:
  - wireguard
  - fedora
  - serveur
  - sysadmin
draft: false
date: 2025-05-06T15:58:26+00:00
---
Au menu d'aujourd'hui :
1. Installer WireGuard ;
2. Faire et comprendre sa première configuration ;
3. Accéder à l'internet mondial via WireGuard ;
4. Rendre le tout persistant ;
5. Bonus : fournir un DNS bloqueur de pub.

## Installer WireGuard
[la doc](https://www.wireguard.com/install/)

```bash
sudo dnf install wireguard-tools
```

## Première config
[la dooc](https://www.wireguard.com/quickstart/)
Avec WireGuard, nous allons venir créer une nouvelle interface réseau (un peu comme une interface ethernet ou wifi).
Pour chaque appareil, nous aurons besoin d'une clef privée et d'une clef publique.

On les génère comme suit :
```bash
umask 077
wg genkey > privatekey
wg pubkey < privatekey > publickey
```
* `umask` permet de fixer les permissions du fichier créé à la prochaine commande exécutée ;
* `>` permet de rediriger les sorties de programme dans un fichier ;
* `<` fait l'inverse : il lit le fichier donné pour le passer en argument de la commande exécutée.

Ou bien en une seule ligne :
```bash
wg genkey | tee privatekey | wg pubkey > publickey
```
* `|` permet de passer le contenu de la sortie à la commande d'après ;
* `tee` écrit le contenu dans le fichier `privatekey` et dans la sortie standard.
### Config du serveur
L'interface que nous allons créer aura le nom de `wg0`, on crée donc le fichier `/etc/wireguard/wg0.conf` qu'on remplit avec :
```toml
[Interface]
PrivateKey = clef_privée_serveur
Address = 10.0.0.1
ListenPort = 51820

# Premier appareil
[Peer]
PublicKey = clef_publique_peer
AllowedIps = 10.0.0.2/32
```
Points clefs à retenir :
* `Address` correspond à l'adresse IP du serveur sur le réseau `wg0`.
* `AllowedIps` correspond au trafic qui sera redirigé depuis le serveur vers le peer.
### Config d'un appareil distant
Sur le client :
```toml
[Interface]
PrivateKey = clef_privee_client
Address = 10.0.0.2/32
DNS = 1.1.1.1

[Peer]
PublicKey = clef_publique_serveur
AllowedIPs = 0.0.0.0/0, ::/0
Endpoint = 192.168.0.162:51820
PersistentKeepalive = 25
```
Points clefs à retenir :
* `Address` correspond à l'adresse IP du client sur le réseau `wg0`.
* `AllowedIps` correspond au trafic qui sera redirigé depuis le peer vers le serveur, ici on demande la totalité.
* `Endpoint` correspond à l'adresse IP du serveur (avec le port en bonus) sur le réseau habituel. Cela peut également être un nom de domaine.
## Accès à Internet

> [!warning] Attention
> Ceci est un risque potentiel de sécurité ! Pensez à paramétrer votre pare-feu.

Pour accéder à l'internet mondial, il faut autoriser le serveur à faire du net forwarding.

### Net forwarding
Prenons une image. 
Depuis une ville A, nous écrivons à Bob qui est dans la ville B. Nous indiquons sur l'enveloppe l'adresse de Bob. Les services postaux vont chercher à délivrer le courrier Bob... dans la ville A. Il ne sera jamais délivré, et nous n'obtiendrons jamais sa réponse.
Il faut autoriser le courrier à voyager entre les villes A et B pour obtenir notre réponse.
Une fois l'autorisation donnée, si les services postaux ne trouvent pas Bob dans la ville A, ils le chercheront dans la ville B.

Sans l'image, il faut autoriser les interfaces réseau à communiquer entre elles.

```bash
sudo sysctl -w net.ipv4.ip_forward=1
sudo sysctl -w net.ipv6.conf.all.forwarding=1
```

### Network Forwarding Table
Une [autre doc](https://docs.pi-hole.net/guides/vpn/wireguard/internal/)
Maintenant que le net forwarding est autorisé, il n'y a plus qu'à le configurer pour l'IPv4 :
```bash
nft add table ip wireguard
nft add chain ip wireguard wireguard_chain {type nat hook postrouting priority srcnat; policy accept;}
nft add rule ip wireguard wireguard_chain counter packets 0 bytes 0 masquerade
```
Et pour l'IPv6 :
```bash
nft add table ip6 wireguard
nft add chain ip6 wireguard wireguard_chain {type nat hook postrouting priority srcnat; policy accept;}
nft add rule ip6 wireguard wireguard_chain counter packets 0 bytes 0 masquerade  
```

## Persistence
C'est bien rigolo tout ça, mais ce n'est pas persistant.
Trois choses à gérer :
1. le lancement de WireGuard ;
2. L'autorisation du net forwarding ;
3. La création des règles de forwarding.

### systemctl
```bash
sudo systemctl start wg-quick@wg0
sudo systemctl enable wg-quick@wg0
```
Donc, on va faire en sorte que ça le devienne : pour le forwarding [la doc de RedHat](https://docs.redhat.com/fr/documentation/red_hat_enterprise_linux/9/html/managing_monitoring_and_updating_the_kernel/using-configuration-files-in-etc-sysctl-d-to-adjust-kernel-parameters_configuring-kernel-parameters-at-runtime#using-configuration-files-in-etc-sysctl-d-to-adjust-kernel-parameters_configuring-kernel-parameters-at-runtime) et pour NFT.

On vient créer `/etc/sysctl.d/95-wg-ipforwarding.conf`
```toml
net.ipv4.ip_forward = 1
net.ipv6.conf.all.forwarding = 1
```

Ce fichier sera exécuté à chaque démarrage du serveur.

Pour le reste, on vient ajouter dans le fichier de configuration du serveur ces deux lignes, dans la section `[Interface]` :
```toml
PostUp = nft add table ip wireguard; nft add chain ip wireguard wireguard_chain {type nat hook postrouting priority srcnat; policy accept;}; nft add rule ip wireguard wireguard_chain counter packets 0 bytes 0 masquerade; nft add table ip6 wireguard; nft add chain ip6 wireguard wireguard_chain {type nat hook postrouting priority srcnat; policy accept;}; nft add rule ip6 wireguard wireguard_chain counter packets 0 bytes 0 masquerade  
PostDown = nft delete table ip wireguard; nft delete table ip6 wireguard
```

## Bloquer les pubs
Pour bloquer (en partie) les pubs, on peut paramétrer le serveur DNS qui les filtre pour nous.
Merci AdGuard pour tes DNS !

Rendez-vous sur leur [site](https://adguard-dns.io/fr/public-dns.html), dépliez "Les adresses de nos serveurs" puis "DNS brut", et faites votre choix !

Dans la section Interface de la configuration client, vous pouvez renseigner le serveur DNS qui vous arrange.

```toml
[Interface]
DNS = 94.140.14.15
```

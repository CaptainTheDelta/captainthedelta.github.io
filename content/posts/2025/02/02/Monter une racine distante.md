---
title: Monter une racine distante
author: Damien
description: Comment monter une racine distante de manière sécurisée, les étapes.
tags:
  - aventure
draft: true
date: 2025-02-02T19:49:31+00:00
---
L'objectif d'aujourd'hui : monter un système de fichiers distant. Petit hic, je veux pouvoir toucher aux fichiers dans `/etc`. Et que lorsque j'utilise mon `user@remote`, vlati pas que j'obtiens un `Permission denied`, pas surprenant, rassurant, mais bien embêtant.

Je dois donc me connecter via root depuis mon PC. J'utilise déjà une clef SSH pour me connecter sur `user`, mais je me suis dit que ce serait mieux de me compliquer la tâche et d'en configurer une autre pour le compte `root`.

Donc, il faut :
1.  Créer un nouveau jeu de clef ;
2.  Autoriser la connexion à distance via root ;
3.  Indiquer la clef ;
4.  Monter le système de fichiers.

## Créer une clef
On génère notre  clef, c'est tout simple.
```
ssh-keygen
```
On nous demande un nom de fichier, ce sera `/home/damien/.ssh/yolooo` et un mot de passe.
## Connexion via root
Les options par défaut de OpennSSH permettent la connexion via clef sur l'utilisateur `root`, pas besoin donc d'aller modifier le fichier `/etc/ssh/sshd_config` pour avoir la ligne `PermitRootLogin prohibit-password`, ni de redémarrer le service via `systemctl restart sshd`. 
## Indiquer la clef
On vient ajouter la clef publique dans le fichier distant `authorized_keys` du `root`.

> [!warning] Emplacement du `home` de `root`
> `root` étant un utilisateur à part, son `home` n'est pas dans `/home/root`, mais directement dans... `/root`. Voilà, je viens de vous éviter 28 minutes de recherches.

Dans mon cas :
```
sudo vim /root/.ssh/authorized_keys
```
On ajoute la clef publique dans le fichier, comprendre : mettre le contenu du fichier `yoloo.pub` à la dernière ligne. 
On s'assure ensuite que le fichier `authorized_keys` ait toujours les bonnes autorisations.
```
sudo chmod 600 /root/.ssh/authorized_keys
```
Ensuite, je teste la connexion avec la clef en particulier :
```
ssh -i .ssh/yolooo root@remote
```
Jusque là, on est bon !

## Monter le système
On a besoin de `sshfs`, à installer de la manière que vous préférez, mais présent sur les dépôts.
Manuellement, ça donne :
```
sudo dnf install sshfs
sudo mkdir /media/remote
sudo chown -R damien:damien /media/remote
sshfs -o IdentityFile=/home/damien/.ssh/yolooo root@remote:/etc/ /media/remote
```

Et ça marche. Purée. Side quest accomplished.
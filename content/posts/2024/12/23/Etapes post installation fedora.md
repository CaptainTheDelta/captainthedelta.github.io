---
title: Etapes post installation fedora
author: Damien
description: 
tags: 
draft: false
date: 2024-12-23T14:46:01+00:00
---
## Le grub
### Le voir
```
sudo grub2-editenv list
```
 On voit `menu_auto_hide=1`, ce qui fait disparaître le menu grub.
```
sudo grub2-editenv - unset menu_auto_hide
```
[fedora project](https://discussion.fedoraproject.org/t/how-to-get-grub-menu-to-shhow/91947)
### Le compléter


## Le wifi
ça marche direct, pas besoin d'aller chercher les drivers comme pour Windows (ASUS PCE-AX1800)

## Remettre sa clef ssh
On remet la clef ssh dans le dossier `~/.ssh/`, on lui passe un coup de `chmod 700`, puis à la prochaine utilisation de ssh on rentre la *passphrase* et le tour est joué.
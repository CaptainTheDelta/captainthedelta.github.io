---
title: GitHub Actions pour hugo
author: Damien
description: Nous mettons en place le déploiement automatique du blog sur GitHub.
tags:
  - hugo
  - github
draft: false
date: 2024-12-25T17:32:55+00:00
---
Nous allons aujourd'hui déployer le blog automatiquement sur GitHub.

Plusieurs choses par rapport à cela : 
1. Nous utiliserons les Pages GitHub :
	1. Préparer le répertoire ;
	2. Configurer la page.
2. Nous utiliserons les Actions GitHub :
	1. Création de la configuration

### Nom de domaine personnalisé
Si vous comptez mettre en place un nom de domaine, vu que cela prend un temps indéterminé, ne faites pas comme moi, et commencez par là.
Il suffit d'indiquer à votre hébergeur que vous désirez pointer votre nom de domaine vers l'une de ces quatre adresses : 
* `185.199.108.153`
* `185.199.109.153`
* `185.199.110.153`
* `185.199.111.153`

Pour plus d'infos, direction la [doc de GitHub Pages](https://docs.github.com/fr/pages/configuring-a-custom-domain-for-your-github-pages-site)?
## GitHub pages
### Création d'une page `github.io`
Comme vous le savez, ou pas, GitHub permet d'héberger un site statique par utilisateur.
Pour en bénéficier, il faut créer le répertoire `[username].github.io`. 
Par exemple, dans mon cas, on passe de CaptainTheDelta à `captainthedelta.github.io`.
### Préparation du répertoire
Pour séparer le répertoire d'hugo des pages générées, on crée la branche `gh-pages`.

> [!warning]
> Je n'invente pas le nom, il est réutilisé par le script que nous utiliserons plus tard. Je vous invite à utiliser le même nom.

### Configuration de la `Page`
Deux choses à configurer. Tout d'abord, rendons-nous dans `Settings → Code and automation → Pages → `
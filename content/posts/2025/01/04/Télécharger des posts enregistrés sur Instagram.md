---
title: Télécharger des posts enregistrés sur Instagram
author: Damien
description: Nous voyons ici comment accéder à une liste de posts enregistrés sur Instagram.
tags:
  - python
  - instagram
draft: true
date: 2025-01-04T02:47:53+00:00
---
Bon, ça va être crade.

1. On va sur la page de la collection ;
2. On va sur l'onglet réseau des outils développeurs, et on reload ;
3. On scroll jusqu'en bas ;
4. On filtre avec : `collection` ;
5. On vient `Enregistrer en tant que HAR` (_Http ARchive_ ; via la petite molette en haut à droite).

Reste plus qu'à :
1. lire le fichier correctement ;
2. en extraire les liens des vidéos ;
3. télécharger les vidéos ;

Bonus :
1. gérer une base de données pour ne pas télécharger inutilement des vidéos déjà téléchargées ;
2. Gérer le scroll et le download via Selenium.

## Enregistrer le fichier
Je pensais au départ que chaque requête était à enregistrer, mais le HAR les enregistre toutes.

Ensuite, après les premières étapes de lecture, je me suis rendu compte que certaines réponses étaient tronquées : dans firefox, j'ai droit à un "La réponse a été tronquée". L'internet est formel, il faut soit passer par un autre OS, soit par un autre navigateur... On retente avec Chromium.


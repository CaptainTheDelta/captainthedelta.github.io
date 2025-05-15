---
title: Installer Ollama & Open WebUI
author: Damien
description: Les intelligences artificielles, utilisées à bon escient, c'est bien. En local, c'est mieux.
tags:
  - ia
  - fedora
  - nvidia
draft: false
date: 2025-04-28T23:50:53+00:00
---
Les intelligences artificielles, utilisées à bon escient, c'est bien. En local, c'est mieux.
Nous allons donc nous pencher sur les étapes nécessaires qui nous permettront ultimement de faire tourner un LLM localement.

Bon globalement, au début, voilà ce que je croyais que je demandais :

![première approche.excalidraw](première%20approche.excalidraw.md)

Après un peu de recherches, d'essais et d'erreurs, j'en arrive à :

![deuxième approche.excalidraw](deuxième%20approche.excalidraw.md)
Ce qui implique un peu plus d'étapes. Procédons dans l'ordre.

> [!warning] Disclaimer
> Ceci n'est pas un guide d'installation ! C'est plutôt un résumé des connaissances glanées sur le chemin (avec des liens vers des vrais guides, normalement maintenus)
## Installer les drivers pour la carte graphique

Globalement, il y a trois options :
1. nouveau
2. akmod-nvidia
3. kmod-nvidia

Le pilote nouveau est l'option libre, mais voulant privilégier les performances, je passerais par l'une des deux autres options.

> **RPMFusion** fournit deux types de paquets pour les pilotes **nvidia** :
> - akmod-nvidia (et variantes) qui à chaque démarrage, vérifie la présence du pilote, et si celui-ci est absent, le compile automatiquement. Cela est très pratique pour les mises à jour du noyau, et est du reste fortement recommandé ;
> - kmod-nvidia (et variantes) lui contient directement le pilote déjà compilé, ce paquet doit être construit pour la version du noyau que vous utilisez. Il peut de ce fait y avoir un certain laps de temps entre la sortie d'un nouveau noyau Fedora, et la disponibilité du kmod-nvidia associé.

La [doc de fedora](https://doc.fedora-fr.org/wiki/Carte_graphique_NVIDIA_:_installation_des_pilotes_propri%C3%A9taires).
## Installer les drivers CUDA

Pas besoin d'en faire tout un fromage. 
```bash
sudo dnf install xorg-x11-drv-nvidia-cuda
```

Pour vérifier que tout fonctionne, exécutons la commande 
```bash
nvidia-smi
```

## Installer Ollama

![installation ollama.excalidraw](installation%20ollama.excalidraw.md)

Trois options pour installer Ollama :
1. Directement sur la machine ;
2. Dans un container commun avec Open WebUI ;
3. Dans un container à part.

La doc pour l'installation sur [github](https://github.com/ollama/ollama/blob/main/docs/linux.md).

J'étais parti dans un premier temps sur la deuxième option, je vais basculer bientôt sur la troisième, plus de flexibilité pour un projet futur.

Dans ces deux cas, Ollama est coupé du reste de la machine car situé dans un container. Seulement, le container n'a pas d'accès direct à la carte graphique. 
### Utiliser la carte graphique depuis un container Podman

Il faut utiliser un CDI, et un bon [tuto](https://www.metal3d.org/blog/2023/podman-et-nvidia/).
Ou bien la doc pour le [nvidia-container-toolkit](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html) suivie de la doc pour le [CDI](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/cdi-support.html) .

En résumé, nous installons un paquet qui permet de générer la description de notre matériel, description que nous fournirons à la création d'un nouveau container :

```bash
sudo podman run --device nvidia.com/gpu=all ...
```

> [!info] Besoin de recherche supplémentaires
> `--security-opt=label=disable`

## Installer Ollama

```bash
sudo podman run --device nvidia.com/gpu=all ...
```
## Installer Open WebUI
La [doc](https://docs.openwebui.com/getting-started/quick-start/).

Après, je n'ai pas encore vu s'il y a besoin de mettre en paramètre la carte graphique pour le container d'Open WebUI, dans la mesure où le frontend n'est pas censé faire du backend...

## Installer des LLM
Cela se passe via l'interface de recherche, dans Open WebUI, il cherche sur le site [ollama.com](https://ollama.com/) et télécharge le modèle correspondant à une taille appropriée (je crois). 
## Conclusion
J'aurai pris du temps, mais j'ai appris deux trois trucs sur le chemin. Et maintenant je peux me passer de ChatGPT.

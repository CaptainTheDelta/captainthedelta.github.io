---
title: Notifier la carte de passage Valetudo
author: Damien
description: 
tags:
  - hass
  - valetudo
draft: true
date: 2025-02-20T14:34:46+00:00
---
Dans la suite de l'article précédent, je veux être notifié lorsque mon robot aspirateur a terminé sa tâche, et comme compte rendu, Home Assistant doit m'envoyer la carte de passage du robot.

Avec Valetudo, par mqtt on obtient les contrôles du robot, et une fausse image contenant pleins d'info. Comme je n'ai pas tout compris mais que je voulais quand même pouvoir lire ces infos, j'ai installé [MQTT Vacuum's Camera](https://github.com/sca075/mqtt_vacuum_camera), disponible sur le [HACS](https://hacs.xyz/).

## Le déclenchement
Tout gentil tout mignon, l'automatisation doit se déclencher cinq minutes après que Dobby (oui mon robot s'appelle Dobby) revienne sur sa base.

On a donc :
```

```
## Obtenir l'image
A première vue, rien de plus simple, une option de l'extension permet de l'enregistrer dans `/config/www/`.

Mais.

Mais si c'était aussi simple, ce le serait trop justement. Activer cette option fait crasher mon Home Assistant. Alors plutôt que trouver le pourquoi du comment, j'ai cherché une roue de secours.

Je prends donc une snapshot de la caméra que me fournit l'extension, et je l'enregistre sous `/config/www/snapshot_dobby.png.

```

```

## La notif
Derrière, la notif est très simple :

```
action: notify.notify
metadata: {}
data:
  message: Dobby a fini
  data:
    image: local/snapshot_dobby.png
```

> [!warning] à noter
> `/config/www/` devient `local/` ! 
> C'est très certainement noté dans la doc, mais j'ai du le sauter probablement...


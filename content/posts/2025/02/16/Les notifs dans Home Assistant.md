---
title: Les notifs dans Home Assistant
author: Damien
description: Petit mémo sur l'utilisation des notifs
tags:
  - hass
  - android
draft: true
date: 2025-02-16T15:52:30+00:00
---
Objectif de cette page : avoir une base de cas d'utilisation que je peux réemployer à l'avenir.

Le lien important, [LA DOC](https://companion.home-assistant.io/docs/notifications/notifications-basic/).

## Informations utiles
```
data:
  permanent: true
```
N'empêche pas les notification d'être swipée, lorsque le téléphone est déverouillé.
## Notification du chauffage dans la salle de bain

```
action: notify.notify
metadata: {}
data:
  title: Salle de bain
  message: "Le radiateur chauffe : {{ states(\"sensor.bathroom_temperature\") }}°C"
  data:
    persistent: true
    tag: heating
    importance: high
    alert_once: true
    notification_icon: mdi:shower
```

```
action: notify.notify
metadata: {}
data:
  message: clear_notification
  data:
    tag: heating
```

## Messages d'erreur du robot aspirateur

```
if:
  - condition: state
    entity_id: sensor.valetudo_scentedstarchywren_error
    attribute: message
    state: No error
then:
  - action: notify.notify
    metadata: {}
    data:
      message: clear_notification
      data:
        tag: dobby
else:
  - action: notify.notify
    metadata: {}
    data:
      message: "{{ states(\"sensor.valetudo_scentedstarchywren_error\")}}"
      data:
        tag: dobby
        persistent: true
        sticky: true
        importance: low
        notification_icon: mdi:progress-wrench
      title: Dobby galère
```
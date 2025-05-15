---
title: Interfacer obsidian pour hugo
author: Damien
description: Dans ce blog, nous mettons en place une méthode pour rédiger des articles avec obsidian et les exporter dans le bon format pour hugo.
tags:
  - obsidian
  - hugo
draft: false
date: 2024-12-19T14:43:22+00:00
---

Dans l'objectif de proposer à l'internet mes galères, je veux prendre des notes et pouvoir les partager. J'aime bien hugo, j'aime bien obsidian, autant faire un truc avec les deux.
## Les templates
Les posts d'hugo embarquent plusieurs paramètres dans le [front matter](https://gohugo.io/content-management/front-matter/), comme le titre, l'auteur, la description, les tags, etc. 
Pour gérer ça intelligemment, nous utiliserons les templates d'obsidian.

Dans le vault, nous créons un dossier `templates` et dedans, une note, `post`. Dans cette note :

```
---
title: "{{Title}}"
author: Damien
description: 
tags: 
date: "{{date:YYYY-MM-DD}}T{{time:HH:mm:ss}}+00:00"
draft: true
---
```

Une fois le fichier préparé, nous nous rendons dans `Core plugins → Templates → Template folder` pour sélectionner le dossier contenant notre template.

## Un nouvel article
Nous aurons maximum un article de blog par jour, donc nous adopterons la hiérarchie :
```
2024
	12
		22
			mon article.md
			img
				img1.png
				img2.png
```

Ne voulant pas créer manuellement l'arborescence, nous allons profiter d'une fonctionnalité à la création de la Daily Note : la possibilité de créer une chaîne de dossiers. Dans `Core plugins → Daily notes → Date format`, nous rentrons `YYYY/MM/DD/[untitled]` et le tour est joué.

> [!info]
> Pour ceux que ça intéresse, le format utilisé par obsidian est celui de [moment.js](https://momentjs.com/docs/#/displaying/format/)

Nous en profitons pour renseigner le template précédemment dans `Core plugins → Daily notes → Template file location`.
## L'export
Ce qu'il nous faut, c'est :
1. Copier les articles dans le dossier `content` d'hugo ;
2. Apporter les corrections de liens nécessaires ;
3. Assurer la publication ;
4. Faire en sorte que ce soit léger de notre côté.

Pour ça, on ne va pas s'embêter, on va faire un script python qui gère tout à notre place.

# Les sources
* un gros merci à 4rkal pour son [travail](https://4rkal.com/posts/obsidianhugo/)
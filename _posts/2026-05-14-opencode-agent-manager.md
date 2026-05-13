---
layout: post
title:  "Opencode Agent Manager"
date:   2026-05-14 10:00:00 +0200
author: cloud
categories: dev tools
tags: ai opencode tools dev
permalink: /2026/05/14/opencode-agent-manager.html
---

Opencode est un outil génial mais gérer ses agents et les skills pour chaque projet peut vite devenir pénible. Je me suis donc fait un outil pour gérer ça : [Opencode Agent Manager](https://github.com/madpowah/Opencode-Agent-Manager).

# Le problème

Si tu utilises [Opencode](https://opencode.ai/), tu sais de quoi je parle. Tu définis des agents avec du YAML en frontmatter dans des fichiers `.md`, chaque agent référence des skills (qui sont juste des dossiers avec des fichiers markdown), et chaque projet a son propre répertoire `.opencode/`. Très vite, tu te retrouves à :

-   Copier-coller des définitions d'agents d'un projet à l'autre
-   Perdre la trace de quel agent a tel skill
-   Modifier manuellement des fichiers YAML à la main
-   Oublier quel projet utilise quelle configuration

Bref, ça marche mais c'est pas ce qu'il y a de plus pratique. Surtout quand tu commences à avoir plusieurs projets avec des configurations différentes.

# La solution : Opencode Agent Manager

Je me suis donc penché sur le problème et j'ai pondu une petite app web pour gérer tout ça proprement. L'idée est simple : une interface graphique pour faire tout ce que tu ferais à la main dans les fichiers, mais en mieux.

## Dashboard

Dès que tu arrives sur l'app, tu as un tableau de bord qui te donne une vue d'ensemble de tes agents, de tes skills et de tes projets. Plus besoin de fouiller dans les dossiers pour savoir ce que tu as configuré.

![Dashboard](/assets/images/Dashboard.jpg)

## Gestion des agents

Tu peux créer, modifier et supprimer des agents directement depuis l'interface. L'éditeur te permet de modifier le contenu YAML OpenCode (mode, température, permissions, tout ça). Et surtout, tu peux assigner ou retirer des skills à un agent en quelques clics.

![Agent Configuration](/assets/images/Agent_conf.jpg)

Quand tu assignes un skill à un agent, le fichier YAML est automatiquement écrit dans le répertoire `.opencode/` du projet. Et si tu retires un skill, il est supprimé du filesystem proprement. Plus de fichiers orphelins.

## Gestion des skills

Les skills, c'est la base du fonctionnement d'Opencode. Tu peux en créer, les catégoriser et les assigner aux agents de ton choix. L'interface te permet de voir d'un coup d'oeil quels skills sont utilisés par quels agents.

## Projets

Chaque projet a son propre chemin `.opencode`. Les agents et les skills sont traqués par projet, ce qui veut dire que tu peux avoir des configurations complètement différentes selon le projet sur lequel tu travailles. Le context switch est propre et tu ne mélanges plus jamais une config de pentest avec une config de développement web.

## Import / Export

Le meilleur pour la fin : tu peux scanner des répertoires existants et importer automatiquement les agents et les skills depuis les fichiers `.md` déjà présents. Ça veut dire que si tu as déjà une config OpenCode qui traîne quelque part, tu n'as pas à tout re-saisir. L'app va lire les fichiers, parser le frontmatter YAML et tout mettre dans la base de données.

![Import](/assets/images/Agent-Import.jpg)

Et bien sûr, tu peux exporter le tout vers n'importe quel projet. L'app écrit les fichiers directement au bon endroit.

## Mode Global vs Mode Projet

La sidebar te permet de switcher entre un mode Global (tous les agents et skills disponibles) et un mode Projet (ce qui est configuré pour le projet courant). Pratique pour ne pas être submergé quand tu travailles sur un projet spécifique.

# Technos utilisées

Côté stack, on trouve :

-   **Backend** : NestJS + TypeScript + Prisma pour l'ORM
-   **Frontend** : Next.js 16 + React 19 + Tailwind CSS
-   **Base de données** : PostgreSQL
-   **State management** : Zustand

Rien de bien fou, mais ça fait le job. L'important c'est que ça marche et que ça me simplifie la vie au quotidien.

# Comment ça marche concrètement

Tu crées un agent avec ses paramètres (mode, température, permissions), tu defines les skills que tu veux lui coller, et l'app se charge d'écrire les fichiers YAML OpenCode dans le répertoire `.opencode/` de ton projet. Si tu supprimes un skill d'un agent, le fichier correspondant est retiré du filesystem. C'est propre, c'est clair, c'est maintenable.

L'import, c'est la feature que j'utilise le plus. Tu balances un dossier, l'app scanne tout, parse le YAML et reconstruit la config dans la base. Après tu n'as plus qu'à assigner les skills aux agents depuis l'interface.

Have fun.

[opencode]: https://opencode.ai/

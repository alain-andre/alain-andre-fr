---
title: Installation de NodeJs sous Ubuntu
author: Alain ANDRE
layout: post
date: 2013-12-27 21:10:36 +0200
categories:
  - Petits tutos
tags:
  - installer nodejs
  - ubuntu
---
Il est possible d'installer NodeJs sous Ubuntu depuis les repos proposés par la distribution. Mais la version n'est pas toujours à jour. NodeJs est la dernière technologie en vogue dans le monde du web et nombre de packages évoluent très vite et on des pré-requis de version du serveur.

Il est possible de charger les sources et tout compiler, oui, mais ça n'est pas très propre -enfin à mon avis- Heureusement les PPA sont à notre disposition.
```bash
  $ sudo add-apt-repository ppa:richarvey/nodejs
  [sudo] password for alain:
  Node.js is a platform built on Chrome's JavaScript runtime for easily building fast, scalable network applications.
  Node.js uses an event-driven, non-blocking I/O model that makes it lightweight and efficient, perfect for data-intensive
  real-time applications that run across distributed devices.

  This repo provides the stable builds for nodejs and npm.

  Original code from:

  nodejs: http://nodejs.org/
  npm: http://npmjs.org/

  Plus d’info : https://launchpad.net/~richarvey/+archive/nodejs
  Appuyez sur [ENTRÉE] pour continuer ou Ctrl-C pour annuler l’ajout

  gpg: le porte-clefs « /tmp/tmp5zckpa/secring.gpg » a été créé
  gpg: le porte-clefs « /tmp/tmp5zckpa/pubring.gpg » a été créé
  gpg: demande de la clef 20B1B760 sur le serveur hkp keyserver.ubuntu.com
  gpg: /tmp/tmp5zckpa/trustdb.gpg : base de confiance créée
  gpg: clef 20B1B760 : clef publique « Launchpad PPA for ric_harvey » importée
  gpg: Quantité totale traitée : 1
  gpg:               importées : 1  (RSA: 1)
  OK
  $ sudo apt-get update
  $ sudo apt-get install nodejs npm
```

Et voilà, vous avez la dernière version de **nodejs** et **npm** sur votre machine ;p

---
layout: post
title: "Un Générateur Angular on Rails"
date: 2014-06-26 10:52:32 +0200
comments: true
categories: 
  - Ruby on Rails
tags:
  - angular
  - angularjs
  - rails
  - templating
---

L'idée est de construire un environnement rails qui permet de créer un projet avec les composantes minimales pour une application AngularJS utilisant Bootstrap de Twitter et une gestion d'authentification. Rails nous propose pour faire cela de créer un **template**. Ce template doit posséder un générateur de *templates AngularJS*; qui permettra ainsi de développer rapidement des **scaffolding** d'application AngularJS.

# Le template rails
Ce template crée une application avec les composantes suivantes :

### L'ajout d'AngularJS dans le pipeline asset avec d'autres modules Angular
- gem 'angularjs-rails', '~> 1.2.16'

### Les directives Angular.js UI Bootstrap dans le pipeline asset
- gem 'angular-ui-bootstrap-rails', '~> 0.10.0'

### Le Bootstrap de Twitter converti en Sass
- gem 'bootstrap-sass', '~> 3.1.1'

### Une gem d'autentification
- gem 'devise', '~> 3.2.4'

### Le support international pour notre application
- gem 'i18n', '~> 0.6.9'

### Les fichiers de traduction pour la gem devise
- gem 'devise-i18n', '~> 0.10.3'

### La gestion non bloquante en AngularJS 
- gem 'angular-ujs', '~> 0.4.13'

### Haml (HTML Abstraction Markup Language)
- gem 'haml', '~> 4.0.5'

Le template va charger les gems, créer des commits et paramétrer l'application pour qu'elle fonctionne tel que. La page se charge automatiquement sur la demande d’authentification de l'application. Une fois l'application générée, il ne reste qu'à créer des templates AngularJS.

```bash
git clone https://github.com/alain-andre/ar-template.git
rails _4.0.0_ new test_app -m ar-template/template.rb --skip-bundle
```

# La génération de templates AngularJS
Une fois notre application créée, il est alors possible de générer le code nécessaire à l'utilisation de templates dans assets en utilisant le générateur grâce à la commande `rails g angular_template`.

Par exemple pour générer un scaffold pour le contrôleur *Livre*
```bash
rails g angular_template livre
  create  app/assets/javascripts/controllers/Livre.js
  create  app/assets/templates/Livre/index.html.haml
  create  app/assets/templates/Livre/show.html.haml
  create  app/assets/templates/Livre/edit.html.haml
  create  app/assets/templates/Livre/new.html.haml
  insert  app/assets/javascripts/init.js
    gsub  app/assets/javascripts/init.js
  insert  app/assets/javascripts/application.js
```
Ceci permet alors de créer, supprimer des Livres ainsi que de les lister.
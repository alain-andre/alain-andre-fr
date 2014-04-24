---
layout: post
title: "Installer lunr-search sur Octopress"
date: 2014-04-24 05:42:21 +0200
comments: true
categories:
  - Ruby on Rails
---

Ce post s'inspire de [jekyll-lunr-js-search](https://github.com/slashdotdash/jekyll-lunr-js-search) et de [octopress-lunr-js-search](https://github.com/yortz/octopress-lunr-js-search/blob/master/plugins/search_generator.rb). Je ne trouvais pas ces plugins faciles à installer/utiliser alors j'ai décidé de créer un thème minimaliste qui l'intégre directement. [Ce thème](https://github.com/alain-andre/octopress-lunr-theme) est basé sur une structure permettant de [mettre à jour Octopress](http://octopress.org/docs/updating/).

{% img /images/ 250 "Présentation de la vue recherche" %}

## Installation du plugin.
Ce thème ne modifie que le fichier **source/_includes/custom/head.html**.
Vous pouvez executer les commandes ci-dessous et vous éviter de perdre beaucoup de temps.

Cette partie explique comment installer rapidement le plugin ainsi que les structures dont il a besoin. On commence par travailler sur une branche des fois que votre thème en place ne supporte pas cette installation rapide.
{% raw %}
```bash Installer le plugin et ses dépendances
  $ git checkout -b octopress-lunr-theme
  $ sed -i "/^end/c\  gem 'json'\n  gem 'nokogiri'\nend" Gemfile
  $ bundle install
  $ git clone https://github.com/alain-andre/octopress-lunr-theme.git .themes/octopress-lunr-theme
  $ # Si vous n'avez pas encore installé de thème, passez la commande suivante : bundle exec rake install
  $ bundle exec rake install["octopress-lunr-theme"] # Dans tous les cas
  $ bundle exec rake new_page['search']
  mkdir -p source/search
  Creating new page: source/search/index.markdown
  $ echo "{% include custom/lunr_search/entries.html %}" >> source/search/index.markdown
  $ cp .themes/octopress-lunr-theme/plugins/octopress_lunr_theme.rb plugins/octopress_lunr_theme.rb
  $ sed -i "/^simple_search:/c\simple_search: #" _config.yml
  $ sed -i "s/^default_asides: \[/default_asides: \[custom\/asides\/lunr_search.html, /" _config.yml
  $ bundle exec rake generate
  $ git add -A
  $ git commit -m "Installation de lunr-js-search"
```
{% endraw %}

Voilà, il ne vous reste qu'à faire `un bundle exec rake preview` pour voir que tout fonctionne. Maintenant il est temps de retourner sur votre branche principale et de voir ce qui a changé dans le fichier **source/_includes/custom/head.html**.
```bash
  $ git checkout master
  $ git merge octopress-lunr-theme
```

## Comment le plugin fonctionne-t-il ?
Il est composé d'un dossier **source/custom/lunr_search/** qui contient les fichiers suivants :
```bash ls source/custom/lunr_search/
  entries.html
  search_results_template.html
```

### Le plugin en lui-même.
Il s'agit du fichier [octopress_lunr_theme.rb](https://github.com/alain-andre/octopress-lunr-theme/blob/master/plugins/octopress_lunr_theme.rb) du dossier `_plugins`. Ce script est à 99% le même que celui de [jekyll-lunr-js-search](https://github.com/slashdotdash/jekyll-lunr-js-search).

Le plugin ruby va indexer **à chaque generation** tout le contenu du site dans un fichier `source/search.json` qui serra basculé par Jekyll dans le dossier public sur la fin de la generation.

### Maintenant on va passer au plugin jquery.
[jquery.lunr.search.js](https://github.com/alain-andre/octopress-lunr-theme/tree/master/source/javascripts/libs/jquery.lunr.search.js) se trouve dans `source/javascripts/libs`.

Ce plugin est composé des scripts suivants : jQuery (déjà présent dans octopress), [lunr.js](http://lunrjs.com/), [Mustache.js](https://github.com/janl/mustache.js), [date.format.js](http://blog.stevenlevithan.com/archives/date-time-format), [URI.js](http://medialize.github.com/URI.js/).

### La modification du fichier head.html
Le fichier [source/_includes/custom/head.html](https://github.com/alain-andre/octopress-lunr-theme/blob/master/source/_includes/custom/head.html) est ce qui va charger le plugin javascript.
```html source/_includes/custom/head.html
<script src="{{ root_url }}/javascripts/libs/search.min.js" type="text/javascript" charset="utf-8"></script>

<script type="text/javascript">
  $(function() {
    $('#search-query').lunrSearch({
      indexUrl: '/search.json', // URL of the `search.json` index data for your site
      results: '#search-results', // jQuery selector for the search results container
      entries: '.entries', // jQuery selector for the element to contain the results list, must be a child of the results element above.
      template: '#search-results-template' // jQuery selector for the Mustache.js template
    });
  });
</script>

{% include custom/lunr_search/search_results_template.html %}
```

### La page de recherche en elle-même.
Il est temps d'ajouter le [formulaire](https://github.com/alain-andre/octopress-lunr-theme/blob/master/source/_includes/custom/asides/lunr_search.html) qui va nous permettre de faire nos recherches.
```html source/_includes/custom/aside/lunr_search.html
<section id="search">
  <form action="/search" method="get">
    <input type="text" id="search-query" name="q" placeholder="Search" autocomplete="off">
  </form>
</section>
```

### La mise place.
Il est necessaire d'ajouter **custom/asides/lunr_search.html** dans la partie asides
```ruby _config
default_asides: [custom/asides/lunr_search.html, ...]
```

Il faut ensuite ajouter le bloc qui recevra les résultats.

Pour cela, je vais créer une page search.
```bash
  $ bundle exec rake new_page['search']
  mkdir -p source/search
  Creating new page: source/search/index.markdown
  $ echo "{% include custom/lunr-search/entries.html %}" >> source/search/index.markdown
```

Et maintenant on peut configurer le plugin :

Il faut ajouter dans le Gemfile les gems json et nokogiri.
```bash Ajout des gems necessaires
  $ sed -i "/^end/c\  gem 'json'\n  gem 'nokogiri'\nend" Gemfile
```
Et finir de donner les options du plugin dans `_config`.
```bash
$ sed -i "/^simple_search:/c\simple_search: #" _config.yml
```

Il est aussi possible de modifier les options du plugin dans une partie lunr_search du fichier _config :
```ruby _config
# lunr
lunr_search:
  excludes: [rss.xml, atom.xml]
  min_length: 3
```

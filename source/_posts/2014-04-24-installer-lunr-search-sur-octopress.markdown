---
layout: post
title: "Installer lunr-search sur Octopress"
date: 2014-04-24 05:42:21 +0200
comments: true
categories: 
  - Ruby on Rails
---

Ce post s'inspire de [jekyll-lunr-js-search](https://github.com/slashdotdash/jekyll-lunr-js-search) et de [octopress-lunr-js-search](https://github.com/yortz/octopress-lunr-js-search/blob/master/plugins/search_generator.rb). Je ne trouvais pas ces plugins facile à installer/utiliser alors j'ai décidé de créer un theme basé sur [boldandblue](https://github.com/johnkeith/boldandblue) qui l'intégre directement. 

## Installation du plugin
Cette partie explique comment installer rapidement le plugin ainsi que les structures dont il a besoin.
´´´bash Installer le plugin et ses dépendances
  $ sed -i "/^end/c\  gem 'json'\n  gem 'nokogiri'\nend" Gemfile
  $ bundle install
  $ git clone .themes/octopress-lunr-theme
  $ bundle exec rake install # Installe le thème de base (si vous n'en avez pas déjà installé un)
  $ bundle exec rake install["octopress-lunr-theme"]
  $ bundle exec rake new_page['search']
  mkdir -p source/search
  Creating new page: source/search/index.markdown
  $ echo "{% include custom/lunr_search/entries.html %}" >> source/search/index.markdown
  $ sed -i "/^simple_search:/c\simple_search: #" _config.yml
  $ sed -i "s/^default_asides: \[/default_asides: \[custom\/asides\/lunr_search.html, /" _config.yml
´´´


## Intégration du plugin dans votre theme.
Tout d'abord, je vais charger le dossier custom/lunr_search/ qui contient les fichiers suivants :
´´´bash ls custom/lunr_search/
  entries.html
  search_results_template.html
´´´

Puis, on va placer ´octopress_lunr_theme.rb´ dans le dossier ´_plugins´. Ce script est à 99% le même que celui de **jekyll-lunr-js-search** 

Le plugin ruby va indexer tout le contenu du site dans un ´search.json´.

Maintenant on va passer au plugin jquery. On place ´jquery.lunr.search.js´ dans ´source/javascripts/libs´.
Ce plugin a besoin des dépendances suivantes :
  - jQuery (déjà présent)
  - [lunr.js](http://lunrjs.com/)
  - [Mustache.js](https://github.com/janl/mustache.js)
  - [date.format.js](http://blog.stevenlevithan.com/archives/date-time-format)
  - [URI.js](http://medialize.github.com/URI.js/)

Si vous préférez, un build de ces dépendances est disponible ici : [search.min.js](https://github.com/slashdotdash/jekyll-lunr-js-search/blob/master/build/search.min.js). 

Maintenant, il faut modifier le fichier source/_includes/custom/head.html
´´´html source/_includes/custom/head.html
<script src="{{ root_url }}/javascripts/libs/search.min.js" type="text/javascript" charset="utf-8"></script>
{% include custom/lunr-search/search-results-template.html %}
´´´

Il est temps d'ajouter le formulaire qui va nous permettre de faire nos recherches.
´´´html source/_includes/custom/aside/lunr_search.html
<section id="search">
  <form action="/search" method="get">
    <input type="text" id="search-query" name="q" placeholder="Search" autocomplete="off">
  </form>
</section>
´´´

Et on ajoute dan _config notre nouveau plugin:
´´´ruby _config
default_asides: [asides/recent_posts.html, ..., custom/asides/lunr_search.html]
´´´

Il faut maintenant ajouter le bloc qui recevra les résultats. 

Pour cela, je vais créer une page search.
´´´bash 
  $ bundle exec rake new_page['search']
  mkdir -p source/search
  Creating new page: source/search/index.markdown
´´´

La page search contiendra le bloc suivant :
´´´html source/search/index.markdown
{% include custom/lunr-search/entries.html %}
´´´

Et maintenant il faut configurer et lancer le plugin :
´´´javascript source/_includes/custom/head.html
<script type="text/javascript">
  $(function() {
    $('#search-query').lunrSearch({
      indexUrl: '/search.json',             // URL of the `search.json` index data for your site
      results:  '#search-results',          // jQuery selector for the search results container
      entries:  '.entries',                 // jQuery selector for the element to contain the results list, must be a child of the results element above.
      template: '#search_results_template'  // jQuery selector for the Mustache.js template
    });
  });
</script>
{% include custom/lunr_search/search_results_template.html %}
´´´

Il faut ajouter dans le Gemfile les gems json et nokogiri, et finir de donner les options du plugin dans ´_config´.
´´´ruby _config
simple_search: # Tout d'abord, je retire 
.
.
# lunr
lunr_search:
  excludes: [rss.xml, atom.xml]
  min_length: 3
´´´

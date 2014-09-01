La gestion des formulaires sous rails est tout simplement magique, tout le monde connait bien le `simple_form` ou ses dérivés qui nous permettent d'afficher ou de modifier l'un de nos models. Mais qu'en est-il lorsque nous avons une liaison avec un autre model ? 
Dans cet article, nous allons voir comment créer en un seul formulaire deux models. Nous allons pour faire original nous ateler à la création d'un `Project` qui `has_many` `Tasks`.

## Déclarations de nos models
```ruby app/models/project.rb
class Project < ActiveRecord::Base
  has_many :tasks
  accepts_nested_attributes_for :tasks, :reject_if => :all_blank, :allow_destroy => true
end
```

```ruby app/models/task.rb
class Task < ActiveRecord::Base
end
```

## Notre controller
```ruby projects_controller.rb
def new
  @project = Project.new
  @task = Task.new
end
```


### Sources
http://archive.railsforum.com/viewtopic.php?id=717
https://github.com/nathanvda/cocoon/wiki/A-guide-on-doing-nested-model-forms

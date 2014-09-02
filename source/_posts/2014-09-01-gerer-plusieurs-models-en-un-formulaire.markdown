La gestion des formulaires sous rails est tout simplement magique. Mais qu'en est-il lorsque nous avons une liaison avec un autre modèle ?

Dans cet article, nous allons voir comment créer en un seul formulaire deux modèles. Nous allons nous atteler à la création d'un `Project` qui `has_many` `Tasks`.

## Déclarations de nos modèles et accès aux attributs
Afin d'éviter d'avoir à tenir compte dans notre contrôleur *ProjectsController* des sous-modèles (Tasks) contenus dans notre modèle principal (Project), nous allons utiliser les [NestedAttributes](http://api.rubyonrails.org/classes/ActiveRecord/NestedAttributes/ClassMethods.html) d'ActiveRecord.

```ruby app/models/project.rb
class Project < ActiveRecord::Base
  has_many :tasks
  accepts_nested_attributes_for :tasks, :reject_if => :all_blank, :allow_destroy => true
end
```

```ruby app/models/task.rb
class Task < ActiveRecord::Base
  belongs_to :project
end
```

Ceci nous permet de laisser notre contrôleur de Project sans vraiment écrire comment seront enregistrées les Tasks. Les paramètres `reject_if` et `allow_destroy` vont nous garantir qu'une tâche ne serra pas créée si elle est vide et pourra être supprimée par le même biais qu'elle a été créée. si nous n'utilisions pas les NestedAttributes, l'action `create` devrait au moins avoir une ligne comme celle-ci pour créer une tâche.
```ruby app/controllers/projects_controller.rb
def create
  ...
  @task = @project.tasks.build(params[:task])
  ...
end
```

Mais nous allons en fait simplement ajouter les attributs dans la whitelist.
```ruby app/controllers/projects_controller.rb
def create
  @project = Project.new(project_params)
 
  respond_to do |format|
    if @project.save
      format.html { redirect_to(@project, :notice => 'Project was successfully created.') }
      format.xml { render :xml => @project, :status => :created, :location => @project }
    else
      format.html { render :action => "new" }
      format.xml { render :xml => @project.errors, :status => :unprocessable_entity }
    end
  end
end

def project_params
  params.require(:project).permit(tasks_attributes: [project_attributes:[]])
end
```

## Les vues
Une vue simple mais qui explique bien comment tout ceci fonctionne de base serrait :
```ruby app/views/projects/_form.html.haml
  = form_for @project do |p| %>
    = p.fields_for :tasks do |t|
```

Mais pour ajouter ou supprimer des Tasks du Project affiché, il nous faudrait faire bien plus. Heureusement, nous avons une Gem qui va grandement nous aider : [cocoon](https://github.com/nathanvda/cocoon). Attention, Cocoon fonctionne avec JQuery.
 

## Les vues
Cocoon nous livre deux méthodes `link_to_add_association` et `link_to_remove_association` qui permettent d'ajouter et de supprimer un sous-modèle. Pour fonctionner, cette Gem a besoin d'un Partial nommé `_[sousModel]_fields.html.haml` pour afficher les sous-modèles.

Ce qui donne dans notre cas les Partials suivants. Je vous réfère à la documentation pour connaître les détails des paramètres que l'on peut passer aux méthodes [link_to_add_association](https://github.com/nathanvda/cocoon/#link_to_add_association) et [link_to_remove_association](https://github.com/nathanvda/cocoon/#link_to_remove_association).

```haml app/views/projects/_form.html.haml
= simple_form_for @project do |p|
  = p.input :name
  %h3 Tâches
  = p.simple_fields_for :tasks do |task|
    = render 'task_fields', :t => task
    = link_to_add_association 'add task', p, :tasks
    = p.submit
```

```haml app/views/projects/_task_fields.html.haml
  t.input :name
  = link_to_remove_association "remove task", t
```

## Lier un projet à une personne déjà existante ?
Imaginons que vos projets aient un utilisateur référent (un Owner). Nous devrions pouvoir sélectionner lors du `link_to_add_association` une personne déjà existante. Et bien c'est possible comme ceci :

```ruby
    = f.association :owner, :collection => Person.all(:order => 'name'), :prompt => 'Choose an existing owner'
  = link_to_add_association 'add a new person as owner', f, :owner
```

## Sources
[railsforum](http://archive.railsforum.com/viewtopic.php?id=717)
[cocoon](https://github.com/nathanvda/cocoon/)

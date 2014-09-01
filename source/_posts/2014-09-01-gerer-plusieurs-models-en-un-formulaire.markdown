La gestion des formulaires sous rails est tout simplement magique. Mais qu'en est-il lorsque nous avons une liaison avec un autre model ? 

Dans cet article, nous allons voir comment créer en un seul formulaire deux models. Nous allons nous ateler à la création d'un `Project` qui `has_many` `Tasks`.

## Déclarations de nos models
Afin d'éviter d'avoir à tenir compte dans notre controller *ProjectsController* des sous-models (Tasks) contenus dans notre model principal (Project), nous allons utiliser les [NestedAttributes](http://api.rubyonrails.org/classes/ActiveRecord/NestedAttributes/ClassMethods.html) d'ActiveRecord.

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

Ceci nous permet de laisser notre controller de Project tel qu'il est plutôt que de le modifier. Par exemple l'action `create` devrait ressembler à ceci si nous n'utilisions pas les NestedAttributes.
```ruby app/controllers/projects_controller.rb
def create
  @project = Project.new(params[:project])
  @task = @project.tasks.build(params[:task])
  
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
```

Mais nous allons la laisser tel que 
```ruby app/controllers/projects_controller.rb
def create
  @project = Project.new(params[:project])
  
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
```

## cocoon
Plutôt que de réécrire notre controller pour 

L'action **new** va définir un Project et une task qui vont être utilisés dans notre vue. L'action **create** va créer un Projet et une tâche. Pour la création de la première tâche, nous utilisons le lien créé par ActiveRecord via *@project.tasks* pour y stocker les données disponibles dans params[:task].

```ruby app/controllers/projects_controller.rb
def new
  @project = Project.new
  @task = Task.new
end


```

## Les vues
```haml app/views/projects/new.html.haml
  %h1 
```

```haml app/views/projects/_form.html.haml
= simple_form_for @project do |f|
  = f.input :name
  %h3 Tâches
  = f.simple_fields_for :tasks do |task|
    = render 'task_fields', :f => task
```

```haml app/views/projects/_task_fields.html.haml

```

### Sources
http://archive.railsforum.com/viewtopic.php?id=717
https://github.com/nathanvda/cocoon/wiki/A-guide-on-doing-nested-model-forms

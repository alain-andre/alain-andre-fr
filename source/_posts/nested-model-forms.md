A guide on doing nested model forms 

One recurring problem when doing Ruby on Rails is nested model forms.

    Nested Model Form: a single form that contains multiple, nested models.

For example a project with its tasks. A nested model form will allow you to create or edit, in 1 form, the project and each of its tasks.

With the use of cocoon, this document will describe how to create a nested model form for some (all) of the possible relations.

All this examples start from the project, so we will just start with a dummy project as follows:

rails g scaffold Project name:string

That's all we need to get started. In these examples i am using simple_form and slim.
The simple :has_many

The simple :has_many is the most occuring relation (unfortunately no scientific data to back that up). The simple :has_many is the typical relation where the child cannot exist without the parent, and where each child can have only one parent. A typical example is the project and his tasks. Each task is unique, each task only exists because of the project. If the project is gone, so are the tasks.

First the models:

class Project < ActiveRecord::Base
  has_many :tasks
  accepts_nested_attributes_for :tasks, :reject_if => :all_blank, :allow_destroy => true
end

class Task < ActiveRecord::Base
end

And then the views. Inside the project _form.html.slim we add

#tasks
  = f.simple_fields_for :tasks do |task|
    = render 'task_fields', :f => task
  .links
    = link_to_add_association 'add task', f, :tasks

Create a partial called projects/_task_fields.html.slim which looks like:

.nested-fields
  = f.input :name
  = f.input :description
  = f.input :done, :as => :boolean
  = link_to_remove_association "remove task", f

The nested :has_many

Actually this is a simple extension of the previous example. For instance, a project with tasks, and each tasks has sub-tasks.

Our models:

class Task < ActiveRecord::Base
  has_many :sub_tasks
  accepts_nested_attributes_for :sub_tasks,  :reject_if => :all_blank, :allow_destroy => true
end

class SubTask < ActiveRecord::Base
end

Edit the projects_task_fields.html.slim :

.nested-fields
  = f.input :name
  = f.input :description
  = f.input :done, :as => :boolean
  .sub_tasks
    = f.simple_fields_for :sub_tasks, :wrapper => 'inline' do |sub_task|
      = render 'sub_task_fields', :f => sub_task
    .links
      = link_to_add_association 'add sub-task', f, :sub_tasks, :render_options => { :wrapper => 'inline' }
  = link_to_remove_association "remove task", f

and add the view projects_sub_tasks.html.slim (which bears a lot of ressemblance to the previous tasks-partial):

.nested-fields
  = f.input :name
  = f.input :description
  = link_to_remove_association "remove sub-task", f

The look-up or create :belongs_to

A simple :belongs_to would mean that the parent would already exist, and a simple look-up (dropdown list) would suffice. If the parent is always unique, it can be solved in the same way as the :has_many. So, let's describe a solution for the case where we either select the item from a list, or create a new one.

Our project has an owner (which is a person):

class Project < ActiveRecord::Base
  belongs_to :owner, :class_name => 'Person'
  accepts_nested_attributes_for :owner, :reject_if => :all_blank
end

class Person < ActiveRecord::Base
end

inside the _projects/_form.html.slim we add the following:

#owner
  #owner_from_list
    = f.association :owner, :collection => Person.all(:order => 'name'), :prompt => 'Choose an existing owner'
  = link_to_add_association 'add a new person as owner', f, :owner

Here we use the built-in way from simple_form to select an association from a look-up list: f.association. But secondly, we place the link_to_add_association that will render the partial projects/_owner_fields.html.slim and create a new Person and link to him.

The partial itself is pretty straightforward:

.nested-fields
  = f.input :name
  = f.input :role
  = f.input :description
  = link_to_remove_association "remove owner", f

Now if you implement this, the form is functioning but not really user-friendly. What we want is that when clicking the add a new person as owner-link, that the drop-down box and the link itself disappear. When we choose to remove the to-be created owner, that the drop-down and add-link reappear. So we need to add a bit of extra javascript. Open up app\javascripts\projects.js and add:

$(document).ready(function() {
  $("#owner a.add_fields").
    data("association-insertion-position", 'before').
    data("association-insertion-node", 'this');

  $('#owner').bind('cocoon:after-insert',
     function() {
       $("#owner_from_list").hide();
       $("#owner a.add_fields").hide();
     });
  $('#owner').bind("cocoon:after-remove",
     function() {
       $("#owner_from_list").show();
       $("#owner a.add_fields").show();
     });
});

The first two lines control where the new partial (the new owner) will appear.

Upon insertion or removal, cocoon will trigger callbacks that are defined on the parent-container of the add-link. The last lines add those callbacks, and these will make sure that the links will and dropdownbox will be hidden or shown again.
The :has_many :through relation

A classic example of this relation are tags. Something has tags, those can be chosen from an exisiting list of tags, one or more, or one could create new tags. This relation is, again, an extension of the previous solution.

This solution is a bit more complicated, because there is a bit more javascript involved here.

The models:

class Project < ActiveRecord::Base
  has_many :project_tags
  has_many :tags, :through => :project_tags, :class_name => 'Tag'

  accepts_nested_attributes_for :tags
  accepts_nested_attributes_for :project_tags
end

class ProjectTag < ActiveRecord::Base
  belongs_to :tag
  belongs_to :project

  accepts_nested_attributes_for :tag, :reject_if => :all_blank
end

class Tag < ActiveRecord::Base
end

inside the projects/_form.html.slim add:

#tags
  = f.simple_fields_for :project_tags do |project_tag|
    = render 'project_tag_fields', :f => project_tag
  = link_to_add_association 'add a tag', f, :project_tags

Create a file projects/_project_tag_fields.html.slim containing:

.nested-fields.project-tag-fields
  #tag_from_list
    = f.association :tag, :collection => Tag.all(:order => 'name'), :prompt => 'Choose an existing tag'
  = link_to_add_association 'or create a new tag', f, :tag
  = link_to_remove_association "remove tag", f

And create another file projects/_tag_fields.html.slim:

.nested-fields
  = f.input :name, :hint => 'how should it be tagged'

When entering all this, it should be working, but it looks a bit confusing. The dropdown box does not disappear (as before). So we need the following bit of javascript to projects.js:

$("#tags a.add_fields").
  data("association-insertion-position", 'before').
  data("association-insertion-node", 'this');

$('#tags').on('cocoon:after-insert',
     function() {
         $(".project-tag-fields a.add_fields").
             data("association-insertion-position", 'before').
             data("association-insertion-node", 'this');
         $('.project-tag-fields').on('cocoon:after-insert',
              function() {
                $(this).children("#tag_from_list").remove();
                $(this).children("a.add_fields").hide();
              });
     });

While this javascript is doing almost the same as with the :belongs_to it is a bit more complicated, because the tag-partial is not yet inside in the page, so the neither is the parent-div. Once that is clear, it is actually quite obvious what it does. I hope. Upon insertion of a new (project)tag, it also adds the callbacks for the insertion and removal of a new tag.
Example Code

All these examples are demonstrated in a working project: cocoon-simple-form-demo.

Hope this helps.

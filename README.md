#Tools Crud
Based on a [lesson from TuringSchool](https://github.com/turingschool/lesson_plans/blob/master/ruby_02-web_applications_with_ruby/forms_and_route_helpers_in_rails.markdown).

## Goals

* Write full CRUD functionality for a Tool object in rails
  * Create
  * Read
  * Update
  * Destroy
* Construct forms using Rails form helpers
* Construct links using Rails link helpers

## Setup

```
$ rails new tools_crud
$ cd tools_crud
```

Consider adding gems for feedback or testing?
```ruby
  gem 'pry'
  gem 'better_errors'
  gem 'mrspec'
  gem 'capybara'
```

Are you "riding Ruby on Rails?"
```
$ rails s
```

Create the Tool Model (models/tool.rb)
```ruby
class Tool < ActiveRecord::Base
end
```

Generate a migration. This will be the only rails generate command you should be using.

```
$ rails g migration create_tools
```

Tools will have the attributes name, price and quantity. I'll add the first to the migration, you add the other two. Here is a list of the migration types: http://api.rubyonrails.org/classes/ActiveRecord/ConnectionAdapters/Table.html

[More on migrations](http://edgeguides.rubyonrails.org/active_record_migrations.html)
```ruby
class CreateTools < ActiveRecord::Migration
  def change
    create_table :tools do |t|
      t.string :name # this adds an attribute named "name", which is a string, it can store values like `"hammer"`
      # add an attribute named "price", which can store values like `15.99`
      # add an attribute named "quantity", which can store values like `10`
    end
  end
end
```

Set up your database.
```
$ rake db:migrate
```

Add a few tools from the console and practice querying using [Active Record](http://guides.rubyonrails.org/active_record_querying.html) methods.

```
$ rails c
Tool.create(name: "Rotary cutter", price: 100.00, quantity: 10)
Tool.create(name: "Hammer", price: 15.99, quantity: 10)
Tool.create(name: "Rake", price: 20.00, quantity: 10)
```

Use similar code in your seed file to create tools without having to use the rails console. Run the rake commands and check out your tools in console again.

```
$ rake db:drop
$ rake db:create
$ rake db:migrate
$ rake db:seed
```

## Routes

Add RESTful routes for Tools:
[Some RESTful info](https://github.com/CodePlatoon/curriculum/blob/master/phase2/rest.md)

```ruby
Rails.application.routes.draw do
  resources :tools
end
```

Run `$ rake routes` and look at the output. This is a roadmap of the seven actions we will need to have a full CRUD app.

```
    Prefix Verb   URI Pattern                    Controller#Action
     tools GET    /tools(.:format)               tools#index
           POST   /tools(.:format)               tools#create
  new_tool GET    /tools/new(.:format)           tools#new
 edit_tool GET    /tools/:id/edit(.:format)      tools#edit
      tool GET    /tools/:id(.:format)           tools#show
           PATCH  /tools/:id(.:format)           tools#update
           PUT    /tools/:id(.:format)           tools#update
           DELETE /tools/:id(.:format)           tools#destroy
```

We will be creating these routes as needed, so delete:
```ruby
resources :tools
```

Try creating only the index route, and check out your routes again.
 ```ruby
resources :tools, only: [:index]
```

From here on, we will be writing our routes manually to get a feel for the "resources magic." Write the index route yourself and check out your routes again.
```
get "/tools", to: "tools#index", as: "tools"
```

Set that to your root path:
```
root 'tools#index'
```

## Controller
Check out your app. Sounds like we need a controller for Tools?

```
$ touch app/controllers/tools_controller.rb
```

Add your class and inherit from ApplicationController:

```ruby
class ToolsController < ApplicationController

end
```

### Index
Are we missing an 'index' action for ToolsController? What does that mean?

Lets create an index method in the controller.

```
  def index

  end
```

####Create a view:

```
$ mkdir app/views/tools
$ touch app/views/tools/index.erb
```

Inside of that file:

```erb
<h1>Tools Index</h1>
```
I like explicitly descriptive headers on pages while setting up minimum functionality. I remove them when I have a functioning application.

####Add content to the view:
Use our controller to get some tools out of our database and store them in an instance variable so we can pass it to our view.
```
  def index
    @tools = Tool.all
  end
```

Harness the power of erb to display each of the tools in our view.

```erb
<% @tools.each do |tool| %>
  <div>
    <%= tool.name %>
    | Quantity: <%= tool.quantity %>
    | Price: <%= tool.price %>
  </div>
<% end %>
```

## Route Helpers

We can use the prefixes from our `rake routes` output to generate routes:

```
    Prefix Verb   URI Pattern                    Controller#Action
     tools GET    /tools(.:format)               tools#index
           POST   /tools(.:format)               tools#create
  new_tool GET    /tools/new(.:format)           tools#new
 edit_tool GET    /tools/:id/edit(.:format)      tools#edit
      tool GET    /tools/:id(.:format)           tools#show
           PATCH  /tools/:id(.:format)           tools#update
           PUT    /tools/:id(.:format)           tools#update
           DELETE /tools/:id(.:format)           tools#destroy
```

If we want to go to `/tools`, we can create a link to `tools_path`. If we want to go to `/tools/new`, we can create a link to `new_tool_path`. Notice that we just add on `_path` to the prefix.

Rails also has a link helper that looks like this:

```erb
<%= link_to "Tool Index", tools_path %>
```

Which will generate the equivalent of:

```erb
<a href="/tools">Tool Index</a>
```

We can add a link to view individual tools with this:

Inside of the view:

```erb
<% @tools.each do |tool| %>
  <div>
    <%= link_to tool.name, tool_path(tool) %>
    | Quantity: <%= tool.quantity %>
    | Price: <%= tool.price %>
  </div>
<% end %>
```

Undefined method 'tool_path'? Looks like we can't "Show" a tool until we create a route for it.

### Show

Add a route for Show. Check out how we did it for index as an example. Still stuck? Ask your neighbor/pair/instructor!

If its working, we should see a new error. Did you get one?

Do we need a 'show' action in our conroller?

```ruby
  def show

  end
```

New error? Missing template? Sounds like a railsy word for a view.

Let's create a view to show the individual tool: `$ touch app/views/tools/show.erb`.
```erb
<h1>Tool Show</h1>
```

####Add content to the view:
Use our controller to access the database again, this time we are looking for a specific tool.

```ruby
def show
  @tool = Tool.find(params[:id])
end
```

Wait, what the heck is params? It's a hash with info from our browser. I'll let these folks [explain a bit more](http://stackoverflow.com/questions/6885990/rails-params-explained)

Consider exploring it a bit wth pry/inspect.

Display that tool on its show page.

```erb
<h3><%= @tool.name %></h3>
<p>Quantity: <%= @tool.quantity %></p>
<p>Price: <%= @tool.name %></p>
```

### New

Ok, back to the index (`/tools`). How can we create a link to go to a form where we can enter a new tool? Let's put that link on our `index.html.erb` page. (Hint: to follow convention, use the method 'new_tool_path')

The link will rely on a new route, controller action, and view.

Since we are going to make and configure a new tool, lets pass our view a tool that hasn't been assigned any attribute values yet.

```
  @tool = Tool.new
```

Add a form to the view. You can write your own HTML form, or you can let rails do some of the work via its [form helpers](http://guides.rubyonrails.org/form_helpers.html)

```erb
<%= form_for(@tool) do |f| %>
  <%= f.label :name %>
  <%= f.text_field :name %>
  <%= f.label :price %>
  <%= f.text_field :price %>
  <%= f.label :quantity %>
  <%= f.text_field :quantity %>
  <%= f.submit %>
<% end %>
```

Fill out the form and submit. Looks like we need more routes again? What http verb is it looking for?

### Create
Go ahead and make the create route/action. We won't be needing a view since this is a post route.

```
def create
  tool = Tool.create(#lets go digging in params)
  redirect_to #some_path?
end
```
Is the new tool in your database?

### Edit

Let's go to the show view and add a link to edit. What will this link look like?

Build what you need until you can get to an edit view.

Add a view for edit which contains a `form_for(@tool)`. This form looks EXACTLY THE SAME as our `new.erb`.
(Stretch Goal: DRY! Implement [partials](http://guides.rubyonrails.org/layouts_and_rendering.html#using-partials))

Looks like our view wants a Tool. Do we want any tool, a new tool, or a specific tool. If it already exists, where is it and how can we get it?

If we submit that form will we get an error? What might it be? Lets fix it.

### Update
  Do we need a route? What is our HTTP verb?
  Do we need a controller action?
  Do we need a view?

```ruby
  def update
    tool = #probably a specific tool
    tool.update(#some things)
    redirect_to #somewhere
  end
```

Did we go to somewhere? Is the tool updated in our database?

### Destroy

Add a link on the tool show view to destroy the tool:

What type of request are we making? What type of request do we want to make?

A bit of syntax:
```erb
<%= link_to "#words", #some_path, method: :#HTTPverb %>
```

Add a `destroy` action in your controller:

```ruby
  def destroy
    #find a Tool?
    #destroy a Tool?
    #go somewhere?
  end
```

Is the tool on your home page? In your database?

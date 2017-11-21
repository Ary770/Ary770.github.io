---
layout: post
title:      "My Projects App (MVC Sinatra Application) "
date:       2017-11-21 02:15:23 +0000
permalink:  my_projects_app_mvc_sinatra_application
---


The Sinatra Portfolio Project was a very exciting journey. I believe it's the first time where everything you've learned up to this point really starts to come together. Very much like different pieces of a puzzle coming together when you start to see the big picture you've been building.

My Projects App is essentially a web app to keep track of all your projects in an organized fashion. It uses Active Record, sessions, user functionality, and password_security. 

The app has four models: User, Project, Idea, and Category. 
A user has many projects and has many categories through projects. 
Projects belong to a User, belong to a Category and has many ideas. Ideas belong to a Project, and a Category has many projects and belongs to a User. 

So I can do something like;

```

categorry = current_user.categories.find_or_create_by(name: params[:category])

project = current_user.projects.build(params[:project])

project.category = category

```

In the first line of code, I'm using active record's method #find_or_create_by to...you guessed it, find or create an instance of Category. Since I'm chaining the method back to current_user, this instance will be associated only with that specific user.

On the second line, I'm creating a new project and at the same time associating it with the current user.

And finally, I'm associating the project with the category I found or created on the first line.

I can later display all that User's categories in the following way:

In the CategoryController:

```

get '/categories' do
    if logged_in?
      @categories = current_user.categories
      erb :'categories/categories'
    else
      redirect to '/login'
    end
  end

```

In categories.erb:

```

<h1>Categories</h1>

  <% @categories.each do |category| %>
    <h2><a href="/categories/<%=category.id%>"><%=category.name%></a></h2>
  <%end%>
    
```

It was a bit complex to think about this relationship, but the more I thought about the practicality of how I wanted my app to work the more simple it was to map the models. 

Thanks to the beauty and "magic" of Active Record, I was able to move quite fast. I already knew how I wanted my app to behave, so I went ahead and started creating my routes in the corresponding controllers. I created a separate controller for Users, Projects, and Categories. It makes it much more organized to separate the routes into different controllers, plus it allows room for expansion in the future. 

For the Views (aside from using bootstrap just for fun :)), I incorporated some additional ruby code to output data in each page dynamically. 

Such as in layout.erb :

```
<% if logged_in? %>

          <ul class="nav nav-pills">
            <li role="presentation" class="active"><a href="/projects">My Projects</a></li>
            <li role="presentation"><a href="/projects/new">Create New Project</a></li>
            <li role="presentation"><a href="/categories">Categories</a></li>
            <li role="presentation"><a href="/logout">Log Out</a></li>
          </ul>

        <% else %>

          <ul class="nav nav-pills">
            <li role="presentation" class="active"><a href="/">Home</a></li>
            <li role="presentation"><a href="/login">Log In</a>
            <li role="presentation"><a href="/signup">Sign Up</a>
          </ul>

        <% end %>
``` 

 if "logged_in?", display the following as menu bar, else, display login and sign up options, etc...

or in show_project.erb :

```
<% if flash.has?(:message) %>
  <div class="alert alert-success" role="alert"><%=flash[:message]%></div>
<% end %>

<% if @project.category %>
  <h2><a href="/categories/<%=@project.category.id%>"><%= @project.category.name %></a></h2>
<% end %>

<h1><%=@project.name%></h1>

  <% if @project.ideas != [] %>
    <h3>Ideas:</h3>
    <% @project.ideas.each do |i| %>
    <ul>
      <li><%= i.text %></li>
    </ul>
    <% end %>
  <% end %>

```

In conclusion, I felt very well prepared when it came to developing the Sinatra app (thanks to the awesome curriculum). Of course, I can always improve on it and build more functionality, but I'm quite happy with it. 

Coding is fun :)







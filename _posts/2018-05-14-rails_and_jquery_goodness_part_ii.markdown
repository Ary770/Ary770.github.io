---
layout: post
title:      "Rails and jQuery Goodness Part II"
date:       2018-05-14 18:45:41 +0000
permalink:  rails_and_jquery_goodness_part_ii
---

*# items/show.html.erb*

```
<h1><%= @item.name %></h1>

<div class= "row">
  <div class="col-sm-6 col-md-4 ">
     <%= render 'image', locals: @item %>
    <div class="card">
      <%= render 'item', locals: @item %>
    </div>
  </div>
</div>

<br>

<% if @item.comments != [] %>
  <h4><a href="#" id="js-comments" data-id="<%= @item.id %>">Show Comments</a></h4>
<% end %>

<div id="js-view-comments">
</div>
```

So this is a Rails show view for a single item. It displays the item's name, renders a couple of partials that contain the item's image, descriptions, inventory, etc. 

The part I want to focus on is this:

```
  <h4><a href="#" id="js-comments" data-id="<%= @item.id %>">Show Comments</a></h4>
```

Here we have a link, that doesn't seem to do anything. Notice the empty 'href' attribute. But this link is a 'hook' that when clicked triggers a JavaScript function with some jQuery and AJAX calls. Let's take a look:

```
<script type="text/javascript" charset="utf-8">
  $( document ).ready(function () {
    $('#js-comments').click(function(e){
      e.preventDefault();
      var id = $(this).data("id")
      $.getJSON(`/items/${id}`, function(responseData) {
        var comments = responseData['comments']
        var commentsList = "";
        comments.forEach(function (commentData) {
          var comment = new Comment(commentData);
          commentsList += comment.buildHtml();
        })
        $('#js-view-comments').html(commentsList);
      })
    });
  });
</script>
```

Did you notice the 'id' attribute on the 'a' tag? With jQuery, we can 'tap into' the link, so to speak, in this case through the id, and attach an event listener that, when clicked, triggers a JS function to perform a particular task. Here we are making a 'GET' request to the server API to get all the comments nested under this single item and then display them on the page. 

Let's break it down:

First, we'll wrap the function with $( document ).ready(), which merely detects the 'readiness' of the page and allows our code to run only once the page is fully loaded. 

Then, we attach an event listener to the link with the id of 'js-comments' that when clicked triggers the callback function:

```
$('#js-comments').click(function(event){
//Some code...
```

Once the element is clicked we first need to prevent the default action of the link tag for our code to be able to run properly: 

```
event.preventDefault();
```


The next step would be to perform a 'GET' request via AJAX to the API to see all the comments on this particular item. But how do we access the `@item.id` attribute inside our function? One way of doing this is by passing a data attribute to the link with the value of the current item id and then retrieving that from within our function. Let's see that link again:  

```
<a href="#" id="js-comments" data-id="<%= @item.id %>">Show Comments</a>
```

Back in our function, we'll save the id to a variable so we can use it in our AJAX call.

```
var id = $(this).data("id");
```

`$(this)` refers to whatever element we attached our event listener. In this case the link element. We then access the data attribute and retrieve the id of the item.

Now we're ready to make the call:

```
  $.getJSON(`/items/${id}`, function(itemData) {
  });
```

Now, you should know there are several ways of doing this. In this case, I chose to use $.getJSON. This method, besides from specifying that we're making a 'GET' request (as opposed to a POST or PATCH) it also tells the server to respond with a JSON format. It's sort of a shortcut for $.ajax() which is more of a 'lower level' method. $.get takes in two arguments, the URI we want to make the call to and a callback function to handle the response of the request. 


Once the request is made, and the server successfully responds with the data we wanted (assuming everything goes as planned) we can manipulate the data and append it to the page. Again, this can be done in many different ways but here are the steps I followed: 

1. Get all the comments nested under the item.

(Bear in mind that every API is different and we need to know beforehand how to navigate through the JSON object.)

`var comments = responseData['comments'];`

2. Create a variable pointing to an empty string to later populate with the data we want to display on the page.

`var commentsList = ' ';`

3. Iterate through all the comments and create a new JS Model Object for each comment.

`var comment = new Comment(commentData);`

For this step, I created an ES6 Class to build the comment object.

```

class Comment {
  constructor(attributes) {
    this.id = attributes.id;
    this.text = attributes.text;
    this.created_at = attributes.created_at;
    this.user = attributes.user;
  }

  userEmail(){
    return this.user.email;
  }

  buildHtml(){
    return `<div class="panel panel-default"><div class="panel-heading"><h6 class="panel-title">${this.user.email}<span class="text-muted" style="float: right"><small>${this.date()}</small></span></h6></div><div class="panel-body"><p>${this.text}</p></div></div>`;
  }

  date(){
    let date = new Date(this.created_at);
    return date.toLocaleDateString();
  }
}
```

As you can see, I added some prototype methods as well to be able to access and format the object data easily.

Which takes us to the next step:

4. Create an HTML string formatting the comment data and add each string to our commentsList variable.

`commentsList += comment.buildHtml();`

`buildHtml() ` introspects on the comment instance and builds an HTML string using template literals.

5. Now that we have all of our adequately formatted comments we can alter the HTML on the page and add our comments!

`$('#js-view-comments').html(commentsList);`

And that's it. jQuery/AJAX awesomeness. A bit complicated at first but once you get the hang of it the possibilities become endless! 



















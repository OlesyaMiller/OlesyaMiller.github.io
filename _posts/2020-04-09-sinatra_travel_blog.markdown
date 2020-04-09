---
layout: post
title:      "Sinatra Travel Blog"
date:       2020-04-09 15:59:36 -0400
permalink:  sinatra_travel_blog
---


I am very excited to present you my Sinatra Travel Blog! I’ve been wanting to build a blog for a while, and now that I have enough knowledge I’m very happy to finally have been able to execute my idea!

I use MVC pattern to divide my app logic. That is also called ‘separation of concerns’.

I have an app folder that contains a models folder, a controllers folder and a views folder.. 

I have three models in my app: User, BlogPost and Comment.

My User has many blog_posts and many comments; my BlogPost belongs to User and has many comments; my Comment belongs to User and to BlogPost. I was able to establish these relationships by inheriting from ActiveRecord::Base. Now that my models know about each other I can start building my app. 

In my controller folder I have four controllers (one for each model and ApplicationController). In ApplicationController I enable sessions and set app/views directory to just ‘views’. For that functionality to be shared with other controllers my other controllers simply inherit from ApplicationController.  

By enabling sessions my app can store the user’s information when they are logged in. I do that by assigning a new key-value pair to the session hash - session[:user_id] = user.id in the post ‘/signup’ route where a user is created in my Users controller. Once the user is logged in, they can see the navigation bar which has following links: ‘My Blog Posts’, ‘New Blog Post’, ‘All Blog Posts’, ‘All Users’ and ‘Sign Out’.  

I use restful routes to provide mapping between HTTP verbs (get, post, put, delete and patch) and CRUD actions (create, read, update and delete).

I use two helper class methods - current_user and is_logged_in? – for a more efficient workflow

```
def self.current_user(session)
      User.find_by(id: session[:user_id])
end
  
def self.is_logged_in?(session)
      !!session[:user_id] 
end
```

The user is able to edit or delete only blogposts and comments that belong to them. I enable that by checking if the comment or the blogpost that is being displayed on the current page belongs to the current user, and if it does the ‘edit’ and ‘delete’ buttons are displayed

```
<% if Helpers.current_user(session) == @blogpost.user %>
    <button><a href="/blogposts/<%= @blogpost.id %>/edit">Edit</a></button>
    <form action="/blogposts/<%= @blogpost.id %>/delete" method="post">
        <input type="hidden" name="_method" value="delete">
        <input type="submit" value="Delete">
    </form>
<% end %>
```

One of the challenges I came across building my app was being able to display only the comments under each blog post that belong to that blog post. 

Here is how I handled that:

In my CreateComments migration I added a blog_post_id  column to my comments table. In my comments form that is dispIayed under each blog post I created a form with an input field with the name “blog_post_id”  and the type “hidden” and assigned the value attribute a default value of the current blogpost’s id

```
<form action="/comments" method="post" id="comment_form">
        <input type="hidden" name="blog_post_id" value="<%= @blogpost.id %>">
        <div class="form-group">
            <textarea class="form-control" form="comment_form" name="content">Enter Your Comment Here</textarea>
        </div>    
        <button type="submit" class="btn btn-info">Submit</button>
</form>
```

That way I was able to store the blog post’s id in the params hash and manipulate it in my post ‘/comments’ controller action

```
post '/comments' do    
        comment = Comment.new(content: params["content"])
        blogpost = BlogPost.find_by_id(params["blog_post_id"])
        if comment && blogpost 
            user = Helpers.current_user(session)
            comment.user = user 
            comment.blog_post = blogpost 
            comment.save 
            redirect to "/blogposts/#{blogpost.id}"
        else
            redirect to '/'
        end
end
```

To display the blog post’s comments I simply iterate over @blogpost.comments array in my blogposts/show.erb file using erb tags

```
<% @blogpost.comments.each do |comment| %>
    <div class="container">
        <div class="border rounded p-3 content">
            <%= comment.content %><br>
            <small><strong>Created: <%= comment.created_at %></strong></small><br>
            <% if comment.updated_at.to_i != comment.created_at.to_i %>
                <small><strong>Updated: <%= comment.updated_at %></strong></small>
            <% end %><br>
            <small><strong>Posted by: <a href="/users/<%= comment.user.id %>"><%= comment.user.username %></a></strong></small>
        </div>    
	. . . . . .

```


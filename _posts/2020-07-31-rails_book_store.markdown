---
layout: post
title:      "Rails Book Store"
date:       2020-07-31 15:00:09 +0000
permalink:  rails_book_store
---


I'm happy to present you my first rails project - an online book store! 

I have four models in my app - Book, User, Purchase and Genre. Book has many purchases and many users through purchases and belongs to User.  Genre has many books, User has many purchases and many books through purchases. Purchase belongs to User and to Book. 

My User model has has_secure_password method that works together with BCrypt gem that is responsible for salting user's password. It also automatically adds password_confirmation attribute to my User model. 

In my app I use all four actions of the CRUD life cycle. 

A user can create book listing and can purchase books. When a user clicks "Purchase This Book" link on the 'users/:user_id/purchases/new' page, the price of the book gets substracted from the user's credit (if the user has enough credit) and the user gets a message saying how much credit they have remaining. Otherwise the user sees a message saying that they don't have enought credit.

```
def purchase_book
        if self.book.price > self.user.credit 
            "You do not have enough credit to purchase this book"
        else
            self.user.credit -= self.book.price 
            self.user.save 
            "Concratulations on purchasing this book! Your remaining credit is #{self.user.credit}"
        end        
end
```

I enabled that functionality by calling #purchase_book method on an instance of a Purchase that returns a string, saving it in a variable and passing that variable as a flash message in my create action in my Purchases controller

```
def create
        purchase = Purchase.create(purchase_params)
        message = purchase.purchase_book 
        redirect_to user_books_path(purchase.user), flash: { message: message }
end
```

I have one scope method in my app to be able to filter books by title

```
scope :search, -> (query){where('title LIKE ?', "%#{query}%")}
```

I have two nested routes in my app - 'users/:user_id/books' and 'users/:user_id/purchases/new'. I handle functionality of these routes in the actions of regular routes by checking if the params hash has a user_id key (only nested routes can have that key). My 'users/:user_id/purchases/new' page has a form that has a hidden user_id field to be able to associate the user with the purchase in the Purchases controller

```
def new
        @purchase = Purchase.new 
        @user = User.find(params[:user_id])
        @purchase.user = @user 
end
```

To make sure only logged in users can access my app pages I defined a before_action custom filter method #require_login in my Application controller.

```
def require_login
        if current_user 
            current_user
        else 
            redirect_to root_path
        end 
end
```

On my 'books/new' page my form has fields_for helper to let the user type in genre name for their book. I created it because my  Book model doesn't have a genre attribute, so it's a responsibily of the Genre model to assign a book a genre name

```
<%= f.fields_for :genre do |g| %>
        <%= g.label "Genre Name" %>
        <%= g.text_field :name %>
<% end %>
```

To get fields_for to work properly I had to include accepts_nested_attributes_for :genre macro in my Book model and pass genre_attributes: [:name] to my book_params

```
def book_params 
        params.require(:book).permit(
            :title, 
            :genre_id, 
            :description, 
            :number_of_pages, 
            :author, :price, 
            genre_attributes: [:name] 
            )
end
```

I have three partials in my project. One is for the form to create and edit the book, the second one is to create and edit a user. My third partial is responsible for storing error messages functionality for my Book and User models

```
<% if l_var.errors.any? %>
    <ul>
        <% l_var.errors.full_messages.each do |message| %>
            <li><%= message %></li>
        <% end %>    
    </ul>
<% end %> 
```

And as you might have guessed it has a local variable to be able to work for two different instances of two different models

```
<%= render partial: "application/error_message", locals: { l_var: @user } %>  #=> users/new.html.rb

<%= render partial: "application/error_message", locals: {l_var: @book} %>  #=> books/new.html.rb
```

I hope you enjoyed reading my blog post! Happy coding!





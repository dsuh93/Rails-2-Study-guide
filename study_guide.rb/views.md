# Views - AAO Study Guide bullet examples
## Demonstrate how to pass instance variables from controllers to HTML views for that controller

    - Concept: In the `Controller`, you must create `instance variables` in your action methods in order to utilize them in your `View` file for that specific action.

``` Ruby
#example from library_demo
    #while in app/controllers/books_controller.rb
    def index
  --> @books = Book.all
      render :index
    end
```
``` C#
    # while in app/views/books/new.html.erb -->
    <h1>all the books!</h1>
    <ul>
    # you can either do 
      <% @books.each do |book| %> 
        <%= render book %> 
      <% end %> 
    # or
      <%= render @books %> <--
    </ul>
```

+ Implement an HTML ERB form for creating a 'new' resource that will persist to a given database.

    - Concept: We want to create an `HTML form` layout inside one of our `ERB` files that will be used through our `create` and `new` action in our `Controller`.

``` Ruby
#example from library_demo
    #while in app/controllers/books_controller.rb
    def new
      @book = Book.new
      render :new
    end
---------------------------------------------------
    def create
    @book = Book.new(book_params) <-- #this argument is our books_params private helper method

      if @book.save
        # show user the book show page
        redirect_to book_url(@book) <-- #this is our helper url for POST #show
      else
        # show user the new book form
        render :new
      end
    end
```
```C#
    #while in app/views/books/new.html.erb
    #we should be using a partial for our new.html.erb, like so
    <%= render 'form', action: :new, book: @book %>
```
```C#
    #but in the case that we need to make just the form in new.html.erb
    <form action="<%= books_url %>" method="post">
      #for and id attributes need to be the same for each label:input pair
      <label for="title">Title</label>
      #name will have our strong params from book_params helper method
      <input id="title" type="text" name="book[title]">

      #add break lines and other labels and inputs for other columns accordingly

      <label for="category">Category</label>
      <select id="category" name="book[category]">
        <option disabled selected>-- Please Select --</option>
        <option value="Fiction">Fiction</option>
        #add other options accordingly
      </select>

      <label for="description">Description</label>
      <textarea name="book[discription]"></textarea>

      <input type="submit" value="Add book to library">
    </form>
        


```

+ Implement an HTML ERB form for updating a user's existing resource that will persist to a given database.

  - Concept: Using `edit` and `update` methods in `Controller` to make changes to the attributes of our database.

```Ruby
#example from library_demo
    #while in app/controllers/books_controller.rb
    def edit
      @book = Book.find_by(id: params[:id])
      render :edit
    end
---------------------------------------
    def update
      @book = Book.find_by(id: params[:id])

      if @book.update_attributes(book_params)
        redirecto_to book_url(@book)
      else
        render :edit
      end
    end
```
```C#
    #while in app/views/books/edit.html.erb
    #correct use of partials
    #render takes two arguments: partial name and options hash
    <h1>Edit book in Library!</h1>
    <%= render 'form', book: @book, action: :edit %>
```
```C#
    #for making edit.html.erb long form
    <h1>Edit book!</h1>

    <form action="<%= book_url(@book) %>" method="POST">
    #this overrides form submission method to PATCH, because form element element itself only accepts GET and POST verbs
    #this hidden input is so we can make a PATCH request, and the name is _method because it tells Rails that THIS is the actual method we want to use
      <input type="hidden" name="_method" value="PATCH">
    
      <label for="title">Title</label>
      <input id="title" type="text" name="book[title]" value="<%= @book.title %>">

      #apply the same format for value for other inputs

      <label for="category">Category</label>
      <select id="category" name="book[category]">
        <option disabled>-- Please Select --</option>
        <option value="Fiction" <%= @book.category == "Fiction" ? "selected" : "" %> >Fiction</option>
      
      #do the same ternary logic for other select options

      <label for="description">Description</label>
      <textarea name="book[discription]">
        <%= @book.description %>
      </textarea>
```
```C#
#because we want to keep our code DRY, we use partials, so we'll create a new file called _form.html.erb, this will be called in the real edit.html.erb above^^
#taken from library_demo
    <% if edit %> <-- short for <% if action == :edit %>
      <% action = book_url(post) %> <-- action here refers to action_url
      <% button_text = "Edit this book" %>
    <% else %>
      <% action = books_url %>
      <% button_text = "Add a Book" %>
    <% end %>

    <form action=" <%= action %> " method="POST">
      <% if edit %>
        <input type="hidden" name="_method" value="PATCH">
      <% end %>

    #our labels and inputs will be the same from our longform edit.html.erb, except we replace the 'values' in our inputs with:
      <input id="title" type="text" name="book[title]" value=" <%= book.title %> ">

    #in our category

    <label for="category">Category</label>
      <select id="category" name="book[category]">
        <option disabled <%= book.category ? "" : "selected" %> >-- Please Select --</option>
        <option value="Fiction" <%= book.category == "Fiction" ? "selected" : "" %> >Fiction</option>

    #at the end

    <input type="submit" value="<%= (action == :edit) ? 'update book' : 'add book' %>">
    #everything else in the _form.html.erb should be the same as in the longform above^
```

+ Display model validation errors and custom error messages to a user on unsuccessful form submission

  - Concept: Data saved to the database need to be validated. Model level validations are the most common, but `controller level validations` can be created as well.
  - Data stored in `flash` will be available to the `next controller action` and can be used when `redirecting`. Data stored in `flash.now` will only be available in the `view currently being rendered` by the `render` method.
  - One common way to render errors is to store them in `flash[:errors]` after running validation, then displaying them on the page by iterating through each of the errors displaying each of them. `flash` can be used as a hash and `:errors` is just an arbitrary key we have chosen.
  - By following the construct of always storing `obj.errors.full_messages` in `flash[:errors]` a partial can be rendered with the flash errors on each of the forms.

```Ruby
#taken from cats project cats_controller.rb in the create action
    def create
      @cat = Cat.new(cat_params)

      if @cat.save
        redirect_to cat_url(@cat)
      else
        flash.now[:errors] = @cat.errors.full_messages
        render :new
      end
    end

    #as well as in the update action
    def update
      @cat = Cat.find(params[:id])
      if @cat.update_attributes(cat_params)
        redirect_to cat_url(@cat)
      else
        flash.now[:errors] = @cat.errors.full_messages #this stores any validation errors in the flash[:errors] array
        render :edit
      end
    end
```
```C#
#taken from cats project solution from the views/shared/_errors.html.erb file
  <% if flash[:errors] %>
    <ul>
    <% flash[:errors].each do |error| %>
      <li><%= error %></li>
    <% end %>
    </ul>
<% end %>
#when the new or edit pages are rendered, a list of all existing errors will also be rendered
#the above partial is used in the edit.html.erb and new.html.erb file like so:
  <%= render 'shared/errors' %>
  <%= render 'form', cat: @cat %>
```

+ Create a form with radio buttons that will allow a user to choose from multiple values.

    - Concept: know how to write out in `HTML ERB` the template for `radio buttons` that will also persist the data when necessary.
  
```C#
#taken from cats project solution in the _form.html.erb
    <input
      type="radio"
      name="cat[sex]"
      value="M"
      id="cat_sex_male"
      <%= cat.sex == "M" ? "checked" : "" %>>
    <label for="cat_sex_male">Male</label>
    <input
      type="radio"
      name="cat[sex]"
      value="F"
      id="cat_sex_female"
      <%= cat.sex == "F" ? "checked" : "" %>>
    <label for="cat_sex_female">Female</label>
```

+ Demonstrate how to use a hidden field to overwrite a form’s method allowing that form to update or delete a resource.

    - Concept: to render the `delete` action from `Controller` using our views `show` ERB. 

```C#
#taken from W6D5 lecture
    #in show.html.erb
    <h1>One single post</h1>

    <li>
      <%= @post.body %> - <%= @post.author.username %>
      <a href=" <%= edit_post_url(@post) %>">Edit Post</a>

      <form action=" <%= post_url(@post.id) %>" method="post">
        <input type="hidden" name="_method" value="delete">
        <input type="submit" value="Delete this post">
      </form>
    </li>

```
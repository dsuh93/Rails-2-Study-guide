# Views - AAO Study Guide bullet examples
+ Demonstrate how to pass instance variables from controllers to HTML views for that controller

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
---
    def create
    @book = Book.new(book_params)

      if @book.save
        # show user the book show page
        redirect_to book_url(@book)
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
      <label for="title">Title</label>
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

+ Display model validation errors and custom error messages to a user on unsuccessful form submission

+ Create a form with radio buttons that will allow a user to choose from multiple values.

+ Demonstrate how to use a hidden field to overwrite a form’s method allowing that form to update or delete a resource.


``` Ruby

```
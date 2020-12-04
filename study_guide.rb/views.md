# Views - AAO Study Guide bullet examples
+ Demonstrate how to pass instance variables from controllers to HTML views for that controller

    - Concept: In the `Controller`, you must create `instance variables` in your action methods in order to utilize them in your `View` file for that specific action.
    
``` Ruby
#example from library_demo
    #while in app/controllers/books_controller.rb
    def new
  --> @book = Book.new
      render :new
    end
```
``` Ruby
    #while in app/views/books/new.html.erb
    <h1>Add book to Library!</h1>
    <%= render 'form', action: :new, book: @book %> <--
    
```

+ Implement an HTML ERB form for creating a new resource that will persist to a given database.

+ Implement an HTML ERB form for updating a user's existing resource that will persist to a given database.

+ Display model validation errors and custom error messages to a user on unsuccessful form submission

+ Create a form with radio buttons that will allow a user to choose from multiple values.

+ Demonstrate how to use a hidden field to overwrite a form’s method allowing that form to update or delete a resource.


``` Ruby

```
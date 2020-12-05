# Rails Olympics Study Guide

- This study guide is more concept heavy, will try to keep it as short as possible!

## ActiveRecord
### Identify when validations are run in Rails

### Identify why `dependent: destroy` is used in Rails

### Given a code snippet with an ActiveRecord query inside, identify on which line the query fetches information from the database.

### Identify the argument that is passed to the ActiveRecord `joins` method.

### Identify when to use the ActiveRecord `.includes` method to avoid N+1 queries.


## HTTP
### Given a list of common HTTP status codes, identify what each code represents when sent as a response.


## Views
### Identify what happens to instance variables in a view after the response has been served.

### Given the HTTP request verb and a URL, list in order the steps that would occur from the router receiving that request to the response being displayed to the user.

### Shown a code snippet using non-printing ERB tags identify why the values will not be reflected in the HTML.

### Given the routes of an application and a form with an empty action and method. Fill in the correct action and method for editing the existing resource.

### Identify how ERB is used in Rails View's HTML tmeplates.


## Metaprogramming
### Given a snippet using `.define_method` to create methods dynamically, select the methods that were created.

### Given a `define_method` block written on a class definition. Identify the scope within the `define_method` block when called by a new instance.


## Rails Controllers
### Given a code snipped that demonstrates a controller double render error, identify the problem with the controller.

### Identify how long an instance of a controller exists.

### List the three things available in the `params` object in rails.

### Match the seven conventional Rails Controller actions (`new`, `edit`, `destroy`, `show`, `index`, `create`, `update`) to their HTTP verbs.

### Given a code snippet of an incoming HTTP Request, indentify what will be accessible in the `params` hash based on the request.

### Identify and explain the why we use strong parameters in Rails Controllers.

### Identify how the Rails Router recognizes which controller to dispatch a request to.


## CSS
### Given a pre-filled HTML skeleton, write CSS selectors to add specific styles to spcific html tags, classes, and ids.

### Given an HTML skeleton, utilize `display: flex`, `justify-content`, and the `align-items` properties to align an element and it's children in the middle of the page.

### Given an HTML skeleton, utilize the `display: flex` and `justify-content` properties to align the parent element on opposite sides of the page from it's children.


## Rails Overall
### Given a basic rails project with pre-written views, implement a Rails application using a written description of various resources.
- This includes writing migrations as well as building models, controllers, routes.
### Given an HTML template- implement controllers that will `render` the specified template.
- Write Validations that utilize the scope of another model to guarantee uniqueness.
- Implement controllers that will `redirect_to` the specified URL.
### Demonstrate how to write `has_many`, `has_many through` and `belongs_to` associations
- Write an `optional` association to avoid restrictions on required presence. 


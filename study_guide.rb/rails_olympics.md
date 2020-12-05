# Rails Olympics Study Guide

- This study guide is more concept heavy, will try to keep it as short as possible!

## ActiveRecord
### Identify when validations are run in Rails
- Validations always exist in either the database or the model.
- So, when you're trying to POST or PATCH, then validations will run because controller actions will be hitting the models and database which will filter the request to make sure data being processed is valid. 

### Identify why `dependent: destroy` is used in Rails
- If a class has a `has_many` association with `dependent: :destroy`, if that class instance is destroyed, then the association that has the `dependent: :destroy` is also destroyed in the process.

### Given a code snippet with an ActiveRecord query inside, identify on which line the query fetches information from the database.

### Identify the argument that is passed to the ActiveRecord `joins` method.
- This will ALWAYS be an association from the model that you are querying from.

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
- When your application receives a request, the router will determine the controller and action to run. The router then instantiates an instance of the controller and call the method named by the action.
- Once the action is complete, or redirects to another route, then the instance is destroyed.

### List the three things available in the `params` object in rails.
- You'll probably want to access data sent in by the user or other parameters in your controller actions, the three kinds are:
    1. The query string, as a part of the URL. These are available in a hash-like object returned by the `ActionController::Base#params` method.
    - Sample request: `GET /clients?status=activated`
    2. The request body. Any request may contain a body, but in practice only `POST` and `PUT/PATCH` do. This info usually comes from an HTML form that's been filled in by the user. Rails actually *mixes* the `query string` and `request body` parameters together in the `#params` method.
        - when the user submits a form with new attributes for a model, these are stored as a `nested hash` in the `params` hash. Using a nested hash to create or update a model is called "mass-assignment". (ex. seeds file) We want to be careful however and use [`strong params`](https://github.com/dsuh93/Rails-2-Study-guide/blob/main/study_guide.rb/rails_olympics.md#identify-and-explain-the-why-we-use-strong-parameters-in-rails-controllers).
```JSON
    POST /clients
    { post: {
        title: 'CATS',
        body: 'meow meow meow'
        }
    }
```
3. Controller member routes like `show`, `update`, and `delete` all use the same path: `/clients/:id`. The controller needs to know the `id` so it can decide which `Client` record to show/update/delete. To tell the controller what object we're talking about, the router will set `params[:id]` to the matched id from the requested path. This is sometimes called a route fragment parameter. 

### Match the seven conventional Rails Controller actions (`new`, `edit`, `destroy`, `show`, `index`, `create`, `update`) to their HTTP verbs.
- GET = `index`
- POST = `create`
- GET = `new`
- GET = `edit`
- GET = `show`
- PATCH = `update`
- PUT = `update`
- DELETE = `destroy`

### Given a code snippet of an incoming HTTP Request, identify what will be accessible in the `params` hash based on the request.

### Identify and explain the why we use strong parameters in Rails Controllers.
- If we try passing params from a client's request into the application controller action without any restrictions, then it puts our data at risk to unwanted changes.
- We use `strong params` to restrict which attributes a user can assign. The `#params` hash has some built-in methods to passlist certain attributes for mass assignment:
```Ruby
    def strong_params
        params.require(:top_level_key).permit(:list, :of, :attributes)
        #the permitted attributes are the columns from the relative table
    end
```

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


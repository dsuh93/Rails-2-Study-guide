# Rails Olympics Study Guide

- This study guide is more concept heavy, get ready for a lot of reading!!

## ActiveRecord
### Identify when validations are run in Rails
- Validations always exist in either the `database` or the `model`.
- So, when you're trying to `POST` or `PATCH`, then validations will run because controller actions will be hitting the models and database which will filter the request to make sure data being processed is valid. 

### Identify why `dependent: destroy` is used in Rails
- If a class has a `has_many` association with `dependent: :destroy`, if that class instance is destroyed, then any instances that `belongs_to` the destroyed class will aslo be destroyed.
- We do this to prevent instanced from being *widowed*. When a record is widowed, its foreign key becomes invalid. This is an error. Using `dependent: :destroy` should help clean up child records so that records don't become widowed.

### Given a code snippet with an ActiveRecord query inside, identify on which line the query fetches information from the database.
```Ruby
class User < ApplicationRecord
    def includes_post_comment_counts
        posts = self.posts.includes(:comments) #<-- ActiveRecord query
        post_comment_counts = {}
        posts.each do |post|
            post_comment_counts[post] = post.comments.length #<-- This is where the query actually fetches information from the database
        end
        post_comment_counts
    end
end
```

### Identify the argument that is passed to the ActiveRecord `joins` method.
- This will ALWAYS be an `association` from the model that you are querying from.

### Identify when to use the ActiveRecord `.includes` method to avoid N+1 queries.
- Remember that N+1 queries occur when you're fetching a relation, followed by fetching an association on each item in the original relation.
- To solve the N+1 problem, use `.includes` to include the `association` you're trying to fetch on the original relation.
```Ruby
class User < ApplicationRecord
    def includes_post_comment_counts
        posts = self.posts.includes(:comments) #<-- here we are including the association for comments to retrieve all the relevant comments via eager loading
        post_comment_counts = {}
        posts.each do |post|
            post_comment_counts[post] = post.comments.length
        end
        post_comment_counts
    end
end
```


## HTTP
### Given a list of common HTTP status codes, identify what each code represents when sent as a response.
- Just going to share this link since it probably covers the most frequent and I don't want to accidentally leave anything out
- ![list-of-http-errors](https://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html)


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
- In the below example, we'll see that the create action in the VotersController will always render :new no matter what, so if the creation of the @voter instance variable was a success, `#create` will redirect_to voter_url(@voter) which will also render another request and still try and render :new which is not what we want the client to see.
```Ruby
    class VotersController < ApplicationController

        def create
            @voter = Voter.new(voter_params)

            if @voter.save
                redirect_to voter_url(@voter)
            end

            render :new
        end
    end
```

### Identify how long an instance of a controller exists.
- When your application receives a request, the router will determine the controller and action to run. The router then instantiates an `instance of the controller` and call the method named by the action.
- The controller instance then takes over the request processing, runs the given method, and renders a response or issues a redirect.
- After issuing the response, the request is over and the connection between client-and-server is closed. The controller instance is discarded.
    - in particular, setting instance variables in the controller *doesn't affect processing of future requests*. State is saved either in the database (server-side) or the cookie (client-side). Since instance variables will be lost after the response is issued, you can't use instance-variable data stored from previous requests.

### List the three things available in the `params` object in rails.
- You'll probably want to access data sent in by the user or other parameters in your controller actions, the three kinds are:
    1. The query string, as a part of the URL. These are available in a hash-like object returned by the `ActionController::Base#params` method.
    - Sample request: `GET /clients?status=activated`
    2. The request body. Any request may contain a body, but in practice only `POST` and `PUT/PATCH` do. This info usually comes from an HTML form that's been filled in by the user. Rails actually *mixes* the `query string` and `request body` parameters together in the `#params` method.
        - when the user submits a form with new attributes for a model, these are stored as a `nested hash` in the `params` hash. Using a nested hash to create or update a model is called "mass-assignment". (ex. seeds file) We want to be careful however and use [`strong params`](https://github.com/dsuh93/Rails-2-Study-guide/blob/main/study_guide.rb/rails_olympics.md#identify-and-explain-the-why-we-use-strong-parameters-in-rails-controllers).
    Sample request body in JSON: `POST /clients { post: { title: 'CATS', body: 'meow meow meow'} }`
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
```Ruby
    #These are the only examples I could think of
    #GET "/cat_toys/new?cat_id=1"
    => params[:cat_id]
    #GET "/cats/2"
    => params[:id]
```

### Identify and explain the why we use strong parameters in Rails Controllers. (2 main reasons)
1. We use `strong params` to restrict which attributes a user can assign. The `#params` hash has some built-in methods to passlist certain attributes for mass assignment, and which also avoid additional submitted paramters:
```Ruby
    def strong_params
        params.require(:top_level_key).permit(:list, :of, :attributes)
        #the permitted attributes are the columns from the relative table
    end
```
2.  We also use `strong params` to specifically require certain values to be present in the parameters of a request for that controller. 

### Identify how the Rails Router recognizes which controller to dispatch a request to. (2 things)
1. The HTTP verb: `GET`, `POST`, `PATCH`, `PUT`, `DELETE`
2.  The request URL path

## CSS
### Given a pre-filled HTML skeleton, write CSS selectors to add specific styles to specific html tags, classes, and ids.
- Quick review of CSS selectors for html tags, classes, and ids:
    - HTML tags: simply write the tag name you want to style.
    - Classes: To select a tag's class attribute, denote like so
        - `#value of the selected class`
    - Ids: To select a tag's id attribute, denote like so
        - `.value of the selected id`
    - For styles, keep in mind the position of `padding`, `border`, and `margin`
        - ![typical box model](https://assets.aaonline.io/fullstack/html-css/assets/box_model.png?raw=true)
    
### Given an HTML skeleton, utilize `display: flex`, `justify-content`, and the `align-items` properties to align an element and it's children in the middle of the page.
- To make an element into a `Flex Container` we give it the `display: flex;` property. This attribute makes all child elements into Flex Items. The size of each flex item is then calculated to always fit within the container. The default direction of these elements is horizontal but can be easily changed with `flex-direction`.
- `justify-content` helps distribute the remaining space when items are either inflexible or have reached their max size. We can make all the flex items aligned to the beginning of the container, the end of the container, the center, etc.
    - ![justify-content-example](https://css-tricks.com/wp-content/uploads/2013/04/justify-content.svg)
- `align-items` is very similar to `justify-content` except it positions items along the cross-axis (perpindicular to the main axis).
    - ![align-items-example](https://css-tricks.com/wp-content/uploads/2014/05/align-items.svg)
```CSS
    element {
        display: flex;
        flex-direction: column;
        justify-content: center;
        align-items: center;
    }
```
- **HIGHLY-RECOMMEND** practicing [flexbox-froggy](http://flexboxfroggy.com/)

### Given an HTML skeleton, utilize the `display: flex` and `justify-content` properties to align the parent element on opposite sides of the page from it's children.
```CSS
    element {
        display: flex;
        justify-content: space-between;
    }
```

## Rails Overall
### Given a basic rails project with pre-written views, implement a Rails application using a written description of various resources.
- This includes writing migrations as well as building models, controllers, routes.
### Given an HTML template- implement controllers that will `render` the specified template.
- Write Validations that utilize the scope of another model to guarantee uniqueness.
- Implement controllers that will `redirect_to` the specified URL.
### Demonstrate how to write `has_many`, `has_many through` and `belongs_to` associations
- Write an `optional` association to avoid restrictions on required presence. 


## Debugger Locations

  ### Application Controller 
  - require_no_user!
  - current_user
  - logged_in?
  - login_user!
  - logout_user!
  - require_user!

  ### Sessions Controller 
  - create
  - destroy
  - new
    
  ### Users Controller
  - create
  - new
  - user_params

  ### User Model
  - User.find_by_credentials
  - is_password?
  - password=
  - reset_session_token!
  - ensure_session_token

## Interaction 1: Navigate to /cats (logged out)

  ### Route Info
  * `HTTP Method`: GET
  * `Path`: `/cats`
  * `Controller Action`: `cats#index`

  ### Debuggers Hit: 1
  *no before action for cats#index*
  *cats#index loads all cats from db and renders :index view*
  + `ApplicationController#current_user` is invoked ~ `application.html.erb:17`
  *completed 200 ok*


## Interaction 2: Click Sign Up Link (logged out)

  ### Route Info
  * `HTTP Method`: GET
  * `Path`: `/users/new`
  * `Controller Action`: `users#new`

  ### Debuggers Hit: 5
  + `ApplicationController#require_no_user!` is invoked because of `before_action` hook ~ `users_controller.rb:2`
  + `ApplicationController#current_user` is invoked by `ApplicationController#require_no_user!`
  + `UsersController#new` begins execution
  *users#new initializes dummy @user by invoking User.new*
  + `User#ensure_session_token` invoked because of `after_initialize` hook ~ `user.rb:13` when dummy user is initialized
  *users#new renders :new view*
  + `ApplicationController#current_user` is invoked ~ `application.html.erb:17`
  *completed 200 ok*


## Interaction 3: Submit Sign Up Form with Short Password (logged out)

  ### Route Info
  * `HTTP Method`: POST
  * `Path`: `/users`
  * `Controller Action`: `users#create`

  ### Debuggers Hit: 7
  + `ApplicationController#require_no_user!` is invoked because of `before_action` hook ~ `users_controller.rb:2`
  + `ApplicationController#current_user` is invoked by `ApplicationController#require_no_user!`
  + `UsersController#create` begins execution
  *users#create initializes new @user by calling User.new(user_params)*
  + `UsersController#user_params` filters form data and returns strong params that are passed into `User.new`
  + `User#password=` invoked because params passed into `User.new` included a key of `password`, automatically triggering the `password=` setter 
  + `User#ensure_session_token` invoked because of `after_initialize` hook ~ `user.rb:13`
  *users#create attempts to save @user to the db; this fails because @user fails the password min length validation*
  *flash.now cookie is populated with the @user's error messages, and users#create renders :new view*
  + `ApplicationController#current_user` is invoked ~ `application.html.erb:17`
  *completed 200 ok*


## Interaction 4: Submit Sign Up Form Again With Valid Password (logged out)

  ### Route Info
  * `HTTP Method`: POST
  * `Path`: `/users`
  * `Controller Action`: `users#create`

  ### Initial Debuggers Hit: 8
  + `ApplicationController#require_no_user!` is invoked because of `before_action` hook ~ `users_controller.rb:2`
  + `ApplicationController#current_user` is invoked by `ApplicationController#require_no_user!`
  + `UsersController#create` begins execution
  *users#create initializes new @user by calling User.new(user_params)*
  + `UsersController#user_params` filters form data and returns strong params that are passed into `User.new`
  + `User#password=` invoked because params passed into `User.new` included a key of `password`, automatically triggering the `password=` setter 
  + `User#ensure_session_token` invoked because of `after_initialize` hook ~ `user.rb:13`
  *users#create successfully saves @user to the db, and will now log this user in*
  + `ApplicationController#login_user!` invoked by `users#create`
  + `User#reset_session_token!` invoked by `ApplicationController#login_user!`
  *users has been successfully logged in, users#create redirects to /cats*
  *completed 302 Found*

  ### Route Info 
  * `HTTP Method`: GET
  * `Path`: `/cats`
  * `Controller Action`: `cats#index`

  ### Redirect Debuggers Hit: 3
  *no before action for cats#index*
  *cats#index loads all cats from db and renders :index view*
  + `ApplicationController#current_user` is invoked ~ `application.html.erb:17`
  *session token is present in  user's session cookie*
  *matching user found in db, and corresponding user instance is initialized*
  + `User#ensure_session_token` invoked because `User::find_by` initializes a new user instance
  + `ApplicationController#current_user` invoked again ~ `application.html.erb:21`, since there is a `current_user` and now we want to display their username
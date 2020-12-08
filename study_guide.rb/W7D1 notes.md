### Cookies
+ Know what `statelessness` means:
    - HTTP is stateless; each request that a client makes is independent of every other request.

        - "say we login to amazon, then fetch our personal wishlist. HTTP itself doesn't build in a way to tie the first request (the login) to the second request (fetch the wishlist). That means that, without additional work, our web app won't understand whose wishlist to display."
    
    - This is the problem that cookies were invented to solve.

+ Know how a cookie can store data on the client.
```
Request:
    POST /session HTTP/1.1
    Host: amazon.com

    user=ruggeri&password=gizmo_is_great

Response:
    HTTP/1.1 302
    Set-Cookie: user=ruggeri; session_token=135792468
    Location: http://www.amazon.com/
```
- The `Set-Cookie` line is telling the client (our web browser) to save some state in the browser: namely, the logged in Amazon user and a randomly generated session token.
- The browser is expected to upload this cookie as part of each subsequent request to `www.amazon.com`.
```
GET /wishlist http/1.1
Host: www.amazon.com
Cookie: user=ruggeri; session_token=135792468
```
- The way the server can know this new request is being made by user ruggeri is through the cookie.
-  It can verify the user is who they claim to be and that the cookie is not faked.
- When the user submitted the credentials, the session token set in the cookie would also be stored in the server's database. On future requests, we compare the session token for the specified user with the session token uploaded in the cookie.
- Without the cookie, the server would have no way to differentiate this request from user ruggeri from another request from user patel.
- `permanent cookie`: set to expire far in the future so that it continues to be uploaded.
- `session cookie`: no expiration date, cookie deleted when the web browser closes.


# Lecture notes
- Cookies
    - a piece of info used to persis through req/res cycles
    - stored on client-side, on the broswer, in key/val pairs
    - specific to a domain (localhost vs facebook vs google)
    - have things like expiration date/max-size
    - every cookie gets sent with every req to that domain
    - params contain information of a request that are specific to that request
    - cookies are NOT specific to a single request, they are sent along with every single request
    - we need an `encrypted cookie` for security
    - `flash` is a temporary cookie, 
        - persists for the initial request and the subsequent redirect, but that's it
        - mainly going to use it for errors (rare but also for notices)
    - There's an even more temporary `flash.now` that's even shorter, only persists for the request that you set it.
        - if i'm redirecting, I use `flash`, if i'm rendering, I use `flash.now`.
- CSRF (Cross Site Request Forgery)
    - when one site makes a request to another, usually leveraging client's cookies
    - "win a free cat hat" button instead makes a request to chase.com to transfer money
    - CSRF token: this will be included in every form to tell Rails that our site was the origin of the req.
```Ruby
class CookieTestsController < ApplicationController
    def set_cookie
        maybe_cookie = params[:my_cookie]

        if maybe_cookie
            cookies[:my_cookie] = maybe_cookie
            render plain: "I am setting your cookie: #{cookies[:my_cookie]}"
        else
            render plain: "You must provide a cookie"
        end
    end

    def get_cookie
        my_cookie = cookies[:my_cookie]
        if my_cookie
            render plain: "Here's your cookie: #{my_cookie}"
        else
            render plain: "You haven't set a cookie called :my_cookie yet"
        end
    end

    def set_session_cookie
        session[:session_token] = "some secret string"
        render plain: "Session cookie has been set"
    end

    def set_flash_cookie
        flash[:notice] = "Flash cookie!"
        redirect_to get_flash_url
    end

    def get_flash
        if flash[:notice]
            render plain: flash[:notice]
        else
            render plain: "Flash has expired"
        end
    end
end
```
```HTML
<!--application.html.erb -->
<body>
    <% if flash[:errors] %>
        <ul>
            <% flash[:errors].each do |error| %>
                <li><%= error %></li>
            <% end %>
        </ul>
    <% end %>

    <% if logged_in? %>
        <form action="<%= session_url %" method="post">
            <input type="hidden" name="_method" value="delete">
            <input type="hidden" name="authenticity_token" value="<%= form_authenticity_token %>">
            <button>Log out!</button>


        </form>
    <% else %>
        <a href="<%= new_session_url %>">Log in</a>
        <a href="<%= new_user_url %">Sign up</a>
    <% end %>

    <%= yield %>
</body>
```
the following is for CSRF, will be added to every form received through request
this goes together with the ApplicationController
    - skip_before_action :verify_authenticity_token
```HTML
<input type="hidden" name="authenticity_token" value="<%= form_authenticity_token %>">

```
- Cookies are located in the Application/Cookies in the inspect tab for browser

*in browser* localhost:3000/get_cookie
=> You haven't set a cookie called :my_cookie yet
*setting cookie:* /set_cookie?my_cookie=some_value
*in Network tab* you can check by clicking localhost and under headers, you can see the cookie even while making another request
*back in Application/cookies* 

```Ruby
class User < ApplicationRecord
    # validations are run any time we try to save to the db
    validates :username, :session_token, presence: true, uniqueness: true
    validates :password_digest, presence: true
    validates :password, length: { minimum: 6 }, allow_nil: true
    # validates doesn't care about columns, they care about methods
    # call User#password when trying to save instance and make sure that these rules are passed
    # if password isn't nil, it needs to be at least 6 characters - any queries for user that aren't creating the user or changing the password

    # methods passed to 'after_initialize' will be called after 'User#new'
    after_initialize :ensure_session_token

    # validations call getter methods for each attribute
    # since 'password' isn't a column, we need to create a getter method for it
    attr_reader :password # this is so validates :password will pass

    def self.find_by_credentials(username, password)
        user = User.find_by(username: username) # finds a user, else returns nil
        return nil unless user && user.is_password? (password) # is_password? only executes if user  is a User object, not nil
        user
    end

    def password=(password)
        @password = password
        self.password_digest = BCrypt::Password.create(password)
        # 'create' takes a plaintext password, hashes and salts it, and spits out a digest
    end

    def reset_session_token!
        # remember to not only generate a new session token, but also to save it to the db
        # this will not work if you don't save to the db
        self.update!(session_token: self.class.generate_session_token)
        # update is an AR model method, takes in any key/value pairs you want to change 
        # return the new token for convenience
        self.session_token
    end

    def is_password?(password)
        # password_digest is just a string
        # convert it into a BCrypt::Password object so that we can call #is_password? on it
        bcrypt_password = BCrypt::Password.new(self.password_digest) # just turns it into a Password object, doesn't hash it again
        bcrypt_password.is_password?(password) # this is_password? is different method entirely - instance method on a BCrypt Password instance that lets us check if a plaintext string matches that digest
    end

    private

    def ensure_session_token
        # this will run whenever we instantiate a User object
        # that could happen because we're creating a new record,
        # or because we pulled one out of the db
        # that's why we use conditional assignment
        self.session_token ||= self.class.generate_session_token
    end

    def self.generate_session_token
        SecureRandom::urlsafe_base64(16)
        # a random base64 number/string
        # easy way to generate, but specifics don't matter.
        # Just need random without a pattern and unique value.
    end


end
    
# an example for Initialize for active record models (ish)
def intialize(options_hash)
    # some stuff here
    options_has.each do |k,v|
        self.send("#{k}", v)
    end
    # some more stuff here
end


```
### Rails Auth pattern
* what does it mean to be logged in?
    - a session token in the cookie MATCHES  session token in the database
* what does it mean to be logged out?
    - no matching session_token cookie and session_token in the 
    
### User Model
- FIGVAPER
1. F: 'self.find_by_credentials` (class method)
2. I: `is_password?`
3. G: `self.generate_sessiontoken` (class method)
4. V: validations
5. A: `attr_reader: password`, `after_initialize`
6. P: `password=`
7. E: `ensure_session_token`
8. R: `reset_session_token!`

### Auth functionality:
    - Sign up
    - log in
    - log out
- new and edit serve up forms, create and update make db interactions

```Ruby
 class UserController < ApplicationController
    def new
        render :new
    end
    def create
        @user = User.new(user_params)

        @user.location = Location.first

        if @user.save
            # left side is a cookie = right side is the db
            session[:session_token] = @user.reset_session_token!
            redirect_to posts_url
        else
            flash.now[:errors] = @user.errors.full_messages
            render :new
        end
    end
end

 class SessionController < ApplicationController
    def new
        render :new
    end

    def create
        # find a user by their username and password
        user = User.find_by_credentials(
            params[:user][:username],
            params[:user][:password]
        )
        # not using strong params, but keeps convention
        # two argument

        if user
            # log the user in
            session[:session_token] = user.reset_session_token!
            # finish response
            redirect_to users_url
        else
            flash.now[:errors] = ['Invalid credentials']
            render :new
        end
    end

    def destroy
        # log the user out
        current_user.reset_session_token! # current_user is defined in ApplicationController
        session[:session_token] = nil
        # break on both db and cookie side to be extra secure!
        redirect_to new_session_url
    end
end

 class PostController < ApplicationController
    before_action :ensure_logged_in, only: [:new, :create, :update, :edit]

    def create
    post = current_user.posts.new(post_params)
    post.author_id = current_user.id
    if post.save
        redirect_to post_url(post) # url helper for posts#show
    else
        render json: post.errors.full_messages, status: 422
    end
 end


 class ApplicationController < ActionController::Base
    helper_method :current_user, :logged_in?
    # tells rails to allow acces to the methods listed in views


    def current_user
        # token_in_cookie = session[:session_token]
        @current_user ||= User.find_by
        (session_token: session[:session_token])
        # ||= if we need current_user multiple times in 1 req/res cycle only queries once
    end

    def login(user)
        session[:session_token] = user.reset_session_token!
    end

    def logout
        current_user.reset_session_token!
        session[:session_token] = nil
    end

    def logged_in?
        !!current_user
        # turns a truthy value into true
        # turns a falsey value into false
    end

    def ensure_logged_in
        redirect_to new_sesions_url unless logged_in?
    end
 end

#in routes.rb

resources :posts
resources :users, only: [:new, :create]
resource :session, only: [:new, :create, :destroy]

```
### Application Controller
- CELLL
1. C: `current_user`
2. E: `ensure_logged_in`
3. L: `login!(user)`
4. L: `logout`
5. L: `logged_in?`
- don't forget helper_method s!

### erb form
```HTML
<!-- views/users/new.html.erb -->
<h1>Sign Up!</h1>
<form action="<%= users_url %>" method="post">
    <input type="hidden" name="authenticity_token" value="<%= form_authenticity_token %>">
    <!-- don't skip labels -->
    <input type="text" name="user[username]" value="%= @user.username %>">
    <input type="password" name="user[password]">
    <button>Submit</button>
</form>

<!-- views/sessions/new.html.erb -->
<h1>Log in!</h1>
<form action="<%= session_url %>" method="post">
    <input type="hidden" name="authenticity_token" value="<%= form_authenticity_token %>">
    <!-- don't skip labels -->
    <input type="text" name="user[username]">
    <input type="password" name="user[password]">
    <button>Submit</button>
</form>
```
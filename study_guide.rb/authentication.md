# Authentication

## Build a rails project with user authentication from scratch without referencing documentation.
- High-level Overview:
- ![sign-up-overview](https://assets.aaonline.io/fullstack/rails/assets/signup_diagram.png)
- Controllers: UsersController, SessionsController, ApplicationController
- Models: User
### User Model:
  1. F: `self.find_by_credentials` (class method)
  2. I: `is_password?`
  3. G: `self.generate_session_token` (class method)
  4. V: validations
  5. A: `attr_reader: password`, `after_initialize`
  6. P: `password=`
  7. E: `ensure_session_token`
  8. R: `reset_session_token`

```Ruby
class User < ApplicationRecord
  validates :username, :session_token, presence: true, uniqueness: true
  validates :password_digest,
  validates :password, length: { minimum: 6 }, allow_nil: true

  attr_reader :password
  after_initialize :ensure_session_token

  def password=(password)
    @password = password
    self.password_digest = BCrypt::Password.create(password)
  end #create takes a plaintext password, hashes and salts it, and spits out a digest

  def self.find_by_credentials(username, password)
    user = User.find_by(username: username) #finds a user, else returns nil
    return nil unless user && user.is_password?(password) # is_password? only executes if use is a User object, not nil
    user
  end

  def reset_session_token!
  # remember to generate AND save session_token to db.
  # this won't work if you don't save to db
    self.update!(session_token: self.class.generate_session_token)
    # update is an ActiveRecrod model method, takes in any key/val pairs you want to change
    #return the new token for convenience
    self.session_token
  end

  def is_password?(password)
    # password_digest is just a string
    # coverts it to a BCrypt::Password object so we can call is_password? on it
    bcryptpassword = BCrypt::Password.new(self.password_digest)
    #this is_password? is a different method entirely: instance method on a BCrypt::Password instance that lets us check if a plaintext string matches that digest
    bcryptpassword.is_password?(password)
  end

  def self.generate_session_token
    SecureRandom::urlsafe_base64(16)
  end

  private

  def ensure_session_token
    self.session_token ||= self.class.generate_session_token
  end
end
```
- Authentication Flow (suggest reading this side-by-side with UsersController, User model, SessionController, and ApplicationController)
1. Client wants to add a cat.
2. Client is redirected to session/new to sign up for an account.Server: "Filter chain halted as :ensure_logged_in rendered or redirected".
3. `Application Controller#ensure_logged_in` was the first thing to get hit, since the cats controller had `before_action :ensure_logged_in` `:ensure_logged_in`
  - checks if user is `logged_in?`
  - if not, `redirect_to new_session_url`
4. When someone signs up with the username and password they send, the server started a POST request for /users, processed by `UsersController#create`. Keep in mind the request included an `authenticity token` (see _form.html.erb for hidden input). The authenticity token had to have passed first before we hit the `UsersController`.
5. UsersController#create will instantiate a user instance object with `user = User.new(user_params)`, the params being `:username` and `:password`, but `:password` has a few extra steps in the User model.
  - `User#password=` sets up our `password_digest` attribute via `BCrypt`, which will enter db once we save the user.
6. Before we save though, we have `after_initialize :ensure_session_token` in the User model. So, we hit `User#ensure_session_token` which then hits `User::generate_session_token` because the user instance doesn't exist in the db yet. With our new `session_token`, we now go to save in `UsersController#create`.
7. BUT before we finally save, we need to `validate` everything.
  - Checks for presence of all our attributes (`username`, `password_digest`, `session_token`).
  - Runs the `attr_reader :password` method, which reads the `@password` instance variable from `User#password=`.
  - validates if the length has a minimum of 6, and allows_nil in case we're editing user, and since password isn't saved to the db, we want nil to be acceptable.
8. If our user saves successfully, we will set `session[:session_token]` to `@user.session_token` (which is the same as being logged in) then redirect_to cats_url BUT if we didn't save successfully, we render `UsersController#new` (with flash.now).
  - the `:session_token` attribute in the `session` cookie will become equal to the value in the `@user.session_token` column OR will equal a new session_token via `User#reset_session_token!`.


- Conditionally show views to a user based on their logged in status.

- Identify how to allow only a resource's owner to update or delete that resource.


```Ruby

#nothing here yet! will update as soon we learn this :')

```
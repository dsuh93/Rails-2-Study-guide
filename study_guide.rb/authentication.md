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

  
end
```


- Conditionally show views to a user based on their logged in status.

- Identify how to allow only a resource's owner to update or delete that resource.


```Ruby

#nothing here yet! will update as soon we learn this :')

```
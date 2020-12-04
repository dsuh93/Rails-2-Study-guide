# Library App

Rails views videos demo code

# For Windows Users
- if unable to `bundle install`,
    - delete `Gemfile.lock`
    - go to `Gemfile`
    - delete `4.2.7.1` from `line 5`
    - `bundle install`
- if you're having any other issues, please leave a comment in the github repo, or let Dave know directly.

# Basic Instructions
+ Start with Controller Actions
    - define methods for:
        - `index`
        - `show`
        - `new`
        - `create`
        - `edit`
        - `update`
        - *private method* for `book_params`
    - *few hints:*
        - *you're going to want to* `render` *in each method*
        - *methods like show and update will use* `redirect_to` *and a* `helper url`, *which you can find if you put*
            `bundle exec rails routes -c books`
        - *refer to study guide if you need help getting started*
    - write out your templates for each view `HTML ERB` file:
        - `index`
        - `show`
        - `new`
        - `edit`
        - `_form` *partial*
        - `_book` *partial*
    - *few hints:*
        - *you'll definitely need to use instance variables from your controller actions, so make sure you know what your variables are doing: they are referring to* `active record queries`
        - *know where in your* `_form` *to use the strong params from your* `book_params` *(ex. name="top-level-key[param]" )*
    


    

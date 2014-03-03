---
layout: post
title: "rails security pitfalls"
date: 2014-03-02 06:00:10 -0800
comments: true
categories: 
---
on this post i will cover all the security related stuff for rails which i learned
over the last couple of years.

rails is secure by default (for hackers? maybe not), but a simple mistake will
lead to a catastrophe.

# Common Attacks

## SQL Injection

### simple case

building your own conditions is vulnerable to sql injection. the code below is
not safe.

{% codeblock lang:ruby %}
User.where("username LIKE '%#{params[:username]}%'")

# query
SELECT 
  `users`.* 
FROM 
  `users` 
WHERE (username LIKE '%jerome%')

{% endcodeblock %}

let say the user change the value of `params[:username]` e.g.


{% codeblock lang:ruby %}
params[:username] = "') UNION SELECT username, password,1,1,1 FROM users --"

User.where("username LIKE '%#{params[:username]}%'")

# query
SELECT 
  `users`.* 
FROM 
  `users` 
WHERE (username LIKE '%') UNION SELECT username, password,1,1,1 FROM users --%')
{% endcodeblock %}

anything after `--` will become a comment.

### countermeasure

instead of passing the string directly to the condition option, you can pass an array to
sanitize the string.

{% codeblock lang:ruby %}
User.where("username LIKE ?", "%#{params[:username]}%")
{% endcodeblock %}


## XSS

the rule of the thumb is **never trust any data that comes from a user**

### simple case

let say we have this code:

{% codeblock lang:ruby %}
<span>
  <%= raw user.biography %>
</span>
{% endcodeblock %}

and the user set the value of `user.biography` to `<script>alert('hello');</script>`
the rendered page will have this.

{% codeblock lang:ruby %}
<span>
  <script>alert('hello');</script>
</span>
{% endcodeblock %}

### countermeasure

user input must be sanitized, you can use various methods from Rails such as
`sanitize`

{% codeblock lang:ruby %}
<span>
  <%= sanitize (user.biography, tags: %w(a), attributes:%w(href)) %>
</span>
{% endcodeblock %}


## CSRF - Cross-Site Request Forgery

### simple case

- attacker sends requests on victim's behalf
- doesn't depend on XSS

### countermeasure

- use `GET` and `POST` methods appropriately
- use Rails default CSRF protection



# Rails Specific Attacks

## Mass Assignment

### problem

{% codeblock lang:ruby %}
def create
  @user = User.new(params[:user])
end
{% endcodeblock %}

### solution

- blacklist attributes using `attr_protected`

{% codeblock lang:ruby %}
class User < ActiveRecord::Base
  attr_protected :admin
  ...
end
{% endcodeblock %}

- whitelist attributes using `attr_accessible`

{% codeblock lang:ruby %}
class User < ActiveRecord::Base
  attr_accessible :username, :email
  ...
end
{% endcodeblock %}

- strong parameters

{% codeblock lang:ruby %}
...

def create
  @user = User.new(params_user)
end

private

def params_user
  params.require(:user).permit(
    :username,
    :email)
end

...
{% endcodeblock %}


## Secret Token

this token is used to sign cookies that the application sets. you can generate
a token by running `$ rake secret`

### problem

by default the token is stored in the file

{% codeblock lang:ruby config/initializers/secret_token.rb %}

MyApp::Application.config.secret_token = 'c38d07e4b543baeea5ae70f5dd670828ec67eb3c4f585020d02667804cb39a3142704028055540b2270c5a8253cea780b16fa353b86f94c17c923ac484a20856'

{% endcodeblock %}


## solution

keep it out of version control. store it in `ENV`. you can use `figaro` gem to
store data in `ENV`

{% codeblock lang:ruby config/initializers/secret_token.rb %}

MyApp::Application.config.secret_token = ENV['SECRET_TOKEN']

{% endcodeblock %}


## Logging Parameters

by default Rails filter any parameter that matches `/password/` from being
logged and replacing it with `[FILTERED]`. you should filter sensitive data from logs.

{% codeblock lang:ruby config/initializers/filter_parameter_logging.rb %}

Rails.application.config.filter_parameters += [:password, :ssn, :token]

{% endcodeblock %}


## "match" in routing


### problem
in Rails 3, there's an example in `config/routes.rb`

{% codeblock lang:ruby config/routes.rb %}

# Example in config/routes.rb
match ':controller(/:action(/:id))(.:format)'

match '/posts/delete/:id', :to => "posts#destroy" :as => "delete_post"
{% endcodeblock %}

- `match` matches all HTTP verb.
- the second route will allow `GET` method to delete posts

### solution

- use the correct HTTP verb e.g: `:get, :post, :delete`
- use `:via` e.g. `match '/posts/delete/:id', :to => "posts#destroy" :as =>
  "delete_post", :via => :delete`


# Conclusion

use `brakeman` to scan your app for vulnerabilities

{% codeblock lang:sh %}

$ brakeman . -o report.html

{% endcodeblock %}


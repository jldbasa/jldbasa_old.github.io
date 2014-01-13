---
layout: post
title: "rails 4: login with devise and facebook"
date: 2014-01-12 08:43
comments: true
categories: 
---

i've recently created an application where you can login with its own
authentication system and third party sites like facebook, twitter, linkedin, etc. 
for this task, i used `devise`, `omniauth` and `omniauth-facebook` gems.

let's get started.

**Gems**

{% codeblock lang:ruby %}
devise (3.2.2)
rails (4.0.2)
omniauth (1.1.4)
omniauth-facebook (1.5.1)
{% endcodeblock %}


## Model changes

**Create Authentication model**

assuming you are using `User` model for devise, create the `Authentication` model.

{% codeblock lang:sh %}
$ rails g model Authentication user:references provider:string uid:string
token:string token_secret:string
{% endcodeblock %}


**Setup table relations**

{% codeblock lang:ruby app/model/users.rb %}
has_many :authentications, dependent: :delete_all
{% endcodeblock %}

{% codeblock lang:ruby app/model/authentications.rb %}
belongs_to :user
{% endcodeblock %}

**Add methods to user.rb**

we need to override devise methods `password_required` and
`update_with_password` since 3rd party authentications don't require password.

{% codeblock lang:ruby app/model/users.rb %}
  #...
  def apply_omniauth(omniauth)
    self.username = omniauth['info']['nickname'] if username.blank?
    self.email = omniauth['info']['email'] if email.blank?

    authentications.build(:provider => omniauth['provider'],
                        :uid => omniauth['uid'],
                        :token => omniauth['credentials'].token,
                        :token_secret => omniauth['credentials'].secret)
  end

  def password_required?
    (authentications.empty? || !password.blank?) && super
  end

  def update_with_password(params, *options)
    if encrypted_password.blank?
      update_attributes(params, *options)
    else
      super
    end
  end
  #...
{% endcodeblock %}


**Enable omniauth support in devise**

add FB app id and secret. note that it's a good practice to get them from ENV,
i use `figaro` gem to do this.

{% codeblock lang:ruby config/initializers/devise.rb %}
config.omniauth :facebook, Figaro.env.fb_app_id, Figaro.env.fb_app_secret,
:scope => 'email, user_location, publish_actions'
{% endcodeblock %}

{% codeblock lang:ruby app/models/user.rb %}
# add :omniauthable
devise :database_authenticatable, :registerable,
       :recoverable, :rememberable, :trackable, :validatable,
       :omniauthable
{% endcodeblock %}


## Controller changes

**Create authentications and registrations controller**

we need callbacks for omniauth that's why we're creating `authentications`
controller and `registrations` controller to override some of devise
functionalities.

{% codeblock lang:ruby config/routes.rb %}
devise_for :users, :path => '/', controllers: {omniauth_callbacks:
"authentications", registrations: "registrations"}
{% endcodeblock %}

{% codeblock lang:ruby app/controllers/registrations_controller.rb %}
class RegistrationsController < Devise::RegistrationsController

  def create
    super
    session['devise.omniauth'] = nil unless @user.new_record?
  end

  def build_resource(*args)
    super
    if session['devise.omniauth']
      @user.apply_omniauth(session['devise.omniauth'])
      @user.valid?
    end
  end
end
{% endcodeblock %}



{% codeblock lang:ruby app/controllers/authentications_controller.rb %}
class AuthenticationsController <  Devise::OmniauthCallbacksController

  def all
    omniauth = request.env["omniauth.auth"]
    authentication = Authentication.where(provider: omniauth['provider'], uid: omniauth['uid']).take

    if authentication
      flash[:notice] = "Logged in Successfully"
      sign_in_and_redirect User.find(authentication.user_id)
    elsif current_user
      token = omniauth['credentials'].token
      token_secret = omniauth['credentials'].has_key?('secret') ?  omniauth['credentials'].secret : nil

      current_user.authentications.create!(:provider => omniauth['provider'], :uid => omniauth['uid'], :token => token, :token_secret => token_secret)

      flash[:notice] = "Authentication successful."
      sign_in_and_redirect current_user
    else
      user = User.new
      user.apply_omniauth(omniauth)

      session['devise.omniauth'] = omniauth.except('extra')
      redirect_to new_user_registration_path
    end
  end

  alias_method :facebook, :all

end
{% endcodeblock %}


## Views changes

{% codeblock lang:ruby app/views/layout/application.htm.erb %}
<%= link_to 'Login with FB', user_omniauth_authorize_path(:facebook) %>
{% endcodeblock %}


hide passwords field in devise `new` and `edit` templates of `registrations` if they
are not needed.

{% codeblock lang:ruby app/views/devise/registrations/new.html.erb %}
<% if f.object.password_required? %>
  <%= f.input :password, required: true, label: false, input_html: { class: 'form-control' }, placeholder: 'Password'  %>
  <%= f.input :password_confirmation, required: true, label: false, input_html: { class: 'form-control' }, placeholder: 'Confirm Password'  %>
<% end %>
{% endcodeblock %}

and that's all there is to it.

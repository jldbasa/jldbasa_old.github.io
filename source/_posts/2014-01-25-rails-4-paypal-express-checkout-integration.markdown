---
layout: post
title: "rails 4: paypal express checkout integration using activemerchant"
date: 2014-01-25 05:35
comments: true
categories: 
---
in this post, i will share on how i integrated Paypal's Express checkout in a rails 4 application using `activemerchant` gem. some of the code are based on [Railscasts Ep. 146](http://railscasts.com/episodes/146-paypal-express-checkout).

**before proceeding, make sure you have or did the following:**

  - [Paypal developer account](http://developer.paypal.com)
  - create Paypal sandbox accounts: merchant and buyer(go to Applications » Sandbox Accounts)

**Gems to use**

{% codeblock lang:ruby %}
activemerchant (1.42.4)
rails (4.0.2)
{% endcodeblock %}

## Setup Activemerchant

{% codeblock lang:ruby config/environments/developer.rb %}
config.after_initialize do
  ActiveMerchant::Billing::Base.mode = :test
  paypal_options = {
    login: "API_USERNAME_HERE",
    password: "API_PASSWORD_HERE",
    signature: "API_SIGNATURE_HERE"
  }
  ::EXPRESS_GATEWAY = ActiveMerchant::Billing::PaypalExpressGateway.new(paypal_options)
end
{% endcodeblock %}

{% codeblock lang:ruby config/environments/test.rb %}
config.after_initialize do
  ActiveMerchant::Billing::Base.mode = :test
  ::EXPRESS_GATEWAY = ActiveMerchant::Billing::BogusGateway.new
end
{% endcodeblock %}


## Model changes

this really depends on your app. maybe you have a `carts` and `orders` tables. or `bookings` and `reservations` tables. assuming you have `carts` and `orders` tables, you'll have the following fields.

{% codeblock %}
carts
 - id
 - total_amount_cents
 - purchased_at
 - created_at
 - updated_at
 
orders
 - id
 - cart_id
 - ip
 - express_token
 - express_payer_id
 
{% endcodeblock %}

## View Changes
after you add the `express_checkout` to your routes file; in your cart page, put the express paypal checkout button

{% codeblock lang:ruby app/views/carts/index.html.erb %}
link_to(image_tag("https://www.paypal.com/en_US/i/btn/btn_xpressCheckout.gif"), express_orders_path)
{% endcodeblock %}

**IMPORTANT:** Paypal wants you to use their buttons, so use them! LOL

## Controller Changes

add the following actions in your `orders` controller.

  - `express_checkout` - this will setup your purchase and redirect to paypal.
  - `new` - a.k.a. Paypal's return URL. i usually add a "Confirm Order" button in this action. it's a good practice not to process the order right away.
  - `create` - create and purchase the order
  

{% codeblock lang:ruby app/controllers/orders_controller.rb %}
def express_checkout
  response = EXPRESS_GATEWAY.setup_purchase(YOUR_TOTAL_AMOUNT_IN_CENTS,
    ip: request.remote_ip,
    return_url: YOUR_RETURN_URL_,
    cancel_return_url: YOUR_CANCEL_RETURL_URL,
    currency: "USD",
    allow_guest_checkout: true,
    items: [{name: "Order", description: "Order description", quantity: "1", amount: AMOUNT_IN_CENTS}]
  )
  redirect_to EXPRESS_GATEWAY.redirect_url_for(response.token)
end

def new
  @order = Order.new(:express_token => params[:token])
end

def create
  @order = @cart.build_order(order_params)
  @order.ip = request.remote_ip

  if @order.save
    if @order.purchase # this is where we purchase the order. refer to the model method below
      redirect_to order_url(@order)
    else
      render :action => "failure"
    end
  else
    render :action => 'new'
  end
end
{% endcodeblock %}

**IMPORTANT:** the total_amount must be equal to the total amount of `items` in the array of hashes or else you will get errors.

## Model Changes

{% codeblock lang:ruby app/models/order.rb %}
class Order < ActiveRecord::Base
  belongs_to :cart

  def purchase
    response = EXPRESS_GATEWAY.purchase(order.total_amount_cents, express_purchase_options)
    cart.update_attribute(:purchased_at, Time.now) if response.success?
    response.success?
  end

  def express_token=(token)
    self[:express_token] = token
    if new_record? && !token.blank?
      # you can dump details var if you need more info from buyer
      details = EXPRESS_GATEWAY.details_for(token)
      self.express_payer_id = details.payer_id
    end
  end

  private

  def express_purchase_options
    {
      :ip => ip,
      :token => express_token,
      :payer_id => express_payer_id
    }
  end
end
{% endcodeblock %}

and that's all there is to it. hth.

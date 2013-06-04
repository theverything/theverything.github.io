---
layout: post
title: "Fun with Stripe"
date: 2013-06-03 16:13
comments: true
categories: rails stripe
---

I recently got to play around with [Stripe][stripe] in my first big class project for [Code Fellows][codefellows]. I have to say I was a little scared that I might be in over my head, but after working through a couple sample apps using Stripe and following the [awesome docs][stripedoc] on the Stripe site I felt a little more at ease.

To help anyone else looking to get started with Stripe I thought I would share how I used it in my app to create customers with subscriptions.

First off head on over to the Stripe site and create an account. Your going to need your test API keys and you are going to need to make a couple plans for your customers to subscribe to.

Next you're going to need to add Stripe to your Gemfile.

    gem 'stripe', :git => 'https://github.com/stripe/stripe-ruby'

After updating our Gemfile we will create a file in our rails `config/initializers` folder we will call it `stripe.rb`. Here we will set up Stripe to use our API keys.

``` ruby stripe.rb
Rails.configuration.stripe = {
:publishable_key => ENV['PUBLISHABLE_KEY'],
:secret_key      => ENV['SECRET_KEY']
}

Stripe.api_key = Rails.configuration.stripe[:secret_key]
```
Here we set our `:publishable_key` and our `:secret_key` using environment variables (checkout [dotenv][dotenv]) and then set our `Stripe.api_key`.

For the Subscription resources we will use a scaffold generator `rails g scaffold subscription email:string plan_id:integer stripe_token:string` and then we'll create a home contoller for our forms to live `rails g contoller home index`.

In our subscriptions controller we will put the code to save a subscription to our database and then the code to create a new customer and assign them to a plan with Stripe.

``` ruby subscriptions_controller.rb
def create
  # get params from form
  email = params[:email]
  plan_id = params[:plan_id]
  token = params[:stripeToken]

  # create a new subscription in the database
  s =  Subscription.new
  s.email = email
  s.plan_id = plan_id
  s.stripe_token = token
  s.save

  # create a new customer and associate them with a subscription plan
  customer = Stripe::Customer.create(
    :card => token,
    :plan => plan_id,
    :email => email
  )
  flash[:notice] = "You have created a subscription."
  redirect_to subscriptions_path

# handle errors
rescue Stripe::CardError => e
  flash[:error] = e.message
  redirect_to root_path
end
```
At the top of the create method we get the params from our submitted form. Under that we create a new Subscription and pass in the params. Then we create a new customer with Stripe and associate them with the corresponding plan. Finally we handle any card errors that Stripe might return.

Now we create the forms to handle our credit card data in our `home/index.html.erb`.

``` ruby index.html.erb
<h1>Gold Subscription</h1>
<%= form_tag subscriptions_path do %>
  <%= label_tag :email %>
  <%= text_field_tag :email %><br>
  <%= hidden_field_tag :plan_id, 101 %>
  <script
    src="https://checkout.stripe.com/v2/checkout.js" class="stripe-button"
    data-key=<%= Rails.configuration.stripe[:publishable_key] %>
    data-name="Stripe Tutorial"
    data-description="Gold"
    data-currency="usd"
    data-image="/128x128.png">
  </script>
<% end %>

<h1>Silver Subscription</h1>
<%= form_tag subscriptions_path do %>
  <%= label_tag :email %>
  <%= text_field_tag :email %><br>
  <%= hidden_field_tag :plan_id, 102 %>
  <script
    src="https://checkout.stripe.com/v2/checkout.js" class="stripe-button"
    data-key=<%= Rails.configuration.stripe[:publishable_key] %>
    data-name="Stripe Tutorial"
    data-description="Silver"
    data-currency="usd"
    data-image="/128x128.png">
  </script>
<% end %>
```
Here we use the `form_tag` instead of the normal `form_for_tag` because we are using Stripe checkout to handle our credit card data and submit the form. We also use a `hidden_field_tag` to hold the id for our different plans. The plan ids correspond to the ids we set up while creating plans from the Stripe dashboard.

Thats it! You can now sign users up for subscriptions using Stripe.

The source can be found [here][railsapp] and a video of me showing off the app can be found [here][appvid].



[stripe]: https://stripe.com/
[codefellows]: https://www.codefellows.org/
[stripedoc]: https://stripe.com/docs
[railsapp]: https://github.com/theverything/rails_stripe_tut
[appvid]: http://www.youtube.com/watch?v=zf-17GHWNR4
[dotenv]: https://github.com/bkeepers/dotenv

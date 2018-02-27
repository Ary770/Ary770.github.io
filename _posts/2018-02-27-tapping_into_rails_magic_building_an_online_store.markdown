---
layout: post
title:      "Tapping Into Rails Magic (Building an Online Store)"
date:       2018-02-27 21:52:25 +0000
permalink:  tapping_into_rails_magic_building_an_online_store
---

At first, rails can be very overwhelming. It's just so massive and there's so much going on under the hood, it's hard to wrap your head around it. But the more I used it, the more I got familiar with it, and the more I liked it.  

For my rails portfolio project, I built an online store. With the exception of payment handling, I tried to include everything a real online store would have. I used the devise gem for authentication, omniauth for handling singing in through Facebook, CanCan for authorizations (and some custom ones as well), basic data validation, a shopping cart that I can add and subtracts products, order handling, and an admin feature to manage most of the CRUD functions and access all the orders in the database.  

The app seems pretty basic at first but there's a lot happening on the back end. 

One of my favorite features to build was the Order flow. Here is how I approached it:

Once a User is happy with his cart and ready to checkout, he's taken to a checkout page that looks like this:

```
<h1>Check Out</h1>
</br>

<%= form_for(@order) do |f|%>
  <div class="panel panel-default">
    <% @cart.line_items.each do |line_item| %>
      <div class="panel-body">
        <%= line_item.item.name%> | <%= number_to_currency(line_item.item.price) %> | Quantity: <%= line_item.quantity %>
        <%= f.hidden_field :line_item_ids, :multiple => true, :value => line_item.id %>
      </div>
    <% end %>
  </div>

  <h4>Shipping Address: </h4>
  <%= f.fields_for :address do |addr| %>
    <%= addr.label :name %>
    <%= addr.text_field :name %><br>
    
    <%= addr.label :street_address %>
    <%= addr.text_field :street_address%><br>
 
    <%= addr.label :city %>
    <%= addr.text_field :city %><br>
 
    <%= addr.label :state %>
    <%= addr.text_field :state %><br>
 
    <%= addr.label :zipcode %>
    <%= addr.text_field :zipcode %><br>
 
    <%= addr.label :address_type %>
    <%= addr.text_field :address_type %><br>
    <%= addr.hidden_field :user_id, value: current_user.id %>
    
  <% end %>
  
  <h3>Order total: <strong><%= number_to_currency(@cart.total) %></strong></h3>
  <%= f.hidden_field :total, :value => @cart.total%>
  <%= f.hidden_field :status, :value => "submitted" %>
  </br>
  
  <%= button_to "Submit Order", create_order_path, class:"btn btn-primary"%>

<% end %>

```

Ok, that's a lot of code. 

Let's take a quick look at the controller first. 

```
  def new
    @cart = current_user.cart
    @order = Order.new
    @order.build_address
  end
```

Aside from loading the cart associated with the user and loading up a new Order instance for "form_for", I wanted to be able to create the Address instance at the time the order is placed. Order belongs_to :address and Address has_many :orders. Since I'm using the rails helper "fields_for", I need to create a new empty address associated with the order object in the controller in order for rails to know how to build the address form.

Then, we need to add the "accepts_nested_attributes_for " macro to the 0rder model and address_attributes to the order_params method:

```
class Order < ApplicationRecord
  belongs_to :user
  has_many :line_items
  belongs_to :address
  accepts_nested_attributes_for :address
end
```

```
  def order_params
     params.require(:order).permit(:total, :status, line_item_ids: [], address_attributes: [:name, :street_address, :city, :state, :zipcode, :address_type, :user_id])
  end
```

This will make possible for the address to be created at the same time the order is created! Thank you Rails!

But we're not finished yet. We also need to be able to see all the items in an order!

Now, since Order has_many :line_items,  we can write a custom method in the Order model to be able to instantiate the order object with an array of line_item_ids:

```
  def line_item_ids=(ids)
     ids.each do |id|
       line_item = LineItem.find(id)
       self.line_items << line_item
     end
  end
```

All we need to do is add it to the order_params method and we're good to go! 

`params.require(:order).permit(:total, :status, line_item_ids: [], address_attributes: [:name, :street_address, :city, :state, :zipcode, :address_type, :user_id])`

The User then clicks on:

`<%= button_to "Submit Order", create_order_path, class:"btn btn-primary"%>`

and is taken to the create action in the controller:

```
  def create
    @order = current_user.orders.build(order_params)
    @order.save
    current_user.cart.update_inventory
    current_user.cart.reset_cart
    current_user.save
    flash[:notice] = "Your Order Has Been Submitted"
    redirect_to order_path(@order)
  end
```

Now the order has been created with all the information needed to ship the order! It has all the items (and quantities for each item) as well as the address to ship to. 

Oh, and because of this line of code in the form:

 `<%= addr.hidden_field :user_id, value: current_user.id %>`

and because a User also has_many addresses, the newly created address is ALSO associated with the current_user. All at the same time. How cool is that...

After the order is created and saved, we update the inventory, reset the cart and save the user. Then the user is redirected to the order_path where he can see the pertaining details of the order he has placed, mainly the order number, date it was placed on, all the items and amounts he purchased, the address where is being shipped to, the status of the order and the order total.

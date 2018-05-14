---
layout: post
title:      "Rails and jQuery Goodness Part I"
date:       2018-05-14 18:23:05 +0000
permalink:  rails_and_jquery_goodness_part_i
---


One of the features I implemented on my Rails and jQuery project was the ability to make asynchronous calls to a local API and dynamically display the data on the page. How? Enter jQuery/AJAX. Using these tools, we can communicate with the server, exchange data, and update the 'DOM' without having to refresh the page! All happening in the background.

Why would we want to do this?

Besides from improving the user experience (since we don't need to wait for the whole page to load), AJAX allows us to request from the server only the bits of data that we actually need. This implementation could save tons of bandwidth and increase the performance and speed of the website. 

OK, so that's AJAX. What is jQuery?

Well first and foremost, jQuery is just a JavaScript library. So it's essentially JS code that other people (smart people with experience) wrote and made available for us to use. With jQuery, we can do things like manipulating the DOM (which in simple terms is the representation of an HTML website's properties), trigger event handlers and...use Ajax! So jQuery in a way is like Ajax's big brother. At least that's one way of thinking of it.

So enough talking, let's look at some code!



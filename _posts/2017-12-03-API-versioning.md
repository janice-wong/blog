---
layout: post
title: API Versioning
---

[2 posts ago](/2017/12/01/creating-and-exposing-an-API), I created a concert API but forgot to version it. Do I even need to? What's the point of versioning APIs?

The point is that ***if*** people were to use my API, and I made changes to it (like change the name of a key), their apps would break. Many companies whose APIs are widely used need their APIs to stay current, but they can't just change them. What do they do? They **create different versions of their API**.

Here's how you do it:

1. In your `routes` file, wrap your route in a `v1` namespace. This Rails shortuct means is:
  * All routes within this namespace will be prefixed with the `/v1` path, and
  * Any controllers within this namespace can be found in the `v1` module

  ```ruby
  Rails.application.routes.draw do
    namespace :v1 do
      get '/concerts' => 'concerts#index'
    end
  end
   ```

2. In your `controllers` folder, create a `v1` folder, and move `concerts_controller.rb` into `v1`.

3. Namespace the `concerts_controller` like so:

  ```ruby
class V1::ConcertsController < ApplicationController
  def index
    concerts_array = []
    Concert.all.each do |concert|
      concerts_array << {
        name: concert.name,
        date: concert.date,
        duration: concert.duration,
        cost: concert.cost.to_f,
      }
    end
      render json: concerts_array.as_json
  end
end
   ```

Now check out your url! I first went to `localhost:3000/concerts` but got the following error:
![](/assets/img/error_3.png)


See what I did wrong? The error says `No route matches [GET] "/concerts"`. If you read my [previous post](/2017-12-02-consuming-an-API), you'll learn that **reading your errors** is everything! I realized I needed to go to `localhost:3000/v1/concerts`, and bada bing, bada boom, it worked!

![](/assets/img/namespace_api.png)

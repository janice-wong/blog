---
layout: post
title: Creating and Exposing an API
---

In this tutorial, we will create an API to expose information about concerts.

Before we start, we should understand (at a high-level) what an API is. **API** stands for Application Programming Interface, but what does that really mean? It's an interface that allows people to control and access a program someone else wrote (shoutout to [this ELI5](https://www.reddit.com/r/learnprogramming/comments/1xvm9l/can_some_eli5_what_an_api_is/)). People often use it to grab data that can be available in the form of JSON. Here's [an example of a JSON response](https://gist.github.com/hrp/900964#file-twitter-json) using the ever so popular Twitter API.

Now let's begin writing our own an API that allows other people to access our concert data.

### Creating the foundation

1. Create a rails API in Terminal using `rails new concert-api --database=postgresql` (I used Postgres as my database, but feel free to use whatever you'd like).

2. Run `rails s` in your terminal and open `localhost:3000` in your browser to ensure you've got Rails up and running. The page should say "Yay! You're on Rails!"

3. Great. Now that you've got Rails working, you can create your database with `rails db:create`.

4. Generate a `Concerts` controller and a `Concert` model. *Recall the naming convention - plural for controllers and singular for models.* My model included the attributes name, date, duration, and cost.

   When creating this, I forgot the syntax for the data structure for "cost". I knew it would be a number with 2 decimal places, so after a quick Google, I was able to update my migration file before actually migrating the database. Here's what it looked like:

```ruby
# 20171201211525_create_concerts.rb

class CreateConcerts < ActiveRecord::Migration[5.1]
  def change
      create_table :concerts do |t|
        t.string :name
        t.datetime :date
        t.integer :duration
        t.decimal :cost, :precision => 8, :scale => 2

        t.timestamps
      end
  end
end
```

### Populating the database

There are two ways to go about this - **seed the database using the `seeds.rb` file** or **create records in `rails console`**. We're going to go the first route.

{:start="5"}

5. Install the [Faker gem](https://github.com/stympy/faker) (one of my favorites) by adding it to your `Gemfile` and running `bundle install` (or `bundle` for short) in Terminal.

6. Then `require` the `'Faker'` gem in `seeds.rb`. Then create multiple entries in the `Concert` table using Faker.

   Take note of the `Faker` syntax below. `Faker` is the module being used for namespacing and `RickAndMorty`, `Date`, and `Number` are classes. Check out [my previous post on modules](/2017/11/27/modules/) for reference.

```ruby
# seeds.rb

require 'Faker'

10.times do
  Concert.create(
      name: Faker::RickAndMorty.character,
      date: Faker::Date.forward(23),
      duration: rand(1..4),
      cost: Faker::Number.decimal(2).to_f
  )
end
```

{:start="7"}

7. In Terminal, `rails db:seed` to seed your database. You should receive no feedback if this was successful. To check if it indeed worked, you can ask `rails console` to return all the Concerts using `Concert.all` or the first concert using `Concert.first`.

### Finish it off

{:start="8"}

8. This API will only have one URL that will show all concerts and their information. In `routes.rb`, create one route using a **`GET` request** to the **`/concerts` url** *routing* to the **`index` method** of the **`Concerts` controller**. (That was a lot of vocab, but it's important to know what each of those are and how to reference them.)

```ruby
# routes.rb

Rails.application.routes.draw do
  get '/concerts' => 'concerts#index'
end
```

{:start="9"}

9. Create the `index` method in the `Concerts` controller to show all concerts and their information in JSON (Javascript Object Notation) format.

```ruby
# concerts_controller.rb

class ConcertsController < ApplicationController
  def index
    concerts_array = []               # create an empty array
    Concert.all.each do |concert|     # for each concert in Concert.all
      concerts_array << {             # push into the concerts_array a hash containing the keys
        name: concert.name,           # name,
        date: concert.date,           # date,
        duration: concert.duration,   # duration, and
        cost: concert.cost.to_f,      # cost
      }
    end
      render json: concerts_array.as_json     # render the populated concerts_array as JSON
  end
end
```

{:start="10"}

10. Back in your browser, check out `localhost:3000/concerts` to ensure your concerts are indeed shown in JSON. It should look something like this:
![](/assets/img/API_example.png)

### And that's it!

You've created and exposed an API that returns information about concerts by:
1. Creating the foundation of a Rails API
2. Populating your database
3. Creating a route to render concert information in JSON

Check out my [next post](/2017-12-02-consuming-an-API) to learn how to use (or consume) this API.

*Note we did not any versioning to our API. Check out [this post](/2017/12/03/API-versioning) if you'd like to see how that's done.*

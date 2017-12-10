---
layout: post
title: RESTful APIs
---

In this post, we will adopt the REST (or RESTful) conventions in our existing Rails API.

Let's first discuss what REST is. REST stands for REpresentational State Transfer, which is essentially a set of rules invented by [Roy Fielding](http://roy.gbiv.com/) to name URLs, controllers, methods, views, and so on. In this approach, requests to particular URLs return particular information. A few things to note about REST:
- These conventions are commonly used to access APIs.
- One **advantage** of REST is that you (as a programmer) don't need to think about what to name your routes. Its naming conventions allow you to focus on other things.
- An even **larger advantage** of REST is that if you know one route, you'll likely know "all" the others, so RESTful APIs allow for much easier access.

## RESTful Routes
There are a standard of 7 routes that perform the 4 CRUD actions - Create, Read, Update, and Delete.

![](/assets/img/7_routes.png)

___

Given the scope of this post is RESTful APIs, we'll focus on the 5 below:

![](/assets/img/5_routes.png)

Note:
- The model is `Photo`.
- Each "action" is the method in the `photos_controller`.

Regarding the routes:
- `get /photos` makes a request to show all photos.
- `post /photos` makes a request to create a photo.
- `get /photos/:id` makes a request to show the photo with id `:id`.
- `PATCH` and `PUT` do the same thing, though I typically use `PATCH`.

### Let's look at an example

In your browser, go to [www.weworkremotely.com/jobs](https://www.weworkremotely.com/jobs). Given what you know about RESTful routes, it's no surprise this `GET` request to the URL `/jobs` displays all available jobs.

Now, click on a job. What do you notice about the URL? It changed to `/jobs/xxxx-<job_title>` where `xxxx` is likely what? You guessed it - the **job id**. This request follows the naming convention of the 3rd RESTful route in the table above, in which the `GET` request to the URL `/jobs/:id` shows the job with id `xxxx`.

Recall the (arguably) largest advantage discussed earlier - if you know one route, you'll likely know the others, making it easier to understand what these routes mean.

## Adding RESTful routes to our API

In the last few blogs, we created a concert API that returned an array of concerts in JSON. We had already added the first RESTful route (index) so in this tutorial, we'll continue adding the `show` and `create` routes.

### Starting with the `show`
1. In your `routes` file, add the `show` route. Recall the structure of this request:

   ```ruby
   get '/concerts/:id' => 'concerts#show'
   ```
   - `get` is the request.
   - `/concerts/:id` is the URL (`id` is a parameter being passed through this request).
   - `concerts` is the controller.
   - `show` is the method in the `concerts_controller`.
2. In your controller, add the `show` method. This method will return the **specific** concert with id `params[:id]`. `params` is a Rails method that returns the parameters hash. In our case, it may look something like `{:id => 2}`.
   - To do this, you can assign a variable (say) `concert_id` to `params[:id]`.
   - Then **find** the Concert with `concert_id` with ActiveRecord's `find_by` method, by assigning `concert` to `Concert.find_by(id: params[:id])`
   - Finally, render `concert` as JSON.

   - After refactoring, my method looked like:

   ```ruby
   def show
     concert = Concert.find(params[:id])
     render json: concert.as_json
   end
   ```

{:start="3"}

3. Test your `show` route and method in your browser by going to `localhost:3000/v1/concerts/xx` where `xx` is some concert ID.
   - Note, you can either find a concert ID by going into Rails console and grabbing `Concert.first` or `Concert.last`. You can also add the ID in your index method, which is what I did. I made `id: concert.id` the first element of each concert hash.
   - If your HTTP request was made successfully, you should see something like:

   ![](/assets/img/concert_50.png)

### Adding the `create` route

Each HTTP request (URL typed in the browser) is a `GET` request. How do we make a `POST` request? Recall we previously made Unirest calls in a Ruby file (`frontend.rb`) in the [Consuming an API](/2017/12/02/consuming-an-API) post (no pun intended). In addition to making `Unirest.get` requests, we can make `Unirest.post` requests! Thanks Unirest.

1. Start by creating the `POST` request in your `routes` file. You can do this by following the naming convention in the table above.
2. This will require a `create` method in your controller, in which we'll create a concert given user input. We'll get to this soon, but can leave it empty for now.

At this point, we can create a `Unirest.post` request in our Ruby file. This request will take in a parameters hash, whose keys are the attribute names and values are user input.

{:start="3"}
3. In `frontend.rb`, feel free to remove or comment out all previously written code (except for `require 'unirest'`).
4. `require 'pp'` at the top to use Ruby's pretty-print - this will pretty up any hash we want to print out.
5. Create an empty parameters hash. I called mine `the_params`.
6. For each of the attributes, ask the user to provide a value. Then save that value in your hash with key `attribute_name`.
7. Pretty print `Unirest.post("localhost:3000/<url>", parameters: the_params).body`. My `frontend.rb` eventually looked like this:

```ruby
require 'unirest'
require 'pp'

system "clear" #=> Clears your terminal

the_params = {}

p "Create a new concert by providing the following:"
p "Name"
the_params[:name] = gets.chomp
p "Date in MM/DD/YYYY format"
the_params[:date] = gets.chomp
p "Duration (hrs) in the form of an integer"
the_params[:duration] = gets.chomp
p "Cost in the form xx.xx"
the_params[:cost] = gets.chomp

p "-" * 100
p "Success! You've created the concert:"

pp Unirest.post("localhost:3000/v1/concerts", parameters: the_params).body
```

I bet you wish we were done, but we're not! Yes, we've created the `POST` request, which would route to the `create` method in our controller, but our `create` method is still empty! The goal is to create a concert whose key-value pairs are in the `the_params` hash that was passed in through the `POST` request.

{:start="8"}
8. In the `create` method, create a Concert where each attribute's value is `params[:<key_name_of_the_params_hash]`.
9. Assign this concert to a variable `concert` and render it as JSON.
   Check out my `create` method below:

```ruby
def create
  concert = Concert.create(
    name: params[:name],
    date: DateTime.strptime(params[:date], "%m/%d/%Y"),
    duration: params[:duration].to_i,
    cost: params[:cost].to_f
  )

  render json: concert.as_json
end
```
- `DateTime.strptime` converts the user's input to Ruby's `timestamp` data type
- My `duration` attribute is of the `Integer` data type, so I need to convert it into an integer given all parameters are passed in as strings. Same goes for `cost` which has a `Float` data type.

{:start="10"}
10. Test out your `POST` request (aka `create` method) by running `frontend.rb`! If you did it correctly, it might look like this:
![](/assets/img/created_concert.png)

You can take a breather now. We're finally done. We have yet to add the last two RESTful routes `update` and `delete`, but those will come soon.

As these RESTful naming conventions become second nature to you, you'll begin to learn how useful they are and why they've become such a popular and standard approach for building APIs.

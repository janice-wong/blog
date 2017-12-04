---
layout: post
title: Consuming an API
---

In the [previous post](/2017/12/01/creating-and-exposing-an-API), we learned how to create and expose an API. Normally, there would be a separate front-end containing HTML, CSS, and JS to display data received from the API, but for the sole purpose of consuming our API, we'll just use a regular ol' Ruby file.

At a high level, we'll:
1. Make a call to our API,
2. Test that our call is successful, and
3. Play around with our response (or data) to return:
  * Concert names
  * Total cost of all concerts
  * Prompt to ask user to select multiple concerts

### Let's begin

1. Ensure the local server for your API is running so that we can make calls to it. You do this by "`cd`-ing" into your API app, and running `rails server` (or `rails s` for short). **If** your local server isn't running, you'll get an error like:

```ruby
Errno::ECONNREFUSED: Failed to open TCP connection to localhost:3000 (Connection refused - connect(2) for "localhost" port 3000)
```

{:start="2"}

2. Create a `frontend.rb` file in the root directory of your project.

### Make the call

{:start="3"}

3. You can use any gem you'd like to consume this API. I used the [Unirest gem](https://github.com/Kong/unirest-ruby). Be sure to install it and require it at the top `frontend.rb`.

4. In `frontend.rb`, use Unirest's `get` method to make a request to your API.

   If you read the `Unirest` documentation, `Unirest.get("<url>")` will return details about the response of the request. These include the code, headers, etc. If you'd like to see what the response looks like, you can run `Unirest.get("<url>")` in `rails console` (or `rails c` for short). In our case, we **only want the body**, hence the `.body` suffix.

```ruby
concerts = Unirest.get("localhost:3000/v1/concerts").body
```

{:start="5"}

5. To see if your response is successful, test your Ruby file by printing some outputs using `concerts`. You should be fine if you don't get any errors.

```ruby
# frontend.rb

require 'unirest'

concerts = Unirest.get("localhost:3000/v1/concerts").body
p concerts.class #=> Array
p concerts #=> [{"name" => "xx", "date" => "xx", etc.}, {}, ...]
```
### Play with the data

{:start="6"}

6. Now that you've got a successful response, you can begin to mess around with the data. Let's try to print all the concert names. I first did this:

```ruby
def concert_names
  concerts.each do |concert|
    p concert.name
  end
end
```

But got the error:
![](/assets/img/error_2.png)

What was wrong? After **reading the error** that said `undefined local variable or method 'concerts'`, I realized that `concerts` could only be accessed within my method if it were an instance variable, so I updated my code:

```ruby
@concerts = Unirest.get("localhost:3000/v1/concerts").body

def concert_names
  @concerts.each do |concert|
    p concert.name
  end
end
```

Did it work yet? Nope. I got ANOTHER error:
![](/assets/img/error_1.png)

Before steam came out of my ears from all these dang errors, I looked back at my API (at `localhost:3000/concerts`) and realized that `name` is really a key in each `concert`, so I updated my method and it worked!

```ruby
def print_all_concerts
  p "Here are all the concerts:"
  @concerts.each do |concert|
    p concert['name']
  end
end
```

{:start="7"}

7. Time to try my hand at printing the total cost of all concerts. My first attempt was my `total_concert_cost`, but eventually refactored to `fancy_total_concert_cost`. **Hint #1**: Ruby methods like [inject](https://ruby-doc.org/core-2.4.0/Enumerable.html#method-i-inject) come in super clutch in times like these.

```ruby
def total_concert_cost
  total = 0
  @concerts.each do |concert|       # for each concert in @concerts
    total += concert['cost']        # add concert cost to running total
  end
  "Concert costs total #{total.round(2)}"
  # can say return total but 'returns' are implicit in Ruby. this is not the case in other languages like JS
end

def fancy_total_concert_cost
  total = @concerts.inject(0) { |sum, concert| sum + concert['cost'] }
  "Concert costs total #{total.round(2)}"
end

p fancy_total_concert_cost #=> "Concert costs total 418.72"
```

{:start="8"}

8. Last task: Create a prompt to allow the user to select multiple concerts. I used the [tty-prompt gem] for this. After reading the documentation, I refactored `concert_names` below and created the prompt. **Hint #2**: The [map Ruby method](https://ruby-doc.org/core-2.2.0/Array.html#method-i-map) is also very useful.

```ruby
def concert_names
  @concerts.map { |concert| concert['name'] }
end

prompt = TTY::Prompt.new
prompt.multi_select("Select concerts:", concert_names)
```

The result looks a little something like this:
![](/assets/img/tty-prompt-example.png)

Pretty snazzy huh?! I think so. Feel free to practice creating other methods that (say) only return concerts that cost less than $50, or return only the concerts within a specific date range.

### Major Lesson Learned

**Read your errors**. Ruby errors are pretty descriptive - they tell you which line returned an error and why you received it, so use this to your advantage! Good luck.

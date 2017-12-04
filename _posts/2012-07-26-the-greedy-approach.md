---
layout: post
title: The Greedy Approach
---

I've begun practicing algorithm problems using [Interview Cake](https://www.interviewcake.com) and solved my first one today (though it wasn't the most efficient). Beware, these are quite difficult - they require you to solve their problems with consideration of **both time *and* space efficiency**.

### The Problem
Say you have an array of yesterday's stock prices where:
- The indices are the minutes past opening time (9:30am local)
- The values are the stock price at that time,

Determine the maximum profit from one purchase and one sale. Note, you must buy *before* you sell.

For example:

```ruby
stock_prices_yesterday = [10, 7, 5, 8, 11, 9]

get_max_profit(stock_prices_yesterday)
# returns 6 (buying for $5 and selling for $11)
```

This problem can be solved in O(n) time and O(1) space.

### My Approach
Here's what I knew:
- This problem would require a loop to iterate through the stock prices in the array.
- I'd need to maintain some variable `max_profit` to hold and maintain the highest profit as I ran through each iteration.

If at each iteration, I could:
- Find the differences between the price of my current index and each of the previous prices
- Update `max_profit` to the maximum of these differences exceeded its current value

I could then find `max_profit`. Using the example above, here's what I would want at each iteration:

```ruby
stock_prices_yesterday = [10, 7, 5, 8, 11, 9]

# 1st iteration
stock_prices = [10, 7]
differences = [7 - 10] = [-3]
max_profit = -3

# 2nd iteration
stock_prices = [10, 7, 5]
differences = [5 - 10, 5 - 7] = [-5, -2]
max_profit = -2

# 3rd iteration
stock_prices = [10, 7, 5, 8]
differences = [8 - 10, 8 - 7, 8 - 5] = [-2, 1, 3]
max_profit = 3

...
```

### My Solution

While I knew this approach wasn't entirely efficient because it involved comparing every price to all the ones before it, I drafted my solution:

```ruby
def get_max_profit(stock_prices_yesterday)
  max_profit = stock_prices_yesterday[1] - stock_prices_yesterday[0]
  stock_prices_yesterday[2..-1].each_with_index do |price, index|
    index += 2
    diff = stock_prices_yesterday[0..index].map { |x| price - x }.max
    max_profit = diff if diff > max_profit
  end
  max_profit
end
```

I'll admit I caved and read through the Interview Cake solution once I was able to draft this successful-ish solution. Side note: 'ish' because
- I didn't solve it in the stated time and space efficiency
- I missed considering the case of a stock_prices array whose length < 2

Anyway, through reading the solution, I learned about the **greedy approach**!

> A **greedy** algorithm iterates through the problem space taking the
optimal solution "so far," until it reaches the end.

> Suppose we could come up with the answer in one pass through the input, by simply updating the ‘best answer so far’ as we went. What **additional values** would we need to keep updated as we looked at each item in our set, in order to be able to update the **‘best answer so far’** in constant time?

In this problem, we would need to keep track of:
- `min_price` seen so far, representing the "additional value"
- `max_profit` given prices seen so far, representing the "best answer so far"

Here is their solution:

```ruby
def get_max_profit(stock_prices_yesterday)
  if stock_prices_yesterday.length < 2
    raise ArgumentError, 'Getting a profit requires at least 2 prices'
  end

  # we'll greedily update min_price and max_profit, so we initialize them to the first price and the first possible profit
  min_price = stock_prices_yesterday[0]
  max_profit = stock_prices_yesterday[1] - stock_prices_yesterday[0]

  stock_prices_yesterday.each_with_index do |current_price, index|

    # skip the first time, since we already used it when we initialized min_price and max_profit
    next if index.zero?

    # see what our profit would be if we bought at the min price and sold at the current price
    potential_profit = current_price - min_price

    # update max_profit if we can do better
    max_profit = [max_profit, potential_profit].max

    # update min_price so it's always the lowest price we've seen so far
    min_price  = [min_price, current_price].min
  end

  max_profit
end
```

You can see here the loop is being iterated through only once and
only 5 variables are holding integer values, thus satisfying both O(n)time and O(1) space.

After having gone through multiple Interview Cake problems since this one, I've definitely seen the benefits of using of the **greedy approach** to solve with efficiency in mind.

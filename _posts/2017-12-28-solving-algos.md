---
layout: post
title: Solving Algorithms
---

Ah, the dreaded, infamous algo problems. Why do they even exist? Who even invited them to the party?

I'll stop before I get too carried away. Despite their unpopular reputation, I actually love algos, even in interviews! They test your ability to reason through a problem, challenging not only how you think, but how you handle stress. In this post, I'll discuss my approach to solving the **Palindrome Problem**, which I've gotten on multiple interviews so it must be a popular one.

## The Problem
Given a string of lowercase letters, determine if it's a palindrome. Solve this both iteratively and recursively.
{:.lead}

### Iterative Solution

I'm going to solve this how I would typically solve this in an interview.

> I'd like to look at an example, say 'madam'. I imagine some type of loop that iterates both forwards and backwards to compare the first and last letter. If those are the same, move inwards and compare the second and second to last letter. Continue moving inwards until you reach the inner most letters. If at any point the letters do not match, return false. Otherwise, return true.

After saying this aloud to help myself and the interviewer make sense of my thought process, I'd write this out in pseudo code:

```ruby
# loop through string forwards and backwards by first defining an index i = 0
# compare first and last letter
# if same, increase i by 1
# if different, return false
# do this half as many times as the length of the string
# return true once the loop is complete - at this point, all of the letters will have been matched
```

At this point, I can write out my code to test its success.
```ruby
def palindrome(str)
  i = 0
  (str.length / 2).times do
    if str[i] == str[str.length - 1 - i]
      i += 1
    else
      return false
    end
  end
  true
end
```

I always make sure to test inputs of varying attributes (in this case, string length) and those that will result in varying outputs. See below:

```ruby
p palindrome('madam')   #=> should return true
p palindrome('madame')  #=> should return false
p palindrome('racecar')  #=> should return true
p palindrome('rececar')  #=> should return false
```

### Recursive Solution

> At each iteration of the iterative solution, I performed the same task - comparing the outermost letters and moving inwards if they matched. This can be replaced by a recursive function whose input updates to `str[1..-2]` after each comparison. `str[1..-2]` represents the remaining string after dropping the outermost letters.

Pseudo-code:
```ruby
# if the first and last letters match, drop them and perform the recursive function on the remaining string
# if the first and last letters don't match, return false
# establish base case - return true if length of the string <= 1
```

Code:
```ruby
def palindrome(str)
  return true if str.length <= 1
  if str[0] == str[-1]
    palindrome(str[1..-2])
  else
    return false
  end
end
```

---

We can always analyze the efficiency of each solution, but I'll leave that scope for another post. It's natural to feel nervous and intimidated, but just keep in mind every whiteboarding experience is a learning opportunity - each will give you awareness of how you think and perform under pressure. Best of luck in all your whiteboarding endeavors :)

---

Here are some resources I love to practice algos:
- [HackerRank](https://www.hackerrank.com) / [CodeFights](https://codefights.com) / [CodeWars](https://www.codewars.com)
- [Project Euler](https://www.projecteuler.net) - pretty mathy, which I love!
- [Interview Cake](https://www.interviewcake.com) - pretty complex, but demonstrates multiple problem-solving approaches with consideration of time and space efficiency


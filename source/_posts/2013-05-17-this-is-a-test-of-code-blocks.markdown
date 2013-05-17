---
layout: post
title: "this is a test of code blocks"
date: 2013-05-17 16:44
comments: true
categories: ruby
---
Check out my fizzbuzz solution.
``` ruby FizzBuzz.rb
(1..100).each do |number|
  output = ""
  output = "fizz" if number.%(3).zero?
  output += "buzz" if number.%(5).zero?
  output = number if output.empty?
  puts output
end
```

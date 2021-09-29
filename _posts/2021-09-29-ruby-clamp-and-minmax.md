---
layout: post
title: "Exploring Ruby's `clamp` and `minmax` methods"
date: 2021-09-29
description: "An introduction into Ruby's Comparable#clamp and Enumerable#minmax methods"
---

I came across an interesting problem recently where I was given a value and an array of "buckets" and I basically needed to return which bucket the value was closest to.

This made me extremely excited at the prospect of having a practical use case for Ruby's `Comparable#clamp` and `Enumerable#minmax`!

In a simplified example I ended up playing around with an implementation similar to this:
```rb
buckets = [1, 2, 3, 4]
0.clamp(*buckets.min_max)
=> 1
```

Super cool stuff! Let's dig a bit deeper into both `clamp` and `minimax`!

## Comparable#clamp

Ruby's [`Comparable#clamp`](https://ruby-doc.org/core-3.0.2/Comparable.html#method-i-clamp) "clamps" a value within a provided minimum and maximum value.

```rb
(1.2).clamp(1.3, 1.5)
=> 1.3

(1.6).clamp(1.3, 1.5)
=> 1.5

(1.4).clamp(1.3, 1.5)
=> 1.4

('a').clamp('b', 'd')
=> "b"
```

As of Ruby 2.7, `clamp` can also take a range:
```rb
6.clamp(3..5)
=> 5

1.clamp(2..)
=> 2

2.clamp(..1)
=> 1
```

## Enumerable#minmax

There's also [`Enumerable#minmax`](https://ruby-doc.org/core-3.0.2/Enumerable.html#method-i-minmax) which gets both the minimum and maximum value in a given enumerable:
```rb
[1,2,3].minmax
=> [1, 3]

%w[b c a].minmax
=> ["a", "c"]
```

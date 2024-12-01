---
layout: post
title:  "Advent of Code Day 1"
date:   2024-12-01 15:00:49 -0500
categories: advent-of-code programming
---

Another December, another [Advent of Code](https://adventofcode.com)! I've been
casually participating in this festive programming challenge for a few years,
though I have never actually finished all of the problems. This year I've decided
to accompany my solutions with explanations. I've learned new things about my
favorite programming language, Ruby, every year that I participate.

Without further ado, here is my [full solution for day 1](https://github.com/lancenewman/advent-of-code/blob/main/2024/day_1/solution.rb).

```rb
# USAGE: $ ruby ./solution.rb input.txt
LINES = File.readlines(ARGV[0])

group_1 = []
group_2 = []
LINES.each do |line|
  id_1, id_2 = line.split.map(&:to_i)
  group_1 << id_1
  group_2 << id_2
end

p "part 1: #{ group_1.sort.zip(group_2.sort).map { |id_1, id_2| (id_1 - id_2).abs }.sum }"

similarity_score = 0
group_2_tally = group_2.tally
group_1.each do |id_1|
  similarity_score += group_2_tally[id_1].nil? ? 0 : id_1 * group_2_tally[id_1]
end

p "part 2: #{similarity_score}"
```

As usual, day 1 was a great warm-up to get back into the logic puzzle programming 
space. When I was first learning Ruby, I was amazed at how many times these 
puzzles were solvable in one line using all of the great methods the 
[Enumerable](https://ruby-doc.org/core-2.7.2/Enumerable.html) mixin. The code
for part one isn't the most readable, but I always have a good time coming up
with clever method chains like this.

Today's cool method comes from part 2 – [`Enumerable#tally`](https://ruby-doc.org/core-2.7.2/Enumerable.html#method-i-tally). This can be used to quickly identify the number of times
that each element appears in an array. It made the bulk of the work for part 2
relatively easy. I didn't spend as much time code-golfing part 2 because my
newborn started to wake up. 

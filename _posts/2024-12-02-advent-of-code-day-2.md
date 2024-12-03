---
layout: post
title: "Advent of Code Day 2"
date: 2024-12-02 08:22:56 -0500
categories: advent-of-code programming
---

TLDR: More enumerable manipulation. Skip right to [my solution](#solution).

## Part 1

Today's [Advent of Code](https://adventofcode.com) puzzle input is a list of
integer arrays, called reports, with each report on a new line. For part 1, a
report is considered "safe" if it is either strictly increasing or decreasing,
and the amount of change between each element is no more than 3.

Let's look at a few examples.

1. `1 2 3 4`
2. `4 4 3 1`
3. `6 4 3 1`

Example 1 is safe because all elements are increasing and each element differs
by no more than 3. Example 2 is _not_ safe because there is no change between
the first two elements. Finally, example 3 is safe because it is strictly
decreasing and all elements differ by no more than 3.

In order to evaluate all of these conditions in one pass through the array, we
need to observe consecutive elements in groups of 3. For each group of
elements `a`, `b`, and `c`:

```ruby
# Check that they are all increasing or decreasing
(a < b && b < c) || (a > b && b > c)

# Check that the middle element, b, is no more than 3 away from the element on
# either side.
((a - b).abs <= 3) && ((b - c).abs >= 3)
```

Simple enough, but how do we iterate through the elements in groups of 3?
Introducing today's cool method, [Enumerable#each_cons](https://apidock.com/ruby/Enumerable/each_cons),
which allows us to do exactly what we need.

```ruby
# #each_cons example
> [1, 2, 3, 4].each_cons(3) { |a, b, c| p "#{a}, #{b}, #{c}" }
"1, 2, 3"
"2, 3, 4"
=> [1, 2, 3, 4]
```

We can then use the `#all?` method to apply our conditions to each group of
elements in the array.

```ruby
def safe?(report)
  report.each_cons(3).all? { |a, b, c| ((a < b && b < c) || (a > b && b > c)) && ((a - b).abs <= 3 && (b - c).abs <= 3) }
end
```

Obviously this isn't super readable but it concisely evaluates all of the report
constraints.

## Part 2

Part 2 introduces a tolerance threshold for the reports. A report is now
considered safe if it meets all constraints with up to one element removed.

There's probably plenty of room for optimization on this part, but my newborn
started waking up so I went with a simple brute-force approach. Basically, we're
just re-using the implementation from part 1 on sub-arrays with offending
elements removed.

```ruby
def safe_with_tolerance?(report)
  return true if safe? report

  report.size.times do |i|
    report_cpy = report.dup
    report_cpy.delete_at i
    return true if safe? report_cpy
  end
  false
end
```

The only annoying part about this was dealing with `#delete_at` mutating the
array that it operates on and returning the deleted element instead of the
array without that element.

## Solution

Full [solution for today's problem on GitHub](https://github.com/lancenewman/advent-of-code/blob/main/2024/day_2/solution.rb).

```ruby
LINES = File.readlines(ARGV[0])
reports = LINES.map { |report| report.split.map(&:to_i) }

# Part 1
def safe?(report)
  report.each_cons(3).all? { |a, b, c| ((a < b && b < c) || (a > b && b > c)) && ((a - b).abs <= 3 && (b - c).abs <= 3) }
end

p "Part 1: #{reports.map { |report| safe? report }.tally[true]}"

# Part 2
def safe_with_tolerance?(report)
  return true if safe? report

  report.size.times do |i|
    report_cpy = report.dup
    report_cpy.delete_at i
    return true if safe? report_cpy
  end
  false
end

p "Part 2: #{reports.map { |report| safe_with_tolerance? report }.tally[true]}"
```

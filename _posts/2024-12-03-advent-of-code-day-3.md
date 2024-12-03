---
layout: post
title: "Advent of Code Day 3"
date: 2024-12-03 07:49:42 -0500
categories: advent-of-code programming
---

TLDR: Regular expressions are awesome, edge cases in these puzzles are not. Skip to [my solution](#solution)

## Part 1

Today we have the first [Advent of Code](https://adventofcode.com) regular
expression challenge of 2024! As usual, the first one was a great warm up and I'm
looking forward to exploring them a bit more later into the month.

Our input for the puzzle is a string representing corrupted computer memory for
a program designed to multiply integers together. For example:

```
switch:%mul(4,5)then[a-4532}do_not()mul(8,4)
```

Our job in part 1 is to extract the `mul(a,b)` instructions, evaluate them by
multiplying `a` and `b`, and finally report the sum of the results. The Ruby
implementation is trivial using today's cool method, [`String#scan`](https://apidock.com/ruby/String/scan).

```ruby
memory.scan(/mul\((\d{1,3}),(\d{1,3})\)/)
```

When `memory` contains the contents of our puzzle input, this will return a 2D
array with each element containing the multiplication operands. The holiday magic
comes from capture groups, denoted by unescaped parentheses. When a regular
expression containing groups is used with `#scan`, the result is the contents of
those groups rather than the total matched string. Let's build up the expression
in parts:

```ruby
# Matches a digit (using \d) between 1 and 3 characters long (0-999). This regex
# is used twice, once for each operand.
/\d{1,3}/

# Matches the entire mul(a,b) expression, using the digit regex above.
/mul\(\d{1,3},\d{1,3}\)/

# Add in capture groups using parentheses to extract each operand
/mul\((\d{1,3}),(\d{1,3})\)/
```

Now all that's left to do is convert the operands to integers, multiply them,
and then calculate the sum. Easy enough with Ruby `Enumerable` built-ins.

```ruby
memory.scan(/mul\((\d{1,3}),(\d{1,3})\)/).map { |operands| operands[0].to_i * operands[1].to_i }.sum
```

That's all there is to it. I don't typically add any complicated edge-case
handling to these solutions because the puzzle makes several guarantees about
the input format (I'm big fan of [YAGNI](https://en.wikipedia.org/wiki/You_aren%27t_gonna_need_it)).

## Part 2

Part 2 introduces additional pre-processing on the corrupted memory before
executing the same evaluation from part 1. There are now additional instructions
we need to pay attention to in the input: `do()` and `don't()`. These enable and
disable evaluation of `mul(a,b)` expressions, respectively. In my solution, I
opted to do some simple string splitting on the new instructions that allows us
to identify memory fragments that are operable (i.e. enabled with `do()`).

The first step is to split the entirety of our puzzle input on any `don't()`
instructions. This gives us an array of memory fragments that start with
disabled instructions. We can further process those fragments to remove the
disabled part, recreating our program memory with only the instructions that are
enabled with `do()`.

The instructions start out enabled, so we can automatically include the first
fragment in the operable memory. After that, we split each fragment on the first
`do()` instruction. The original memory was split on `don't()`, so the entire
fragment after the first `do()` is operable.

```ruby
fragments = memory.split("don't")
operable_memory = fragments[0]
fragments[1..-1].each do |fragment|
    fragment_parts = fragment.split("do()", 2)
    next if fragment_parts.size <= 1
    operable_memory += fragment_parts[1]
end
```

There's probably a way to do this using a more complicated regular expression
such that all of the processing can be completed with a single `#scan` call, but
I've found that trying to do more than one or two things with a single regex
introduces a level of complexity that typically isn't worth it.

## Solution

Full [solution for today's problem on GitHub](https://github.com/lancenewman/advent-of-code/blob/main/2024/day_3/solution.rb).

```ruby
memory = File.read(ARGV[0])

def part_1(memory)
  # String#scan does the heavy lifting by using regular expression capturing.
  memory.scan(/mul\((\d{1,3}),(\d{1,3})\)/).map { |operands| operands[0].to_i * operands[1].to_i }.sum
end

p "Part 1: #{part_1 memory}"

def part_2(memory)
  fragments = memory.split("don't()")

  # Operation enabled at start, so we automatically include the first fragment
  operable_memory = fragments[0]
  fragments[1..-1].each do |fragment|
    # The first element of this array is always disabled because it immediately
    # follows a don't() instruction. If there's only one element, then the
    # entire fragment is disabled.
    fragment_parts = fragment.split("do()", 2)
    next if fragment_parts.size <= 1

    # We split on don't() originally, so everything after the first do() is operable
    operable_memory += fragment_parts[1]
  end

  # Now that we've reconstructed memory to only include operable statements,
  # it can be processed just like part 1.
  part_1 operable_memory
end

p "Part 2: #{part_2 memory}"
```

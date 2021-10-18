---
layout: post
title:  "The Beauty of Programming"
categories: rust_shell
---

There is one question that every CS major must eventually ask themselves:

## Why study computer science?

Inevitably, the 2021 answer to this ticks at least one of three boxes:

1. You like problem-solving.
2. You like money.
3. Your parents (or a similarly influential set of authority figures) want you to like money.

Doubtless, we've all been told, explicitly or implicitly,
that you need to check off Point 1 before you should
even consider a career in software. 
If you don't enjoy your work, you will rapidly burn out
and become a husk of an office worker, soullessly
clacking away at a keyboard to enrich some faraway
billionaires.

But I also believe that
as software engineers, the programming languages that 
we use in our daily lives can profoundly shape our
understanding of Point 1. The digital world that
we live in runs on languages like Java and C++,
and as much as we stand in awe of the pillars
that they erect, we also grimace at the pains
that they bring us. At this point, it's become
somewhat of a trope: Null pointer exceptions,
templates, segfaults, factories, getters and setters.

With OOP having entrenched itself as the canonical
style of programming in industry, scores of future
programmers enter university programs, trudging
through *Introduction To Computer Science* courses
and wondering if there's another way.

## Functional Programming: An Arcane Art

UC Berkeley's introductory CS course, CS61A,
instilled in me a vision of the greener grass
on the other side. Unlike many of my classmates,
I enrolled in it without having printed a single
"Hello world!" in my life. I had no preconceptions
about what programming was or what it could be,
and while students with prior programming
exposure to the canonical OOP languages could do 
loop-de-loops around `for` loops,
almost none of us were prepared for the absolute
f*ckery that was function currying.

{%- highlight python -%}
def create_adder(x):
    def add(y):
        return x + y
    return add 

add_2 = create_adder(2)
add_2(3) == 5
{%- endhighlight -%}

Some of us enjoyed it. Most of us hated it. Few
understood *why* we were learning this arcane
art, when we could just as easily write `2 + 3 == 5`.

The tragedy is that the beauty of functional programming
in CS61A is rapidly snuffed out by the practical reality
of Berkeley's following programming course, CS61B, which
presents to students the absolute carnage of writing
large-scale projects in the OOP posterboy language of
Java. The vast majority of Berkeley CS students,
or at least my peers, take the Pythonic exercises
of our first semester to be something akin
to a tiger jumping through hoops: it's cool, but
useless; why curry functions when loops and if-else
statements can get you everything you want?

The sad part is, this kind of thinking isn't even
entirely wrong. A software engineer could theoretically
spend a lifetime without ever having to curry
a single function, but it's tough to imagine a modern
day career in programming that entirely avoids
the object-oriented paradigm.

## The Java Experience

Let us remind ourselves how to print "Hello world!" in Java
again.

{%- highlight java -%}
class Main {
    public static void main(String[] args) {
        System.out.println("Hello world!");
    }
}
{%- endhighlight -%}

Many a programmer has asked themselves something roughly along
the lines of: 
"The !%#&? Why must it be so complicated?"

The Java experience is built around worshipping the
object-oriented mode of thinking; even the simplest program
imaginable must be built within a class to properly compile.
And classes notwithstanding, it's painfully true that
Java is quite a verbose language. Queue the memes of
buying a second monitor just to properly program in
Java.

As a new, budding programmer, I was horrified to think
that this could be my future. After experiencing the pristine
conciseness of Python, I loathed to think that a
career in programming could consist of wading through
boilerplate. I missed the pythonic shortcuts: 
sane iterators, list comprehensions, the whole #!.

But I could also appreciate the security that Java's
static typing afforded me; I grew to appreciate the
compiler always watching my back, making sure I
never tried to subtract strings. And so, ever so slowly,
I was coaxed into OOP-design principles, writing factory
functions, getters and setters, and of course always
dutifully inserting `if (x == null) return null;`
at the beginning of my functions.

## The Haskell Promise

I had been barely coding for 3 months when I first
stumbled upon [Advent of Code](https://adventofcode.com).
Long story short, it's a series of coding challenges
that come out in the days preceding Christmas, and
gradually builds in intensity such that it can provide
a happy spot for almost any programmer.

After completing the day's AoC challenge, I would always
go on r/adventofcode and see what solutions the world's
best programmers could come up with. Consistently,
I always found the FP solutions to be the most
concise and beautiful ones.

Let's take a look at the 
[2020 Day 6](https://adventofcode.com/2020/day/6) problem.
Spoilers: For the first part, you essentially tokenize the input by double
newlines (`\n\n`), and sum over all unique characters that
appear at least once in each token.
For the second part, you do the same but instead sum
over unique characters that appear on all lines
in each token.

My solution was as follows:

{%- highlight python -%}
from functools import reduce

with open('inputs/day6.txt', 'r') as file:
    groups = file.read().split('\n\n')
    num_anyone = 0
    num_everyone = 0
    for g in groups:
        p_sets = [set(s) for s in g.split()]
        S_anyone = reduce(set.intersection, p_sets)
        S_everyone = reduce(set.union, p_sets)
        num_anyone += len(S_anyone)
        num_everyone += len(S_everyone)
    print(f"The answer to part 1 is {num_everyone}")
    print(f"The answer to part 2 is {num_anyone}")
{%- endhighlight -%}

Not the best, but I was pretty happy with where I ended up.
But I was always struck by the alien elegance of Haskell
solutions. Let's look at one, courtesy of u/mount-cook
on Reddit:

{%- highlight haskell -%}
import Data.List
import Data.List.Split

parse :: String -> [[String]]
parse = map lines . splitOn "\n\n"

solve1 :: [[String]] -> Int
solve1 = sum . map (length . foldl1 union) 

solve2 :: [[String]] -> Int
solve2 = sum . map (length . foldl1 intersect)

day06a = show . solve1 . parse
day06b = show . solve2 . parse
{%- endhighlight -%}

This solution computes pretty much the same result
as mine, but does it without any help from loops
and with no need for any mutable variables. Awesome!

Of course, Python's flexibility means that an entirely
FP Python solution would've been possible, but I always
found myself returning to the crutch of loops when
formulating my solutions, even for problems that lended
themselves very naturally to functional solutions.
Still, coding programs like these (or seeing others' code)
 gave me a much needed
breather from the inflexible, uncompromising OOP
approach that Java outlines.
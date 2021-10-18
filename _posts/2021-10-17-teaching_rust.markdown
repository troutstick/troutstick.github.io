---
layout: post
title:  "The Beauty of Programming"
categories: rust_shell
---

# Why study computer science?

Inevitably, the answer to this ticks at least one of three boxes:

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
+++
title = "Functional programming with match"
date = 2025-12-05
tags = ["coding", "python", "functional-programming"]
draft = false
+++

Python 3.9 became [EOL](https://devguide.python.org/versions/) last week, which means you now have no reason to
not write `match` statements anymore. While python's `match` pattern
matching isn't as sophisticated or flexible as Elixir's or Haskell's,
it's still a massive improvement over other mainstream
languages.


## Recap {#recap}

This is a recap if you're not a pythonista; unpacking has existed
since the early days. You've probably already done things like this

```python
a, b = (1, 2)
print(a + b)
```

That's great, and you've probably already tried iterable unpacking ([PEP 3132](https://peps.python.org/pep-3132/))

```python
first, *rest = "abc"
print(first == 'a')
```

There is a minor issue that if there is no pattern to be matched you'd
need some sort of error-handling, which makes the code unnecessarily
verbose and defeats the purpose of unpacking; `match` remedies that
issue.


## Some examples {#some-examples}

You might have heard of the [99 problems in prolog](https://www.ic.unicamp.br/~meidanis/courses/mc336/2009s2/prolog/problemas/), which was later
rewritten for lisp and other languages. It's one of the best resources
for learning about list operations, recursion, and functional
programming in general. Python is no match (pun intended) compared to prolog, but it can still operate on lists in a somewhat
similar fashion.

Another thing to note is that python does <span class="underline">not</span> opimize tail calls,
which means you shouldn't actually recurse like this, and if you're
adament, you should at least increase the recursion limit and accept
the crashes

```python
from sys import setrecursionlimit
setrecursionlimit(2000)
```


### Last element {#last-element}

What if `my_list[-1]` didn't exist? You can define it like this:

```python
def last(my_list: list) -> list:
    match my_list:
        case [last_item]:
            return last_item
        case [_, *tail]:
            return last(tail)
        case _:
            raise Exception("😕")

assert last([1,2,3]) == 3
```

The first case is the simple single-element case and the second case
matches bigger lists. We discard the "head" of the list and recurse
with the tail. Last case is for anything else like empty lists.


### Consecutive Duplicate Elimination {#consecutive-duplicate-elimination}

Or "compression" as it's called in the exercise.

```python
def compress(my_list: list) -> list:
    match my_list:
        case [] | [_]:
            return my_list
        case [x, y, *rest] if x == y:
            return compress([x, *rest])
        case [x, y, *rest]:
            return [x] + compress([y, *rest])
```

and you can test it out for yourself

```python
assert compress([1,1,1,1,2,3,3,1,1,4,5,5,5,5]) == [1,2,3,1,4,5]
```

The first case reads "If `my_list` has 0 or 1 element, return it as it
is," which is fairly self-explanatory. The beauty of python's `match`
lies in the 2nd case where we use a "guard" for comparison. This terse
syntax relieves you of having to create separate if
statements.[^fn:1]


### Other cases {#other-cases}

I won't spoil other problems for you. If you haven't gone through that
problem list, definitely try it with python. A lot of `itertools` and
`functools` already do the heavy lifting for you. After your leetcode
and [aoc](https://adventofcode.com/), try these problems for a change to review your FP fundamentals.


## Real-world example {#real-world-example}

While list destructuring is already plenty helpful, the line of code
that made me want to write this post was [from the langchain
docs](https://docs.langchain.com/oss/python/integrations/chat/ollama):

```python
if isinstance(result, AIMessage) and result.tool_calls:
    print(result.tool_calls)
```

This is _understandable_, and doesn't look "ugly" at all, but as soon as I read
`if isinstance...` my first question was "Why not match?"

```python
match result:
    case AIMessage(tool_calls=calls):
        print(calls)
```

Ignoring the fact that the line isn't long, there are fewer
characters, etc, you also have a clear view of what the value within
the object is and you can bind it to another name in that line.


## Why match isn't everywhere {#why-match-isn-t-everywhere}

Despite being around since Python 3.10, which is 4 years by now, the
average developer still treats it like switch statements instead of
actually utilizing its pattern matching capabilities. My speculation
is that the average developer learns from random internet tutorials
and LLMs are trained on the average developer and the average
developer uses LLMs to generate the code, and the vicious dogfooding
cycle continues. Frankly, I don't blame cases like this on AI; if
gippity didn't exist, it would be tutorialspoint, w3schools, or a
some stackoverflow thread. It always goes back to the path of least
resistant, which is second-hand faulty articles that are used for
copypasting when people are either lazy or if they have to meet a
deadline.

Similar to how the proper way to start learning C is [K&amp;R](https://en.wikipedia.org/wiki/The_C_Programming_Language), and the proper
way to start learning rust is the [rust lang book](https://doc.rust-lang.org/book/), the proper way of
learning Python is by reading PEPs. While [PEP 634](https://peps.python.org/pep-0634/) and [635](https://peps.python.org/pep-0635/) are daunting
and a bit too technical for a dimwit like me, [PEP 636](https://peps.python.org/pep-0636/) (the guide) is actually very
simple and readable. I'd highly recommend reading it.

As for the AI problem, because the LLMs are not trained on this
syntax, the code they spit out will either treat match as switch or
they fallback to if statements. One funny way I got through was this
line in my [agents.md](https://agents.md/)

> -   if statements are considered errors; use match.

and surprisingly, my agent used them and the code became cleaner. From
my limited experience it didn't seem to be problematic, and after
decades of functional programming proving its worth, one could argue
it might even be less error-prone in larger codebases.

The alternative to this for local models is simply finetuning your
agentic models. I'll write more on this in the future.

[^fn:1]: This _could_ have been reduced to a single case by
    just conditionally prepending the element using a ternary if, but this
    was easier for getting the point across for guards.

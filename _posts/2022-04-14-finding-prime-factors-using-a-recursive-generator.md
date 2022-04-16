---
layout: post
title:  "Finding Prime Factors Using a Recursive Generator"
tags: [python, generators]
comments: false
---

Python's generators with their `yield` and `yield from` statements allow for
elegant and terse (and lazy) solutions to recursive problems.

The following is a generator-based implementation of the recursive algorithm
to find prime factors of a whole number greater than one.

```python
from math import isqrt

def prime_factors(number):
    for i in range(2, isqrt(number) + 1):
        if number % i == 0:
            yield i
            yield from prime_factors(number // i)
            break
    else:
        yield number
```

All that is needed for the generator to yield values from another generator
(or an iterator, or even a simple iterable) is the `yield from` statement.

Let's test the function:

```pycon
>>> assert list(prime_factors(4)) == [2, 2]
>>> assert list(prime_factors(5)) == [5]
>>> assert list(prime_factors(6)) == [2, 3]
```

```pycon
>>> list(prime_factors(99))
[3, 3, 11]
>>> list(prime_factors(100))
[2, 2, 5, 5]
>>> list(prime_factors(101))
[101]
```

Seems to work alright. Of course in theory, the recursive solution has
the potential to hit the limit in the form of Python's default
maximum recursion depth:

```pycon
>>> assert list(prime_factors(2**980)) == [2] * 980    # OK
>>> assert list(prime_factors(2**990)) == [2] * 990    # error
...
RecursionError: maximum recursion depth exceeded while calling a Python object
```

## Note

Since we can use the `yield` and the `return` statements within the same
function, our generator function can be simplified a tiny bit by returning
directly from the loop and trimming away the `else` clause:

```python
def prime_factors(number):
    for i in range(2, isqrt(number) + 1):
        if number % i == 0:
            yield i
            yield from prime_factors(number // i)
            return
    yield number
```

## Note 2

As the `prime_factors` function calculates both `number % i` and `number // i`,
a question arises whether we could optimize performance by using
the [`divmod`][docs_divmod] builtin:

```pycon
def prime_factors(number):
    for i in range(2, isqrt(number) + 1):
        div, mod = divmod(number, i)
        if mod == 0:
            yield i
            yield from prime_factors(div)
            return
    yield number
```

The answer is "no". For example, in the case of
`list(prime_factors(1111111111111111))`, the version with `divmod` takes about
twice as much time -- most likely because both `div` and `mod` are being
calculated with each iteration of the for-loop, whereas the original
implementation calculates the integer division in just a small fraction of the
loop's iterations.

[docs_divmod]: https://docs.python.org/3/library/functions.html#divmod

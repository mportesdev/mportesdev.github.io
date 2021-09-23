---
layout: post
title:  "LBYL vs EAFP Timing Comparison"
tags: [python, timing]
comments: false
---

This is a simple experiment in Python comparing performance of two alternative
solutions to the same problem. The task is simple: remove all occurrences of a
value from a list.

In the following examples, we will have a list named `numbers` containing
integers from 0 to 4, and we will remove all occurrences of `4` from the list
in-place. 

This will be the [LBYL][lbyl] approach:

```python
while 4 in numbers:
    numbers.remove(4)
```

And this will be the [EAFP][eafp] approach:

```python
while True:
    try:
        numbers.remove(4)
    except ValueError:
        break
```

In theory, the former approach should be less efficient compared to the latter
because with each iteration, the loop condition will perform an additional item
look-up in the list. Let's verify this assumption.

## Processing a short list

Scenario:
- list length: 5,000 items
- number of repetitions: 100,000

The [`timeit.timeit`][timeit] function will be used to perform the operation
many times repeatedly and measure the execution time.

```python
from timeit import timeit

setup = 'numbers = list(range(5)) * 1000'

lbyl_code = '''
while 4 in numbers:
    numbers.remove(4)
'''

eafp_code = '''
while True:
    try:
        numbers.remove(4)
    except ValueError:
        break
'''

lbyl_time = timeit(lbyl_code, setup=setup, number=100_000)
eafp_time = timeit(eafp_code, setup=setup, number=100_000)

print(f'LBYL: {lbyl_time:.3f} s')
print(f'EAFP: {eafp_time:.3f} s')
```

Output of three sample runs of the program:

```
LBYL: 6.232 s
EAFP: 5.892 s
```

```
LBYL: 6.221 s
EAFP: 5.953 s
```

```
LBYL: 6.233 s
EAFP: 5.903 s
```

We can see that the `try-except` approach is indeed faster, but the difference
is not significant.

## Processing a long list

Scenario:
- list length: 50,000 items
- number of repetitions: 1


The [`codetiming`][codetiming] third-party library will be used to perform the
operation once without repetition and measure the execution time.

```python
from codetiming import Timer

numbers = list(range(5)) * 10_000

with Timer(text='LBYL: {:.3f} s'):
    while 4 in numbers:
        numbers.remove(4)


numbers = list(range(5)) * 10_000

with Timer(text='EAFP: {:.3f} s'):
    while True:
        try:
            numbers.remove(4)
        except ValueError:
            break
```

Output of three sample runs of the program:

```
LBYL: 6.104 s
EAFP: 2.999 s
```

```
LBYL: 6.334 s
EAFP: 3.010 s
```

```
LBYL: 6.099 s
EAFP: 3.008 s
```

We can see that the difference in execution time is significant in this case:
the `try-except` approach is twice as faster.

## Final note

The LBYL solution in this simple example is less efficient because of the
additional `4 in numbers` list item look-up. This operation's average time
complexity is `O(n)`.

Thank you for reading, and feel free to contact me if you think something is
wrong or missing in this blog post.

[lbyl]: https://docs.python.org/3/glossary.html#term-lbyl
[eafp]: https://docs.python.org/3/glossary.html#term-eafp
[timeit]: https://docs.python.org/3/library/timeit.html#timeit.timeit
[codetiming]: https://pypi.org/project/codetiming/

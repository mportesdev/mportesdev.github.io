---
layout: post
title:  "LBYL vs EAFP Timing Comparison"
date:   2021-08-16
categories: articles
---

This is a simple experiment in Python comparing performance of two alternative
solutions to the same problem.

The task is simple: remove all occurrences of a value from a list. In the
following examples, we will have a list named `numbers` containing integers from
0 to 4, and we will remove all occurrences of `4` from the list in-place. 

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

print('LBYL:', timeit(lbyl_code, setup=setup, number=100_000))


eafp_code = '''
while True:
    try:
        numbers.remove(4)
    except ValueError:
        break
'''

print('EAFP:', timeit(eafp_code, setup=setup, number=100_000))
```

Output of three sample runs of the program:

```
LBYL: 6.231663892016513
EAFP: 5.89169003101415
```

```
LBYL: 6.220698317018105
EAFP: 5.952736995008308
```

```
LBYL: 6.2331863199942745
EAFP: 5.903339722019155
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

with Timer(text='LBYL: {}'):
    while 4 in numbers:
        numbers.remove(4)


numbers = list(range(5)) * 10_000

with Timer(text='EAFP: {}'):
    while True:
        try:
            numbers.remove(4)
        except ValueError:
            break
```

Output of three sample runs of the program:

```
LBYL: 6.104196606989717
EAFP: 2.998941622005077
```

```
LBYL: 6.334021889982978
EAFP: 3.0097643220215105
```

```
LBYL: 6.0991898470092565
EAFP: 3.008402475999901
```

We can see that the difference in execution time is significant in this case:
the `try-except` approach is twice as faster.

## Conclusion

The LBYL solution in this simple example is less efficient because of the
additional `4 in numbers` list item look-up. This operation's average time
complexity is `O(n)` which means it tends to take significantly more time in
longer lists.

Thank you for reading, and feel free to contact me if you think something is
wrong or missing in this blog post.

[lbyl]: https://docs.python.org/3/glossary.html#term-lbyl
[eafp]: https://docs.python.org/3/glossary.html#term-eafp
[timeit]: https://docs.python.org/3/library/timeit.html#timeit.timeit
[codetiming]: https://pypi.org/project/codetiming/

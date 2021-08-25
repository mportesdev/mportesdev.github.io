---
layout: post
title:  "Joining sequences in Python"
date:   2021-08-10
categories: articles
---

In this short article we will explore some of the numerous ways Python
offers to concatenate sequences such as lists, tuples, byte arrays etc.

## Joining sequences of different types

Let's assume we have the following data:

```python
numbers = [1, 2]
strings = ('three', 'four')
binary = b'\x05\x06'
```

and we want to join `numbers`, `strings` and `binary` together into a single
flat list `[1, 2, 'three', 'four', 5, 6]`.

### Traditional approach

The most *obvious way to do it* is arguably a list comprehension:

```python
data = [numbers, strings, binary]
joined = [item for sequence in data for item in sequence]
```

```pycon
>>> joined
[1, 2, 'three', 'four', 5, 6]
```

which is equivalent to a regular nested loop:

```python
joined = []
for sequence in data:
    for item in sequence:
        joined.append(item)
```

which in turn can be simplified by using the `list.extend` method:

```python
joined = []
for sequence in data:
    joined.extend(sequence)    # or: joined += sequence
```

```pycon
>>> joined
[1, 2, 'three', 'four', 5, 6]
```

Another interesting technique is a list literal with the `*` unpacking notation:

```pycon
>>> [*numbers, *strings, *binary]
[1, 2, 'three', 'four', 5, 6]
```

but this one becomes rather impractical if we have many sequences to join.

### Functional approach

Alternatively, out iterable sequences can be concatenated using
[`itertools.chain`][docs_chain]:

```python
from itertools import chain
```

```pycon
>>> list(chain(*data))
[1, 2, 'three', 'four', 5, 6]
```

or

```pycon
>>> [*chain(*data)]
[1, 2, 'three', 'four', 5, 6]
```

which is equivalent to

```pycon
>>> list(chain.from_iterable(data))
[1, 2, 'three', 'four', 5, 6]
```

or

```pycon
>>> [*chain.from_iterable(data)]
[1, 2, 'three', 'four', 5, 6]
```

## Joining sequences of the same type

In addition to all the solutions demonstrated above, sequences of the same type
can typically be concatenated by simply summing them together. In the following
scenario, we have all our data stored in lists:

```python
numbers = [1, 2]
strings = ['three', 'four']
floats = [5.0, 6.0]
```

Again, we intend to join the numbers and strings into a single flat list
`[1, 2, 'three', 'four', 5.0, 6.0]`. The first solution is obvious:

```pycon
>>> numbers + strings + floats
[1, 2, 'three', 'four', 5.0, 6.0]
```

### Functional approach

The same result can be achieved by taking advantage of some standard library
functions. One possible way to sum sequences of the same type is by
cumulatively adding them using the [`functools.reduce`][docs_reduce] function:

```python
from functools import reduce

data = [numbers, strings, floats]
```

```pycon
>>> reduce(lambda x, y: x + y, data)
[1, 2, 'three', 'four', 5.0, 6.0]
```

or

```pycon
>>> from operator import add
>>> reduce(add, data)
[1, 2, 'three', 'four', 5.0, 6.0]
```

Alternatively, the built-in function [`sum`][docs_sum] can be used as well:

```pycon
>>> sum(data, [])
[1, 2, 'three', 'four', 5.0, 6.0]
```

**Note:** the [official documentation][docs_sum] suggests to use
`itertools.chain` rather than `sum` to concatenate iterables.

## Final notes

- All solutions shown in the first section can also be used to join sequences
retrieved from iterators. For example:
```pycon
>>> [*map(int, '12'), *(n ** 2 for n in (3, 4))]
[1, 2, 9, 16]
```
(Technically, iterators themselves are not considered
[sequences][docs_sequence] as they don't support `len`, indexing and slicing.)

- Feel free to contact me if you know of another interesting technique to
join sequences in Python.

[docs_chain]: https://docs.python.org/3/library/itertools.html#itertools.chain
[docs_reduce]: https://docs.python.org/3/library/functools.html#functools.reduce
[docs_sum]: https://docs.python.org/3/library/functions.html#sum
[docs_sequence]: https://docs.python.org/3/glossary.html#term-sequence

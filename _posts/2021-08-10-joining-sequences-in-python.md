---
layout: post
title:  "Joining sequences in Python"
date:   2021-08-10
categories: articles
---

In this short article I would like to explore some of the numerous ways Python
offers to concatenate sequences such as lists, tuples, byte arrays etc.

## Joining sequences of different types

Let's assume we have the following data:

```python
data = [[1, 2], ('three', 'four'), bytearray(b'\x05\x06')]
```

Now we want to join all the iterable sequences stored in `data` together into
a single list. In other words, we want to flatten `data`.

### Idiomatic approach

I think the most *obvious way to do it* is a list comprehension:

```pycon
>>> flat = [item for sequence in data for item in sequence]
>>> flat
[1, 2, 'three', 'four', 5, 6]
```

which is equivalent to a regular nested loop:

```pycon
>>> flat = []
>>> for sequence in data:
...     for item in sequence:
...         flat.append(item)
... 
>>> flat
[1, 2, 'three', 'four', 5, 6]
```

which in turn can be simplified by applying the `list.extend` method:

```pycon
>>> flat = []
>>> for sequence in data:
...     flat.extend(sequence)
... 
>>> flat
[1, 2, 'three', 'four', 5, 6]
```

Another interesting technique would be a list literal with the `*` unpacking
notation:

```pycon
>>> [*data[0], *data[1], *data[2]]
[1, 2, 'three', 'four', 5, 6]
```

but that would become impractical if we had many sequences in `data`.

### Functional approach

Alternatively, out iterable sequences can be concatenated using
[`itertools.chain`][docs_chain]:

```pycon
>>> from itertools import chain

>>> flat = list(chain(*data))
>>> flat
[1, 2, 'three', 'four', 5, 6]
```

As an equivalent, we might again use a slightly more compact list literal with
the star notation:

```pycon
>>> flat = [*chain(*data)]
>>> flat
[1, 2, 'three', 'four', 5, 6]
```

## Joining sequences of the same type

In addition to the solutions demonstrated above, sequences of the same type can
typically be concatenated by simply summing them together. In the following
scenario, we have all our data stored in lists:

```python
data = [[1, 2], ['three', 'four'], [5, 6]]
```

Again, we intend to join the numbers and strings into a single flat list. The
first solution is obvious but not very practical:

```pycon
>>> data[0] + data[1] + data[2]
[1, 2, 'three', 'four', 5, 6]
```

There are however more elegant techniques to achieve the same.

### Functional approach

One possible way to sum sequences of the same type is by cumulatively adding
them using the [`functools.reduce`][docs_reduce] function:

```pycon
>>> from functools import reduce
>>> flat = reduce(lambda x, y: x + y, data)
>>> flat
[1, 2, 'three', 'four', 5, 6]
```

**Note:** You can use [`operator.add`][docs_add] instead of the lambda function.

Alternatively, the built-in function [`sum`][docs_sum] can be used as well:

```pycon
>>> flat = sum(data, [])
>>> flat
[1, 2, 'three', 'four', 5, 6]
```

**Note:** the [official documentation][docs_sum] suggests to use
`itertools.chain` rather than `sum` to concatenate iterables.

## Final notes

- All solutions in the first section can also be used to join sequences
retrieved from iterators. (Technically, iterators themselves are not considered
[sequences][docs_sequence] as they don't support `len`, indexing and slicing.)

- All solutions in the second section use addition by `+` internally, as can be
shown by applying them to incompatible types:
```pycon
>>> from functools import reduce
>>> from operator import add
>>> [1] + (2,)
TypeError: can only concatenate list (not "tuple") to list
>>> reduce(add, [[1], (2,)])
TypeError: can only concatenate list (not "tuple") to list
>>> sum([[1], (2,)], [])
TypeError: can only concatenate list (not "tuple") to list
```

- Feel free to contact me if you know of another interesting technique to
join sequences in Python.

[docs_chain]: https://docs.python.org/3/library/itertools.html#itertools.chain
[docs_reduce]: https://docs.python.org/3/library/functools.html#functools.reduce
[docs_add]: https://docs.python.org/3/library/operator.html#operator.add
[docs_sum]: https://docs.python.org/3/library/functions.html#sum
[docs_sequence]: https://docs.python.org/3/glossary.html#term-sequence

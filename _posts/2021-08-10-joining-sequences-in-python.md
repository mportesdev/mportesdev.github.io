---
layout: post
title:  "Joining sequences in Python"
date:   2021-08-10
categories: articles
---

In this short article I would like to explore some of the numerous ways Python
offers to concatenate sequences such as lists, tuples, byte arrays etc.

## 1. Joining sequences of different types

Let's assume we have the following data:

```python
numbers = [1, 2]
strings = ('three', 'four')
binary = b'\x05\x06'
```

Now we want to join `numbers`, `strings` and `binary` together into
a single list.

### Idiomatic approach

I think the most *obvious way to do it* is a list comprehension:

```pycon
>>> data = [numbers, strings, binary]
>>> joined = [item for sequence in data for item in sequence]
>>> joined
[1, 2, 'three', 'four', 5, 6]
```

which is equivalent to a regular nested loop:

```pycon
>>> joined = []
>>> for sequence in data:
...     for item in sequence:
...         joined.append(item)
... 
>>> joined
[1, 2, 'three', 'four', 5, 6]
```

which in turn can be simplified by applying the `list.extend` method:

```pycon
>>> joined = []
>>> for sequence in data:
...     joined.extend(sequence)
... 
>>> joined
[1, 2, 'three', 'four', 5, 6]
```

Another interesting technique would be a list literal with the `*` unpacking
notation:

```pycon
>>> [*numbers, *strings, *binary]
[1, 2, 'three', 'four', 5, 6]
```

but this one becomes rather impractical if we have many sequences to join.

### Functional approach

Alternatively, out iterable sequences can be concatenated using
[`itertools.chain`][docs_chain]:

```pycon
>>> from itertools import chain

>>> joined = list(chain(*data))
>>> joined
[1, 2, 'three', 'four', 5, 6]
```

As an equivalent, we might again use a slightly more compact list literal with
the star notation:

```pycon
>>> joined = [*chain(*data)]
>>> joined
[1, 2, 'three', 'four', 5, 6]
```

## 2. Joining sequences of the same type

In addition to all the approaches demonstrated above, sequences of the same type
can typically be concatenated by simply summing them together. In the following
scenario, we have all our data stored in lists:

```python
numbers = [1, 2]
strings = ['three', 'four']
floats = [5.0, 6.0]
```

Again, we intend to join the numbers and strings into a single list. The
first solution is obvious:

```pycon
>>> numbers + strings + floats
[1, 2, 'three', 'four', 5.0, 6.0]
```

### Functional approach

The same result can be achieved by taking advantage of functions from the
standard library. One possible way to sum sequences of the same type is by
cumulatively adding them using the [`functools.reduce`][docs_reduce] function:

```pycon
>>> from functools import reduce
>>> data = [numbers, strings, floats]
>>> joined = reduce(lambda x, y: x + y, data)
>>> joined
[1, 2, 'three', 'four', 5.0, 6.0]
```

**Note:** You can use [`operator.add`][docs_add] instead of the lambda function.

Alternatively, the built-in function [`sum`][docs_sum] can be used as well:

```pycon
>>> joined = sum(data, [])
>>> joined
[1, 2, 'three', 'four', 5.0, 6.0]
```

**Note:** the [official documentation][docs_sum] suggests to use
`itertools.chain` rather than `sum` to concatenate iterables.

## Final notes

- All solutions shown in section 1 can also be used to join sequences
retrieved from iterators. (Technically, iterators themselves are not considered
[sequences][docs_sequence] as they don't support `len`, indexing and slicing.)
For example:
```pycon
>>> [*map(int, '12'), *(n ** 2 for n in (3, 4))]
[1, 2, 9, 16]
```

- Feel free to contact me if you know of another interesting technique to
join sequences in Python.

[docs_chain]: https://docs.python.org/3/library/itertools.html#itertools.chain
[docs_reduce]: https://docs.python.org/3/library/functools.html#functools.reduce
[docs_add]: https://docs.python.org/3/library/operator.html#operator.add
[docs_sum]: https://docs.python.org/3/library/functions.html#sum
[docs_sequence]: https://docs.python.org/3/glossary.html#term-sequence

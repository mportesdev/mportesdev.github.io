---
layout: post
title:  "Joining sequences in Python"
date:   2021-08-06
categories: articles
---

Let's explore some of the ways Python offers to concatenate sequences like
lists, tuples, byte arrays etc.

## Joining sequences of the same type

In the following basic scenario, we have some lists referenced by names
("assigned to variables").

```python
numbers = [1, 2]
strings = ['three', 'four']
```

This allows us to concatenate the lists in a straightforward and explicit
fashion.

_Simple solution:_ addition with the `+` operator

```python
>>> numbers + strings
[1, 2, 'three', 'four']
```

_Fancy solution:_ list literal with the `*` unpacking notation

```python
>>> [*numbers, *strings]
[1, 2, 'three', 'four']
```

In a more general scenario, we would have the lists stored in a collection.

```python
my_lists = ([1, 2], ['three', 'four'])
```

We may or may not know in advance how many lists there are. This prevents us
from using the _simple_ and _fancy_ solutions shown above. We need more
versatile approaches to join the lists.

_Idiomatic solution:_ list comprehension with a nested for-loop

```python
>>> [item for list_ in my_lists for item in list_]
[1, 2, 'three', 'four']
```

_Hacky solution:_ summing the lists

```python
>>> sum(my_lists, [])
[1, 2, 'three', 'four']
```

Note however that the [official documentation][docs_sum] suggests to use
[itertools.chain][docs_chain] rather than `sum` to concatenate iterables.
Let's do exactly that.

_Functional solution:_ chaining the lists

```python
>>> from itertools import chain
>>> list(chain(*my_lists))
[1, 2, 'three', 'four']

>>> # or: a starry list literal again, if you like
>>> [*chain(*my_lists)]
[1, 2, 'three', 'four']
```

[docs_sum]: https://docs.python.org/3/library/functions.html#sum
[docs_chain]: https://docs.python.org/3/library/itertools.html#itertools.chain

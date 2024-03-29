---
layout: post
title:  "Regex Named Groups in Python"
tags: [python, regex]
comments: false
---

A quick refresher on the syntax of named groups (also known as named
subpatterns) in regular expressions.

There are three alternative notations of [named groups in PCRE][wiki]:

- `(?P<name>...)`
- `(?<name>...)`
- `(?'name'...)`

The first one originates in Python and is the only one supported by the
[`re`][re] standard library module:

```python
import re

# match date in format YYYY/M/D

regex = re.compile(
    '(?P<year>[0-9]{4})'
    '/'
    '(?P<month>[0-9]{1,2})'
    '/'
    '(?P<day>[0-9]{1,2})'
)

match = regex.match('1969/7/20')
```

```pycon
>>> match['month']
'7'
>>> match.groupdict()
{'year': '1969', 'month': '7', 'day': '20'}
```

The other two notations are not recognized and raise an
`re.error: unknown extension` error.

You can further experiment with the above example by going to
[https://regex101.com/r/UE4Zex/1](https://regex101.com/r/UE4Zex/1) and switching
between the `PCRE` and `Python` regex flavor.

[wiki]: https://en.wikipedia.org/wiki/Perl_Compatible_Regular_Expressions#Named_subpatterns
[re]: https://docs.python.org/3/library/re.html#regular-expression-syntax

---
layout: post
title:  "Regex Named Groups in Python"
date:   2021-09-07
---

There are the following three alternative notations of
[named groups in PCRE][wiki]:

```regexp
(?P<name>...)
(?<name>...)
(?'name'...)
```

Only the first one is recognized by Python's [`re`][re] standard library module:

```python
import re

# match date in format YYYYMMDD

regex = re.compile(
    r"(?P<year>[0-9]{4})"
    r"(?P<month>[0-9]{2})"
    r"(?P<day>[0-9]{2})"
)

match = regex.match('20210907')
```

```pycon
>>> match.groupdict()
{'year': '2021', 'month': '09', 'day': '07'}
```

The other two raise an `sre_constants.error: unknown extension` error.

## Final note

You can further experiment with the above example by going to
[regex101.com][regex101] and switching between the `PCRE` and `Python` regex
flavor.

Thank you for reading, and as always, feel free to contact me if you think
something is wrong or missing in this blog post.

[wiki]: https://en.wikipedia.org/wiki/Perl_Compatible_Regular_Expressions#Named_subpatterns
[re]: https://docs.python.org/3/library/re.html#regular-expression-syntax
[regex101]: https://regex101.com/r/xF7IuH/1

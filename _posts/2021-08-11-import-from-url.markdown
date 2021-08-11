---
layout: post
title:  "How to Import Python Code from URL"
date:   2021-08-11
categories: articles
---

This experiment was inspired by a discussion on Slack.
The question was how to make a single Python file, sitting in a remote
GitHub repository, easily importable on local machine, without having to
manually save it to disk or copy-paste its contents to a new file.

In the following code example, we will write a function that:
 - retrieves Python source code from a URL
 - stores it in a temporary file
 - programmatically imports this file
 - returns the resulting module object

We will only be using the modules from the [standard library][stdlib].

```python
from importlib.util import spec_from_file_location, module_from_spec
from tempfile import NamedTemporaryFile
from urllib.request import urlopen


def module_from_url(module_name, url):
    # get contents of URL
    with urlopen(url) as response:
        url_contents = response.read()

    # store contents in temporary .py file
    with NamedTemporaryFile(suffix='.py') as temp_file:
        temp_file.write(url_contents)
        temp_file.seek(0)

        # import temporary file as Python module
        spec = spec_from_file_location(module_name, temp_file.name)
        module = module_from_spec(spec)
        spec.loader.exec_module(module)

    return module
```

Now let's take our function for a ride. We will retrieve and import a piece
of Python source code and then call one of the functions it defines.

```
>>> url = 'https://raw.githubusercontent.com/asweigart/codebreaker/master/primeSieve.py'
>>> primes = module_from_url('primes', url)
>>> primes.primeSieve(20)
[2, 3, 5, 7, 11, 13, 17, 19]
```

[stdlib]: https://docs.python.org/3/library/

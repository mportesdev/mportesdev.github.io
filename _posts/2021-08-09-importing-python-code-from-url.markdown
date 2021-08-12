---
layout: post
title:  "Importing Python Code from URL"
date:   2021-08-09
categories: articles
---

I got the idea for this experiment when reading a discussion on Slack.
The original question was how to make a single Python file, sitting somewhere
in a remote GitHub repository, easily importable on local machine,
without having to manually save it to disk or copy-paste its contents to
a new file.

As an exercise, let's do exactly that. We are going to craft a function that:
 - retrieves Python source code from a URL
 - stores it in a temporary file
 - programmatically imports this file as a Python module
 - returns the resulting module object

The following is the most straightforward implementation I could come up with:

```python
from importlib.util import module_from_spec, spec_from_file_location
from tempfile import NamedTemporaryFile
from urllib.request import urlopen


def module_from_url(name, url):
    # get contents of URL
    with urlopen(url) as response:
        url_contents = response.read()

    # store contents in temporary .py file
    with NamedTemporaryFile(suffix='.py') as temp_file:
        temp_file.write(url_contents)
        temp_file.seek(0)

        # import temporary file as Python module
        spec = spec_from_file_location(name, temp_file.name)
        module = module_from_spec(spec)
        spec.loader.exec_module(module)

    return module
```

As you can see, I chose to only use the tools available in the
[standard library][stdlib]. I wanted this code snippet to have no dependencies
and be easily copy-pasted and tried.
(yes, that's ironic, if you read the first paragraph again)

Now let's take our function for a test ride. We are going to retrieve and import
[this file][primeSieve] from one of Al Sweigart's repositories (note however
that we use the URL of its raw contents) and then call one of the functions
it defines.

```
>>> url = 'https://raw.githubusercontent.com/asweigart/codebreaker/master/primeSieve.py'
>>> prime_sieve = module_from_url('prime_sieve', url)
>>> prime_sieve.primeSieve(20)
[2, 3, 5, 7, 11, 13, 17, 19]
```

All seems to work fine.

Notes:
 - Before returning from `module_from_url`, it might be a good idea to
   cache the dynamically imported module by setting
   `sys.modules[module_name] = module`.
 - This is just a quick and naive exercise. Don't use it in any serious way.
 - Feel free to contact me if you have an idea for a simpler solution,
   e.g. one without a temporary file.


[stdlib]: https://docs.python.org/3/library/
[primeSieve]: https://github.com/asweigart/codebreaker/blob/master/primeSieve.py

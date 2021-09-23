---
layout: post
title:  "Importing Python Code from URL"
tags: [python, imports]
comments: false
---

I got the idea for this experiment when reading a discussion on Slack.
The original question was how to make a single Python file, sitting somewhere
in a remote GitHub repository, importable on a local machine without having
to manually save it to disk or copy-paste its contents to a new file.

As a fun exercise, let's do exactly that. We are going to write a function that:
- retrieves Python source code from a URL
- stores it in a temporary file
- programmatically imports this file as a Python module
- returns the resulting module object

The following is the most straightforward implementation I could come up with:

```python
from importlib.util import module_from_spec, spec_from_file_location
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

I chose to only use the tools available in the standard library, as I wanted
this code snippet to have no dependencies and be easily copy-pasted and tried.
(yes, that's kind of ironic if you read the first paragraph again)

Now let's take our function for a test ride. We are going to retrieve and import
a file from Al Sweigart's [codebreaker][codebreaker] repository and then call
one of the functions it defines, namely `primeSieve` that implements the
[sieve of Eratosthenes][sieve].

```
>>> url = 'https://raw.githubusercontent.com/asweigart/codebreaker/master/primeSieve.py'
>>> prime_sieve = module_from_url('prime_sieve', url)

>>> prime_sieve.primeSieve(50)
[2, 3, 5, 7, 11, 13, 17, 19, 23, 29, 31, 37, 41, 43, 47]
```

The import works as expected and so does the imported function.

Let's try another example. We will import the main file from the
[simplematch][simplematch] project and then call the `match` function:

```
>>> url = 'https://raw.githubusercontent.com/tfeldmann/simplematch/main/simplematch.py'
>>> simplematch = module_from_url('simplematch', url)

>>> pattern = '{artist} - {year:int} - {album}'
>>> string = 'ZZ Top - 1985 - Afterburner'

>>> simplematch.match(pattern, string)
{'artist': 'ZZ Top', 'year': 1985, 'album': 'Afterburner'}
```

Again, everything seems to work just fine.

## Final notes

- Before returning from the `module_from_url` function, it might be a good idea
to cache the dynamically imported module by setting
`sys.modules[module_name] = module`, so that e.g.
`simplematch is sys.modules['simplematch']` is true (consistently with modules
imported the usual way).

- This is just a quick and naive exercise. Don't use it in any serious way.

Thank you for reading, and feel free to contact me if you have an idea for a
simpler solution (e.g. one without a temporary file) or if you think something
is wrong or missing in this blog post.

[codebreaker]: https://github.com/asweigart/codebreaker
[sieve]: https://en.wikipedia.org/wiki/Sieve_of_Eratosthenes
[simplematch]: https://github.com/tfeldmann/simplematch

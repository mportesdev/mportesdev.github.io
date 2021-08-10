August 9, 2021

# How to import Python code from URL

```python
from importlib.util import spec_from_file_location, module_from_spec
from tempfile import NamedTemporaryFile
from urllib.request import urlopen


def module_from_url(module_name, url):
    # get the contents of URL as bytes
    url_contents = urlopen(url).read()

    # store the contents in a temporary file
    with NamedTemporaryFile(suffix='.py') as temp_file:
        temp_file.write(url_contents)
        temp_file.seek(0)

        # import the temporary file programmatically as a Python module
        spec = spec_from_file_location(module_name, temp_file.name)
        module = module_from_spec(spec)
        spec.loader.exec_module(module)

    return module
```

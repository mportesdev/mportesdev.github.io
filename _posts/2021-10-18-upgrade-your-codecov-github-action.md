---
layout: post
title:  Upgrade Your codecov GitHub Action
tags: [github actions, test coverage]
comments: false
---

If you use GitHub Actions for automated testing and continuous integration of
you project, chances are you also measure the test coverage, and that you
upload the results to [Codecov][codecov].

Codecov offers their own [action on Actions Marketplace][action] which
can be easily integrated into a workflow. For example, this is what one
of a job's steps may look like in YAML:

```
    - name: Upload coverage to Codecov
      uses: codecov/codecov-action@v1
```

One thing worth noting is that these days, __codecov-action__ version 1
is being deprecated in favor of version 2. (This change is related
to the deprecation of the underlying coverage report
uploader. Additional details can be found [here][blog].)

In the best-case scenario, all it should take to upgrade is simply changing
`v1` to `v2` in the action specifier (see the above snippet)
in your workflow's YAML file. There is however one inconsistency that
I observed in the behaviour of the two versions with regard to
Python/pytest.

The thing is, __codecov-action__ searches for a coverage report in XML
format by default. At the same time, the `pytest-cov` plugin does not
write an XML file by default. It must be either configured to do so or
a `--cov-report=xml` option must be passed to `pytest`.

When an XML coverage report is not found, the deprecated __codecov-action__
version 1 will simply run `coverage xml` which will generate the report
_ex post_ from the existing `.coverage` file.
Example from an actual workflow's output:

```
==> Python coveragepy exists ...
    -> Running coverage xml
```

and subsequently:

```
==> Searching for coverage ...
    -> Found 1 reports
```

On the other hand, the new __codecov-action__ version 2 is not smart
enough to do that. After updating the action from `v1` to `v2`, I noticed
that my workflow's output contains an error message:

```
['info'] Searching for coverage files...
['error'] There was an error running the uploader: No coverage files located...
```

After I added the aforementioned `--cov-report=xml` option to the
`pytest --cov` command, the coverage data upload started working again.

# Final Note

If you use __codecov-action__ in your GitHub Actions, don't forget to upgrade,
and subsequently check your workflow's output for error messages. Note that
by default, __codecov-action__ will exit with zero exit code even if
the above shown error occurs. This can result in a successful workflow run
without having the relevant coverage data uploaded.

Thank you for reading, and feel free to contact me if you think something
is wrong or missing in this blog post.

[codecov]: https://about.codecov.io/
[action]: https://github.com/marketplace/actions/codecov
[blog]: https://about.codecov.io/blog/codecov-uploader-deprecation-plan/

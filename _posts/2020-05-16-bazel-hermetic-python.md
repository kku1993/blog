---
layout: post
title:  "Hermetic Python with Bazel"
date:   2020-05-16
categories: tech, python, bazel
---

This post walks through how to hermetically build a python interpreter in bazel
and set up the toolchain.

## Motivation

When sharing a bazel workspace with multiple collaborators, I want to make sure
everyone is using the same python version. This is particularly important when
some the pip packages we use are sensitive to python minor version (e.g.
[`pysopg2-binary`](https://github.com/psycopg/psycopg2/issues/721#issuecomment-392317620)).

One solution is to check-in pre-compiled python interpreter binaries as
suggested in [1] and [2]. However, I still need to deal with developers on
different operating systems (macOS vs Ubuntu) and the thought of having binary
blobs next to source code makes me sad.

## My Solution

The idea is to download and build the python interpreter from source and add
the binary to bazel's toolchain. You can see an example here:
[https://github.com/kku1993/bazel-hermetic-python](https://github.com/kku1993/bazel-hermetic-python).

### Step 1: Build Python Interpreter

In `WORKSPACE`, I use an `http_archive` rule to create an external repository
where the python interpreter source code is fetched and built. I configured the
installation to store the artifacts in the external workspace and exposed a
symlink to the resulting binary via `exports_files`.

Note: you can achieve the same result with a custom [repository
rule](https://docs.bazel.build/versions/master/skylark/repository_rules.html).

```python
# WORKSPACE

load("@bazel_tools//tools/build_defs/repo:http.bzl", "http_archive")

# Special logic for building python interpreter with OpenSSL from homebrew.
# See https://devguide.python.org/setup/#macos-and-os-x
_py_configure = """
if [[ "$OSTYPE" == "darwin"* ]]; then
    ./configure --prefix=$(pwd)/bazel_install --with-openssl=$(brew --prefix openssl)
else
    ./configure --prefix=$(pwd)/bazel_install
fi
"""

http_archive(
    name = "python_interpreter",
    urls = ["https://www.python.org/ftp/python/3.8.3/Python-3.8.3.tar.xz"],
    sha256 = "dfab5ec723c218082fe3d5d7ae17ecbdebffa9a1aea4d64aa3a2ecdd2e795864",
    strip_prefix = "Python-3.8.3",
    patch_cmds = [
        "mkdir $(pwd)/bazel_install",
        _py_configure,
        "make",
        "make install",
        "ln -s bazel_install/bin/python3 python_bin",
    ],
    build_file_content = """
exports_files(["python_bin"])
filegroup(
    name = "files",
    srcs = glob(["bazel_install/**"], exclude = ["**/* *"]),
    visibility = ["//visibility:public"],
)
""",
)
```

### Step 2: Set Up Python Toolchain

Nothing special here, just need to set up `py_runtime` and `toolchain`.

```python
# BUILD.bazel

py_runtime(
    name = "python3_runtime",
    files = ["@python_interpreter//:files"],
    interpreter = "@python_interpreter//:python_bin",
    python_version = "PY3",
    visibility = ["//visibility:public"],
)

py_runtime_pair(
    name = "my_py_runtime_pair",
    py2_runtime = None,
    py3_runtime = ":python3_runtime",
)

toolchain(
    name = "my_py_toolchain",
    toolchain = ":my_py_runtime_pair",
    toolchain_type = "@bazel_tools//tools/python:toolchain_type",
)

# WORKSPACE

register_toolchains("//:my_py_toolchain")
```

At this point, any `py_binary` rules you have should run with the custome
interpreter we've just built!

### Bonus: Getting pip_import to work

Note: This is a work in progress with an [open
PR](https://github.com/bazelbuild/rules_python/pull/312)

In order for `pip_import` to install packages with the correct python version,
we need to specify the custom interpreter.

```python
# WORKSPACE

pip_import(
    name = "py_deps",
    requirements = "//:requirements.txt",
    python_interpreter_target = "@python_interpreter//:python_bin",
)
```

## Links

- [1] [https://stackoverflow.com/questions/52228140/how-to-hermetically-include-python-interpreter-in-bazel-to-build-python-librar](https://stackoverflow.com/questions/52228140/how-to-hermetically-include-python-interpreter-in-bazel-to-build-python-librar)
- [2] [https://github.com/abergmeier-dsfishlabs/bazel_hermetic_test/tree/master/third_party/python3](https://github.com/abergmeier-dsfishlabs/bazel_hermetic_test/tree/master/third_party/python3)

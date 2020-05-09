---
layout: post
title:  "Protobufs in Python with Bazel"
date:   2020-05-08
categories: tech, python, protobuf, bazel
---

Currently, there's no canonical implementation of `py_proto_library` in `bazel`.
This post shows how to build protobufs for python using the version from the
[protobuf team](https://github.com/protocolbuffers/protobuf).

## Import Protobuf Dependency

If you haven't done so already, add the following to your `WORKSPACE` to import
the protobuf repo as an external dependency.

```python
# WORKSPACE

load("@bazel_tools//tools/build_defs/repo:git.bzl", "git_repository")

git_repository(
    name = "com_google_protobuf",
    remote = "https://github.com/protocolbuffers/protobuf",
    tag = "v3.10.0",
)

load("@com_google_protobuf//:protobuf_deps.bzl", "protobuf_deps")

protobuf_deps()
```

## Compile Protobufs to Python Library

To invoke the protobuf compiler on `x.proto`, we use the
[`py_proto_library`](https://github.com/protocolbuffers/protobuf/blob/bb3460d71b2f2cd75f10efe94d739e15561c2ccf/protobuf.bzl#L404)
rule from [protobuf repo](https://github.com/protocolbuffers/protobuf).

```python
# lib/BUILD.bazel

load("@com_google_protobuf//:protobuf.bzl", "py_proto_library")

py_proto_library(
    name = "my_py_proto",
    srcs = ["x.proto"],
    visibility = ["//visibility:public"],
)
```

## Using the Compiled Protobuf

The output of `py_proto_library` is a set of `.py` files whose names are the
same as the `.proto` files but with a `_pb2` extension. For example, `x.proto`
becomes `x_pb2.py`. For more info, see [protobuf
documentation](https://developers.google.com/protocol-buffers/docs/reference/python-generated#invocation).

These generated python files are at the same relative path to the root of the
workspace as the `.proto` file that generated them. So in your python code, you
can import them using the relative path. For example:

```python
from lib.x_pb2 import MyMessage
```

## Well-Known Types

To include [well-known
types](https://developers.google.com/protocol-buffers/docs/reference/google.protobuf)
in your protobuf messages, add `@com_google_protobuf//:protobuf_python` as a
dependency to your `py_proto_library` targets.

For example:

```python
py_proto_library(
    name = "my_py_proto",
    srcs = ["x.proto"],
    deps = ["@com_google_protobuf//:protobuf_python"],
    visibility = ["//visibility:public"],
)
```

## Relevant Links

- Alternative implementation by gRPC team: [link](
  https://github.com/grpc/grpc/blob/f7591a3426becbed20dbd1a80beb9c1e2ca1a738/bazel/python_rules.bzl#L89)
- Alternative implementation by `stackb`:
  [link](https://github.com/stackb/rules_proto)
- Discussion on `py_proto_library` for `bazel`: [link](
  https://github.com/bazelbuild/bazel/issues/3935)

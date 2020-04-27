---
layout: post
title:  "Go Modules in Bazel"
date:   2020-04-26
categories: tech, golang, bazel
---

Recently, I wanted to import a go library into my [bazel](https://bazel.build/)
workspace that uses a moduled version of another library that already exists as
an unmoduled version in the workspace. Here's how I made
[gazelle](https://github.com/bazelbuild/bazel-gazelle) generate right
dependencies.

# Setup

My workspace already contains the unmoduled `github.com/google/go-github`
dependency.

I want to import `github.com/bradleyfalzon/ghinstallation`, which depends on
`github.com/google/go-github` module `v29`.

# Solution

Here's my `WORKSPACE` file that successfully imported the dependencies:

```python
# The unmoduled go-github.
go_repository(
    name = "com_github_google_go_github",
    importpath = "github.com/google/go-github",
    commit = "80d007d8679f58cc1bcc9647849501219ea831ca",
)

# v29 go-github module.
go_repository(
    name = "com_github_google_go_github_v29",
    importpath = "github.com/google/go-github/v29",
    sum = "h1:IktKCTwU//aFHnpA+2SLIi7Oo9uhAzgsdZNbcAqhgdc=",
    version = "v29.0.3",
)

go_repository(
    name = "com_github_bradleyfalzon_ghinstallation",
    importpath = "github.com/bradleyfalzon/ghinstallation",
    tag = "v1.1.1",
)
```

Without the separately named `com_github_google_go_github_v29` repository rule,
`gazelle` is going to resolve the go import path
`github.com/google/go-github/v29` to a non-existent `v29` package under
`@com_github_google_go_github`, as shown in the following error message:

> ERROR: Analysis of target '@com_github_bradleyfalzon_ghinstallation//:go_default_library' failed; build aborted: no such package '@com_github_google_go_github//v29/github': BUILD file not found in directory 'v29/github' of external repository @com_github_google_go_github. Add a BUILD file to a directory to mark it as a package.

Ideally I would only use one version of the library in my codebase to avoid
surprises. But it's great that bazel and gazelle are flexible enough to support
multiple versions of the same dependency!

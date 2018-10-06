# Bazel rules for LaTeX

This repository provides [Bazel](https://bazel.build/) rules for LaTeX,
strongly inspired by [Klaus Aehlig's blog post](http://www.linta.de/~aehlig/techblog/2017-02-19.html)
on the matter.

Instead of depending on the host system's copy of LaTeX, these rules
download [a modular copy of TeXLive from GitHub](https://github.com/ProdriveTechnologies/texlive-modular).
By using fine-grained dependencies, you will only download portions of
TeXLive that are actually used in your documents.

# Using these rules

Add the following to `WORKSPACE`:

```python
http_archive(
    name = "bazel_latex",
    sha256 = "<checksum>",
    strip_prefix = "bazel-latex-<release>",
    url = "https://github.com/ProdriveTechnologies/bazel-latex/archive/v<release>.tar.gz",
)

load("@bazel_latex//:repositories.bzl", "latex_repositories")

latex_repositories()
```

And add the following `load()` directive to your `BUILD` files:

```python
load("@bazel_latex//:latex.bzl", "latex_document")
```

You can then use `latex_document()` to declare documents that need to be
built. Commonly reused sources (e.g., templates) can be placed in
[`filegroup()`](https://docs.bazel.build/versions/master/be/general.html#filegroup)
blocks, so that they don't need to be repeated.

```python
latex_document(
    name = "my_report",
    srcs = glob([
        "chapters/*.tex",
        "figures/*",
    ]) + [":company_style"],
    main = "my_report.tex",
)

filegroup(
    name = "company_style",
    srcs = glob([
        ...
    ]),
)
```

As `pdflatex` is effectively invoked as if within the root of the
workspace, all imports of resources (e.g., images) must use the full
path relative to the root.

A PDF can be built by running:

```
bazel build :my_report
```

It can be viewed using your system's PDF viewer by running:

```
bazel run :my_report_view
```

# Using packages

By default, `latex_document()` only provides a version of TeXLive that
is complete enough to build the most basic documents. Whenever you use
`\usepackage{}` in your documents, you must also add a corresponding
dependency to your `latex_document()`. This will cause Bazel to download
and expose those packages for you. Below is an example of how a document
can be built that depends on the Hyperref package.

```python
latex_document(
    name = "hello",
    srcs = ["@bazel_latex//packages:hyperref"],
    main = "hello.tex",
)
```

This repository provides bindings for most commonly used packages.
Please send pull requests if additional bindings are needed.

# Platform support

These rules have been tested to work on:

- macOS Mojave 10.14, building locally.
- Ubuntu 18.04, building locally.
- Ubuntu 18.04, building on a Debian 9 based
  [Buildbarn](https://github.com/EdSchouten/bazel-buildbarn) setup.
- Windows 10 1803, building on a Debian 9 based
  [Buildbarn](https://github.com/EdSchouten/bazel-buildbarn) setup.

These rules are known not to work on:

- Windows 10 1803, building locally.

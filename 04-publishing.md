---
title: "Publishing our Python Packages"
teaching: 30
exercises: 0
---

TODO: replace build and twine with hatch. local build with hatch. demo push to test-pypi.
TODO: pip install from github
TODO: vcs based versioning with git tags and hatch (like setuptools-vcs)
TODO: Update github action to use hatch and push to test-pypi
TODO: Change demo package name to <packageName>_<githubUserName>, check that this does not exist in test-pypi
TODO: Create a test-pypi API token

:::::::::::::::::::::::::::::::::::::::::::::::::::::::::: questions 

- What services can we use to publish our software? 
- What steps must we take to accomplish this?

::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::: objectives

- Learn how to version our software and create releases.
- Learn how to publish our software on PyPi, the Python Packaging Index.

::::::::::::::::::::::::::::::::::::::::::::::::

## Publishing our Python Packages

Following our lesson on [building and installing](03-building-and-installing.Rmd) Python
packages, we should now have a package that can be installed using `pip`. With just a
little more work, we can now publish our software on a few free online services:
GitHub for source/version control, and PyPI (the Python Packaging Index) for package
hosting. The latter is the default repository used by `pip`, so by hosting our software
there, other users will be able to install our software to any machine of their
choosing, and will be able to add it as a dependency to their own projects.

Before discussing how to use these services, we'll briefly cover 'semantic versioning':
a method for adding meaningful versions to our software containing compatibility
information, and we'll look at some of the preparatory steps we should take before
publishing our code.

## Versions and Releases

Users of our software will expect it to stay consistent over time. If we were to
suddenly rename a function, or change what arguments it takes, this could break our
user's code, which in turn could break another developer's code, etc.

A good solution to this problem is to use 'semantic versioning', in which the version
number contains compatibility information in a widely agreed-upon format. Using semanic
versioning, we will periodically make a new 'release' of our code, each time
updating its version number, and the way we change the number informs our users
of whether the changes to our code will break compatibility. In its most basic
form, semantic versioning requires 3 numbers, separated by periods, such as 1.2.3.
These numbers stand for MAJOR.MINOR.PATCH, and are typically updated
right-to-left. The meaning of each number is:

- PATCH: We increment the patch number whenever we make an update that doesn't add or
  remove functionality. It can include things such as backwards-compatible bug fixes, 
  internal code restructuring, and performance improvements.
- MINOR: The minor version should be incremented whenever we add new
  backwards-compatible features to our code. This can include the addition of new
  functions and classes. Whenever we increment the minor version, the patch version is
  reset to zero.
- MAJOR: The major version should be incremented whenever we make a change that breaks
  backwards compatibility. This could include changing a function name or signature,
  or removing something from the public API. It can also include changing a function's
  behaviour in a significant way. Whenever we increment the major version, both the
  minor version and patch version should be reset to zero.



We can add a version to our code by adding a `__version__` to our top-most `__init__.py`
file:

```python
# file: epi_models/__init__.py

__version__ = "1.2.3"
```

This should also be reflected in our `pyproject.toml`:

```toml
# file: pyproject.toml

[project]
version = "1.2.3"
```

To avoid having to keep these up-to-date manually, we'll see later how to automatically
generate version info using `git` tags.

Something to note about the major and minor version numbers is that they only need to
be updated if we change the _public API_ of our software. Therefore, if we change the
behaviour of any functions or classes intended for internal use, and the public API is
unchanged, we only need to update the patch number.



### Unstable Versions

Software with a major version of 0 is usually understood to be 'unstable'. This means
that anything can change between minor versions, and that no promises can be made for
backwards compatibility. This is common for software in early stages of development,
as the public API may undergo many iterations before being finalised.

For your own software, it may be a good idea to keep the major number at zero while
you're hashing out the public API and building a userbase. Once you've settled on a
stable API, or you've developed a sizable userbase that won't appreciate having to
regularly update their code to match your changing standards, you should consider
setting your version to 1.0.0.

If one of the dependencies in your project is still in an unstable state, it may be
a good idea to fix the minor version number instead of using the compatible release
comparator `~=`:

```toml
dependencies = [
    "unstable_project >= 0.6.2, == 0.6.*"
]
```

This is because the `~= 0.6` will get the latest release which is at least `0.6.0` but
less than`1.0.0`, but in this case version `0.7` might break backwards compatibility
with `0.6`. Note that some unstable projects may also break backwards compatibiliy in
their patch releases. In these cases, it may be better to fix even the patch number:

```toml
dependencies = [
    "unstable_project == 0.6.2"
]
```

### Deprecation

It is considered good manners to give our users plenty of warning if we plan to remove
features that they might depend upon in our next major release. For this reason, many
developers will implement deprecation warnings to alert users that the API is due to
change in future, and that they should update their code accordingly. For example:

```python
import warnings

def myfunc():
    warnings.warn(
        "myfunc is deprecated, and wil be removed in version 4",
        DeprecationWarning,
    )
    return 42
```

When adding deprecations to our code, we should update the minor version, and include
them in our changelog. When we follow through on our deprecations and remove features,
this should also be clearly stated in the changelog of our major release.

### Extra: Advanced Versioning

:::::::::::::::::::::::::::: instructor

This section may be skipped.

:::::::::::::::::::::::::::::::::::::::

There are many more fine-grained versioning techniques that may be employed as our
projects grow in complexity, although these are not universally adopted and may only be
applicable to very large projects with a detailed review process. The full list of
version types accepted by tools such as `pip` is listed in [PEP 440][PEP 440], while the
formal specification for semantic versioning can be found at [semver.org][semver]. For
example, we may provide pre-release specifiers:

- 1.1.0a0: The first alpha version of 1.1.0. This is intended for use by developers of
  the project and (perhaps) a small group of users, and may be subject to large changes.
  1.1.0a0 comes after 1.0.N, but before 1.1.0. A second alpha version may be listed as
  1.1.0a1.
- 1.1.0b0: The beta version of 1.1.0. This follows 1.1.0aN, and is intended for use by a
  wider pool of users for further testing and feedback. This is usually close to the
  finished product, but some large changes remain possible.
- 1.1.0rc0: The release-candiate version of 1.1. Sometimes called a 'preview' version,
  it is not expected that further significant changes will be made in version 1.1, but
  some small changes may be included. It comes after 1.1.0bN, but before 1.1.0.

The number following the pre-release specifier is optional, with no number interpretted
as 0 (for example, `1.0.0a == 1.0.0a0`). A dot, hyphen, or underscore separator is also
allowed, such as `1.0.0-a1`, `1.1.0.beta.3`, or `1.1.0-rc.2`. We may also make
_post-releases_ if we wish to make minor edits after a release:

- 1.1.0post0: An update after a release that does not change the distributed software.
  For example, a change in documentation or release notes. It comes after 1.1.0 but before
  1.1.1.

Similarly, post-release specifiers may have dot, hypen or underscore separators, and
may exclude the trailing number. The use of 'rev' or 'r' in place of 'post' is also
common. Note that a post-release is _not_ the appropriate place for a quick bug fix
after a full release -- that should be an update to the patch number.

Finally, it is possible to create local versions in cases where a developer has made
progress on a new development but has not finalised their work in a new release. If you
use a tool such as Git to control your versioning, it may automatically generate
the local version. In general, there is no ordering to local versions. A local version
is specified with a `+` followed by a dot-separated list of numbers, letters, and
hyphens, for example `1.1.0+001` or `1.2.0-alpha-630-g60eca14`.

## Preparing to Publish

### README Files

Before sharing our software with the world, it is a good idea to let others know what it
is and how it works! A good README file should explain what a project is for, how users
can install it, and how to use it. For example, we may use the following file
`README.md`, written using markdown:

```markdown
    # epi_models

    A collection of Python utilities for solving epidemiology problems.

    ## Installation

    The project can be installed using \`pip\`:

    ```
    $ pip install epi_models
    ```

    To install from this repo:

    ```
    $ git clone github.com/username/learn-hatch
    $ cd epi_models
    $ pip install .
    ```

    ## Usage

    The project offers a command line interface, which can be used as
    follows:

    ```
    $ epi_models SIR input.yaml
    ```

    `SIR` can be exchanged for a model of your choice, and `input.yaml`
    is an input file with the following entries...
```

This should be included at the top level of our project:

<code>
&#128193; learn-hatch<br>
|<br>
|\_\_\_\_&#128220; pyproject.toml<br>
|\_\_\_\_&#128220; README.md<br>
\ \ \ \ \ &#128193; src<br>
\ \ \ \ \ |<br>
\ \ \ \ \ |\_\_\_\_&#128230; epi\_models<br>
\ \ \ \ \ \ \ \ \ \ |<br>
\ \ \ \ \ \ \ \ \ \ |\_\_\_\_&#128220; \_\_init\_\_.py<br>
\ \ \ \ \ \ \ \ \ \ |\_\_\_\_&#128220; \_\_main\_\_.py<br>
\ \ \ \ \ \ \ \ \ \ |\_\_\_\_&#128193; models<br>
\ \ \ \ \ \ \ \ \ \ |\_\_\_\_&#128193; plotting<br>
</code>

It should be included in our package metadata by adding the following line in our
`pyproject.toml`:

```toml
# file: pyproject.toml

[project]
readme = "README.md"
```

### Licensing

It is important for our software to have a license so that any potential users know
what their rights are with regards to usage and redistribution. This can also provide us
with legal protections in some cases.

There are several open source licenses available, and they can be applied to our
project simply by adding a `LICENSE`, `LICENSE.txt`, or `LICENSE.md` file to the top
level of our project:

<code>
&#128193; learn-hatch<br>
|<br>
|\_\_\_\_&#128220; pyproject.toml<br>
|\_\_\_\_&#128220; README.md<br>
|\_\_\_\_&#128220; LICENSE.md<br>
\ \ \ \ \ &#128193; src<br>
\ \ \ \ \ |<br>
\ \ \ \ \ |\_\_\_\_&#128230; epi\_models<br>
\ \ \ \ \ \ \ \ \ \ |<br>
\ \ \ \ \ \ \ \ \ \ |\_\_\_\_&#128220; \_\_init\_\_.py<br>
\ \ \ \ \ \ \ \ \ \ |\_\_\_\_&#128220; \_\_main\_\_.py<br>
\ \ \ \ \ \ \ \ \ \ |\_\_\_\_&#128193; models<br>
\ \ \ \ \ \ \ \ \ \ |\_\_\_\_&#128193; plotting<br>
</code>



One of the simplest and most widely used licenses is the MIT License, which is very
permissive. It requires users of your software to retain its copyright notice if
they redistribute or modify it (or 'substantial portions' of it), but otherwise allows
users to do what they wish. It also ensures the author is not liable for anything
the users do with their software:

```
MIT License

Copyright (c) [year] [fullname]

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```

Information on alternative open source licenses can be found at
[choosealicense.com][licenses]. Note that your choice of license may be determined
by your dependencies. For example, if your code uses software licensed under the GNU
GPLv3.0 license, it too should be licensed under GPLv3.0.

We should include our license in our `pyproject.toml` file as follows:

```toml
[project]
license = {file = "LICENSE.md"}
```

We can instead simply state the name of the license:

```toml
[project]
license = {text = "MIT License"}
```

If you're developing code on behalf of an organisation, it's worth checking in with
their legal team (or at the very least, your supervisor) before deciding on a license.


## Publishing our Software on GitHub

`git` is a powerful version control tool that allows us to track changes to our source
code over time. This lesson is not the place for teaching how to use `git`, but it is
highly recommended that you use it for managing your Python projects.

GitHub is an online service for hosting `git`-based software projects, and it is a great
way to share our code and collaborate with others.


Once our code is hosted on GitHub, we can create releases and assign them a tag. This
tag should be the version number of the release. We will see later how to use GitHub
Actions to automatically publish our code on other services whenever a new release is
created.

Whenever we make new releases of our code, we should write a detailed changelog that
describes all of the changes since the last version. This should include new features, 
bug fixes, removals, and deprecations. A good changelog document should include all
code changes in reverse chronological order (so the latest changes should be first in
the list), and should credit anybody who contributed to the code.

With our code hosted on GitHub, it is possible for others to install it using `pip`:

```bash
$ pip install "git+https://github.com/user/project"
```

However, this will simply get the latest commit to the main branch, which may be a
work-in-progress and not a true release. We can specify a particular release by
providing the version number after an `@` sign:

```bash
$ pip install "git+https://github.com/user/project@1.2.3"
```

Note that we can also supply a branch name or a commit hash here:

```bash
$ pip install "git+https://github.com/user/project@branch"
$ pip install "git+https://github.com/user/project@1a2b3c4"
```

Projects hosted on GitHub can also be added to dependencies in `pyproject.toml` as so:

```toml
dependencies = [
    "mypkg @ git+https://github.com/user/mypkg@1.2.3",
]
```

However, it is relatively slow for `pip` to install in this way, as it must clone the
whole repository. We can instead install from a snapshot by pointing `pip` towards an
archived `.zip`:

```bash
$ pip install "https://github.com/user/project/archive/1.2.3.zip"
```

```toml
dependencies = [
    "mypkg @ https://github.com/user/mypkg/archive/1.2.3.zip",
]
```

However, this is still slower than installing from a dedicated Python package
repository such as PyPI, which hosts pre-built `.whl` files, and it only allows our
users to specify a single version of our code. This means that they may miss out on
crucial patch releases. In general, it is a good idea to use GitHub to host our source
code and to manage its development, and to use a service like PyPI to host packages that
our users can install.


## Extra: Consistent Versioning with `hatch-vcs`

An issue with using GitHub to create new releases is that the project version can
easily become desynced. The version needs to be specified in three places:

- `git` tags, determined by releases on GitHub
- The `version` field in `pyproject.toml`
- `__version__` in our `__init__.py`

It is possible to have all three determined by `git` tags using `hatch-vcs`. This
can be set as a requirement of the build system:

```toml
# file: pyproject.toml

# Build system configuration
[build-system]
requires = ["hatchling", "hatch-vcs"]

build-backend = "hatchling.build"
```

With this, we no longer need to provide an entry for `project.version`, and instead
should add:

```toml
# file: pyproject.toml

[project]
name = "epi_models"
# version = "1.2.3" # no longer needed!
dynamic = ["version"]
```

Following this, we should add the following section elsewhere in `pyproject.toml`:

```toml
# file: pyproject.toml

# Hatch versioning configuration
[tool.hatch.version]
source = "vcs"

# Version control system (VCS) versioning
[tool.hatch.version.vcs]
tag-pattern = "v*"  # Git tags starting with 'v' will be used for versioning
fallback-version = "0.0.0"

# Version file location for VCS
[tool.hatch.build.hooks.vcs]
version-file = "src/epi_models/_version.py"
```

Now, when we install or build the project, `hatch-vcs` will create a new file
`_version.py` inside of our built package. If our git tag is `1.2.3`, this will contain:

```python
# file generated by setuptools_scm
# don't change, don't track in version control
__version__ = version = '1.2.3'
__version_tuple__ = version_tuple = (1, 2, 3)
```

This can be retrieved at runtime by adding the following to `__init__.py`:

```python
# file: __init__.py

from epi_models._version import __version__

```

Now, our Git tags, `__version__`, and `pyproject.toml` `version` will automatically be
kept in sync.


We will need to add the dynamically generated `_version.py` to our `.gitignore`

```bash
# file: .gitignore

src/epi_models/_version.py
```

### Adding git tags

Before adding a new tag we should commit all our current changes and push them to our remote repo.

Then we can add a new tag like so:

```bash
# list tags
git tag

# Add new tag
git tag -a 0.1.0 -m "Initial minor release"

# Push the tag to github
git push origin 0.1.0

# list tags
git tag
```

Now that we have a new tag let's re-install the package and check the version:

```bash
pip install -e .

epi_models --version
```




## PyPI, the Python Packaging Index

[PyPI][PyPI] is the official package repository for the Python community. It is
equivalent to CRAN, used for the R programming language. This is the repository that
`pip` uses when we install remote packages from the command line. It is recommended to
upload packages here if we want our projects to reach a wider audience.

![PyPI_screenshot](fig/pypi.png){alt="The Python Packaging Index"}

To begin, you will need to create an account on [PyPI][PyPI] and its sister-site,
[TestPyPI][TestPyPI] -- the latter is used to check that our packages have been
uploaded and set up properly before committing to the standard repository.

To help people find our project after it's uploaded, we should add some keywords they
can search for. We can also add 'classifiers' that categorise our project and describe
its intended audience. A list of possible classifiers are available on the
[PyPI website][classifiers]. These should be added to our `pyproject.toml`:

```toml
# file: pyproject.toml

[project]

# keywords should contain search terms so users can
# find our project
keywords = [
    "epimediology",
    "virology",
    "SIR",
    "SEIR",
    "Compartmental",
    "Model",
]

# Classifiers are a standard set of categories for
# our project, including which level of development
# we're at, who our intended audience is, which
# license we're using, etc.
classifiers = [
    "Development Status :: 3 - Alpha",
    "Programming Language :: Python :: 3",
    "Intended Audience :: Science/Research",
    "Topic :: Scientific/Engineering :: Bio-Informatics",
    "License :: OSI Approved :: MIT License",
]

```

Before discussing how to get our project hosted on PyPI, we'll quickly discuss 'wheel'
files, which are the standard way for Python to distribute packages.

## Wheel files

When we install packages using `pip`, it first creates a 'wheel' file, with the
file extension `.whl`. For example:

```bash
$ cd epi_models
$ pip install .
```

```result
... installs dependencies ...

Building wheels for collected packages: epi-models
  Building wheel for epi-models (pyproject.toml) ... done
  Created wheel for epi-models: filename=epi_models-0.1.0-py3-none-any.whl size=5378 sha256=005bcad72e7f6cc773f4180960e402e3bdbdf0fcb6fb683a019d9521775edfc8
  Stored in directory: /tmp/pip-ephem-wheel-cache-69r0h48p/wheels/d9/4d/ac/bbc1437fd83635787dd0fb8c3df8da61fc7b57f6eaa2b0d17b
Successfully built epi-models
Installing collected packages: epi-models
Successfully installed epi-models-0.1.0
```

The installation goes through several steps:

- `pip` analyses our package and creates a `.whl` file.
- The `.whl` file is stored in a temporary directory.
- The package is installed to our system from the `.whl` file.

After installation, we can find our installed package at
`/path/to/my/env/lib/python3.8/site-packages/epi_models`, and within we'll find that
every file has been compiled to Python bytecode, contained within `__pycache__`
directories. So what is a wheel file, and how does it help with this process?

A wheel is a standard package distribution format, defined in [PEP 427][PEP 427]. It
is essentially a `.zip` file with a descriptive name that contains our package.
When we installed our package above it created the following wheel file:

```
epi_models-0.1.0-py3-none-any.whl
```

The filename has seveal components separated by dashes. In order, these are:

- The package name
- The package version
- Language implementation, such as `py3`, `py27`, or `py2`. Packages that are compatible
  with any Python version will be `py2.py3`. Packages that are only compatible with a
  particular Python implementation, such as CPython, may be listed `cpy38`.
- The ABI tag, which stands for 'Application Binary Interface'. This specifies the
  binary compatibility of the CPython API, and may take a value such as `abi3`. This
  won't be relevant unless you're distributing compiled extensions written in C, and
  will be `none` if your package is pure Python. This is beyond the scope of these
  lessons.
- Platform tag, which specifies which operating system your package is compatible with.
  `any` means it will run on Windows, Mac or Linux, while it may say something like
  `macosx_10_9_x86_64` if it includes compiled C extensions for Mac OSX 10.9 on an
  x86_64 architecture. The `manylinux` tag encompasses many different popular Linux
  distros.

When we upload our package to PyPI, we will do so in the form of a wheel file, which
contains all Python files in our package along with any compiled binaries for extensions
written in C. When a wheel file is installed, Python files are compiled to bytecode, and
the package is installed to the `site-packages` of our environment.

As the wheel file contains compatibility information in its name, `pip` can quickly
check to see whether any wheels uploaded to PyPI are compatible with our system
whenever we try to install something. In order to meet the needs of as many users as
possible, it's also possible for us to upload multiple wheels for each release -- one
for each targeted Python version and operating system.

## `hatch build`

So how do we create a wheel file to upload? The Hatch library can be used to build and publish wheels.

```bash
$ pip install hatch
```

As we already have a `pyproject.toml`, `build` has everything it needs to create a
wheel file. It can be called simply using:

```bash
$ hatch build
```

This will create a new directory `./dist` containing the following:

<code>
&#128193; dist<br>
|<br>
|\_\_\_\_&#128220; epi_models-0.1.0-py3-none-any.whl<br>
|\_\_\_\_&#128220; epi_models-0.1.0.tar.gz<br>
</code>

We can see the contents of the `.whl` file by unzipping it:

```bash
$ cd dist
$ unzip dist/epi_models-0.1.0-py3-none-any.whl
```

We'll see that it contains our package along with a second directory
`epi_models-0.1.0.dist-info`, which contains package metadata such as our license
and README file. Note that we should remove the unzipped directories from `./dist`
before the upload stage.

The second file `hatch build` created is a `.tar.gz` file -- a gzip-compressed tarball. This
is a 'source distribution', which is used as a backup by `pip` if it can't find a
suitable wheel file to install.

### Publish Wheels to test-pypi

The tool for uploading our package to PyPI is `hatch publish`.

Note: before we upload our package we will need to make sure we are using unique package names.

Change you package name from to `githubusername-epi_modules`.

You will need to change the package name in the following locations:
- pyproject.toml
    - project.name
    - project.scripts entrypoint
- The `src/epi_models` dir
- Any relative imports in the modules

Test that the package can still be installed.

```bash
pip uninstall epi_modules

pip install -e .

epi_modules --version
```


Lets commit those changes and add a new tag.

```bash
# list tags
git tag

# Add new tag
git tag -a 0.1.1 -m "Edit package name"

# Push the tag to github
git push origin 0.1.1

# list tags
git tag
```

We can test the distribution of our package by uploading to TestPyPI.

First we will need to create a "token" on test-pypi to use as a temporary password.

```bash
$ hatch publish --repo testpypi --user __token__ --auth <your_token_here>
```

To avoid entering you token on the cmd line you can provide details as env variables

```bash
export HATCH_INDEX_USER=__token__ 
export HATCH_INDEX_AUTH=<your-token>
```

For more authentication options see the [Hatch documentation](https://hatch.pypa.io/latest/how-to/publish/auth/).


Note that there is a registration process for TestPyPI, and a separate one for PyPI.

Once uploaded to TestPyPI we can check that everything looks correct using the web interface.

If you choose to `pip` download from TestPyPI you may need to specify that the dependencies are gotten 
from PyPI proper by using the `--extra-indec-url` flag. This is because the dependencies may not be available through TestPyPI.

```bash
$. pip install -i https://test.pypi.org/pypi/ --extra-index-url https://pypi.org/simple githubusername-epi_models==0.1.0
```


The process is the same for publishing to PyPi (but please don't publish this test project!).

Users will then be able to install our package to any machine
by calling the following:

```bash
$ pip install epi_models
```

Similarly, it can be included as a dependency to other Python projects:

```toml
# file: pyproject.toml

[project]
dependencies = [
    "epi_models",
]
```

As we continue to develop our software, we should upload each new version to PyPI.
Our users will then be able to request a particular version:

```toml
# file: pyproject.toml

[project]
dependencies = [
    "epi_models ~= 1.2",
]
```

## Extra: Automating Package Publishing with GitHub Actions

We saw earlier how to ensure that the version of our package was stored only in the
Git tags. We can similarly set up our GitHub project to automatically publish our
package whenever we make a new release. We can use GitHub Actions for this, which is a
service used for continuous integration. It can achieve tasks such as:

- Automatically run tests on a variety of platforms each time a new commit is pushed
  or version is published.
- Enforce a code style automatically.
- Publish work on each release.

GitHub Actions are controlled using a YAML file. Though the full specification is beyond
the scope of this course, the following file will cause GitHub to automatically upload
our package to PyPI with each new release:

```yaml
# Provide a name for the workflow
name: Upload Python Package

# Tell GitHub when to run the action
# This will run every time a new release is published
on:
  release:
    types: [published]

jobs:
  deploy:
    # Run the workflow on the latest Ubuntu version
    runs-on: ubuntu-latest
    steps:
      # This will checkout our GitHub project and enter
      # the directory
      - uses: actions/checkout@v3
      # This will set up a Python environment
      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.x'
      # Here we update pip to the latest version and
      # install 'hatch'
      - name: Install dependencies
        run: |
          python -m pip install --upgrade pip
          pip install hatch
      # Here we run build to create a wheel and a
      # .tar.gz source distribution.
      - name: Build package
        run: hatch build
      # Finally, we use a pre-defined action to publish
      # our package 
      - name: Publish package
        uses: pypa/gh-action-pypi-publish@release/v1
        with:
          user: __token__
          password: ${{ secrets.PYPI_API_TOKEN }}
```

This should be included in the file `.github/workflows/publish.yml`, where the `.github`
directory is located at the top level of our project (at the same level as
`pyproject.toml`). In order for this to work, we'll need to set up a
`PYPI_DEPLOYMENT_TOKEN` in the 'secrets' section of our project's GitHub settings.
A guide to doing this may be found on the [PyPI website][pypi_token].

:::::::::::::::::::::: callout

You should _never_ push sensitive information such as passwords directly to
your GitHub projects!

::::::::::::::::::::::::::::::

With this set up, our projects will be automatically pushed to PyPI each time we 
make a new release.

::::::::::::::::::::::::::::::::::::: keypoints

- Versioning our projects is important so that our users know what's compatible.
- GitHub is a powerful service for hosting our projects and managing their development.
- Each new release of our packages should be uploaded to PyPI using `hatch build` and `hatch publish`.

::::::::::::::::::::::::::::::::::::::::::::::::

---
title: "Building and Installing Packages using hatch"
teaching: 20
exercises: 0
---


:::::::::::::::::::::::::::::::::::::: questions 

- How can we manage our Python environment?
- How can we install our own packages?

::::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::: objectives

- Use `conda` to manage Python environments
- Understand what happens when we install a package
- Use `pip` and `hatch` to install packages to our local environment

::::::::::::::::::::::::::::::::::::::::::::::::

## Introduction

In the first lesson, we showed how to use the `PYTHONPATH` environment variable to
enable us to import our modules and packages from anywhere on our system. There are
a few disadvantages to this method:

- If we have two different versions of a package on our system at once, it can be
  tedious to manually update `PYTHONPATH` whenever we want to switch between them.
- If we have multiple Python environments on our system (using tools such as `venv` or
  `conda`), setting `PYTHONPATH` will affect all of them. This can lead to unexpected
  dependency conflicts that can be very hard to debug.
- If we share our software with others and require them to update their own
  `PYTHONPATH`, they will need to install any requirements for our package separately,
  which can be error prone.

It would be preferable if we could install our package using `pip`, the same way that
we would normally install external Python packages. However, if we enter the top level
directory of our project and try the following:

```bash
$ cd /path/to/my/workspace/epi_models
$ python -m pip install .
```

We get the following error:

```output
ERROR: Directory '.' is not installable. Neither 'setup.py' nor 'pyproject.toml' found.
```

In order to make our project installable, we need to add the either the file
`pyproject.toml` or `setup.py` to our project. For modern Python projects, it is
recommended to write only `pyproject.toml`. This was introduced by
[PEP 517][PEP 517], [PEP 518][PEP 518] and [PEP 621][PEP 621]  as a standard way to
define a Python project, and all tools that build, install, and publish Python packages
are expected to use it.


By making our project `pip`-installable, we'll also make it very easy to publish our
packages on public repositories -- this will be covered in our
[lesson on package publishing](./04-publishing.Rmd). After publishing our work, our
users will be able to download and install our package using `pip` from any machine
of their chocie!

To begin, we'll introduce the concept of a 'Python environment', and how these can help
us manage our workflows.

## Managing Python Environments

When working with Python, it can sometimes be beneficial to install packages to an
isolated environment instead of installing them globally. Usually, this is done to
manage competing dependencies:

- Project B might depend upon Project A, but may have been written to use version 1.0.
- Project C might also depend upon Project A, but may instead only work with version
  2.0.
- If we install Project A globally and choose version 2.0, then Project B will not
  work. Similarly, if we choose version 1.0, Project C will not work.

A good way to handle these sorts of conflicts is to instead use _virtual environments_
for each project. A number of tools have been developed to manage virtual environments,
such as `venv`, which is a standard built-in Python tool, and `conda`, which is a
powerful third-party tool. 

:::::::::::::::::::::: callout

You can `pip install` packages into a `conda` virtual environment.

::::::::::::::::::::::::::::::

If we're using Linux, we can find which Python environment we're using by calling:

```bash
$ which python
```

If we're using the default system environment, the result is something like the
following:

```output
/usr/bin/python
```

To create a new `conda` environment called "packaging" with Python v3.12 installed, we can call:

```bash
# Create a new environment
$ conda create -n packaging python=3.12 matplotlib hatch -c conda-forge

# It is a good idea to clean up cached and unused files 
$ conda clean --all

# You can always check available envs with
$ conda env list
```

Now we can activate our new conda environment:

```bash
# Start the env
$ conda activate packaging 

# To check which packages are installed in the env run:
$ conda list
```

Checking which Python we're running should now give a different result:

```bash
$ which python
```
```output
/Users/username/miniforge3/envs/packaging/bin/python
```

If we now install a new package, it will be installed within our new virtual environment
instead of being installed to the system libraries. For example:

```bash
$ pip install numpy
```

We can check the location of `numpy` using `pip show`:

```bash
pip show numpy
```

`site-packages` is a standard location to store installed Python packages. 


If we no longer wish to use this virtual environment, we can return to the base
environment by calling:

```bash
$ conda deactivate
```

Virtual environments are very useful when we're testing our code, as they allow us to
create a fresh Python environment without any of the installed packages we normally use
in our work. This will be important later when we add dependencies to our package, as
this allows us to test whether our users will be able to install and run our
code properly using a fresh environment.


## An Overview of TOML files

`pyproject.toml` is a TOML file, which stands for 'Tom's Obvious Minimal Langauge'
(named for its developer, Thomas Preston-Werner, who cofounded GitHub). There are many
configuration file formats in common usage, such as YAML, JSON, and INI, but the Python
community chose TOML as it provides some benefits over the competition:

- Designed to be human writable and human readable.
- Can map unambiguously to a hash table (a `dict` in Python).
- It has a formal specification, so has an unambiguous set of rules.

A TOML file contains a series of `key = value` pairs, which may be grouped into sections
using a header enclosed in square brackets, such as `[section name]`. The values are
typed, unlike some other formats where all values are strings. The available types are
strings, integers, floats, booleans, and dates. It is possible to store lists of values
in arrays, or store a series of key-value pairs in tables. For example:

```toml
# file: mytoml.toml

int_val = 5
float_val = 0.5
string_val = "hello world"
bool_val = true
date_val = 2023-01-01T08:00:00
array = [1, 2, 3]
inline_table = {key = "value"}

# Section headings allow us to define tables over
# multiple lines
[header_table]
name = "John"
dob = 2002-03-05


# We can define subtables using dot notation
[header_table.subtable]
foo = "bar"
```

We can read this using the `tomllib` library in Python:


```python
>>> import tomllib
>>> with open("mytoml.toml", "r") as f:
...     data = toml.load(f)
>>> print(data)
```

The result is a dictionary object, with TOML types converted to their corresponding
Python types:

```result
{
    'int_val': 5,
    'float_val': 0.5,
    'string_val': 'hello world',
    'bool_val': True,
    'date_val': datetime.datetime(2023, 1, 1, 8, 0),
    'array': [1, 2, 3],
    'inline_table': {'key': 'value'},
    'header_table': {
        'name': 'John', 
        'dob': datetime.date(2002, 3, 5),
        'subtable': {
            'foo': 'bar'
        }
    }
}
```


## Installing our package with `pyproject.toml`

First, we will show how to write a relatively minimal `pyproject.toml` file so that
we can install our projects using `pip`. We will then cover some additional tricks that
can be achieved with this file:

- Use alternative directory structures
- Include any data files needed by our code
- Generate an executable so that our scripts can be run directly from the command
  line
- Configure our development tools.

To make our package `pip`-installable, we should add the file `pyproject.toml` to the
top-level `learn-hatch` directory:

<code>
&#128193; learn-hatch<br>
|<br>
|\_\_\_\_&#128220; pyproject.toml<br>
|<br>
|\_\_\_\_&#128193; src<br>
\ \ \ \ \ |<br>
\ \ \ \ \ |\_\_\_\_&#128230; epi\_models<br>
\ \ \ \ \ \ \ \ \ \ |<br>
\ \ \ \ \ \ \ \ \ \ |\_\_\_\_&#128220; \_\_init\_\_.py<br>
\ \ \ \ \ \ \ \ \ \ |\_\_\_\_&#128220; \_\_main\_\_.py<br>
\ \ \ \ \ \ \ \ \ \ |<br>
\ \ \ \ \ \ \ \ \ \ |\_\_\_\_&#128193; models<br>
\ \ \ \ \ \ \ \ \ \ |\ \ \ \ |<br>
\ \ \ \ \ \ \ \ \ \ |\ \ \ \ |\_\_\_\_&#128220; \_\_init\_\_.py<br>
\ \ \ \ \ \ \ \ \ \ |\ \ \ \ |\_\_\_\_&#128220; SIR.py<br>
\ \ \ \ \ \ \ \ \ \ |\ \ \ \ |\_\_\_\_&#128220; SEIR.py<br>
\ \ \ \ \ \ \ \ \ \ |\ \ \ \ |\_\_\_\_&#128220; SIS.py<br>
\ \ \ \ \ \ \ \ \ \ |\ \ \ \ |\_\_\_\_&#128220; utils.py<br>
\ \ \ \ \ \ \ \ \ \ |<br>
\ \ \ \ \ \ \ \ \ \ |\_\_\_\_&#128193; plotting<br>
\ \ \ \ \ \ \ \ \ \ \ \ \ \ \ |<br>
\ \ \ \ \ \ \ \ \ \ \ \ \ \ \ |\_\_\_\_&#128220; \_\_init\_\_.py<br>
\ \ \ \ \ \ \ \ \ \ \ \ \ \ \ |\_\_\_\_&#128220; plot\_SIR.py<br>
\ \ \ \ \ \ \ \ \ \ \ \ \ \ \ |\_\_\_\_&#128220; plot\_SEIR.py<br>
\ \ \ \ \ \ \ \ \ \ \ \ \ \ \ |\_\_\_\_&#128220; plot\_SIS.py<br>
</code>


The first section in our `pyproject.toml` file should specify which build system we
wish to use, and additionally specify any version requirements for packages used to
build our code. 

We will choose to use `hatchling`, which requires the following:

```toml
# file: pyproject.toml

[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"
```

- `requires` is set to a list of strings, each of which names a dependency of the build
  system and (optionally) its minimum version. This uses the same version syntax as
  `pip`.
- `build-backend` is set to a sub-module of `hatchling` which implements the
  [PEP 517][PEP 517] build interface.

With our build system determined, we can add some metadata that defines our project.
At a minimum, we should specify the name of the package, its version, and our
dependencies:


```toml
# file: pyproject.toml

# Build system configuration
[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"

# Project metadata
[project]
name = "epi_models"
version = "0.1.0"
dependencies = [
    "matplotlib",
]

# Hatch build configuration
[tool.hatch.build]
source = "src"

```

That's all we need! We'll discuss versioning in our
[lesson on publishing](04-publishing.Rmd). With this done, we can install our package
using:

```bash
$ pip install .
```

This will automatically download and install our dependencies, and our package will be
importable regardless of which directory we're in.

The installed package can be found in the directory
`/Users/username/miniforge3/envs/packaging/lib/python3.12/site-packages/epi_models`.

If we look inside our installed package, we'll see that our files have been copied, and
there is also a `__pycache__` directory:

```bash
$ ls /path/to/my/env/lib/python3.12/site-packages/epi_models
```

```results
__init__.py  __main__.py  models  plotting  __pycache__
```

The `__pycache__` directory contains Python bytecode, which is a lower-level version of
Python that is understood by the Python Virtual Machine (PVM). All of our Python code
is converted to bytecode when it is run or imported, and by pre-compiling our package
it can be imported much faster. If we look into the directories `models` and `plotting`,
we'll see those have been compiled to bytecode too.

If we wish to uninstall, we may call:

```bash
$ pip uninstall epi_models
```

We can also create an 'editable install', in which any changes we make to our code are
instantly recognised by any codes importing it -- this mode can be very useful when
developing our code, especially when working on documentation or tests. 

```bash
$ pip install -e .
```

:::::::::::::::::::::::::::::::: callout

The ability to create editable installs from a `pyproject.toml`-only build was
standardised in [PEP 660][PEP 660], and only recently implemented in `pip`. You may
need to upgrade to use this feature:

```bash
$ python -m pip install --upgrade pip
```

::::::::::::::::::::::::::::::::::::::::

There are many other options we can add to our `pyproject.toml` to better describe our
project. [PEP 621][PEP 621] defines a minimum list of possible metadata that all build
tools should support, so we'll stick to that list. Each build tool will also define
synonyms for some metadata entries, and additional tool-specific metadata. Some of
the recommended core metadata keys are described below:

```toml
# file: pyproject.toml


# Build system configuration
[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"

# Project metadata
[project]
name = "epi_models"
version = "0.1.0"
dependencies = [
    "matplotlib",
]

# A simple summary of the project
description = "My wonderful Python package"


# Full description of the project.
# Should be the path to your README file, relative to pyproject.toml
readme = "README.md"

# The Python version required by the project
requires-python = ">=3.8"

# The license of your project.
# Can be provided as a file or a text description.
# Discussed in the lesson on publishing
license = {file = "LICENSE.md"}
# or...
license = {text = "BDS 3-Clause License"}



# Authors / maintainers
# Each entry can have a name and/or an email
authors = [
    {name = "My Name", email = "my.email@email.net"},
    {name = "My Friend", email = "their.email@email.net"},
]

# dependencies: Array of Strings
# A list of requirements for our package
dependencies = [
    "matplotlib",
]

# Project URLs
[project.urls]
homepage = "https://github.com/username/learn-hatch"
documentation = "https://github.com/username/learn-hatch"
repository = "https://github.com/username/learn-hatch"

# Hatch build configuration
[tool.hatch.build]
source = "src"



```

TODO: Discuss including package data file with hatch.

## Installing Scripts

If our package contains any scripts and/or a `__main__.py` file, we can run those from
anywhere on our system after installation:

```bash
$ python -m epi_models
$ python -m epi_models.plotting.plot_SIR
```

With a little extra work, we can also install a simplified interface that doesn't
require `python -m` in front. This is how tools like `pip` can be invoked using two
possible methods:

```bash
$ python -m pip  # Invoke with python
$ pip             # Invoke via console-scripts entrypoint
```

This can be achieved by adding a table `scripts` under the `[project]` header:

```toml
# file: pyproject.toml

# Add entry points
[project.scripts]
epi_models = "epi_models.__main__:main"
```

This syntax means that we should create a console script `epi_models`, and that running
it should call the function `main()` from the file `epi_models/__main__.py`. This will
require a slight modification to our `__main__.py` file. All that's necessary is to
move everything from the script into a function `main()` that takes no arguments,
and then to call `main()` at the bottom of the script:

```python
# file: main.py

def main():
    # Put the __main__ script here...

main()
```

This will allow us to run our package as a script directly from the command line

```bash
$ python -m pip install .
$ epi_models --help
```

Note that we'll still be able to run our code using the longer form:

```bash
$ python -m src/epi_models --help
```

If we have multiple scripts in our package, these can all be given invidual console
scripts. However, these will also need to have a function name as an entry point:

```toml
# file: pyproject.toml

[project.scripts]
epi_models = "epi_models.__main__:main"
epi_models_sir = "epi_models.plotting.plot_SIR:main"
```

TODO: remove this? 

So how do these scripts work? When we activate a virtual environment, a new entry is
added to our `PATH` environment variable linking to `/path/to/my/env/bin/`:

```bash
PATH = "/path/to/my/env/bin:${PATH}"
```

After installing our console scripts, we can find a new file in this directory with
the name we assigned to it. For example, `/path/to/my/env/bin/epi_models`:

```python
#!/path/to/my/env/bin/python
# -*- coding: utf-8 -*-
import re
import sys
from epi_models.__main__ import main
if __name__ == '__main__':
    sys.argv[0] = re.sub(r'(-script\.pyw|\.exe)?$', '', sys.argv[0])
    sys.exit(main())
```

Installing our project has automatically created a new Python file that can be run as a
command line script due to the hash-bang (`#!`) on the top line, and all it does it
import our main function and run it. As it's contained with the `bin/` directory of
our Python environment, it's available for use as long we're using that environment,
but as soon as we call `deactivate`, it is removed from our `PATH`.

## Setting Dependency Versions

TODO: make this an info box.

Earlier, when setting `dependencies` in our `pyproject.toml`, we chose to specify
a minimum requirement for `numpy`, but not for `pyyaml`:

```toml
# file: pyproject.toml

dependencies = [
    "numpy >= 1.20",
    "pyyaml",
]
```

This indicates that `pip` should install any version of `numpy` greater than 1.20, but
that any version of `pyyaml` will do. If our installed `numpy` version is less than
1.20, or if it isn't installed at all, `pip` will upgrade to the latest version that's
compatible with the rest of our installed packages and our Python version. We'll cover
software versioning in more detail in the [lesson on publishing](05-publishing.Rmd), but
now we'll simply cover some ways to specify which software versions we need:

```toml
"numpy >= 1.20"  # Must be at least 1.20
"numpy > 1.20"   # Must be greater than 1.20
"numpy == 1.20"  # Must be exactly 1.20
"numpy <= 1.20"  # Must be 1.20 at most
"numpy < 1.20"   # Must be less than 1.20
"numpy == 1.*"   # Must be any version 1
```

If we separate our clauses with commas, we can combine these requirements:

```toml
# At least 1.20, less than 1.22, and not the release 1.21.3
"numpy => 1.20, < 1.22, != 1.21.3"
```

A useful shorthand is the 'compatible release' clause:

```toml
"numpy ~= 1.20"  # Must be a release compatible with 1.20
```

This is equivalent to:

```toml
"numpy >= 1.20,  == 1.*"
```

That is, we require anything which is version 1, provided it's greater than 1.20.
This would include version 1.25, but exlude version 2.0. We'll come back to this later
when we discuss [publishing](05-publishing.Rmd).

## Optional Dependencies

Sometimes we might have dependencies that only make sense for certain kind of user.
For example, a developer of our library might need any libraries we use to run
unit tests or build documentation, but an end user would not. These can be
added as `optional-dependencies`:

```toml
# file: pyproject.toml

[project.optional-dependencies]
test = [
    "pytest >= 5.0.0",
]
doc = [
    "sphinx",
]
```

These dependencies can be installed by adding the name of each optional dependency
group in square brackets after telling `pip` what we want to install:

```bash
$ pip install ".[test]"     # Include testing dependencies
$ pip install ".[doc]"     # Include documentation dependencies
$ pip install ".[test,doc]" # Include all dependencies
```



::::::::::::::::::::::::::::: keypoints

- We can configure our projects for `pip` installation by setting up a `pyproject.toml`
  file.
- Simple projects require very little to be added to this file, but there are many
  optional extras we can add.

:::::::::::::::::::::::::::::::::::::::

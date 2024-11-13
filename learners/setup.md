---
title: Setup
---

This lesson teaches how to create and publish packages in Python. We assume a basic
understanding of Python, and assume learners are comfortable using simple commands in a
Bash shell. The lesson assumes learners are working in Linux, but we hope to update this
to include other operating systems in future. For the final lesson on publishing Python
code, it will also be helpful to understand how to use Git to manage software projects.
A lesson on Git is available with the [Software Carpentires][sc-git].

Some sections are marked 'extra', and these contain non-essential information that may
be of interest to some learners. These sections do not need to be covered in order to
understand the core content of the course.


## Python environments

In this workshop we will use `conda` to manage Python environments and install packages.

Our preferred flavour of `conda` is provided by Miniforge3, but you could also use Miniconda.

### Windows

Windows users will need to install `GitBash` which is a linux-like terminal that comes pre-installed with Git.
You will use `conda` from within the `GitBash` shell.

Install:
- [GitBash](https://gitforwindows.org/)
- [Miniforge3](https://conda-forge.org/miniforge/)

Windows checks:

- Open the `GitBash` terminal app.
- Check for `(base)` prefix in your prompt. 
- If `(base)` is not present, open the "miniforge prompt" app and run `conda init bash`
- Run `conda --version`


### Mac

Install:
- [Miniforge3](https://conda-forge.org/miniforge/)

Mac checks:

- Open a terminal and run `git --version`. You may be prompted to install git.
- Run `conda --version`

## Configure Git

If you are setting up Git for the first time, you will need to set your user name and email.

```bash
$ git config --global user.name "Alfredo Linguini"
$ git config --global user.email "a.linguini@ratatouille.fr"
```

Please use your own name and email address instead of Alfredo's. And make sure you use the same email associated with your GitHub account.

We will also set the default editor as `nano`.

```bash
$ git config --global core.editor "nano -w"
```

## Setup Github

### Create a GitHub account

Create a new account at [GitHub.com](github.com}) if you do not have one.

Use an email address that you will always have access to (not a work or university address)

### Configure SSH access

If you do not have ssh set up for git [create a new ssh key](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent) and [add the key](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/adding-a-new-ssh-key-to-your-github-account) to your github account.

On windows you should do this from the GitBash shell.

To test that your key is correctly configured run:

```bash
ssh -T git@github.com
```


## VS Code setup

VS Code is a popular Interactive Development Environment (IDE) that we will use to edit our project.

Install:
- [VS Code](https://code.visualstudio.com/Download)

Windows users:
- Open VS Code and set GitBash as default terminal

VS Code has a marketplace of community developed extensions. Here are a few that you may find useful when working on Python projects.

Optional extensions:
- python language support
- git lens
- remote ssh
- github pull requests
- ruff

## Create Test-Pypi account

PyPi is the Python Package Index, it hosts publically available python packages.

We will practice uploading packages on a clone of PyPi called `test-pypi`.

Setup your test-pypi account:
- Set up a new account on [test.pypi.org](https://test.pypi.org/)
- Confirm your email address.
- Set up 2-factor authentication.

## Create project repo

Create a new repo in your github account called `learn-hatch` and clone it to your local machine.

Tutor to discuss `.gitignore` and licences.
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


TODO: Updated setup instructions

## Windows

- Install GitBash
- Install miniforge


Windows checks:

- Open GitBash
- Check for (base) and conda version
- If base not present, open miniforge prompt and run `conda init bash`

## Mac

- Install miniforge

Mac checks:

- git --version
- conda version

## VS Code setup

Install VS Code

Windows users:
- Open VS Code and set GitBash as default terminal

Install Extensions:
- python language support
- git lens
- remote ssh
- github pull requests
- ruff

## Conda config

conda update -n base -c conda-forge conda

TODO: Check for other recommended settings

## Git config

Set global variables:
- name
- email
- default editor

## Create Github account

- create ssh key and add to github account
- test key wit ssh -T git@github.com

## Create env for this workshop

Clone demo project repo + all branches

New conda env from env.yml

`conda env create -f environment.yml`

conda env list

cleanup tarballs etc

activate env

Check packages with conda list

# Create Pypi account

may need to create token to link to github
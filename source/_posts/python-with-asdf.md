---
title: python setup with asdf
date: 2021-08-27 17:26:47
tags:
- tech
- python
---

### on asdf

I first discovered the [asdf](https://asdf-vm.com/) tool when I was trying to manage different version of Elixir language on my machine. Later I discovered that I can use the same tool for managing versions of other languages (and tools!) using asdf plugins.

### on python setup

Setting up python locally is easy.

Setting up python multiple projects, each using a different version of python, along with their possibly conflicting dependencies is complicated. There are various tools like [conda](https://docs.conda.io/en/latest/), [pyenv](https://github.com/pyenv/pyenv), [virtualenv](https://virtualenv.pypa.io/en/latest/) which tackle different (or overlapping) sub-sets of this problem.

I found the combination of pyenv (to manage different versions of python) + virtualenv (to separate out the dependencies of projects) the most tenable. Then I discovered that asdf can be a drop in replacement for pyenv. I also use [virtualfish](https://virtualfish.readthedocs.io/en/latest/), the fish shell wrapper over virtualenv

Please install asdf's [python-plugin](https://github.com/danhper/asdf-python.git)
```shell
# Install python version
asdf install python <version>

# Set python version to be used globally
asdf global python 3.9.5

# Set python version to be used in the current folder
asdf local python 3.7.11
```

Once the requisite python version installed, a virtual environment can be created as follows
```shell
vf new -p python3.7.11 my-virtual-env
```
_Please note_: If the python version is not specified virtualfish always takes up the global version irrespective of the currently set local python version. This maybe a bug in my configuration.

### on setting python for homebrew

[Homebrew](https://brew.sh/) likes to install and manage its own version of python. It will often get installed as a transitive dependency of some tool (pgcli in my case). It bugged me that I had 2 very similar versions of python (3.9.5 vs 3.9.6 iirc) installed on the system, one maintained by brew the other by asdf. Luckily I found a [post](https://towardsdatascience.com/homebrew-and-pyenv-python-playing-pleasantly-in-partnership-3a342d86319b) that lists how to use a pyenv installed python version with homebrew. I did the same, et voila homebrew now uses the python version maintained by asdf.

```shell
ln -s ~/.asdf/installs/python/3.9.6 $(brew --cellar python)/3.9.6
```
_Please note_: If there's no python version managed by brew, then we may need to create the directory
```
mkdir $(brew --cellar python)
```

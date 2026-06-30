# Exercise 04 - MiniPack

Building a Python package that can be installed with pip. This involves understanding
the standard packaging structure, metadata files, and the build tools that produce
distributable archives.

- A Python package is a directory with an `__init__.py` that makes it importable
- `setup.cfg` (or `pyproject.toml`) holds the package metadata — name, version, author, etc.
- `python -m build` produces a `.whl` (wheel) and `.tar.gz` (sdist) in `dist/`
- `build.sh` automates the build; it should upgrade pip and setuptools first

## Directory Structure

```
ex04/
├── build.sh
├── setup.cfg          # metadata
├── setup.py           # minimal shim (can just call setup())
├── README.md
├── LICENSE.md
└── my_minipack/
    ├── __init__.py
    ├── progress.py    # ft_progress from m00 ex10
    └── logger.py      # log decorator from m02 ex02
```

The directory name `my_minipack` must match the package name exactly. The `__init__.py`
makes it a package — it can be empty, or it can re-export symbols for convenience.

## setup.cfg

The declarative metadata file:

```ini
[metadata]
name = my-minipack
version = 1.0.0
author = schmitzi
author_email = schmitzi@student.42vienna.com
description = Howto create a package in Python.
long_description = file: README.md
long_description_content_type = text/markdown
license = GPLv3
classifiers =
    Development Status :: 3 - Alpha
    Intended Audience :: Developers
    Topic :: Education
    License :: OSI Approved :: GNU General Public License v3 (GPLv3)
    Programming Language :: Python :: 3
    Programming Language :: Python :: 3 :: Only

[options]
packages = find:
python_requires = >=3.7
```

`find:` tells setuptools to automatically discover all packages (directories with
`__init__.py`) rather than listing them manually.

## setup.py

A minimal shim — just enough to invoke setuptools:

```python
from setuptools import setup

if __name__ == "__main__":
    setup()
```

All the real config lives in `setup.cfg`. This file is still needed for editable
installs and some tooling compatibility.

## build.sh

```bash
#!/bin/bash
pip install --upgrade pip setuptools wheel build
python -m build
```

`python -m build` runs the PEP 517 build frontend, which produces both the wheel
(`.whl`) and the source distribution (`.tar.gz`) in `dist/`. Make it executable:
`chmod +x build.sh`.

## What a Wheel Is

A `.whl` file is a zip archive with a specific naming convention:
`my_minipack-1.0.0-py3-none-any.whl`

- `py3` — works with any Python 3 implementation
- `none` — no ABI dependency (pure Python, no compiled extensions)
- `any` — platform independent

Pip installs wheels by unpacking them into `site-packages`. Source distributions
(`.tar.gz`) require a build step on the target machine; wheels don't.

## Import Path

Once installed, the modules are importable as:

```python
import my_minipack.progress
import my_minipack.logger

# or
from my_minipack.progress import ft_progress
from my_minipack.logger import log
```

The `__init__.py` in `my_minipack/` controls what happens when you `import my_minipack`
directly. You can leave it empty or add `from .progress import ft_progress` to make
`my_minipack.ft_progress` work.

## Verifying the Install

```bash
pip install ./dist/my_minipack-1.0.0-py3-none-any.whl
pip show -v my_minipack   # check metadata
pip list                   # confirm it appears
```

Test in a venv to confirm it installs cleanly without depending on anything else in
your environment:

```bash
python -m venv tmp_env
source tmp_env/bin/activate
pip install ./dist/my_minipack-1.0.0-py3-none-any.whl
python -c "from my_minipack.logger import log; print('ok')"
```

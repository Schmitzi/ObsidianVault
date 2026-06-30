# Exercise 00 - $PATH

Setting up a Python environment correctly is something that bites everyone at least once.
The problem is that most systems ship with Python 2 or a random version of Python 3 on
the PATH, and you need a specific version isolated from the system. This exercise is about
learning to manage that using pip/conda.

- System Python and project Python need to stay separate
- `pip` manages packages, `conda` manages packages *and* Python versions
- A `requirements.txt` is how you communicate dependencies to other people (and your future self)
- The `pip freeze` output is the ground truth for what's actually installed

## The Commands You Need

The five things the exercise asks you to find:

**List installed packages and versions:**

```bash
pip list
# or more verbosely:
conda list
```

**Show metadata for a specific package:**

```bash
pip show numpy
```

This gives you version, author, dependencies, install location — useful for debugging
version conflicts.

**Remove a package:**

```bash
pip uninstall numpy
```

**Reinstall a package:**

```bash
pip install numpy
```

**Freeze the environment to requirements.txt:**

```bash
pip freeze > requirements.txt
```

`pip freeze` outputs every installed package pinned to an exact version. Redirecting
it into a file gives you a snapshot that anyone can reproduce with `pip install -r requirements.txt`.

## Why Conda

`conda` goes further than pip — it manages the Python interpreter itself, not just
packages. The bootcamp has you create a named environment so you can switch between
Python versions per project without anything leaking into each other.

```bash
conda create --name 42AI-schmitzi python=3.7 numpy pandas pycodestyle jupyter
conda activate 42AI-schmitzi
which python   # should point inside the conda env, not /usr/bin/python
python -V      # should say 3.7.x
```

Once the environment is active, `pip` installs into it, not the system. Deactivate
with `conda deactivate`.

## Gotcha: pip vs conda install

Inside a conda environment, prefer `conda install` over `pip install` when the package
is available on conda — it handles binary compatibility better. For packages only on
PyPI, `pip install` inside the active env is fine.

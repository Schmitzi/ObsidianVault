# Installing packages

To install and set conda, I used this command:

```sh
MYPATH="$HOME/.local/share/miniconda3"
curl -LO "https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh"
sh Miniconda3-latest-Linux-x86_64.sh -b -p $MYPATH
$MYPATH/bin/conda init zsh
$MYPATH/bin/conda config --set auto_activate_base false
source ~/.zshrc
conda create --name 42AI python=3.7 jupyter pandas pycodestyle numpy
conda activate 42AI
```

There is also an installation script included

## Pip

Pip is a Python package manager. We need to use a few commands to get familiar with it.

- *Print out a list of installed packages and their version:*

```
pip list

Package 				Version
--------------------------------------  ----------
absl-py					  0.7.0
```

- *Show the metadata of a package, use:*

```
pip show numpy

(42AI) ➜  00 git:(main) ✗ pip show numpy
Name: numpy
Version: 1.21.5
Summary: NumPy is the fundamental package for array computing with Python.
Home-page: https://www.numpy.org
Author: Travis E. Oliphant et al.
Author-email: 
License: BSD
Location: /home/schmitzi/.local/share/miniconda3/envs/42AI/lib/python3.7/site-packages
Requires: 
Required-by: Bottleneck, mkl-fft, mkl-random, numexpr, pandas
```

- *To remove a package, use:*

```
pip uninstall numpy

(42AI) ➜  00 git:(main) ✗ pip uninstall numpy
Found existing installation: numpy 1.21.5
Uninstalling numpy-1.21.5:
  Would remove:
    /home/schmitzi/.local/share/miniconda3/envs/42AI/bin/f2py
    /home/schmitzi/.local/share/miniconda3/envs/42AI/bin/f2py3
    /home/schmitzi/.local/share/miniconda3/envs/42AI/bin/f2py3.7
    /home/schmitzi/.local/share/miniconda3/envs/42AI/lib/python3.7/site-packages/numpy-1.21.5.dist-info/*
    /home/schmitzi/.local/share/miniconda3/envs/42AI/lib/python3.7/site-packages/numpy/*
Proceed (Y/n)? y
  Successfully uninstalled numpy-1.21.5
```

- *To (re)install it, use:*


```
pip install numpy

(42AI) ➜  00 git:(main) ✗ pip install numpy
Collecting numpy
  Downloading numpy-1.21.6-cp37-cp37m-manylinux_2_12_x86_64.manylinux2010_x86_64.whl (15.7 MB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 15.7/15.7 MB 31.4 MB/s eta 0:00:00
Installing collected packages: numpy
Successfully installed numpy-1.21.6
```

- *To make a `requirements.md`, freezing version numbers of modules, use:*

```
pip freeze > requirements.txt

anyio @ file:///tmp/build/80754af9/anyio_1644481697509/work/dist
argon2-cffi @ file:///opt/conda/conda-bld/argon2-cffi_1645000214183/work
argon2-cffi-bindings @ file:///tmp/build/80754af9/argon2-cffi-bindings_1644569681205/work
attrs @ file:///croot/attrs_1668696182826/work
Babel @ file:///croot/babel_1671781930836/work
......
......
......
webencodings==0.5.1
websocket-client @ file:///tmp/build/80754af9/websocket-client_1614804260725/work
widgetsnbextension @ file:///tmp/build/80754af9/widgetsnbextension_1645009354203/work
zipp @ file:///croot/zipp_1672387121353/work
```


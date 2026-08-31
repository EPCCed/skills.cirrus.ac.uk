---
name: python
description: >
  Guide for installing and managing Python environments on Cirrus HPC system.
  Use this skill whenever a user asks about Python on Cirrus, setting up Conda/Miniforge,
  using uv, installing pip packages, mpi4py on HPC,
  virtual environments on Cirrus, or troubleshooting Python package builds.
  Also trigger for questions about Cray Python, build isolation, pyproject.toml.
compatibility: >
  Cirrus. Requires access to an Cirrus login node and Python
  environment tools such as Miniforge, pip, or uv.
metadata:
  author: cirrus-sc
  version: "1.0"
  source_url: https://docs.cirrus.ac.uk/user-guide/python/
---

# Python on Cirrus

Python and packages can be managed with `pip`, `conda`, or `uv`. **pip starting from the `cray-python` module** is the recommended approach. All workflows should use **virtual environments** to isolate dependencies.

Use:
- **Cray Python** as the starting point.
- **pip** for reproducible environment management, binary packages, and complex dependency sets.
- Always install virtual environments on the /epccfs file system to make them accessible on the compute nodes

## Critical Rules

- Always use a virtual environment for every project.
- Always use `python3 -m pip` rather than bare `pip`.
- Prefer pip on Cirrus, starting from the `cray-python` module.
- Do not run `conda init`; activate Miniforge manually.

---

## Cray Python (Recommended)

Python is available via the modules system:

```bash
module avail               # list all available modules
module load PrgEnv-gnu     # Load GCC compiler environment to ensure build compatibility for source packages
module load cray-python
which python3              # /opt/cray/pe/python/3.11.7/bin/python
```

The cray-python release contains

- python
- numpy
- scipy
- mpi4py
- dask

### Virtual environments with Cray Python

This example is for user "auser" in project "t01". These should be replaced with your
actual username and project ID.

```bash
mkdir -p ~/epccfs/t01/t01/auser/myvenv
python3 -m venv --system-site-packages /work/t01/t01/auser/myvenv
source /epccfs/t01/t01/auser/myenv/bin/activate
python3 -m pip install pandas
python3 -m pip list
deactivate
```

> **Tip:** Always use `python3 -m pip` rather than bare `pip` to ensure you're targeting the correct environment.

---

## Conda: Miniforge

Use [Miniforge](https://github.com/conda-forge/miniforge), not the main Anaconda distribution (which requires a license).

This example is for user "auser" in project "t01". These should be replaced with your
actual username and project ID.

### Install

```bash
cd /epccfs/t01/t01/auser
curl --location --remote-name \
  "https://github.com/conda-forge/miniforge/releases/latest/download/Miniforge3-$(uname)-$(uname -m).sh"
bash Miniforge3-$(uname)-$(uname -m).sh
rm Miniforge3-$(uname)-$(uname -m).sh
```

`$(uname)` → OS name (e.g. `Linux`); `$(uname -m)` → architecture (e.g. `x86_64`).

**Do not run `conda init`** — it modifies shell startup scripts and can cause complications. Instead, activate manually:

```bash
source /epccfs/t01/t01/auser/miniforge3/bin/activate
```

### Create and use environments

Never install packages into `base`. Always create a named environment:

```bash
(base) $ conda create --name myenv python=3.10
(base) $ conda activate myenv
(myenv) $ conda install scipy
```

To create from a YAML spec file:

```bash
conda env create --file environment.yml
```

Useful commands: `conda list` (list packages), `conda deactivate` (exit environment).

---

## Building Python Packages from Source on `aarch64`

Some packages require build from source. The sections below cover the main friction points.

### Compilers

The Cray Python distribution is build using GCC from the `PrgEnv-gnu` module so you should load this
module before installing any Python packages that build from source:

```bash
module load PrgEnv-gnu
```

### MPI libraries (`mpi4py`)

Use the `mpi4py` available via the `cray-python` module as it has been built against Cray MPICH
and is compatible with the Slingshot 11 interconnect.

---

## Quick Decision Guide

| Situation | Recommended approach |
|-----------|---------------------|
| General Python environment management | `cray-python` module with venv + pip. e.g. `python3 -m venv --system-site-packages /work/t01/t01/auser/myvenv` |
| MPI (mpi4py) | Use `mpi4py` from the `cray-python` module |

---

## Useful Resources

- [Cirrus documentation](https://docs.cirrus.ac.uk)
- [Cirrus Python documentation](https://docs.cirrus.ac.uk/user-guide/python/)
- [Miniforge on GitHub](https://github.com/conda-forge/miniforge)
- [Anaconda.org package search](https://anaconda.org)
- [mpi4py docs](https://mpi4py.readthedocs.io)

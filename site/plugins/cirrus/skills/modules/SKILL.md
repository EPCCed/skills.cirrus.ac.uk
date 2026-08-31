---
name: modules
description: >
  Guide for using the modules system, programming environments, compilers, and profiling tools
  on Cirrus UK National Compute Resource.
  Use this skill whenever a user asks about loading modules on Cirrus, the Cray Programming
  Environment (PrgEnv-gnu, PrgEnv-cray, PrgEnv-intel, PrgEnv-aocc), compiler wrappers (cc, CC, ftn),
  or Cray perftools.
  Also trigger for questions about linking MPI or scientific libraries on Cirrus, or
  troubleshooting compiler and build issues on the HPE Cray EX4000 system — even if the user
  does not explicitly say "modules".
compatibility: >
  Cirrus. Requires access to an Cirrus login node and the
  Cray Programming Environment module system.
metadata:
  author: cirrus-sc
  version: "1.0"
  source_url: https://docs.cirrus.ac.uk/user-guide/
---

# Modules and Compilers on Cirrus

Cirrus is an HPE Cray EX4000 system and use the **Cray Programming Environment** for managing compilers, MPI, and scientific libraries through a modular software system.

---

## Module Commands

| Command | Effect |
|---------|--------|
| `module avail` | List all available modules |
| `module load <name>` | Load a module into the current session |
| `module unload <name>` | Unload a module from the current session |
| `module list` | List all currently loaded modules |
| `module restore` | Switch back to default login environment module setup |
| `module spider <name>` | Search for a module and show details |

**Gotchas:**
- Users should never use `module purge` on Cirrus as it will break the Cray Programming Environment setup

---

## Programming Environments

### Available environments

```bash
module avail PrgEnv           # quick list
module spider PrgEnv       # detailed view
```

```
   PrgEnv-aocc/8.6.0    PrgEnv-cray/8.6.0 (L)    PrgEnv-gnu/8.6.0    PrgEnv-intel/8.6.0
```

**`PrgEnv-gnu` is usually recommended** — good performance, familiar behaviour, all dependencies loaded automatically:

```bash
module load PrgEnv-gnu
module list

Currently Loaded Modules:
  1) craype-x86-turin   3) craype-network-ofi       5) xpmem/1.0.1-1.5_1_gfb6998056825   7) epcc-setup-env     9) gcc-native/14.2  11) cray-dsmml/0.3.1   13) cray-libsci/25.03.0
  2) libfabric/2.3.1    4) perftools-base/25.03.0   6) cse_env/0.2                       8) load-epcc-module  10) craype/2.7.34    12) cray-mpich/8.1.32  14) PrgEnv-gnu/8.6.0
```

Key components loaded by `PrgEnv-gnu`:

| Module | Purpose |
|--------|---------|
| `gcc-native/14.2` | GCC 14.2 compiler suite |
| `libfabric` | Communication library for Slingshot 11 high-speed interconnect |
| `cray-libsci` | Scientific and math libraries (BLAS, LAPACK, etc.) |
| `cray-mpich` | MPI libraries |

### Cray compiler wrappers

With a `PrgEnv` loaded, use the **Cray compiler wrappers** rather than invoking the compiler directly:

| Wrapper | Language | GCC equivalent |
|---------|----------|----------------|
| `cc` | C | `gcc` |
| `CC` | C++ | `g++` |
| `ftn` | Fortran | `gfortran` |

The wrappers automatically link to loaded Cray PE libraries (MPI, LibSci, etc.). Do not mix wrapper commands and raw GCC commands in the same build.

To see exactly what the wrapper passes to the underlying compiler:

```bash
ftn -craype-verbose -o hello hello.f90
# Shows full flags, include paths, and linked libraries
```

---

## Profiling Tools

### gprof

Available once PrgEnv-gnu is loaded:

```bash
gprof ./my_application.x
```

### Cray Perftools

```bash
module load perftools-base
pat_run ./my_application.x
```

Full docs: [HPE Cray Performance Tools](https://cpe.ext.hpe.com/docs/latest/performance-tools/index.html)

---

## Quick Reference

| Goal | Command |
|------|---------|
| Recommended build environment | `module load PrgEnv-gnu` |
| Compile C | `cc -O3 myfile.c` |
| Compile C++ | `CC -O3 myfile.cpp` |
| Compile Fortran | `ftn -O3 myfile.f90` |
| Debug wrapper flags | `cc -craype-verbose ...` |
| Profile with Cray tools | `module load perftools-base` → `pat_run` |
| Spack builds | See [Spack guide](https://docs.cirrus.ac.uk/software-tools/spack/)  |

---

## Related Resources

- [Cirrus software environment guide](https://docs.cirrus.ac.uk/user-guide/sw-environment/)
- [Cirrus application developer environment guide](https://docs.cirrus.ac.uk/user-guide/development/)
- [Cirrus Spack guide](https://docs.cirrus.ac.uk/software-tools/spack/)
- [Cray Programming Environment docs](https://cpe.ext.hpe.com/docs/latest)

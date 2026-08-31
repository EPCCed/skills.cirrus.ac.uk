---
name: cirrus-mpi
description: >
  Guide for using MPI on Cirrus HPC system.
  Use this skill whenever a user asks about MPI on Cirrus, running multi-node MPI jobs,
  Cray MPICH, cray-mpich, PrgEnv-gnu, PrgEnv-cray, PrgEnv-intel or PrgEnv-aocc for MPI, the PMI or PMIx process
  manager interface, srun --mpi flags (cray_shasta, pmi2, pmix), installing OpenMPI or
  MPICH with conda or from source, mpi4py on Cirrus, libfabric and Slingshot 11 MPI
  performance, or why mpirun and mpiexec should not be used on Cirrus.
  Also trigger for questions about linking MPI libraries, compiler wrappers with MPI
  (mpicc, mpicxx, mpif90), or any multi-node communication setup on an HPE Cray system.
compatibility: >
  Cirrus. Requires access to an Cirrus login node and the
  Cray MPI environment.
metadata:
  author: cirrus-sc
  version: "1.0"
  source_url: https://docs.cirrus.ac.uk/user-guide/development/#message-passing-interface-mpi
---

# MPI on Cirrus

MPI (Message Passing Interface) enables parallel communication across compute nodes. On
Cirrus, MPI must communicate with the **Slingshot 11 (SS11)**
high-speed interconnect via `libfabric` to achieve optimal latency and bandwidth.

```
MPI application → libfabric → Slingshot 11 NIC → network
```

> **Never use `mpirun` or `mpiexec`.** Always launch MPI applications with `srun` so
> you can supply the correct `--mpi` flag for the PMI type. Using `mpirun`/`mpiexec`
> bypasses Slurm's process management and will not work correctly.

## Critical Rules

- Always start MPI jobs with `srun` on Cirrus.
- Never use `mpirun` or `mpiexec` on Cirrus.
- Always choose the `--mpi` value that matches the MPI implementation in use.
- Load MPI through a Cray `PrgEnv` or a Conda/MPI build that is explicitly configured
  for Slingshot 11.
- Do not mix raw compiler invocations (e.g. gcc, gfortran, ifx, icx) with Cray compiler wrappers in the same build.

---

## MPI Libraries

### Default MPI: Cray MPICH

The recommended MPI is **Cray MPICH**, provided by the `cray-mpich` module. It is loaded
automatically for all users when they login.

### OpenMPI

**Important:** Only OpenMPI 5 is compatible with the Slingshot 11 interconnect. If you are
building OpenMPI, then you must use OpenMPI 5 or newer - OpenMPI 4 will not work on Cirrus.

---

## PMI — Process Manager Interface

MPI processes need a Process Manager Interface (PMI) to coordinate ranks across nodes
through Slurm. The correct `--mpi` flag to `srun` depends on which MPI library is in use.

List available PMI types on the system:

```bash
srun --mpi=list
# cray_shasta
# none
# pmi2
# pmix  (pmix_v4)
```

### Choosing the right `--mpi` flag

| MPI library | `--mpi` flag | Notes |
|-------------|-------------|-------|
| Cray MPICH (default) | `cray_shasta` | Default; usually works without specifying |
| OpenMPI ≥ 5.0 | `pmix` | Required for modern OpenMPI |

```bash
# Cray MPICH (cray_shasta is the default, but explicit is clearer)
srun --mpi=cray_shasta ./mpi_app

# OpenMPI ≥ 5.0
srun --mpi=pmix ./mpi_app
```

> If you do not need to specify a custom PMI and are using Cray MPICH, `srun` without
> `--mpi` will use `cray_shasta` by default.

---

## Installing a Different MPI Version

The system Cray MPICH is recommended for performance. If you need a different version
(e.g. for software that requires OpenMPI), you will need to build your own version.

### Build OpenMPI from source

For maximum control or when Slingshot integration is needed with OpenMPI:

- [OpenMPI v5 build instructions](https://docs.open-mpi.org/en/v5.0.x/installing-open-mpi/quickstart.html#building-from-source)

### mpi4py

For Python MPI, see the [Python guide](https://docs.cirrus.ac.uk/user-guide/python/). You should use
the mpi4py installation available in the `cray-python` module which has been built against Cray MPICH.

---

## Quick Reference

| Goal | Command / setting |
|------|------------------|
| Run an MPI job (Cray MPICH) | `srun --mpi=cray_shasta ./mpi_app` |
| Run an MPI job (OpenMPI ≥5) | `srun --mpi=pmix ./mpi_app` |
| List available PMI types | `srun --mpi=list` |
| Use mpi4py | `module load cray-python` |

---

## Related Resources

- [Cirrus application development environment](https://docs.cirrus.ac.uk/user-guide/development/)
- [Cray Programming Environment docs](https://cpe.ext.hpe.com/docs/latest)

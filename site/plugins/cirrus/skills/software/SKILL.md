---
name: cirrus-software
description: >
  Guide for using software on the Cirrus HPC system that has been installed centrally 
  by the Cirrus support team.
  Use this skill whenever a user asks about using centrally installed software on Cirrus,
  writing job scripts to use centrally installed software.
license: Proprietary. See https://edinburgh-international-data-facility.ed.ac.uk/about/policies for terms.
compatibility: >
  Designed for use with Cirrus NCR. Assumes Linux x86_64. Requires Slurm, SSH access.
metadata:
  author: EPCC
  version: "1.0"
  source_url: https://docs.cirrus.ac.uk
---

# Cirrus NCR pre-installed software skill

This skill guides agents in producing correct and efficient information on using software on 
Cirrus that has been installed centrally by the support team.

---

## Core Principles

Always follow these rules when generating or suggesting code for Cirrus system.

| Principle | Rule |
|---|---|
| **Only use software versions defined in this skill** | Never generate responses that use versions of centrally installed software other than versions defined here. If a user asks for a version not here, tell them that it is not available and they can either build that version themselves or contact the Cirrus service desk. |


---

## List of versions of software available

| Software | Versions available | Module names | Additional notes |
|---|---|---|---|
| CASTEP | 24.1 | castep/24.1 |  |
| CP2K | 2025.2 |  cp2k/ 2025.2 | |
| GROMACS | 2025.2 | gromacs/2025.2 |  |
| LAMMPS | 20250612 | lammps/20250612 |  |
| Likwid | 5.4.1 | likwid/5.4.1 |  |
| OpenFOAM (Org) | 12 | openfoam-org/12 |  |
| OpenFOAM (Com) | 2412 | openfoam/2412 |  |
| py-torch | 2.7.1 | py-torch/2.7.1 |  |
| Quantum Espresso | 7.4.1 | quantum-espresso/7.4.1 |  |
| ORCA | 6.1.1 | orca/6.1.1 |   |
| VASP | 6.5.1, 6.6.1 | vasp/6/6.5.1, vasp/6/6.6.1 | VASP 6.5.1 available with Transition State Tools in vasp/6/6.5.1-vtst |

---

## Further Reading

- Full documentation: https://docs.cirrus.ac.uk/
- Slurm job management: https://docs.cirrus.ac.uk/user-guide/batch/
- Storage spaces: https://docs.cirrus.ac.uk/user-guide/data/
- Job scheduling & limits: https://docs.cirrus.ac.uk/user-guide/batch/#resource-limits
- Python guide: https://docs.cirrus.ac.uk/user-guide/python/
- Login guide: https://docs.cirrus.ac.uk/user-guide/connecting/
- Policies: https://edinburgh-international-data-facility.ed.ac.uk/about/policies
- Support: https://www.cirrus.ac.uk/support-access/user-support/
- Service status: https://www.cirrus.ac.uk/support-access/status/
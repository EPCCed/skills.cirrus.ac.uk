---
name: cirrus-hpc-cai-ode
description: >
  Guidance for writing, reviewing, and running AI-generated code responsibly on
  Cirrus NCR UK national HPC resource. Use this skill whenever generating, adapting, or debugging
  code intended to run on Cirrus, writing Slurm job scripts, managing
  storage, installing software, or ensuring compliance with EIDF policies.
license: Proprietary. See https://edinburgh-international-data-facility.ed.ac.uk/about/policies for terms.
compatibility: >
  Designed for use with Cirrus NCR. Assumes Linux x86_64. Requires Slurm, SSH access.
metadata:
  author: EPCC
  docs: https://docs.cirrus.ac.uk/
  support: support@cirrus.ac.uk
  status: https://www.cirrus.ac.uk/support-access/status/
---

# Cirrus NCR HPC Responsible AI-Generated Code Skill

This skill guides agents in producing correct, safe, and policy-compliant code
for the Cirrus HPC facility.

---

## Core Principles

Always follow these rules when generating or suggesting code for Cirrus system.

| Principle | Rule |
|---|---|
| **Shared resource respect** | Never generate code that runs heavy workloads directly on login nodes |
| **Accurate resource requests** | Always estimate realistic `--time`, `--nodes`, `--tasks-per-node`, `--cpus-per-task` in job scripts |
| **Storage awareness** | Use the correct storage area; never assume data persists after project end |
| **Policy compliance** | All generated code must be consistent with the EIDF Terms and Conditions of Access |
| **Architecture awareness** | Cirrus is an RHEL **x86_64** HPE Cray EX4000 system. More details at: https://docs.cirrus.ac.uk/user-guide/hardware/  |
| **Verify before submit** | Always review AI-generated scripts before `sbatch` — especially resource flags |

---

## Quick Decision Table

| Task | Correct approach |
|---|---|
| Run a short test command | `srun --time=00:05:00 [--nodes=1 --tasks-per-node=1] <cmd>` |
| Run a batch workload | Write a script; submit with `sbatch` |
| Install Python packages | Conda (Miniforge) or `uv`; never `pip install --user` in $HOME |
| Share data with project members | Write to /epccfs/<project ID>/<project ID>/shared |
| Share data with all users | Write to /epccfs/<project ID>/shared |
| Check quota on /epccfs | `df -h <dir>` |

---

## Slurm Job Scripts

### Cirrus

#### Checking for valid budget codes

You can check in SAFE by selecting Login accounts from the menu, select the login account you want to query.

Under Login account details you will see each of the budget codes you have access to listed e.g. e123 resources and then under Resource Pool to the right of this, a note of the remaining budget in coreh.

When logged in to the machine you can also use the command

```
sacctmgr show assoc where user=$LOGNAME format=account,user,maxtresmins
```

This will list all the budget codes that you have access to e.g.

```
   Account       User   MaxTRESMins
---------- ---------- -------------
      e123      userx         cpu=0
 e123-test      userx

```

This shows that userx is a member of budgets e123 and e123-test. However, the cpu=0 indicates that the e123 budget is empty or disabled. This user can submit jobs using the e123-test budget.

#### Serial job

```bash
#!/bin/bash

# Slurm job options (name, compute nodes, job time)
#SBATCH --job-name=Example_Serial_Job
#SBATCH --time=0:20:0                     # Required: maximum time is 2 days in standard partition
#SBATCH --ntasks=1                        # Required: set a single task

#SBATCH --account=[budget code]           # Required: replace "[budget code]" with the correct budget for the job
#SBATCH --partition=standard              # Required: using standard memory nodes, would use "highmem" for high memory nodes
#SBATCH --qos=standard                    # Required: runtime 2 days or less, would use "long" for jobs up to 4 days

# Change to the submission directory
cd $SLURM_SUBMIT_DIR

# Enforce threading to 1 in case underlying libraries are threaded
export OMP_NUM_THREADS=1

# Launch the serial job
srun ./my_serial_executable.x
```

#### Parallel MPI job

```
#!/bin/bash

#SBATCH --job-name=Example_MPI_Job
#SBATCH --time=0:20:0                 # Required: maximum time is 2 days in standard partition
#SBATCH --nodes=2.                    # Required: set the number of nodes you want to use
#SBATCH --tasks-per-node=288          # Required: set the number of cores per node to use per node
#SBATCH --cpus-per-task=1             # Required: set the stride between MPI process placement, usually set to 288 divided by the value of `--tasks-per-node`
#SBATCH --exclusive                   # Required: when using more than 1 node the `--exclusive` option must be specified

#SBATCH --account=[budget code].      # Required: replace "[budget code]" with the correct budget for the job
#SBATCH --partition=standard          # Required: using standard memory nodes, would use "highmem" for high memory nodes
#SBATCH --qos=standard.               # Required: runtime 2 days or less, would use "long" for jobs up to 4 days

export OMP_NUM_THREADS=1              # Required: prevents any threaded system libraries from automatically using threading

# Launch the parallel job
#   srun picks up the process count and distribution from the sbatch options
srun --hint=nomultithread --distribution=block:block ./my_mpi_executable.x
```

**Gotchas:**
- `module load cray-mpich` is not needed in job scripts on Cirrus

---

## Storage Spaces

All storage is **working storage — not backed up**. Data is typically deleted 1 month after project end.

| Path | Purpose | Available on | Retention |
|---|---|---|---|---|
| `/home/<PROJECT>/<PROJECT>/<USER>` | Critical data and source files | Login nodes only | Project end + 1 month |
| `/epccfs/<PROJECT>/<PROJECT>/<USER>`  | Data required for jobs running on the system | Login nodes, Compute nodes | Project end + 1 month |

**Critical reminders:**
- `$HOME` is for scripts and configs — **not large datasets**.
- `$TMPDIR` on compute nodes is a **tmpfs RAM disk** — very fast but limited. Its use reduces available memory on a node.
- Never assume `$TMPDIR` data survives between jobs.
- Backup important results off-system before the project end date.

---

## Login Nodes — What NOT to Do

Login nodes are **shared** and must not be used for compute-intensive or long-running work.

| Allowed on login node | NOT allowed on login node |
|---|---|
| Editing files | Running model training |
| Compiling small programs | Running data preprocessing pipelines |
| Submitting/monitoring jobs | Running benchmarks or tests |
| File transfer and compression | Long `python` / `bash` loops |

> Using `squeue -i` or `watch squeue` excessively disrupts **all users**. Use `squeue --me` once to check, or set a reasonable interval.

---

## Software and Environments

### Modules

```bash
module avail              # List available modules
module load cray-python   # Load Cray Python (pre-installed)
```

---

## Connecting to Cirrus

```bash

```

**Gotchas:**
- Login nodes are assigned randomly; you should not usually request a specific login node.
- Do NOT leave persistent `tmux`/`screen` sessions on login nodes — they violate security policy and may be terminated without warning.

---

## Managing Jobs

```bash
squeue --me                        # View your running/pending jobs
sacct                              # View current and completed jobs
scancel                            # Cancel a job
salloc --ntasks=1 --time=00:30:00  # Reserve a compute core interactively (always set --time)
```

**Gotchas:**
- Always cancel `salloc` allocations with `scancel <JOBID>` when finished.
- Use `--time-min` and `--time` together to enable more flexible backfill scheduling.

---

## Responsible AI-Generated Code Checklist

Before submitting any AI-generated code or job script to Cirrus:

- [ ] **Resource requests are realistic** — `--time`, `--nodes` match your actual workload
- [ ] **No heavy computation on the login node** — all intensive work is inside a job script
- [ ] **Correct storage variable used** — inputs/outputs go to `/epccfs`, not `/home`
- [ ] **Architecture is correct** — code compiles/runs on x86_64
- [ ] **No persistent sessions** — `tmux`/`screen` used only within a job, not left on login nodes
- [ ] **Quota checked** — storage usage is within limits before staging large datasets
- [ ] **Data backed up** — important results are copied off-system; nothing is assumed to persist
- [ ] **Policy compliance** — usage is consistent with the EIDF Terms and Conditions of Access
- [ ] **Job output reviewed** — check `sacct` or output files after a job completes


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
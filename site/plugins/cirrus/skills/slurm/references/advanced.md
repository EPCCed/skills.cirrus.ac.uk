# Slurm: Advanced Topics

## Monitoring: `sacct`

`sacct` shows current and recently completed jobs including exit codes and resource usage.
Unlike `squeue` (active jobs only), `sacct` provides a historical record. It is highly
customisable — see the [Slurm documentation](https://slurm.schedmd.com/sacct.html) for fields.

```bash
sacct                         # recent jobs for your user
sacct -j <JOBID>              # specific job
sacct --format=JobID,JobName,State,ExitCode,Elapsed,MaxRSS
```

Exit code format is `<app_exit>:<signal>`. `0:15` = killed by signal (e.g. time limit hit).
`1:0` = application exited with error code 1.

---

## Debugging a Running Job: `srun --jobid`

Attach an interactive shell to a running job without interfering with it:

```bash
# Find the job ID
squeue --me

# Attach (--overlap allows the new step to share the existing allocation)
srun --nodes=1 --ntasks-per-node=1 --cpus-per-task=1 --jobid=<JOBID> --overlap --pty /bin/bash -l

# Inspect, then exit — the original job continues
exit
```

---

## Multi-node Jobs

Use `--nodes` to request more than one node. `srun` launches processes across all allocated
nodes automatically. This example specifies 288 MPI processes per node - 1152 MPI processes
in total across 4 nodes.

```bash
#!/bin/bash
#SBATCH --job-name=my_job
#SBATCH --nodes=4
#SBATCH --ntasks-per-node=288
#SBATCH --cpus-per-task=1
#SBATCH --exclusive
#SBATCH --time=00:05:00
#SBATCH --partition=standard
#SBATCH --qos=standard
#SBATCH --account=<budgetID>   # Replace "<budgetID>" with your budget code

sun --hint=nomultithread --distribution=block:block my_mpi_app.x
```

Useful environment variables set by Slurm for multi-node jobs:

| Variable | Value |
|----------|-------|
| `$SLURM_JOB_NUM_NODES` | Number of nodes allocated |
| `$SLURM_NODELIST` | Hostnames of all allocated nodes |
| `$SLURM_NTASKS` | Total tasks across all nodes |
| `$SLURM_NODEID` | Index of the node the current process is on (0-based) |

**Important:** `--exclusive` is required when submitting multi-node jobs on Cirrus.

---

## Hybrid MPI/OpenMP Jobs

Combines MPI (between processes) with OpenMP (threads within each process). Key directives:
- `--ntasks-per-node` — MPI ranks per node
- `--cpus-per-task` — CPU cores per rank (for OpenMP threads)
- `OMP_NUM_THREADS=$SLURM_CPUS_PER_TASK` — always set this so threads match the allocation

Product of `--ntasks-per-node` × `--cpus-per-task` = cores used per node.

**Cirrus** — natural mapping: 1 MPI rank per CCX × 8 OpenMP threads (36 MPI processes per node):

```bash
#!/bin/bash
#SBATCH --nodes=2
#SBATCH --ntasks-per-node=36
#SBATCH --cpus-per-task=8
#SBATCH --exclusive
#SBATCH --time=01:00:00
#SBATCH --partition=standard
#SBATCH --qos=standard
#SBATCH --account=<budgetID>   # Replace "<budgetID>" with your budget code

export OMP_NUM_THREADS=$SLURM_CPUS_PER_TASK
srun --hint=nomultithread --distribution=block:block ./my_hybrid_application.x
```

For finer core-binding control, see `--cpu-bind` in the [srun man page](https://slurm.schedmd.com/srun.html).

---

## Interactive Allocations: `salloc`

`salloc` reserves resources as a named allocation and lets you run multiple `srun` commands
against it from the login node — useful for several short interactive commands without
repeatedly waiting in the queue.

```bash
salloc --nodes=1 --ntasks-per-node=1 --cpus-per-task=1 --time=00:10:00 --partition=standard --qos=standard --account=<budgetID>
# Granted job allocation <JOBID>

srun hostname
srun cpuinfo
# ... run more commands ...

scancel <JOBID>    # release when done — always do this
```

> **Always release `salloc` allocations with `scancel` when finished.** An idle allocation
> holds resources other users cannot access and consumes your project's coreh allocation.

---

## Scheduler Flexibility

The scheduler assigns jobs to gaps in the resource schedule. More flexibility = earlier start.

### Flexible time: `--time-min`

Sets the minimum time needed while `--time` sets the maximum. Enables backfill scheduling —
your job can run in a shorter gap than `--time` but no shorter than `--time-min`:

```bash
#SBATCH --time=12:00:00
#SBATCH --time-min=01:00:00
```

The job runs for as long as resources allow, up to `--time`.

### Flexible node count: `--nodes` range

```bash
#SBATCH --nodes=1-4
```

The scheduler starts the job with as many nodes as are available between min and max.
Use `$SLURM_JOB_NUM_NODES` inside the script to handle the variable count.

---

## Resource Isolation with `--exclusive`

`--exclusive` behaves differently at the step vs job level.

### At the step level (`srun` command) — safe and common

Prevents concurrent `srun` steps from over-subscribing the allocation:

```bash
srun --ntasks=1 --gpus=1 --exclusive step_a &
srun --ntasks=1 --gpus=1 --exclusive step_b &
wait
```

Without `--exclusive`, both steps inherit the full job allocation and may conflict.

### At the job level (`#SBATCH` directive) — use with caution

Prevents other jobs from sharing the same physical node. **You are charged for the whole
node regardless of how many cores you actually request.**

On Cirrus: a node has 288 cores. If you request `--nodes=1 --ntasks-per-node=1 --cpus-per-task=1 --exclusive`,
you are charged for all 288 cores even though you are only using 1 core. Only use when your workload is sensitive to co-tenant noise or requires exclusive access to all NUMA domains / memory.

```bash
#SBATCH --exclusive
```

---

## Using the high memory nodes

If you want to use the high memory nodes, you must use `--partition=highmem` and `--qos=highmem`:

```bash
#SBATCH --qos=highmem
#SBATCH --partition=highmem
```

High memory jobs often see longer queue times than standard memory jobs

---

## Large Jobs

For jobs requiring **128 or more nodes** you must use `--qos=largescale`:

```bash
#SBATCH --qos=largescale
#SBATCH --partition=standard
```

---

## Job Dependencies (Advanced)

The basics guide covers `singleton`. Additional types:

| Type | Behaviour |
|------|-----------|
| `afterok:<ID>` | Start only if job `<ID>` completed with exit code 0 |
| `afterany:<ID>` | Start after job `<ID>` finishes, regardless of exit code |
| `afternotok:<ID>` | Start only if job `<ID>` failed |
| `singleton` | Only one job with this name+user runs at a time |

Capture job IDs in scripts with `--parsable`:

```bash
JOBID_1=$(sbatch --parsable job1.sh)
JOBID_2=$(sbatch --parsable --dependency=afterok:${JOBID_1} job2.sh)
JOBID_3=$(sbatch --parsable --dependency=afterok:${JOBID_2} job3.sh)
```

View dependency status:
```bash
squeue --me --Format="JobID,Name,StateCompact:6,TimeUsed,ReasonList,Dependency:32"
```

---

## Job Arrays (Advanced)

Control concurrency and step size:

```bash
#SBATCH --array=1-100%4     # max 4 tasks running simultaneously
#SBATCH --array=0-90:10     # tasks 0, 10, 20, ... 90
```

Large arrays with generous `--time` can exhaust a project's reserved credit even when the
overall allocation is not spent. Prefer concurrent `srun` steps for many short tasks.

---

## QOS and Resource Limits

```bash
sacctmgr show qos standard              # QOS settings for standard partition
sacctmgr show user $(whoami) withassoc   # your accounts, QOS, and limits
```

Slurm reserves credits based on **requested** resources × **requested** walltime when a job
is queued or running. Only actual consumption is charged after completion, but the
reservation holds until the job finishes. This is why large arrays with long `--time` can
exhaust credit before the actual allocation is spent.

To resolve limit errors: wait for running jobs to complete, then resubmit with tighter
`--time` values. Check allocation in the [portal](https://portal.cirrus.ac.uk).

---

## Job Requeues and Restarts

Jobs are automatically requeued after node faults (not after user cancellation or script errors).
On requeue: same job ID, restarts from the beginning (no automatic checkpointing).

```bash
# Check if job has been restarted
echo $SLURM_RESTART_COUNT    # 0 = first run, >0 = restarted
```

If a job exhausts the retry limit it enters `PENDING` with reason `JobHoldMaxRequeue` —
cancel with `scancel` and resubmit.

To opt out of automatic requeuing (recommended if restarting could cause duplicate writes
or repeated API calls):

```bash
#SBATCH --no-requeue
```
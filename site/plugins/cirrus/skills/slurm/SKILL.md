---
name: slurm
description: >
  Guide for submitting, monitoring, and managing HPC jobs using the Slurm workload manager
  on Cirrus.
  Use this skill whenever a user asks about Slurm on Cirrus, writing sbatch scripts,
  requesting CPU resources, srun or salloc, job arrays, job dependencies,
  multi-node jobs, hybrid MPI/OpenMP jobs, QOS limits, scheduler flexibility (--time-min,
  --nodes range), --exclusive, sacct, polling intervals, job accounting, or why a job
  is stuck in PENDING, failing, or hitting resource limits.
  Also trigger for questions about acceptable use of the Slurm queue, what PENDING reasons
  mean, how to chain jobs, or how to debug a running job — even if the user doesn't
  explicitly say "Slurm".
compatibility: >
  Cirrus. Requires access to an Cirrus login node, Slurm
  commands, and the scheduler environment.
metadata:
  author: cirrus-sc
  version: "1.0"
  source_url: https://docs.cirrus.ac.uk/user-guide/batch/
---

# Slurm on Cirrus

Cirrus uses the [Slurm Workload Manager](https://slurm.schedmd.com/)
to schedule jobs on compute nodes. Jobs are submitted to a queue and run when the requested
resources become available.

> **Never poll the queue rapidly.** Running `squeue` or `sinfo` in a tight loop (`watch`
> with a short interval, or `--iterate`) floods the scheduler and slows job scheduling for
> every user. Minimum polling interval from scripts: **60 seconds**. Disruptive polling
> is a breach of the acceptable use policy and may result in account suspension.

## Critical Rules

- Never poll `squeue`, `sinfo`, or any scheduler status command in a tight loop.
- Always submit jobs with `sbatch` or launch commands with `srun`; do not use `mpirun`
  or `mpiexec` on Cirrus.
- Always set explicit `--time`, `--nodes`, and `--ntasks-per-node` when applicable.
- Use `--exclusive` only when the workload truly requires an entire node or when you are submitting multi-node jobs.
- Slurm commands to run or submit jobs **must** specify a partition (`--partition` option) and QoS (`--qos` option).
- Slurm commands to run or submit jobs **must** specify an budget to charge to using the `--account` option. The budget is often the same as your project ID.
- Do not request GPUs on Cirrus.

---

## System Setup

| System | Cores per node | Notes |
|--------|------------------|---------------|
| Cirrus | 144 per socket (2 sockets per node) |  |

Max walltime on all systems: **24 hours**. See the [job scheduling page](https://docs.cirrus.ac.uk/user-guide/batch/) for partition and QOS limits.

---

## Monitoring

```bash
squeue --me                    # your jobs only
sinfo                          # partition and node state (general impression only)
sacct                          # current and recently completed jobs with exit codes
```

Common job states: `R` = running, `PD` = pending, `CG` = completing, `F` = failed, `TO` = timed out.

---

## Submitting Jobs

### Key information

- **Accidental node reservation:** `--exclusive` as an `#SBATCH` directive reserves an entire node regardless of actual usage. You are charged for the whole node. Use only when your workload genuinely requires it.
- **Multi-node jobs require `--exclusive`:** If you are running multi-node jobs, you must use `--exclusive`.
- When launching parallel jobs using `srun` within a job script, the option `--hint=nomultithread` should be added to the `srun` command to ensure that only physical cores are used.
- When launching parallel jobs using `srun` within a job script, the option `--distribution=block:block` will typically be used to ensure physical cores are assigned sequentially to MPI processes - this arrangement is important for good MPI collective performance on Cirrus

### Batch jobs (`sbatch`)

```bash
sbatch my_job.sh
```

**Cirrus — single core job:**
```bash
#!/bin/bash
#SBATCH --job-name=my_job
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=1
#SBATCH --cpus-per-task=1
#SBATCH --time=00:05:00
#SBATCH --partition=standard
#SBATCH --qos=standard
#SBATCH --account=<budgetID>   # Replace "<budgetID>" with your budget code

hostname
cpuinfo
```

Always set `--time` — shorter walltimes usually mean shorter queue waits.

### Interactive jobs (`srun`)

```bash
# Run a single command on a compute node
srun --nodes=1 --ntasks-per-node=1 --cpus-per-task=1 --partition=standard --qos=standard --account=<budgetID> --time=00:02:00 cpuinfo

# Start an interactive shell (job ends when you close the terminal)
srun --nodes=1 --ntasks-per-node=1 --cpus-per-task=1 --time=00:15:00 --partition=standard --qos=standard --account=<budgetID> --pty /bin/bash --login
```

### Running multiple tasks in parallel on a single node

**Cirrus — single full node job:**

```bash
#!/bin/bash
#SBATCH --job-name=my_job
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=288
#SBATCH --cpus-per-task=1
#SBATCH --exclusive
#SBATCH --time=00:05:00
#SBATCH --partition=standard
#SBATCH --qos=standard
#SBATCH --account=<budgetID>   # Replace "<budgetID>" with your budget code

hostname
sun --hint=nomultithread --distribution=block:block my_mpi_app.x
```

---

## Managing Jobs

### Job arrays

```bash
#SBATCH --array=1-4                # tasks 1, 2, 3, 4
#SBATCH --output=my_array_%a.out   # %a = task ID
#SBATCH --array=1-100%4            # limit to 4 concurrent tasks
#SBATCH --array=0-90:10            # step: 0, 10, 20 ... 90

# Inside script, task ID is:
echo $SLURM_ARRAY_TASK_ID
```

Array tasks appear as `JOBID_TASKID` in `squeue`. Cancel one task: `scancel JOBID_TASKID`.
Cancel the whole array: `scancel JOBID`.

> For many short tasks, prefer concurrent `srun` steps in one batch job over a large array
> — reduces scheduler overhead and avoids exhausting credit reservations.

### Job dependencies

```bash
# Run job2 only after job1 succeeds (exit code 0)
JOBID_1=$(sbatch --parsable job1.sh)
JOBID_2=$(sbatch --parsable --dependency=afterok:${JOBID_1} job2.sh)

# Only one job with this name runs at a time
sbatch --dependency=singleton my_job.sh
sbatch --dependency=singleton my_job.sh   # waits for the first to finish
```

View dependencies: `squeue --me --Format="JobID,Name,StateCompact:6,ReasonList,Dependency:32"`

Other types: `afterany` (regardless of exit code), `afternotok` (only if failed). See the [sbatch man page](https://slurm.schedmd.com/sbatch.html).

### Cancelling jobs

```bash
scancel <JOBID>         # cancel a job or array task
scancel <JOBID_TASKID>  # cancel one array task
scancel --me            # cancel all your jobs
```

---

## Advanced Topics

See **`references/advanced.md`** for full detail on:
- `sacct` for job history and exit codes
- Attaching an interactive shell to a running job (`srun --jobid --overlap`)
- Multi-node jobs (`--nodes`, `--ntasks-per-node`)
- Hybrid MPI/OpenMP jobs (`--ntasks-per-node`, `--cpus-per-task`, `OMP_NUM_THREADS`)
- Interactive allocations (`salloc`)
- Scheduler flexibility (`--time-min`, `--nodes` range)
- `--exclusive` at the job level — when it's needed and what it costs
- QOS limits, `sacctmgr`, and allocation limit errors
- Job requeues, restarts, and `SLURM_RESTART_COUNT`
- Large jobs (128+ nodes)

Read this file when helping with any of these topics.

---

## Troubleshooting

See **`references/troubleshooting.md`** for a full symptom → cause → fix reference covering:
- Job submission errors (`QOS policy`, `Invalid GRES`, `node configuration not available`, `invalid account`)
- PENDING reasons (`Priority`, `Dependency`, `PartitionTimeLimit`, `ReqNodeNotAvail`, `AssocGrpGRESMinutesLimit`, `JobHoldMaxRequeue`)
- Job failures (`TIMEOUT`, `OUT_OF_MEMORY`, `NODE_FAIL`, `FAILED` with non-zero exit code)
- Job shows `COMPLETED` but results are missing or wrong

Read this file when a user is asking why their job won't start, is failing, or is behaving unexpectedly.

---

## Quick Reference

| Goal | Command |
|------|---------|
| View your jobs | `squeue --me` |
| Check job history + exit codes | `sacct` |
| Submit batch job | `sbatch my_job.sh` |
| Interactive command | `srun --nodes=1 --ntasks-per-node=1 --cpus-per-task=1 --time=00:05:00 <cmd>` |
| Interactive shell | `srun --nodes=1 -ntasks-per-node=1 --cpus-per-task=1 --time=00:15:00 --pty /bin/bash --login` |
| Reserve allocation | `salloc --nodes=1 -ntasks-per-node=1 --cpus-per-task=1 --time=00:10:00` |
| Cancel job | `scancel <JOBID>` |
| Cancel all my jobs | `scancel --me` |
| Chain jobs | `sbatch --parsable` + `--dependency=afterok:<ID>` |
| Limit array concurrency | `--array=1-100%4` |
| Attach to running job | `srun --jobid=<ID> --overlap --pty /bin/bash -l` |
| Check QOS limits | `sacctmgr show qos` |
| Check my accounts | `sacctmgr show user $(whoami) withassoc` |

---

## Related Resources

- [Cirrus job scheduling page](https://docs.cirrus.ac.uk/user-guide/batch/)
- [Slurm sbatch man page](https://slurm.schedmd.com/sbatch.html)
- [Slurm srun man page](https://slurm.schedmd.com/srun.html)
- [Slurm QOS documentation](https://slurm.schedmd.com/qos.html)
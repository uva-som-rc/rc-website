+++
description = ""
title = "SLURM Job Manager"
draft = false
date = "2019-05-28T17:45:12-05:00"
tags = ["hpc","rivanna","parallel-computing","supercomputer","allocations","queues","storage"]
categories = ["userinfo"]
images = [""]
author = "Staff"  
type = "rivanna"

+++

# Overview

<img style="margin-left:2rem; margin-bottom:2rem; float:right; max-width:30%;" src="/images/rivanna/slurm-workload-manager.png" />

Rivanna is a multi-user, managed environment.  It is divided into frontends, which are directly accessible by users, and compute nodes, which must be accessed through the resource manager.  We use the **Simple Linux Utility for Resource Management (SLURM)**, an open-source tool that performs cluster management and job scheduling for Linux clusters. Jobs are submitted to the resource manager, which queues them until the system is ready to run them. SLURM selects which jobs to run, when to run them, and how to place them on the compute node, according to a predetermined site policy meant to balance competing user needs and to maximize efficient use of cluster resources. SLURM divides a cluster into logical units called partitions (generally known as queues in other systems). Different partitions may contain different nodes, or they may overlap; they may also impose different resource limitations. The UVA HPC environment provides several partitions and there is no default; each job must request a partition. To determine which queues are available, log in to the HPC System and type 
``` 
qlist
```
at a Linux command-line prompt.  For the memory and core limits on each queue, use the command
```
qlimits
```

# Local Queue Configuration

Several queues (partitions) are available for different types of jobs.  One queue is restricted to single-node (serial or threaded) jobs; another for multinode parallel programs, and others are for access to specialty hardware such as large-memory nodes or nodes offering GPUs.  For the current queue configuration and policies on Rivanna please see its homepage. 

# SLURM Architecture

SLURM has a controller process (called a daemon) on a head node and a worker daemon on each of the compute nodes. The controller is responsible for queueing jobs, monitoring the state of each node, and allocating resources. The worker daemon gathers information about its node and returns that information to the controller. When assigned a user job by the controller, the worker daemon initiates and manages the job. SLURM provides the interface between the user and the cluster. SLURM performs three primary tasks:

1. Manage the queue(s) of jobs and settles contentions for resources;
1. Allocate a subset of nodes or cores for a set amount of time to a submitted job;
1. Provide a framework for starting and monitoring jobs on the subset of nodes/cores.

To submit a job to the cluster, you must request the appropriate resources and specify what you want to run with a SLURM Job Command File. In most cases, this batch job file is simply a bash or other shell script containing directives that specify the resource requirements (e.g. the number of cores, the maximum runtime, partition specification, etc.) that your job is requesting along with the set of commands required to execute your workflow on a subset of cluster compute nodes.  Batch job scripts are submitted to the SLURM Controller to be run on the cluster. When the script is submitted to the resource manager, the controller reads the directives, ignoring the rest of the script, and uses them to determine the overall resource request.  It then assigns a priority to the job and places it into the queue.  Once the job is assigned to a worker, the job script is run as an ordinary shell script on the "master" node, in which case the directives are treated as comments.  For this reason it is important to follow the format for directives exactly.

The remainder of this tutorial will focus on the SLURM command line interface. More detailed information about using SLURM can be found in the [official SLURM documentation](https://slurm.schedmd.com/documentation.html).

# Job Scripts

Jobs are submitted through a job script, which is a shell script (usually written in `bash`).  Since it is a shell script it must begin with a "shebang"

```
#!/bin/bash
```

This is followed by a preamble describing the resource requests the job is making.  Each request begins with `#SBATCH` followed by an option.

```
#SBATCH --time=12:00:00
#SBATCH -A myaccount
```

After all resource requests have been established through `#SBATCH`, the script must describe exactly how to run the job.  

```
module purge
module load gcc/7.1.0
./mycode
```

In our installation of SLURM, the default starting directory is the directory from which the batch job was submitted, so it may not be necessary to change directories.

- - - 

# Configurable Options in SLURM

SLURM refers to processes as "tasks."  A task may be envisioned as an independent, running process.  SLURM also refers to cores as "cpus" even though modern cpus contain several to many cores.  If your program uses only one core it is a single, sequential task.  If it can use multiple cores on the same node, it is generally regarded as a single task, with multiple cores assigned to that task.  If it is a distributed code that can run on multiple nodes, each process would be a task.

## Options

Note that most SLURM options have two forms, a short (single-letter) form that is preceded by a single hyphen and followed by a space, and a longer form preceded by a double hyphen and followed by an equals sign.  In a job script these options are preceded by a pseudocomment `#SBATCH`.  They may also be used as command-line options on their own, especially with ijob.

| Option | Usage |
|---|---|
| Number of nodes  | `-N <n>` or `--nodes=<n>` |
| Number of tasks  | `-n <n>` or `--ntasks=<n>` |
| Number of processes (tasks) per node  | `--ntasks-per-node=<n>` |
| Total memory per node in megabytes  | `--mem=<M>` |
| Memory per core in megabytes  | `--mem-per-cpu=<M>` |
| Wallclock time  | `-t d-hh:mm:ss` or `--time=d-hh:mm:ss` |
| Partition (queue) requested  | `-p <part>` or `--partition <part>` |
| Account to be charged  | `-A <account>` or `--account=<allocation>` |

<br>
{{< highlight >}}
   The <b>--mem</b> and <b>--mem-per-cpu</b> options are mutually exclusive.  Job scripts should specify one or the other but not both.</alert>
{{< /highlight >}}

- - -

## Environment variables

These are the most basic; there are many more.  By default SLURM changes to the directory from which the job was submitted, so the `SLURM_SUBMIT_DIR` environment variable is usually not needed.

```
SLURM_JOB_ID
SLURM_SUBMIT_DIR
SLURM_JOB_PARTITION
SLURM_JOB_NODELIST
```

SLURM passes all environment variables from the shell in which the `sbatch` or `salloc` (and `ijob`) command was run. To override this behavior in an `sbatch` script, add

```
#SBATCH --export=NONE
```

To export specific variables use

```
#SBATCH --export=var1,var2,var3
```

Optionally export and set with

```
#SBATCH --export=variable1=value1,variable2=value2
```

## Standard Output and Standard Error

By default, SLURM combines standard output and standard error into a single file, which will be named slurm-\<jobid\>.out.  You may rename standard output with

```
#SBATCH -o <outfile> or --output=<outfile>
```

To separate standard error from standard output, you must rename both.

```
#SBATCH -e <errfile> or --error=<errfile>
```


# Submitting a Job

Job scripts are submitted with the sbatch command, e.g.:

```
% sbatch hello.slurm
```

The job identification number is returned when you submit the job, e.g.:

```
% sbatch hello.slurm
Submitted batch job 18341
```

# Submitting an Interactive Job

If you wish to run a job directly from the shell that exceeds the limits on the frontends, you can run an interactive job to obtain a login shell on a compute node.  You can also use an interactive job if you wish to use a graphical user interface (GUI) such as the Matlab Desktop, RStudio, or the ANSYS Workbench.  The recommended method is to use the locally-written command ijob:

```
ijob <options>
```

ijob is a wrapper around the SLURM commands salloc and srun, set up to start a bash shell on the remote node.  The options are the same as the options to salloc, so most commands that can be used with #SBATCH can be used with ijob.  The request will be placed into the queue specified:

```
% ijob -c 1 -A mygroup -p standard --time=1-00:00:00
salloc: Pending job allocation 25394
salloc: job 25394 queued and waiting for resources
```

There may be some delay for the resource to become available.

```
salloc: job 25394 has been allocated resources
salloc: Granted job allocation 25394
```

The allocated node(s) will remain reserved as long as the terminal session is open, up to the walltime limit, so it is extremely important that users exit their interactive sessions as soon as their work is done so that their nodes are returned to the available pool of processors and the user is not charged for unused time.  Also, unlike a batch job, an interactive job will be terminated if you exit your shell for any reason, including shutting down your local computer from which your login originates.

```
% exit 
salloc: Relinquishing job allocation 25394
```

A full list of options is available from SchedMD's salloc documentation, or you may run

```
ijob --help
```
at the command line.

# Displaying Job Status

The squeue command is used to obtain status information about all jobs submitted to all queues. Without any specified options, the squeue command provides a display which is similar to the following:

```
JOBID     PARTITION     NAME       USER    ST  TIME      NODES    NODELIST(REASON)

----------------------------------------------------------------------------------

12345     parallel      myHello    mst3k   R   5:31:21   4        udc-ba33-4a, udc-ba33-4b, uds-ba35-22d, udc-ba39-16a

12346     standard      bash       mst3k   R   2:44      1        udc-ba30-5
```

The fields of the display are clearly labeled, and most are self-explanatory. The TIME field indicates the elapsed walltime (hrs:min:secs) that the job has been running. Note that JOBID 12346 has the name bash, which indicates it is an interactive job. In that case, the TIME field provides the amount of walltime during which the interactive session has be open (and resources have been allocated). The ST field lists a code which indicates the state of the job. Commonly listed states include:

* PD PENDING: Job is waiting for resources;
* R RUNNING: Job has the allocated resources and is running;
* S SUSPENDED: Job has the allocated resources, but execution has been suspended.

A complete list of job state codes is available here.

To check on your job's status

```
% sstat <jobid>
```

For detailed information

```
% scontrol show job <jobid>
```

To request an estimate of when your pending job will run

```
% squeue --start -j <jobid>
```

# Canceling a Job

SLURM provides the scancel command for deleting jobs from the system using the job identification number:

```
% scancel 18341
```

If you did not note the job identification number (JOBID) when it was submitted, you can use squeue to retrieve it.

```
% squeue -u mst3k

JOBID     PARTITION      NAME         USER     ST    TIME   NODES    NODELIST(REASON)
-------------------------------------------------------------------------------------
18341     standard          myHello      mst3k    R     0:01   1        udc-ba30-5
```

To cancel all of your jobs

```
% scancel -u mst3k
```

To cancel all of your pending jobs

```
scancel -t PENDING -u mst3k
```

To cancel all jobs with a specified name

```
scancel --name myjob
```

To restart (cancel and rerun)

```
scontrol requeue <jobid>
```

For further information about the squeue command, type `man squeue` on the cluster front-end machine or see the SLURM Documentation.

# Job Arrays

A large number of jobs can be submitted through one request if all the files used follow a strict pattern.  For example, if input files are named input_1.dat, ... , input_1000.dat, we could write a job script requesting the appropriate resources for a single one of these jobs with

```
#!/bin/bash
#SBATCH --ntasks=1
#SBATCH --time=1:00:00
#SBATCH --output=result_%a.out
#SBATCH --partition=standard

/myprogram < input_${SLURM_ARRAY_TASK_ID}.dat
```

In the output file name, %a is the placeholder for the array ID.  We submit with

```
% sbatch --array=1-1000 myjob.sh
```

The system automatically submits 1000 jobs, which will all appear under a single job ID with separate array IDs.  The SLURM_ARRAY_TASK_ID environment variable can be used in your command lines to label individal subjobs.

The placeholder %A stands for the overall job ID number in the #SBATCH preamble lines, while %a represents the individual task number.  These variables can be used with the --output option.  In the body of the script you can use the regular environment variable SLURM_TASK_ID if you wish to differentiate different job IDs and SLURM_ARRAY_TASK_ID for the jobs within the array.

To submit a range of task IDs with an interval

```
% sbatch --array=1-1000:2
```

To submit a list of task IDs

```
% sbatch --array=1,3,9,11,22
```

## Using Files with Job Arrays

For more complex commands, you can prepare a file containing the text you wish to use. Your job script can read the file line by line.  In the following example, you must number your subtasks starting from 1 sequentially.  You must prepare the `options_file.txt` in advance and each line must be the options you wish to pass to your program.  
```
#!/bin/bash
#
#SBATCH --ntasks=1
#SBATCH --partition=standard
#SBATCH --time=3:00:00
#SBATCH --array=1-1000

OPTS=$(sed -n "${SLURM_ARRAY_TASK_ID}"p options.txt)

./myprogram $OPTS
```
The double quotes and curly braces are required.

## Canceling Individual Tasks in an Array

One task

```
% scancel <jobid>_<taskid>
```

A range of tasks

```
% scancel <jobid>_[<taskid1>-<taskid2>]
```

A list of tasks

```
% scancel <jobid>_[<taskid1>,<taskid2>,<taskid3>]
```

# Specifying Job Dependencies

With the sbatch command, you can invoke options that prevent a job from starting until a previous job has finished. This constraint is especially useful when a job requires an output file from another job in order to perform its tasks. The --dependency option allows for the specification of additional job attributes. For example, suppose that we have two jobs where job_2 must run after job_1 has completed. Using the corresponding SLURM command files, we can submit the jobs as follows:

```
% sbatch job_1.slurm 
Submitted batch job 18375 
% sbatch --dependency=afterok:18375 job_2.slurm
```

Notice that the --dependency has its own condition, in this case afterok. We want job_2 to start only after the job with id 18375 has completed successfully. The afterok condition specifies that dependency. Other commonly-used conditions include the following:

* after: The dependent job is started after the specified job_id starts running;
* afterany: The dependent job is started after the specified job_id terminates either successfully or with a failure;
* afternotok: The dependent job is started only if the specified job_id terminates with a failure.

More options for arguments of the dependency condition are detailed in the manual pages for sbatch found here or by typing `man sbatch` at the Linux command prompt.

We also are able to see that a job dependency exists when we view the job status listing, although the explicit dependency is not stated, e.g.:

```
% squeue
JOBID    PARTITION     NAME      USER     ST     TIME    NODES   NODELIST(REASON)
---------------------------------------------------------------------------------
18375    standard      job_2.sl  mst3k    PD     0:00    1       (Dependency)
18374    standard      job_1.sl  ms53k    R      0:09    1       udc-ba30-5
```

# Job Accounting Data

When submitting a job to the cluster for the first time, the walltime requirement should be overestimated to ensure that SLURM does not terminate the job prematurely. After the job completes, you can use sacct to get the total time that the job took. Without any specified options, the sacct command provides a display which is similar to the following:

```
JobID    JobName        Partition    Account    AllocCPUS   State       ExitCode
------------------------------------------------------------------------
18347    hello2.sl+     standard     default    1           COMPLETED    0:0
18347    .batch         standard     default    1           COMPLETED    0:0
18348    hello2.sl+     standard     default    1           COMPLETED    0:0
18348    .batch         standard     default    1           COMPLETED    0:0
```

To include the total time, you will need to customize the output by using the format options. For example, the command

```
% sacct --format=jobID --format=jobname --format=Elapsed --format=state
```

yields the following display:

```
JobID         JobName     Elapsed    State
------------------------------------------
18347         hello2.sl+  00:54:59   COMPLETED
18347.batch   batch       00:54:59   COMPLETED
18347.0       orted       00:54:59   COMPLETED
18348         hello2.sl+  00:54:74   COMPLETED
18348.batch   batch       00:54:74   COMPLETED
```

The Elapsed time is given in hours, minutes, and seconds, with the default format of hh:mm:ss. The Elapsed time can be used as an estimate for the amount of time that you request in future runs; however, there can be differences in timing for a job that is run several times. In the above example, the job called python took 21 minutes, 27 seconds to run the first time (JobID 18352.0) and 16 minutes, 8 seconds the last time (JobID 18353.2). Because the same job can take varying amounts of time to run, it would be prudent to increase Elapsed time by 10% to 25% for future walltime requests. Requesting a little extra time will help to ensure that the time does not expire before a job completes

# Sample SLURM Command Scripts
In this section are a selection of sample SLURM command files for different types of jobs.  For more details on running specific software packages, please see the software pages.

## Basic Serial Program

This example is for running your own serial (single-core) program.  It assumes your program is in the same folder from which your job script was submitted.

```
#!/bin/bash
#SBATCH -N 1
#SBATCH -n 1
#SBATCH -t 10:00:00
#SBATCH -p standard
#SBATCH -A mygroup

./mycode.exe
```

### MATLAB

This example is for a serial (one core) Matlab job.

```
#!/bin/bash
#SBATCH -N 1
#SBATCH -n 1
#SBATCH -t 01:00:00
#SBATCH -o output_filename
#SBATCH -p standard
#SBATCH -A mygroup

module load matlab

matlab -nojmv -nodisplay -nosplash -singleCompThread -r "Mymain(myvar1s);exit"
```

### Python

This script runs a Python (version 3) program.

```
#!/bin/bash
#SBATCH -n 1
#SBATCH -t 01:00:00
#SBATCH -o myRprog.out
#SBATCH -p standard
#SBATCH -A mygroup

module load anaconda3

python myscript.py
```

### R

This is a SLURM job command file to run a serial R batch job.

```
#!/bin/bash
#SBATCH -n 1
#SBATCH -t 01:00:00
#SBATCH -o myRprog.out
#SBATCH -p standard
#SBATCH -A mygroup

module load R

Rscript myRprog.R
```


## Job Scripts for Parallel Programs

### Distributed Memory Jobs

If the executable is a parallel program using the Message Passing Interface (MPI), then it will require multiple processors of the cluster to run. This information is specified in the SLURM nodes resource requirement. The script mpiexec is used to invoke the parallel executable. This example is a SLURM job command file to run a parallel (MPI) job using the OpenMPI implementation:

```
#!/bin/bash
#SBATCH --nodes=2 
#SBATCH --ntasks-per-node=16
#SBATCH --time=12:00:00
#SBATCH --output=output_filename
#SBATCH --partition=parallel 
#SBATCH -A mygroup

module load gcc
module load openmpi

srun ./parallel_executable
```

In this example, the SLURM job file is requesting two nodes with sixteen tasks per node (for a total of thirty-two processors).  Both OpenMPI and IntelMPI are able to obtain the number of processes and the host list from SLURM, so these are not specified.  In general, MPI jobs should use all of a node so we'd recommend ntasks-per-node=20 on the parallel partition, but some codes cannot be distributed in that manner so we are showing a more general example here.

SLURM can also place the job freely if the directives specify only the number of tasks. In this case do not specify a node count.  This is not generally recommended, however, as it can have a significant negative impact on performance.

```
#!/bin/bash
#SBATCH --ntasks=8
#SBATCH --time=12:00:00
#SBATCH --output=output_filename
#SBATCH --partition=parallel 
#SBATCH -A mygroup

module load gcc
module load openmpi

srun ./parallel_executable
```

### MPI over an odd number of tasks

```
#!/bin/bash
#SBATCH --ntasks=97
#SBATCH --nodes=5
#SBATCH --ntasks-per-node=20
#SBATCH --time=12:00:00
#SBATCH --output=output_filename
#SBATCH --partition=parallel 
#SBATCH -A mygroup

module load gcc 
module load openmpi 
srun ./parallel_executable
```

### Threaded Jobs (OpenMP or pthreads)

SLURM considers a task to correspond to a process.  Specifying a number of cpus (cores) per node ensures that they are on the same node.  SLURM does not set standard environment variables such as OMP_NUM_THREADS or NTHREADS, so the script must transfer that information explicitly.  This example is for OpenMP:

```
#!/bin/bash
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=20
#SBATCH --time=12:00:00
#SBATCH --output=output_filename
#SBATCH --partition=standard
#SBATCH -A mygroup

module load gcc
export OMP_NUM_THREADS=$SLURM_CPUS_PER_TASK
./threaded_executable
```

### Hybrid

The following example runs a total of 32 MPI processes, 4 on each node, with each task using 5 cores for threading.  The total number of cores utilized is thus 160.

```
#!/bin/bash
#SBATCH --ntasks=32
#SBATCH --ntasks-per-node=4
#SBATCH --cpus-per-task=5
#SBATCH --time=12:00:00
#SBATCH --output=output_filename
#SBATCH --partition=parallel
#SBATCH -A mygroup

module load openmpi/gcc
export OMP_NUM_THREADS=$SLURM_CPUS_PER_TASK
srun ./hybrid_executable
```

## GPU-Intensive Computation

The `gpu` queue provides access to compute nodes equipped with K80, P100, V100, and RTX2080 NVIDIA GPU devices.

{{< highlight >}}
   In order to use GPU devices, the jobs must to be submitted to the <b>gpu</b> partition and must include the <b>--gres=gpu</b> option.</alert>
{{< /highlight >}}

**Example:**
```
#!/bin/bash
#SBATCH --ntasks=1
#SBATCH --time=12:00:00
#SBATCH --output=output_filename
#SBATCH -A mygroup
#SBATCH --partition=gpu
#SBATCH --gres=gpu:p100:2

module load tensorflow3
python myAI.py
```
The second argument to `gres` can be `k80`, `p100`, `v100`, or `rtx2080` for the different GPU architectures.  The third argument to `gres` specifies the number of devices to be requested.  If unspecified, the job will run on the first available GPU node with a single GPU device regardless of architecture.

# CPU and Memory Usage
Sometimes it is important to determine if you used all cores effectively and if enough memory was allocated to the job. There are separate SLURM commands for running jobs and completed jobs.

## Running job

Use `sstat` to get CPU and memory usage for a running job.

```
$ sstat -j <jobid> --format=AveCPU,MaxRSS
    AveCPU     MaxRSS
---------- ----------
76-07:19:+  95534696K
```

The CPU efficiency can be calculated by dividing the total core time in `AveCPU` by the number of requested cores and run time. The above example was obtained for a job that had been running for about 4 days on 20 cores. Therefore, the CPU efficiency is:
```
(76 + 7/24)
----------- = 95%
  4 * 20
```
which indicates good usage of all 20 cores.

The maximum memory used is given by `MaxRSS` (about 91 GB).

For more options, please read the manual `man sstat`.

## Completed job

The `seff` command reports the CPU and memory usage of a completed job. 

Please use this for _completed_ jobs only - efficiency statistics may be misleading for _running_ jobs.

```
$ seff <jobid>
Job ID: <jobid>
Cluster: shen
User/Group: mst3k/users
State: COMPLETED (exit code 0)
Nodes: 1
Cores per node: 10
CPU Utilized: 4-03:27:36
CPU Efficiency: 84.11% of 4-22:15:20 core-walltime
Job Wall-clock time: 11:49:32
Memory Utilized: 3.65 GB
Memory Efficiency: 4.16% of 87.89 GB
```

If you need a more detailed analysis of CPU/memory usage, please contact us for help.

# Usage Report

## PI
The PI of an allocation account can see SU usage of all group members via the following command:

```
list-group-usage -A <your_allocation> [-S YYYY-MM-DD] [-E YYYY-MM-DD]
```

The `-S` and `-E` flags for the start date and end date are optional. The default values are, respectively, the beginning of the current month and now. You will see a report as follows:

```
################################################################################
#
# Includes fund 000 (mst3klab)
# Generated on Mon Aug 31 00:00:00 2020.
# Reporting fund activity from 2020-08-01 to Now.
#
################################################################################
Beginning Balance:           100000.000
Ending Balance:               99000.000
############################### Debit Summary ##################################
User   Jobs   SUs
------ ------ ---------
mst1k  500      500.000
mst2k  200      300.000
mst3k  100      200.000
############################### End of Report ##################################
```

## Non-PI
Regular users can run the previous command, but it will only show your own usage. You may use the `sreport` command for the total _CPU time_ of your group members:

```
$ sreport cluster UserUtilizationByAccount Start=2020-08-01 Accounts=<your_allocation> -t Hours
```

Note that this may be different from the actual allocation usage, since `sreport` is unaware of our SU charge policy, but can serve as an estimate.

[Documentation](https://slurm.schedmd.com/sreport.html)

---
title: "Scheduling jobs"
teaching: 45
exercises: 30
questions:
- "What is a scheduler and why are they used?"
- "How do I launch a program to run on any one node in the cluster?"
- "How do I capture the output of a program that is run on a node in the cluster?"
objectives:
- "Run a simple Hello World style program on the cluster."
- "Submit a simple Hello World style script to the cluster."
- "Use the batch system command line tools to monitor the execution of your job."
- "Inspect the output and error files of your jobs."
keypoints:
- "The scheduler handles how compute resources are shared between users."
- "Everything you do should be run through the scheduler."
- "A job is just a shell script."
- "If in doubt, request more resources than you will need."
---

An HPC system might have thousands of nodes and thousands of users. How do we decide who gets what
and when? How do we ensure that a task is run with the resources it needs? This job is handled by a
special piece of software called the scheduler. On an HPC system, the scheduler manages which jobs
run where and when.

The scheduler used in this lesson is SLURM. Although SLURM is not used everywhere, running jobs is
quite similar regardless of what software is being used. The exact syntax might change, but the
concepts remain the same.

## Running a batch job

The most basic use of the scheduler is to run a command non-interactively. Any command (or series 
of commands) that you want to run on the cluster is called a *job*, and the process of using a
scheduler to run the job is called *batch job submission*.

In this case, the job we want to run is just a shell script. Let's create a demo shell script to 
run as a test.

> ## Creating our test job
> 
> Using your favourite text editor, create the following script and run it. Does it run on the
> cluster or just our login node?
>
>```
>#!/bin/bash
>
> echo 'This script is running on:'
> hostname
> sleep 120
> ```
> {: .bash}
{: .challenge}

There is a distinction between running the job through the scheduler and just "running it"on the login node. To submit this job
to the scheduler, we use the `sbatch` command.

```
[yourUsername@turing1 ~]$ sbatch example-job.sh
```
{: .bash}
```
Submitted batch job 8878780
```
{: .output}

And that's all we need to do to submit a job. Our work is done -- now the scheduler takes over and
tries to run the job for us. While the job is waiting to run, it goes into a list of jobs called 
the *queue*. To check on our job's status, we check the queue using the command `squeue`.

```
[yourUsername@turing1 ~]$ squeue -u yourUsername
```
{: .bash}
```
JOBID     PARTITION NAME     USER      ST      TIME      NODES NODELIST(REASON)
8878781   main      example- hpc-0061  R       0:05      1     coreV2-25-057
```
{: .output}

We can see all the details of our job, most importantly that it is in the "R" or "RUNNING" state.
Sometimes our jobs might need to wait in a queue ("PENDING") or have an error. The best way to check
our job's status is with `squeue`. Of course, running `squeue` repeatedly to check on things can be
a little tiresome. To see a real-time view of our jobs, we can use the `watch` command. `watch`
reruns a given command at 2-second intervals. This is too frequent, and will likely upset your system
administrator. You can change the interval to a more resonable value, for example 60 seconds, with the
`-n 60` parameter. Let's try using it to monitor another job.

```
[yourUsername@turing1 ~]$ sbatch example-job.sh
[yourUsername@turing1 ~]$ watch -n 60 squeue -u yourUsername
```
{: .bash}

You should see an auto-updating display of your job's status. When it finishes, it will disappear
from the queue. Press `Ctrl-C` when you want to stop the `watch` command.

## Customizing a job

The job we just ran used all of the scheduler's default options. In a real-world scenario, that's
probably not what we want. The default options represent a reasonable minimum. Chances are, we will
need more cores, more memory, more time, among other special considerations. To get access to these
resources we must customise our job script.

Comments in UNIX (denoted by `#`) are typically ignored. But there are exceptions. For instance the
special `#!` comment at the beginning of scripts specifies what program should be used to run it
(typically `/bin/bash`). Schedulers like SLURM also have a special comment used to denote special
scheduler-specific options. Though these comments differ from scheduler to scheduler, SLURM's
special comment is `#SBATCH`. Anything following the `#SBATCH` comment is interpreted as an
instruction to the scheduler.

Let's illustrate this by example. By default, a job's name is the name of the script, but the `-J`
option can be used to change the name of a job.

Submit the following job (`sbatch example-job.sh`):

```
#!/bin/bash
#SBATCH -J new_name

echo 'This script is running on:'
hostname
sleep 120
```

```
[yourUsername@turing1 ~]$ squeue -u yourUsername
```
{: .bash}
```
JOBID     PARTITION NAME     USER      ST      TIME      NODES NODELIST(REASON)
8878784   main      new_name hpc-0061  R       0:03      1     coreV2-25-057
```
{: .output}

Fantastic, we've successfully changed the name of our job!

> ## Setting up email notifications
> 
> Jobs on an HPC system might run for days or even weeks. We probably have better things to do than
> constantly check on the status of our job with `squeue`. Looking at the
> [online documentation for `sbatch`](https://slurm.schedmd.com/sbatch.html) (you can also google
> "sbatch slurm"), can you set up our test job to send you an email when it finishes?
> 
> Hint: you will need to use the `--mail-user` and `--mail-type` options.
{: .challenge}

### Resource requests

But what about more important changes, such as the number of CPUs and memory for our jobs? One 
thing that is absolutely critical when working on an HPC system is specifying the resources 
required to run a job. This allows the scheduler to find the right time and place to schedule our 
job. If you do not specify requirements (such as the amount of time you need), you will likely be
stuck with your site's default resources, which is probably not what we want.

The following are several key resource requests:

* `-n <ncores>` - How many cores/tasks does your job need? 

* `-c <cores per task>` - How many cores per task does your job need?

* `-p <partition>` - Which partition/queue does your job need?

* `--mem=<megabytes>` - How much memory on a node does your job need in megabytes? You can also
  specify gigabytes using by adding a little "g" afterwards (example: `--mem=5g`)

* `--time <days-hours:minutes:seconds>` - How much real-world time (walltime) will your job take to
  run? The `<days>` part can be omitted.

Note that just *requesting* these resources does not make your job run faster! We'll talk more 
about how to make sure that you're using resources effectively in a later episode of this lesson.

> ## Submitting resource requests
>
> Submit a job that will use 4 cores or tasks, 4 gigabytes of memory, and 5 minutes of walltime.
{: .challenge}

> ## Job environment variables
>
> When SLURM runs a job, it sets a number of environment variables for the job. One of these will
> let us check our work from the last problem. The `SLURM_NTASKS` variable is set to the
> number of cores we requested with `-c`. Using the `SLURM_NTASKS` variable, modify your job
> so that it prints how many cores have been allocated.
{: .challenge}

Resource requests are typically binding. If you exceed them, your job will be killed. Let's use
walltime as an example. We will request 30 seconds of walltime, and attempt to run a job for two
minutes.

```
#!/bin/bash
#SBATCH -t 0:0:30

echo 'This script is running on:'
hostname
sleep 120
```

Submit the job and wait for it to finish. Once it is has finished, check the log file.

```
[yourUsername@turing1 ~]$ sbatch example-job.sh
[yourUsername@turing1 ~]$ watch -n 60 squeue -u yourUsername
[yourUsername@turing1 ~]$ cat slurm-38193.out
```
{: .bash}
```
This job is running on:
gra533
slurmstepd: error: *** JOB 38193 ON gra533 CANCELLED AT 2017-07-02T16:35:48 DUE TO TIME LIMIT ***
```
{: .output}

Our job was killed for exceeding the amount of resources it requested. Although this appears harsh,
this is actually a feature. Strict adherence to resource requests allows the scheduler to find the
best possible place for your jobs. Even more importantly, it ensures that another user cannot use
more resources than they've been given. If another user messes up and accidentally attempts to use
all of the CPUs or memory on a node, SLURM will either restrain their job to the requested resources
or kill the job outright. Other jobs on the node will be unaffected. This means that one user cannot
mess up the experience of others, the only jobs affected by a mistake in scheduling will be their
own.

## Cancelling a job

Sometimes we'll make a mistake and need to cancel a job. This can be done with the `scancel`
command. Let's submit a job and then cancel it using its job number.

```
[yourUsername@turing1 ~]$ sbatch example-job.sh
[yourUsername@turing1 ~]$ squeue -u yourUsername
```
{: .bash}
```
Submitted batch job 8878787

JOBID     PARTITION NAME     USER      ST      TIME      NODES NODELIST(REASON)
8878787   main      new_name hpc-0061  R       0:03      1     coreV2-25-057
```
{: .output}

Now cancel the job with it's job number. Absence of any job info indicates that the job has been
successfully cancelled.

```
[yourUsername@turing1 ~]$ scancel 8878787
[yourUsername@turing1 ~]$ squeue -u yourUsername
```
{: .bash}
```
JOBID     PARTITION NAME     USER      ST      TIME      NODES NODELIST(REASON)
```
{: .output}

> ## Cancelling multiple jobs
>
> We can also all of our jobs at once using the `-u` option. This will delete all jobs for a
> specific user (in this case you). Note that you can only delete your own jobs.
>
> Try submitting multiple jobs and then cancelling them all with `scancel -u yourUsername`.
{: .challenge}

### Interactive jobs

Sometimes, you will need a lot of resource for interactive use. Perhaps it's our first time running
an analysis or we are attempting to debug something that went wrong with a previous job.
Fortunately, SLURM makes it easy to start an interactive job with `salloc`:

```
[yourUsername@turing1 ~]$ salloc
```
{: .bash}
```
salloc: Pending job allocation 8878796
salloc: job 8878796 queued and waiting for resources
salloc: job 8878796 has been allocated resources
salloc: Granted job allocation 8878796
This session will be terminated in 7 days. If your application requires
a longer excution time, please use command "salloc -t N-0" where N is the
number of days that you need.
```
{: .output}

```
[yourUsername@coreV2-25-057 ~]$ 
```
{: .bash}

You should be presented with a shell prompt on a compute node. Note that the prompt will likely change to reflect your
new location, in this case the worker node we are logged on. You can also verify this with
`hostname`.

When you are done with the interactive job, type `exit` to quit your session.

### SLURM Partitions
Your application may require a GPU or a node with higher memory than our standard compute node.  If this is the case, then you can request a specific partition or queue.  These have been segmented from our standard compute resources and grouped together by hardware type.

By default, you will be allocated a node in the main partition. To request another parition, use the `--partition` option.

> ## Requesting a high memory node
>
> Submit an interactive job requesting a high memory node.  
>
> Hint: You will need to use the `-p himem` option.
{: .challenge}

If you need a GPU, then you will need to do two things.  First, request the gpu partition.  Then, request the number of GPUs(N) using `--gres=gpu:N`. Let's run through an example using `salloc` to request a single GPU.

```
[yourUsername@turing1 ~]$ salloc -p gpu --gres=gpu:1
```
{: .bash}
```
salloc: Pending job allocation 8878800
salloc: job 8878800 queued and waiting for resources
salloc: job 8878800 has been allocated resources
salloc: Granted job allocation 8878800
This session will be terminated in 7 days. If your application requires
a longer excution time, please use command "salloc -t N-0" where N is the
number of days that you need.
```
{: .output}

```
[yourUsername@coreV5-21-v100-001 ~]$ 
```
{: .bash}

If you would like to review more in-depth examples, please visit the Turing Case Studies section on our documentation site <a href="https://docs.hpc.odu.edu/#turing-case-studies" title="Turing Case Studies" here</a>

{% include links.md %}

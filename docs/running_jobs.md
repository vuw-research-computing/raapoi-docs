
## Batch jobs

To run a batch job (aka a job that runs unattended) you use the _sbatch_ command.  A simple example would look something like this:

  `sbatch myjob.sh`

In this example the sbatch command runs the file myjob.sh, the contents of this file, also known as a "batch submit script" could look something like this:

```
 #!/bin/bash
 #SBATCH --cpus-per-task=2
 #SBATCH --mem-per-cpu=2G
 #SBATCH --partition=parallel
 #SBATCH --time=3-12:00
 #SBATCH -o /nfs/home/username/project1.out
 #SBATCH -e /nfs/home/username/project1.err
 #SBATCH --mail-type=BEGIN,END,FAIL
 #SBATCH --mail-user=me@email.com

 module load python/3.6.3
 python3 project1.py

```

This will request 2 CPUs and 4GB of memory (2GB per CPU) and a runtime of 3 days
12 hours.  We are requesting that this job be run on the parallel  partition, it
will then load the environment module for python version 3.6.3 and run a python
script called project1.py.  Any output from the script will be placed in your
home directory in a file named project1.out and any error information in a file called project1.err.  If you do not specify an output or error file, the default files will have the form of Slurm-jobID.o and Slurm-jobID.e and will be located in the directory from which you ran _sbatch_.

NOTE:  We have this example script available to copy on the cluster, you can type the following to copy it to your home directory:

  `cp /home/software/tools/examples/batch/myjob.sh ~/myjob.sh`

The ~/ in front of the file is a short-cut to your home directory path.  You will want to edit this file accordingly.

## Interactive jobs

One of the basic job submittal tools is the command srun

For example, say I want to start a job to run an interactive R session. Once logged into the cluster I can:

```
  module load R/3.5.1
  srun --pty --cpus-per-task=2 --mem=2G  --time=08:00:00 --partition=bigmem R
```

So what does this all mean?

The _module load_ command will introduce the environment necessary to run a particular program, in this case R version 3.5.1
The _srun_ command will submit the job to the cluster.  The _srun_ command has many parameter available, some of the most common are in this example and explained below

* --pty - Required to run interactively
* --cpus-per-task=2 - requests 2 CPUs, can also use the -c flag, eg. -c 2
* --mem=2G - requests 2 GigaBytes (GB) of RAM.
* --time=08:00:00 - requests a runtime of up to 8 hours (format is DAYS-HOURS:MINUTES:SECONDS), this is important in case the cluster or partition has a limited run-time, for example if an outage window is approaching.  Keep in mind time is a resource along with CPU and Memory.  
* --partition=bigmem - requests a certain partition, in this case it requests the bigmem partition, see the section on using cluster partitions for more information.
* R - the command you wish to run, this could also be matlab, python, etc. (just remember to load the module first)

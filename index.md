
The Rāpoi HPC Cluster (aka raapoi) is a Uni-wide computing
resource that uses the Slurm resource manager to schedule jobs and reserve
resources.  Similar to most modern compute clusters, raapoi requires you to
request CPU, Memory and Time for your job.  If you do not request these
resources, you will be given the minimal defaults (1 CPU, 1 GB Memory, 1 hour
runtime), which may not be enough to
run your job.  The good news about resource reservations is that the resources
you request are guaranteed to be yours, the bad news is if you request too
little memory or time, your job may terminate prematurely and if you request too
few CPUs then your job may run slowly.

Raapoi is made up of partitions.  A partition is a set of compute nodes
(servers) and each partition has its own configuration and hardware profile.
The partition on which you run your jobs will depend on the type of workflow or
job you intend to submit.

The cluster employs the module environment to allow researchers to customize
their environment with their required applications and languages.  

The documentation contained in this wiki cover most of what you will need to
know to start running jobs, but if you need more help, we will be creating some
tutorials with step-by-step instructions for our most popular apps and
languages.

# Accessing the cluster

_Access is via SSH_

*  Hostname: raapoi.vuw.ac.nz
*  IP Address: 130.195.19.14
*  Port: 22
*  Username: Your VUW username
*  Password: Your VUW password

*NOTE:* On-campus wired network connection or VPN.  VPN is required if
connecting from campus wifi or from off-campus. Some users have had issues with
using the hostname and instead need to use the IP address, eg
`harrelwe@130.195.19.14`


#### SSH Clients
_Mac OSX SSH Clients_
You can use the built-in Terminal.app or you can download iTerm2 or XQuartz. XQuartz is required to be installed if you wish to forward GUI applications (matlab, rstudio, xstata, sas, etc), aka X forwarding.

* Terminal.app is the default application for command-line interface
  * To login using the built-in Terminal.app on Mac, go to
    * Applications --> Utilities --> Terminal.app
    * Or use Spotlight search (aka Command-Space)
* [iTerm2](https://www.iterm2.com/) is a good replacement for the default Terminal app
* [XQuartz](https://www.xquartz.org/) is a Xforwarding application with its own terminal.  XQuartz can be used in conjuction with the Terminal.app for GUI apps.  NOTE: Mac users should run the following command: `sudo defaults write org.macosforge.xquartz.X11 enable_iglx -bool true`   We have found that this allows some older GUI applications to run with fewer errors.
 

NOTE:  Once at the command prompt you can type the following to login (replace "username" with your VUW user):

`ssh username@raapoi.vuw.ac.nz`

_Windows SSH Clients_

* Recommended Clients:
  * [Git Bash](https://gitforwindows.org/) is a great option and is part of the Git for Windows project
  * [MobaXterm](https://mobaxterm.mobatek.net/) is a good option, especially if you require access to GUI applications such as MATLAB or xStata.  This also has a built-in SFTP transfer window.


# Basic Commands
#### The _vuw_ Commands

In an effort to make using raapoi just a bit easier, CAD staff have created commands to help you view useful information.  We call these the _vuw_ commands.  This is because all the commands begin with the string _vuw_.  This makes it easier to see the commands available to you.  If, at a command prompt you type _vuw_ followed immediately by two _TAB_ keys you will see a list of available commands beginning with _vuw_.  Go ahead and type vuw-TAB-TAB to see for yourself.

The commands available as of this update are:

* _vuw-help_:            Prints this help information
* _vuw-job-report_:      Provides some summary information about a job
* _vuw-quota_:           Prints current storage quota and usage
* _vuw-partitions_:      Prints a list of available partitions and the availability of compute nodes
* _vuw-alljobs_:         Prints a list of all user jobs
* _vuw-myjobs_:          Prints a list of your running or pending jobs
* _vuw-job-history_:     Show jobs finished in last 5 days


#### Linux Commands

Raapoi is built using the Linux operating system. Access is primarily via command line interface (CLI) as opposed to the graphical user interfaces (GUI) that you are more familiar with (such as those on Windows or Mac) Below are a list of common commands for viewing and managing files and directories (replace the file and directory names with ones you own):

**ls** - This command lists the contents of the current directory
* _ls -l_ This is the same command with a flag (-l) which lists the contents with more information, including access permissions
* _ls -a_ Same ls command but this time the -a flag which will also list hidden files. Hidden files start with a . (period)
* _ls -la_ Stringing flags together

**cd** - This will change your location to a different directory (folder)
* _cd projects/calctest_proj_
* Typing _cd_ with no arguments will take you back to your home directory

**mv** - This will move or rename a file

* _mv project1.txt project2.txt_
* _mv project2.txt projects/calctest_proj/_

**rm** - This will delete a file

* _rm projects/calctest_proj/projects2.txt_
* _rm -r projects/calctest_proj/code_
The _-r_ flag recursively removes files and directories

**mkdir** - This will create a new directory
* _mkdir /nfs/home/myusername/financial_

Other Commands you may use: _alias, awk, cat, export, for, grep, gzip, if, less, sed, tar, while_

#### Learning the Linux Shell

A good tutorial for using linux can be found here:
[Learning the linux shell](http://linuxcommand.org/lc3_learning_the_shell.php)

# Storage and quotas

Currently users have 2 main storage areas:

* __/nfs/home/USERNAME__ - This is your Home Directory, each user has a *50 GB* quota limit.

* __/nfs/scratch/USERNAME__ - This is you scratch space, each user has a *5 TB* quota limit 

To view your current quota and usage use the _vuw-quota_ command, for example:

```
harrelwe@raapoi-master:~$ vuw-quota 

Quotas for home directory

                       Storage  Usage (GB)  Quota (GB)     % Used 
            /nfs/home/harrelwe        0.10       50.00      0.20%
```


# Using Partitions

A partition is a collection of compute nodes, think of it as a sub-cluster or
slice of the larger cluster.  Each partition has its own rules and
configurations.  

For example, the quicktest partition has a maximum job run-time of 1 hour, whereas the partition
bigmem has a maximum runtime of 10 days.  Partitions can also
limit who can run a job.  Currently any user can use any partition but there
may come a time when certain research groups purchase their own nodes and they are
given exclusive access.

To view the partitions available to use you can type the vuw-partitions
command, eg

```
harrelwe@raapoi-master:~$ vuw-partitions 

VUW CLUSTER PARTITIONS
PARTITION  AVAIL  TIMELIMIT  NODES  STATE NODELIST
quicktest*    up    1:00:00      1   idle c03n01

PARTITION AVAIL  TIMELIMIT  NODES  STATE NODELIST
bigmem       up 10-00:00:0      2   idle c10n01,c11n01

PARTITION AVAIL  TIMELIMIT  NODES  STATE NODELIST
parallel     up 10-00:00:00      6  down* c04n01,c05n04,c06n[01-04]
parallel     up 10-00:00:00     27   idle
c03n[02-04],c04n[02-04],c05n[01-02],c07n[01,03-04],c08n[01-04],c09n[01-04],c12n[01-04],c13n[01-04]

NOTE: This utility is a wrapper for the Slurm command:
      sinfo -p PARTITION
```      

Notice the STATE field, this describes the current condition of nodes within the
partition, the most common states are defined as:

* __idle__ - nodes in an idle state have no jobs running, all resources are available
for work
* __mix__ - nodes in a mixed state have some jobs running, but still have some
resources available for work
* __alloc__ - nodes in an alloc state are completely full, all resources are in use.
* __drain__ - nodes in a drain state have some running jobs, but no new jobs can be
run.  This is typically done before the node goes into maintenance
* __maint__ - node is in maintenance mode, no jobs can be submitted
* __resv__ - node is in a reservation.  A reservation is setup for future maintenance
or for special purposes such as temporary dedicated access
* __down__ - node is down, either for maitnenance or due to failure

Also notice the _TIMELIMIT_ field, this describes the maximum runtime of a
partition.  For example, the quicktest partition has a maximum runtime of 1 
hour and the parallel partition has a max runtime of 10 days.

#### Partition Descriptions

_Partition: quicktest_

This parttion is for quick tests of code, environment, software builds or
similar short-run jobs.  Since the max time limit is 1 hour it should not take
long for your job to run.  This can also be used for near-on-demand interactive
jobs.

* Maximum CPU available per task: 24
* Maximum memory available per task: 62G
* Maximum Runtime: 1 hour

_Partition: bigmem_

This partition is primarily useful for jobs that require very large shared
memory (generally greater than 125 GB).  These are known as memory-bound jobs.

* Maximum CPU available per task: 48
* Maximum memory available per task: 1 TB (Note: maximum CPU for 1 TB is 40)
* Maximum Runtime: 10 days

_Partition: parallel_

This partition is useful for parallel workflows, either loosely coupled or jobs
requiring MPI or other message passing protocols for tightly bound jobs.

* Maximum CPU available per task: 64
* Maximum memory available per task: 125G
* Maximum Runtime: 10 days

_Cluster Defaults_

Please note that if you do not specify CPU, Memory or Time in your job request
you will be given the cluster defaults which are:

* Default CPU: 2
* Default Memory: 2 GB
* Default Time: 1 hour

You can change these with the -c, --mem and --time parameters to the srun and
sbatch commands.  Please see this documentation for more information about srun
and sbatch.

# Preparing your environment

Raapoi has an extensive library of applications and software available. There are numerous programming languages and libraries (R, Julia, Python, lua, OpenMPI, blas, etc) as well as dozens of applications (Matlab, Stata, etc).  We also keep older versions of software to ensure compatibility.

Because of this, raapoi developers use a tool called module to allow a user to load a specific version of an application, language or library and start using it for their work. The _module_ command will show you what software is available to load, and will add the software to your environment for immediate use. To show all software available to load type the following:

  `module avail`

You will see a long list of available modules to load, including a path, eg _lua/5.3.5_
However, instead of searching through a long list, if you know you want to use lua, you can find the path with the keyword subcommand:

  `module keyword lua`

If you want to know more about a particular module you can use the _whatis_ or
_show_ subcommand.  Some modules have this available, for instance:

```
harrelwe@raapoi-master:~$ module show trimmomatic/20190304 
-------------------------------------------------------------------------------------------------------------
   /home/software/vuwrc/modulefiles/trimmomatic/20190304:
-------------------------------------------------------------------------------------------------------------
load("java/jdk/1.8.0_121")
setenv("TM_HOME","/home/software/apps/trimmomatic/20190304/bin")
```

```
  module whatis R/CRAN/3.5

  R/CRAN/3.5          : Adds the R library path to the pre-built CRAN modules
```

#### Adding or loading software

Once you have found the module path you can load the software:

  `module load lua/5.3.5`

After the module loads you can type srun --pty lua at a prompt, or add it to the path of your lua script (the RC team recommends using /usr/bin/env instead of an absolute path).

Showing/listing the module environment modifications
You can discover what the module will load into your environment you can run
module show, for example here is what R adds:

```
module show R/3.5.1
--------------------------------------------------
   /home/software/vuwrc/modulefiles/R/3.5.1:
--------------------------------------------------

whatis("Adds the R language path to your environment ")
prepend_path("PATH","/home/software/apps/R/3.5.1/bin")
```

#### Listing loaded modules

To see what modules you have loaded into your environment you can run the command:

`module list`  

By default you will have the config module loaded (please do not unload that module).  For example, here are the modules I have loaded in my environment when I wrote this section:

```
module list

Currently Loaded Modules:
  1) config   2) tassel/3   3) python/3.7.0   4) python/modules/3.7

```

# Running jobs

#### Batch jobs

To run a batch job (aka a job that runs unattended) you use the _sbatch_ command.  A simple example would look something like this:

  `sbatch myjob.sh`

In this example the sbatch command runs the file myjob.sh, the contents of this file, also known as a "batch submit script" could look something like this:

```
 #!/bin/bash
 #SBATCH --cpus-per-task=2
 #SBATCH --mem-per-cpu=2G
 #SBATCH --partition=parallel
 #SBATCH --time=3-12:00
 #SBATCH -o /home/username/project1.out
 #SBATCH -e /home/username/project1.err
 #SBATCH --mail-type=BEGIN,END,FAIL
 #SBATCH --mail-user=me@email.com

 module load python/3.6.3
 python3 project1.py

```

This will request 2 CPUs and 4GB of memory (2GB per CPU) and a runtime of 3 days 12 hours.  We are requesting that this job be run on the parallel  partition, it will then load the environment module for python version 3.6.3 and run a python script called project1.py.  Any output from the script will be placed in your home directory in a file named project1.out and any error information in a file called project1.err.  If you do not specify an output or error file, the default files will have the form of Slurm-jobID.o and Slurm-jobID.e and will be located in the directory from which you ran _sbatch_.

NOTE:  We have this example script available to copy on the cluster, you can type the following to copy it to your home directory:

  `cp /home/software/vuwrc/examples/batch/myjob.sh ~/myjob.sh`

The ~/ in front of the file is a short-cut to your home directory path.  You will want to edit this file accordingly.

#### Interactive jobs

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

# Parallel processing

Running a job in parallel is a great way to utilize the power of the cluster.  So what is a parallel job/workflow?

* Loosely-coupled jobs (sometimes referred to as embarrassingly or naively parallel jobs) are processes in which multiple instances of the same program execute on multiple data files simultaneously, with each instance running independently from others on its own allocated resources (i.e. CPUs and memory). Slurm job arrays offer a simple mechanism for achieving this.
* Multi-threaded programs that include explicit support for shared memory processing via multiple threads of execution (e.g. Posix Threads or OpenMP) running across multiple CPU cores.
* Distributed memory programs that include explicit support for message passing between processes (e.g. MPI). These processes execute across multiple CPU cores and/or nodes, these are often referred to as tightly-coupled jobs.
* GPU (graphics processing unit) programs including explicit support for offloading to the device via languages like CUDA or OpenCL.

It is important to understand the capabilities and limitations of an application in order to fully leverage the parallel processing options available on the cluster. For instance, many popular scientific computing languages like Python, R, and Matlab now offer packages that allow for GPU, multi-core or multithreaded processing, especially for matrix and vector operations

#### Job Array Example
Here is an example of running a job array to run 50 simultaneous processes:

  `sbatch array.sh`

The contents of the array.sh batch script looks like this:

```
  #!/bin/bash
  #SBATCH -a 1-50
  #SBATCH --cpus-per-task=1
  #SBATCH --mem-per-cpu=2G
  #SBATCH --time=00:10:00
  #SBATCH --partition=parallel
  #SBATCH --mail-type=BEGIN,END,FAIL
  #SBATCH --mail-user=me@email.com

  module load fastqc/0.11.7

  fastqc --nano -o $TMPDIR/output_dir seqfile_${SLURM_ARRAY_TASK_ID}

```

So what do these parameter mean?:

* _-a_ sets this up as a parallel array job (this sets up the "loop" that will be run
* _--cpus-per-task_ requests the number of CPUs per array task, in this case I just want one CPU per task, we will use 50 in total
* _--mem-per-cpu_ request 2GB of RAM per CPU, for this parallel job I will request a total of 100GB RAM (50 CPUs * 2GB RAM)
* _--time_ is the max run time for this job, 10 minutes in this case
* _--partition_ assigns this job to a partition
* _module load fastqc/0.11.7_: Load software into my environment, in this case fastqc
* _fastqc --nano -o $TMPDIR/output_dir seqfile_${SLURM_ARRAY_TASK_ID}_ Run fastqc on each input data file with the filenames seqfile_1, seqfile_2...seqfile_50

Running the array.sh script will cause the SLURM manager to spawn 50 parallel jobs.



#### Multi-threaded or Multi-processing Job Example

Multi-threaded or multi-processing programs are applications that are able to
execute in parallel across multiple CPU cores within a single node using a
shared memory execution model. In general, a multi-threaded application uses a
single process (aka “task” in Slurm) which then spawns multiple threads of
execution. By default, Slurm allocates 1 CPU core per task. In order to make use
of multiple CPU cores in a multi-threaded program, one must include the
--cpus-per-task option.  Below is an example of a multi-threaded program
requesting 12 CPU cores per task and a total of 256GB of memory. The program itself is responsible for spawning the appropriate number of threads.

```
  #!/bin/bash
  #SBATCH --nodes=1
  #SBATCH --ntasks=1
  #SBATCH --cpus-per-task=12 # 12 threads per task
  #SBATCH --time=02:00:00 # two hours
  #SBATCH --mem=256G
  #SBATCH -p bigmem
  #SBATCH --output=threaded.out
  #SBATCH --job-name=threaded
  #SBATCH --mail-type=BEGIN,END,FAIL
  #SBATCH --mail-user=me@email.com
  # Run multi-threaded application
  module load java/1.8.0-91
  java -jar threaded-app.jar
```

#### MPI Jobs

Most users do not require MPI to run their jobs but many do.  Please read on if you want to learn more about using MPI for tightly-coupled jobs.

MPI (Message Passing Interface) code require special attention within Slurm. Slurm allocates and launches MPI jobs differently depending on the version of MPI used (e.g. OpenMPI, MPICH2). We recommend using OpenMPI version 2.1.1 or later to compile your C code (using mpicc) and then using the mpirun command in a batch submit script to launch parallel MPI jobs. The example below runs MPI code compiled by OpenMPI 2.1.1:

```
  #!/bin/bash
  #SBATCH --nodes=3
  #SBATCH --tasks-per-node=8 # 8 MPI processes per node
  #SBATCH --time=3-00:00:00
  #SBATCH --mem=4G # 4 GB RAM per node
  #SBATCH --output=mpi_job.log
  #SBATCH -p parallel
  #SBATCH --mail-type=BEGIN,END,FAIL
  #SBATCH --mail-user=me@email.com

  module load openmpi
  echo $SLURM_JOB_NODELIST
  mpirun -np 24 mpiscript.o

```

This example requests 3 nodes and 8 tasks (i.e. processes) per node, for a total of 24 tasks.  I use this number to tell mpirun how many processes to start, -np 24

NOTE:  If using python or another language you will also need to add the --oversubscribe parameter to mpirun, eg.

  `mpirun --oversubscribe -np 24 mpiscript.py`

More information about running MPI jobs within Slurm can be found here here: http://slurm.schedmd.com/mpi_guide.html.

# Managing Jobs

#### Cancelling a Job

To cancel a job, first find the jobID, you can use the _vuw-myjobs_ (or _squeue_) command to see a list of your jobs, including jobIDs.  Once you have that you can use the _scancel_ command, eg

   `scancel 236789`

To cancel all of your jobs you can use the -u flag followed by your username:

   `scancel -u harrelwe`


#### Viewing Job statistics

If you want to get a quick view of all the jobs completed within the last 5 days you can use the _vuw-history_ command, for example:

```
  vuw-job-history

MY JOBS WITHIN LAST 5 days
       JobID      State    JobName  MaxVMSize    CPUTime 
------------ ---------- ---------- ---------- ---------- 
2645          COMPLETED       bash              00:00:22 
2645.extern   COMPLETED     extern      0.15G   00:00:22 
2645.0        COMPLETED       bash      0.22G   00:00:20 
2734          COMPLETED       bash              00:07:40 
2734.extern   COMPLETED     extern      0.15G   00:07:40 
2734.0        COMPLETED       bash      0.22G   00:07:40 
```

You can also get a report of your completed jobs using the _sacct_ command.  For example if I wanted to get a report on how much memory my job used I could do the following:

   `sacct --units=G --format="MaxVMSize" -j 2156`

* MaxVMSize will report the maximum virtual memory (RAM plus swap space) used by my job in GigBytes ( --units=G ) 
* -j 2156 shows the information for job ID 2156
* type _man sacct_ at a prompt in engaging to see the documentation on the _sacct_ command


#### Viewing jobs in the Queue


To view your running jobs you can type _vuw-myjobs_  eg:


```
           [harrelwe@node142 ~]$ vuw-myjobs
             JOBID PARTITION     NAME     USER ST       TIME  NODES NODELIST(REASON)
           7921967 quicktest     bash harrelwe  R       0:12      1 c03n01
```

As you can see I have a single job running on the node c03n01 on the quicktest partition

You can see all the jobs in the queues by running the _vuw-alljobs_ command.  This will produce a very long list of jobs if the cluster is busy.

#### Job Queuing (aka Why is my job running?)

When a partition is busy, jobs will be placed in a queue.  You can observe this
in the _vuw-myjobs_ and _vuw-alljobs_ commands.  The STATE of your job will be PENDING, this means it is waiting for resources or your job has been re-prioritized to allow other users access to run their jobs (this is called fair-share queueing).

The resource manager will list a reason the job is pending, these reasons can include:

* **Priority** - Your job priority has been reduced to allow other users access to the cluster.  If no other user with normal priority is also pending then your job will start once resources are available.  Possible reasons why your priority has been lowered can include:  the number of jobs you have run in the past 24-48 hours; the duration of the job and the amount of resources requested.  The Slurm manager uses fair-share queuing to ensure the best use of the cluster.  You can google fair-share queuing  if you want to know more
* **Resources**- There are insufficient resources to start your job.  Some combination of CPU, Memory, Time or other specialized resource are unavailable.  Once resources are freed up your job will begin to run.  
Time:   If you request more time than the max run-time of a partition, your job will be queued indefinitely (in other words:  it will never run).  Your time request must be less than or equal to the Partition Max Run-Time.  Also if a special reservation is placed on the cluster, for instance prior to a scheduled maintenance, this too will reduce the available time to run your job.  You can see Max Run-Time for our partitions described in this document.  CAD or ITS Staff will alert all users prior to any scheduled maintenance and advise them of the time restrictions.
* **QOSGrpCPULimit** - This is a Quality of Service configuration to limit the number of CPUs per user.   The QOSMax is the maximum that can be requested for any single job.  If a user requests more CPUs than the QOSMax for a single job then the job will not run.  If the user requests more than QOSMax in 2 or more jobs then the subsequent jobs will queue until the users running jobs complete.
* **PartitionTimeLimit** - This means you have requested more time than the maximum runtime of the partition.  This document contains information about the different partitions, including max run-time.  Typing _vuw-partition_ will also show the max run-time for the partitions available to you.
* **ReqNodeNotAvail** - 99% of the time you will receive this code if you have asked for too much time. This frequently occurs when the cluster is about to go into maintenance and a reservation has been placed on the cluster, which reduces the maximum run-time of all jobs.  For example, if maintenance on the cluster is 1 week away, the maximum run-time on all jobs needs to be less than 1 week, regardless if the configured maximum run-time on a partition is greater than 1 week.  To request time you can use the --time parameter.  Another issue is if you request too much memory or a CPU configuration that does not exist on any node in a partition.  
* **Required node not available (down, drained or reserved)** - This is related to ReqNodeNotAvail, see above.

# Connecting to Cloud Providers

#### Amazon AWS

A feature-rich CLI is available in raapoi.  To use it you need to load the appropriate module and its module dependencies:

  `module load amazon/aws/cli`

Before you proceed you will need to configure your environment with your Access Key ID and Secret Access Key, both of which will be sent to you once your account is created or linked.  The command to configure your environment is `aws configure`  You only need to do this once, unless of course you use more than one user/Access Key.  Most users can simply click through the region and profile questions (using the default of "none").  If you do have a specific region this should be relayed along with your access and secret keys.

Once you have the appropriate environment in place and your configuration setup you can use the aws command, plus an appropriate sub-command (s3, emr, rds, dynamodb, etc) and supporting arguments. 

More information on the CLI can be found here:
[http://docs.aws.amazon.com/cli/latest/userguide/cli-chap-using.html](http://docs.aws.amazon.com/cli/latest/userguide/cli-chap-using.html)

##### Transferring Data to/from Amazon (AWS) S3

To transfer data from S3 you first need to setup your AWS connect, instructions for that can be found above.
Once that is done you should be able to use the aws commands to copy data to and from your S3 storage.  

For example if I wanted to copy data from my S3 storage to my project directory I could do the following:
```
  tmux
  module load amazon/aws/cli
  cd /nfs/scratch/harrelwe/project
  aws s3 cp s3://mybucket/mydata.dat mydata.dat
```
To copy something to storage simply reverse the file paths, eg.
```
  aws s3 cp mydata.dat s3://mybucket/mydata.dat 
```
The above starts a tmux session to allow the transfer to continue even if I
disconnect from the cluster (type `tmux attach` to reconnect).  I then load the
modules necessary to use the AWS commands.  I change directory to my project
space and use the aws s3 cp command to copy from S3.  More information on using
aws can be found here:
[http://docs.aws.amazon.com/cli/latest/reference/s3/index.html#cli-aws-s3](http://docs.aws.amazon.com/cli/latest/reference/s3/index.html#cli-aws-s3)

##### Working with AWS Data Analysis Tools

Amazon has a number of data analytics and database services available.  Using the command line utilities available in raapoi, researchers can perform work on the eo cluster and transfer data to AWS to perform further analysis with tools such as MapReduce (aka Hadoop), RedShift or Quicksight.

A listing of available services and documentation can be found at the following:
[https://aws.amazon.com/products/analytics/](https://aws.amazon.com/products/analytics/)

#### Google Cloud (gcloud) Connections 

The Google Cloud SDK is available in raapoi.  This includes a command-line interface for connecting to gloud services.  To get started, first load the environment module.  You can find the path with the `module spider` command.  As of this writing the current version can be loaded thusly:

  module load google/cloud/sdk/212.0.0

This will give you access to the `gcloud` command.  To setup a connection to your gcloud account use the init sub-command, eg.

  `gcloud init --console-only`

Follow the instructions to authorize your gcloud account.  Once on the Google website, you will be given an authorization code which you will copy/paste back into the raapoi terminal window.

##### Transferring Data to/from Google Cloud (gcloud)

To transfer data from gcloud storage you first need to setup your gcloud credentials, instructions for that can be found above.  Once that is done you should be able to use the `gsutil` command to copy data to and from your gcloud storage.  

For example, if I wanted to copy data from gcloud to my project directory I could do the following:
```
  tmux
  module load google/cloud/sdk/212.0.0
  cd /nfs/scratch/harrelwe/project
  gsutil cp gs://mybucket/mydata.dat mydata.dat
```
To copy something to storage simply reverse the file paths, eg.
```
  gsutil cp mydata.dat gs://mybucket/mydata.dat 
```

The above starts a tmux session to allow the transfer to continue even if I
disconnect from the cluster (type `tmux attach` to reconnect).  I then load the
modules necessary to use the gsutil commands.  I change directory to my project
space and use the gsutil cp command to copy from gcloud.  More information on
using gcloud can be found here:
[https://cloud.google.com/sdk/gcloud/](https://cloud.google.com/sdk/gcloud/)

##### Working with GCloud Data Analysis Tools

Google Cloud has a number of data analytics and database services available.  Using the gcloud command line utilities available on raapoi, researchers can perform work on the cluster and transfer data to gcloud to perform further analysis with tools such as Dataproc (Hadoop/Spark), BigQuery or Datalab (Visualization)

A listing of available services and documentation can be found at the following:
[https://cloud.google.com/products/](https://cloud.google.com/products/)

#### DropBox Cloud Storage

__NOTE:__ Dropbox has upload/download limitations and we have found that once your file gets above 50GB in size the transfer will have a better chance of timing out and failing.

Configuring your Dropbox account on raapoi

__Step A:__  On your local laptop or desktop start your browser and login to your Dropbox account

__Step B:__ On Raapoi type the following:

   `module load dropbox`

__Step C:__ Setup account credentials (You should only need to do this once):

Run the following command from Raapoi

   `dbxcli account`

You will now see something like the following:

  1. Go to https://www.dropbox.com/1/oauth2/authorize?client_id=X12345678&response_type=code&state=state
  2. Click "Allow" (you might have to log in first).
  3. Copy the authorization code.
  Enter the authorization code here:

__Step D:__

Copy the URL link listed in Step C1 and paste it into the web browser that you started in Step A

This will provide you with a long access code (aka hash).  Now copy that access code and paste it into your Raapoi terminal after Step C3 where it is asking for Enter the authorization code here 

Now hit enter or return.  You should see that you are now logged in with your Dropbox credentials

##### Basic Dropbox commands

Remember to load the dropbox environment module if you have not already (see `module spider` for the path)

Now type `dbx` or `dbxcli` at a prompt.  You will see a number of sub-commands, for instance ls, which will list the contents of your Dropbox, eg

  `dbxcli ls`

##### Downloading from Dropbox

Downloading uses the subcommand called: get.   The basic format for get is:

  `dbxcli get fileOnDropbox fileOnRaapoi`

For instance, if I have a datafile called 2018-financials.csv on Dropbox that I want to copy to my project folder I would type:

  `dbxcli get 2018-financials.csv /nfs/scratch/harrelwe/projects/finance_proj/2018-financials.csv`

##### Uploading to Dropbox

Uploading is similar to downloading except now we use the subcommand: put.  The basic format for put is:

  `dbxcli put fileOnRaapoi fileOnDropbox`

For example I want to upload a PDF I generated from one of my jobs called final-report.pdf I would type:

  `dbxcli put final-report.pdf final-report.pdf`

This will upload the PDF and name it the same thing, if I wanted to change the name on Dropbox I could:

  `dbxcli put final-report.pdf analytics-class-final-report.pdf`

#### AARNET Cloudstor

All VUW researchers have access to the AARNET (Australia’s Academic and Research
Network) Cloudstor service which provides __1 TB__ of space to each researcher.  To
use this service first login and download an appropriate client to your laptop
or desktop (or smarthone if you wish):

[Cloudstor Login](https://apac01.safelinks.protection.outlook.com/?url=https%3A%2F%2Fcloudstor.aarnet.edu.au%2Fsimplesaml%2Fmodule.php%2Fdiscopower%2Fdisco.php%3FentityID%3Dhttps%253A%252F%252Fcloudstor.aarnet.edu.au%252Fsimplesaml%252Fmodule.php%252Fsaml%252Fsp%252Fmetadata.php%252Fdefault-sp%26return%3Dhttps%253A%252F%252Fcloudstor.aarnet.edu.au%252Fsimplesaml%252Fmodule.php%252Fsaml%252Fsp%252Fdiscoresp.php%253FAuthID%253D_d8e2f7cbdf882a033df5b33385b5936c08e0e133a1%25253Ahttps%25253A%25252F%25252Fcloudstor.aarnet.edu.au%25252Fsimplesaml%25252Fmodule.php%25252Fcore%25252Fas_login.php%25253FAuthId%25253Ddefault-sp%252526ReturnTo%25253Dhttp%2525253A%2525252F%2525252Fcloudstor.aarnet.edu.au%2525252Fplus%2525252F%26returnIDParam%3Didpentityid&data=02%7C01%7Cwes.harrell%40vuw.ac.nz%7Cefa10aeb0f1d42a6cd2408d6953c9496%7Ccfe63e236951427e8683bb84dcf1d20c%7C0%7C0%7C636860484751891881&sdata=nkN7OI6g%2BKs%2FLoMG6tOtoWqk1%2FU1gbxZK%2FjCd8NNZ6c%3D&reserved=0)

NOTE: Within the Cloudstor web login settings you will need to create a Cloudstor Password, this is the password you will use to login on raapoi, it does not use your VUW credentials for the command line login.

Once you have setup your cloudstor (aka ownCloud) credentials you can use them
to sync data to and from Raapoi.  For example, if I wanted to sync my project
space to Cloudstor I would do the following from raapoi login node:

```
  tmux
  cd /nfs/scratch/harrelwe
  owncloudcmd -u wes.harrell@vuw.ac.nz project1 https://cloudstor.aarnet.edu.au/plus/remote.php/webdav/
```

The above sequence starts a tmux session to allow the transfer to continue even if I disconnect from the cluster
(type `tmux attach` to reconnect), I then change directory (_cd_) to my scratch space and then use the command `owncloudcmd` to sync my project directory called project1 to the Cloudstor service.  You will be prompted for your Cloudstor password, you can also embed this within the command line using the -p paramter.  

# Using Containers

Researchers can use Docker or Singularity containers within the cluster.  This is a great way to run difficult-to-compile applications or to share workflows among colleagues.

#### Running an interactive container

User can run within a container interactively, this is great for testing code before running a job.  Here is an example of running within a docker container that has the blockchain software called BlockSci:

```
module load singularity
srun --pty -c 4 --mem=16G bash
singularity pull docker://tislaamo/blocksci
singularity shell blocksci.simg
```

Once you have typed the _singularity shell_ command you will be within the
container and can type the commands available from within the container such as
the BlockSci utility **blocksci_parser**

#### Running a container in batch

Running a batch job with containers is similar to running a regular job, but will ultimately depend on how the container was created, so your mileage may vary.  Here is an example batch submit script that will run the BlockSci software that was created in an Ubuntu docker image, lets name the submit file runContainer.sh:

```
#SBATCH -J blockchain_parser
#SBATCH -c 4
#SBATCH --mem=16G
#SBATCH --mailtype=BEGIN,END,FAIL
#SBATCH --mail-user=myemail@email.net
#SBATCH --time=12:00:00

module load singularity
singularity exec blocksci.simg blocksci_parser --output-directory bitcoin-data update disk --coin-directory bitcoin
```

Now to run the file you can:

```
module load singularity
singularity pull docker://tislaamo/blocksci
sbatch runContainer.sh
```

Note that _singularity shell_ is primarily for interactive use and _singularity exec_ (or possibly _singularity run_) are for executing the applications that were built within the container directly.  It is important to know how the container was created to make effective use of the software.

# Using Jupyter Notebooks

#### Starting and Working with a Jupyter Notebook

__Step 1:__ The best way to start jupyter is with a batch submit script.  We have created an example script.  You can copy this script from one available on the cluster, just type the following:
```
cp /home/software/vuwrc/examples/jupyter/notebook.sh notebook.sh
```
If you are using Anaconda and have installed it in the default location you need to use the following submit file instead:
```
cp /home/software/vuwrc/examples/jupyter/notebook-anaconda.sh notebook-anaconda.sh
```

__NOTE__: We also have sbatch scripts for R (IRKernel)) notebooks.  These are called *R-notebook.sh* and *R-notebook-anaconda.sh*

This script is ready to run as is, but we recommend editing it to satisfy your own CPU, memory and time requirements.  Once you have edited the file you can run it thusly:

```
sbatch notebook.sh
```

or if using Anaconda:

```
sbatch notebook-anaconda.sh
```
__NOTE:__ If you copied the R notebook script, replace notebook.sh with R-notebook.sh

This will submit the file to run a job.  It may take some time for the job to
run, depending on how busy the cluster is at the time.  Once the job begins to
run you will see some information in the file called notebook-JOBID.out (JOBID
will be the actual jobid of this job, eg notebook-478903.out.  If you view this
file (users can type `cat notebook-JOBID.out` to view the file onscreen).  You will see a line such as:

 *The Jupyter Notebook is running at: http://130.195.19.20:47033/?token=SOME-RANDOM-HASH*

The 2 important pieces of information here are the IP address, in this case *130.195.19.20* and the port number, *47033*.   These numbers should be different for you since the port number is random, although the IP Address may be the same since we have a limited number of compute nodes. Also notice after the ?token= you will see a random hash.  This hash is a security feature and allows you to connect to the notebook.  You will need to use these to view the notebook from your local machine.  

__Step 2:__ To start working with the notebook you will need to tunnel a ssh
session.  In your SSH tunnel you will use the cluster login node (raapoi.vuw.ac.nz)
to connect to the compute node (in the example above the compute node is at
address 130.195.19.20) and transfer all the traffic back and forth between your computer and the compute node).  

__Step 2a from a Mac:__

Open a new session window from Terminal.app or other terminal utility such as Xquartz and type the following:

```
ssh -L <PORT_NUMBER>:<IP_ADDRESS>:<PORT_NUMBER> username@raapoi.vuw.ac.nz
```

For example:

```
ssh -L 47033:130.195.19.20:47033 harrelwe@raapoi.vuw.ac.nz
```

Once you are at a prompt you can go to Step 3

__Step 2b: from Windows__

We recommend tunnelling using Git Bash, which is part of the [Git for Windows project](https://gitforwindows.org/) or [MobaXTerm](https://mobaxterm.mobatek.net/).  There are 2 methods for tunneling in Moba, one is command line, the other is GUI-based.

_Method 1 (Git Bash or MobaXterm):_
Command-line, start a local Git Bash or MobaXterm terminal (or try the GUI method, below)

From the command prompt type:
```
ssh -L <PORT_NUMBER>:<IP_ADDRESS>:<PORT_NUMBER> username@raapoi.vuw.ac.nz
```

For example:

```
ssh -L 47033:130.195.19.20:47033 harrelwe@raapoi.vuw.ac.nz
```

Once you are at a prompt you can go to Step 3


_Method 2 (MobaXterm):_
GUI-based, go to the Tunneling menu:

Now click on *New SSH Tunnel*

When you complete the creation of your tunnel click __Start all tunnels__.  Enter your password and reply "yes" to any questions asked about accepting hostkeys or opening firewalls.  You can safely exit the tunnel building menu.

__Step 3__

Now open your favorite web browser and then use the URL from your job output file and paste it in your browsers location bar, for example my full URL was:

  __http://130.195.19.20:47033/?token=badef11b1371945b314e2e89b9a182f68e39dc40783ed68e__

__Step 4__

One last thing you need to do is to replace the IP address with the word *localhost*.  This will allow your browser to follow the tunnel you just opened and connect to the notebook running on an engaging compute node, in my case my address line will now look like this:

  __http://localhost:47033/?token=badef11b1371945b314e2e89b9a182f68e39dc40783ed68e__

Now you can hit return and you should see your notebook running on an Engaging compute node.

__NOTE__: when you are done with your notebook, please remember to cancel your
job to free up the resources for others, hint: *scancel*

If you want more information on working with Jupyter, there is good
documentation, here: [Jupyter Notebooks](http://jupyter-notebook.readthedocs.io/en/latest/)

# Examples

#### Simple Python program using virtualenv and pip

First we need to create a working directory and move there
```
mkdir python_test
cd python_test
```
Next we load the python 3 module and use python 3 to create a python virtualenv.  This way we can install pip packages which are not installed on the cluster
```
module load python/3.6.6
python3 -m venv mytest
```

Activate the `mytest` virtualenv and use pip to install the `webcolors` package
```
source mytest/bin/activate
pip install webcolors
```

Create the file test.py with the following contents using nano
```
import webcolors
from random import randint
from socket import gethostname

colour_list = list(webcolors.css3_hex_to_names.items())
requested_colour = randint(0,len(colour_list))
colour_name = colour_list[requested_colour][1]

print("Random colour name:", colour_name, " on host: ", gethostname())
```

Alternatively download it with wget:
```
wget https://raw.githubusercontent.com/eResearchSandpit/vuwrc/master/examples/python_venv/test.py
```

Using nano create the submissions script called python_submit.sh with the following content - change `me@email.com` to your email address.
```
#!/bin/bash
#
#SBATCH --job-name=python_test
#SBATCH -o python_test.out
#SBATCH -e python_test.err
#
#SBATCH --cpus-per-task=1
#SBATCH --mem-per-cpu=1G
#SBATCH --time=10:00
#
#SBATCH --mail-type=BEGIN,END,FAIL
#SBATCH --mail-user=me@email.com

module load python/3.6.6

source mytest/bin/activate
python test.py
```

Alternatively download it with wget
```
wget https://raw.githubusercontent.com/eResearchSandpit/vuwrc/master/examples/python_venv/python_submit.sh
```

To submit your job to the Slurm scheduler
```
sbatch python_submit.sh
```

Check for your job on the queue with `squeue` though it might finish very fast.  The output files will appear in your working directory.

# Parallel processing

Running a job in parallel is a great way to utilize the power of the cluster.  So what is a parallel job/workflow?

* Loosely-coupled jobs (sometimes referred to as embarrassingly or naively parallel jobs) are processes in which multiple instances of the same program execute on multiple data files simultaneously, with each instance running independently from others on its own allocated resources (i.e. CPUs and memory). Slurm job arrays offer a simple mechanism for achieving this.
* Multi-threaded programs that include explicit support for shared memory processing via multiple threads of execution (e.g. Posix Threads or OpenMP) running across multiple CPU cores.
* Distributed memory programs that include explicit support for message passing between processes (e.g. MPI). These processes execute across multiple CPU cores and/or nodes, these are often referred to as tightly-coupled jobs.
* GPU (graphics processing unit) programs including explicit support for offloading to the device via languages like CUDA or OpenCL.

It is important to understand the capabilities and limitations of an application in order to fully leverage the parallel processing options available on the cluster. For instance, many popular scientific computing languages like Python, R, and Matlab now offer packages that allow for GPU, multi-core or multithreaded processing, especially for matrix and vector operations.  If you need help with the design of your parallel workflow, send us a message in the raapoi-help Slack channel.

## Job Array Example
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



## Multi-threaded or Multi-processing Job Example

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

## MPI Jobs

Most users do not require MPI to run their jobs but many do.  Please read on if you want to learn more about using MPI for tightly-coupled jobs.

MPI (Message Passing Interface) code require special attention within Slurm. Slurm allocates and launches MPI jobs differently depending on the version of MPI used (e.g. OpenMPI, MPICH2). We recommend using OpenMPI version 2.1.1 or later to compile your C code (using mpicc) and then using the mpirun command in a batch submit script to launch parallel MPI jobs. The example below runs MPI code compiled by OpenMPI 2.1.1:

```
  #!/bin/bash
  #SBATCH --nodes=3
  #SBATCH --tasks-per-node=8 # 8 MPI processes per node
  #SBATCH --time=3-00:00:00
  #SBATCH --mem=4G # 4 GB RAM per node
  #SBATCH --output=mpi_job.log
  #SBATCH --partition=parallel
  #SBATCH --constraint="IB"
  #SBATCH --mail-type=BEGIN,END,FAIL
  #SBATCH --mail-user=me@email.com

  module load openmpi
  echo $SLURM_JOB_NODELIST
  mpirun -np 24 mpiscript.o

```

This example requests 3 nodes and 8 tasks (i.e. processes) per node, for a total of 24 tasks.  I use this number to tell mpirun how many processes to start, -np 24

_NOTE:_ We highly recomend adding the `--constraint="IB"` parameter to your MPI job as
this will ensure the job is run on nodes with an Infiniband interconnect.

_ALSO NOTE:_  If using python or another language you will also need to add the --oversubscribe parameter to mpirun, eg.

  `mpirun --oversubscribe -np 24 mpiscript.py`

More information about running MPI jobs within Slurm can be found here here: http://slurm.schedmd.com/mpi_guide.html.

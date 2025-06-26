# Frequently Asked Questions

## Login node 
* **_Why can't I run programs directly on the login node?_**
    * _Rāpoi_ _login_ is a shared resource and running programs directly on the login node puts unnecessary strain.
    The recommended method is to use `srun` or `sbatch`. Please request an interactive session to work on the cluster, e.g., writing programs, debugging, or even data transfer. 


---

## Requesting Resources
*  **_I don't want to interfere with other people work, what does it mean "Other users are prevented from using resources you request, even if you don't use them"?_**  
    * The system is shared, you will very rarely have full use of a node.  You need to be careful to request resources, leaving the extra space available to others.  For example, say you submitted a job to bigmem which asked for 800GB of ram and 10 CPUs.  Your job would end up on the node with 1000GB ram as only that one would fit it - if your job actually only used 300GB of ram - the extra 500GB of ram you requested would be "wasted" no one else could use it.  So, another user with a job requesting 600GB of ram would have to wait for your job to end even if there was space for it to run alongside yours.

    * The same issue occurs with CPU requests.   It can be very hard to accurately estimate memory and cpu needs before running your job.  If your job has a short run time (less than ~10 hours), you can just request more than you need and check the memory usage afterward to guide further jobs.  If your job has a long run time (several days), you should run a test job with a  short runtime (a few hours) to estimate your needs first. 


---

## Job Requirements    
* **_How do I know resource requirement for my job?_**
    * The best and most reliable method of determining resource requirement is from testing. #Note: It is possible to find out a ballpark figure for a particular software in terms of memory, cpus, or time, for example: MATLAB (~16 GB memory to initialize), see documentation or consult in slack help channel.
    * Select a test job carefully. As a rule of thumb, a test job should not run for more than 15 mins. Perhaps you could use smaller input, coarse parameters or use a subset of the calculations. 
    * Make sure your test job is quick to run and quick to start. The later can be ensured by keeping resource requirement to be small (mem or cpu).
    * Often a good first test to run, is to execute your job serially e.g., using only 1 CPU. These jobs should be easier to debug, and quicker to run.
    * Generally it is okay to request roughly 20% more time and memory than you think the job will use, any more than that can impact on other users.


---

## Visualisation
* **_How do I plot/visualise my results/data on Rāpoi?_**
    * Generally speaking, the best practice is to use Rāpoi for your heavy computing workloads, then transfer your results/data to a local machine to do any plotting/visualisation. An exception to this might be if the visualisation process itself is computationally intensive, the dataset is large and difficult to move, and/or requires specialised hardware (like a gpu).
    * The Rāpoi operating system is primarily designed for command line use, and thus doesn't include most of the software libraries that support graphical interfaces. This typically makes the installation of visualisation software a time consuming process.
    * Should you really need to plot/visualise on Rāpoi, there are several things you probably need to do. The first will be to have a suitable ssh client (see the section on the [Accessing the Cluster](accessing_the_cluster.md) page) and enable X11 forwarding by adding the `-X` option when you `ssh` into Rāpoi. Other steps can vary greatly depending on your local operating system and exactly what you want to do. If you need assistance, reach out on the [raapoi-help slack channel](https://uwrc.slack.com).


---

## MPI Tips
* **_Allocating tasks, threads and MPI processes - or how to parse the meanings of "ntasks", "cpus-per-task" and "nodes" in my sbatch scripts? source: [C.E.C.I hpc docs](https://support.ceci-hpc.be/doc/_contents/SubmittingJobs/SlurmFAQ.html#Q05)_**
    * you use mpi and do not care about where those cores are distributed: `--ntasks=16`
    * you want to launch 16 independent processes (no communication): `--ntasks=16`
    * you want those cores to spread across distinct nodes: `--ntasks=16` `--ntasks-per-node=1` or `--ntasks=16 --nodes=16`
    * you want 16 processes to spread across 8 nodes to have two processes per node: `--ntasks=16 --ntasks-per-node=2`
    * you want 16 processes to stay on the same node: `--ntasks=16 --ntasks-per-node=16`
    * you want one process that can use 16 cores for multithreading: -`-ntasks=1 --cpus-per-task=16`
    * you want 4 processes that can use 4 cores each for multithreading: `--ntasks=4 --cpus-per-task=4`

* **_How do I increase the memory available to Java based applications?_**
    * Add the command line option `-Xms<n>` where `<n>` should be replaced with the desired initial size of the memory allocation pool. For example, `-Xms512m` will result in an initial memory allocation pool of 512MB.
    * Add the command line option `-Xmx<n>` where `<n>` should be replaced with the desired maximum size of the memory allocation pool. For example, `-Xmx4096m` will set the maximum size of the memory allocation pool as 4096MB (i.e. 4GB).
    * The value supplied to these options obviously should not exceed the memory requested for a job through the srun/sbatch arguments.
    * See the [java documentation](https://docs.oracle.com/javase/6/docs/technotes/tools/windows/java.html) for more details.


* **_How do I know if I should use ntasks, cpus-per-tasks, etc.? ([1](https://www.mcs.anl.gov/research/projects/mpi/tutorial/index.html))_**

    For demo purposes, you could try this mpi example on Rāpoi: [example_mpi](https://hackmd.io/@vuwrd1/HkDYM2YOyg)

    * A little complicated as it depends on whether your program needs tasks or cores 
    ([1](<https://scicomp.stackexchange.com/questions/27409/requesting-less-than-a-node-with-slurm#:~:text=1-,tl%3Bdr,-For%20multiprocessing%20(MPI>),[2](https://hpc-uit.readthedocs.io/en/latest/jobs/slurm_parameter.html#:~:text=Requesting%20Resources)). 
    
    A simple difference to understand between *MPI* and *OpenMP* is that an *MPI based program* will be launced several times and communicates via messgae passing, while an *OpenMP based program* will only be launched once and will then launch several threads which communicate via shared memory. In case of a shared memory job, it is required that tasks run on the same node while a message passing job doesn't care about nodes as long as the tasks can communicate (via Infiniband, Ethernet, etc.). A general guide could be:

    * For *multiprocessing* (MPI, message parssing) use `ntasks`.
    * For *multithreading* (OpenMP, pthreads) use `cpus-per-task` - use 1 for MPI. For parallelized applications benchmark this is the number of threads.
    * For hybrid codes, you need both options and probably also want to tune `ntasks-per-node`; refers to the number of (MPI) processes per node. For OpenMP, only 1 task is necessary.
    * In nodes with *hyper-threading* enabled, use `--ntasks-per-core=1` to avoid distributing cores across sockets.
    * `--nodes=<num_nodes>` with more than 1 node is useful for jobs with distributed-memory (e.g. MPI).

    The `--ntasks` option of SLURM specifies how many tasks your program will launch, which could be threads of independent instances of the MPI program. However, SLURM assumes that when you say `--ntasks` you mean tasks which communicate by message passing and in case your machine has 12 cores but you requested 13 tasks, it will happily launch 12 tasks on one node and 1 on another node. (I don't think this behaviour is guaranteed. SLURM could also throw all 13 tasks on one node with 12 CPUs and let the CPU schedule the tasks. You can get more fine-grained control using `--ntasks-per-core` and `--ntasks-per-node`.)

    In case of a multithreaded program, then you want to use `--cpus-per-task` instead and set `--ntasks = 1` (or leave it unspecified, as it defaults to 1). This way if you request 13 CPUs but the maximum available is 12, your job will just be rejected.

    !!! note
        Check if your script uses MPI, if it does, it would benefit from running on multiple nodes simultaneously. If not, it doesn't make sense to request more than one node. For example, you made a mistake in your MPI code, you are running non-optimised simple applications on HPC like python or R scripts without utilising parallelism. 

    - OpenMP is a multiprocessing library is often used for programs on shared memory systems. Shared memory describes systems which share the memory between all processing units (CPU cores), so that each process can access all data on that system.

    - If your job requires more than the default available memory per core (2GB/core on a 256 core node) you should adjust this need with the following command: `#SBATCH --mem-per-cpu=10GB`. When doing this, the batch system will automatically allocate 50 cores or less per node (for a 500GB node).

---


## VS Code tips
* **_How do I clean up my VS Code Server processes on Rāpoi?_**
    * Unfortunately, VS Code seems to have somewhat poor process management when used to connect to Rāpoi for remote code development. In particular, it tends to leave a large number of processes sitting on the cluster occupying resources (even after you leave/close the session on your local machine). [This link from the vscode docs](https://code.visualstudio.com/docs/remote/troubleshooting#_cleaning-up-the-vs-code-server-on-the-remote) tells you how to cleanup your VS Code Server. In particular, they recommend running `kill -9 $(ps aux | grep vscode-server | grep $USER | grep -v grep | awk '{print $2}')` to kill server processes. If you are a user of VS Code, please do this whenever we you finish a session.


---

## Dependent Jobs
* **_How do I start a job that depends on the previous job finishing successfully?_**
    * Submit your first job `sbatch submit1.sl`. This will print a job id on your terminal `Submitted batch job 1042939`. 
    * Now submit the job you would like to start once the previous job has completed successfully, using: `sbatch --depend=afterok:1042939 submit2.sl`
    * Similar can be achieved for array jobs. 
    * If you need assistance, reach out on the [raapoi-help slack channel](https://uwrc.slack.com).

    Here is an example:
    ```
    $ sbatch submit.sl 
    Submitted batch job 1042939
    $ sbatch --depend=afterok:1042939 submit.sl
    Submitted batch job 1042940
    $ squeue --me
    JOBID PARTITION     NAME     USER ST       TIME  NODES NODELIST(REASON)
    1042940 quicktest dependen janedoe PD       0:00      1 (Dependency)
    1042939 quicktest dependen janedoe  R       0:37      1 itl02n01

    ```
    



    
    From the [Bioinformatics Workbook](https://bioinformaticsworkbook.org/Appendix/HPC/SLURM/submitting-dependency-jobs-using-slurm.html#gsc.tab=0)


    | Argument | Description  |
    |-------|-------|
    | after    | The job begins executing after the specified job have begun executing |
    | afterany    | The job begins executing after the specified job have terminated |
    | aftercorr    | A task of this job array can begin executing after the corresponding task ID in the specified job has completed successfully |
    | afternotok    | This job can begin execution after the specified jobs have terminated in some failed state |
    | afterok    | This job can begin execution after the specified jobs have successfully executed |
    | singleton    | This job can begin execution after any previously launched jobs sharing the same job name and user have terminated |



---


## COMSOL
* **_How do I simulate COMSOL model?_**
    * [COMSOL Docs](https://www.comsol.com/support/knowledgebase/1001)



    | Parameter       | SLURM                      | COMSOL                        |
    |-----------------|----------------------------|-------------------------------|
    | --nn            | Total number of Slurm Tasks | Total number of Compute Nodes |
    |                 | across all Slurm Nodes      | across all Hosts              |
    | --nnhost        | Number of Slurm Tasks per   | Number of compute nodes to    |
    |                 | Slurm Node                  | run on each host              |
    |                 | (*A COMSOL instance resides |                               |
    |                 | in each Slurm Task, and     |                               |
    |                 | communicates with other     |                               |
    |                 | Slurm Tasks using MPI)      |                               |
    | -np             | Number of Cores / CPUs used | Number of Cores / CPUs to be  |
    |                 | by each Slurm Task          |  used by each compute node    |
    

    * For e.g., let's assume a workload distribution of starting two COMSOL instances each using four CPUs/cores on a single node would look like:
    
    ```
    #SBATCH --nodes=1
    #SBATCH --ntasks-per-node=2
    #SBATCH --cpus-per-task=4
    
    <your stuff here>
    
    # COMSOL execution command
    /path/to/executable/comsol batch -nn 2 -nnhost 2 -np 4 \
    -inputfile <your${INPUTFILE}> ..... >> rest of the arguments
    
    ```


---

  
## Job Start    
* **_How do I know when will my job start?_** 
    * You can find start time for some of the jobs in pending state using:

    ```
    squeue --start -j <JobID>

    ```


---



## Full Node
* **_How do I request a whole node?_**
    
    We request users to be very careful with their requirements for such a job. However, here's a simple guide to follow [[1](https://hpc-uit.readthedocs.io/en/latest/jobs/slurm_parameter.html#:~:text=Requesting%20Resources)]:
    
    | Parameter       | Function                      | 
    |-----------------|----------------------------|
    | --nodes=<num+nodes>            | Start a parallel job for 
    | | a distributed memory system on several nodes | 
    | --ntasks-per-node=<num_procs> | Number of (MPI) processes per node.
    | |  Maximum number depends on cores per node |
    | --cpus-per-task=1 | User one CPU core per task |
    | --exclusive | Job will not share nodes with other running jobs.
    | | You don't need to specify memory 
    | | as you will get all available on the node. |

    To distribute your job:
    
    | Parameter | Function |
    |--|--|
    | --ntasks=<num_proc> | Total number of (MPI) processes. |
    | | Equal to the number of cores. |
    | --mem-per-cpu=<MB> | Memory (RAM) per requested CPU core. |

    You should run a few tests to see that your job is requested cpus that it can actually utilise efficiently. Try to run your job on 1, 2, 4, 8, 16, etc. cores to see when the runtime for your job starts tailing off. 

---

## Job Failed
* **_What should I do when my job fails (or hangs)?_**
    
    See if you can get your job to run by yourself when you follow these steps:
    
    * Make sure the job is a scaled down version of a huge problem. For example, it can be a single parameter of a huge parameter sweep study. A rule of thumb should be a job that can finish in about 10 - 15 mins. 
    * Make sure you have added `#SBATCH --error = slurm_%j.err` to your submission script.
    * Look through the file to get a hint of the possible error that caused your job to fail.
    * Check slurm exit code by `sacct -o Exitcode -j <JobId>`, look up on the web for help around the exit code. Exit codes: `0:53`. For example, here error `0:53` often means that something wasn't readable or writable.docs
    * Look for some common problems, e.g., OOM which means out-of-memory and try to increase memory request; permissions error - look for if files/folders referenced in your script exist by manually checking the directories.
    * Resubmit your batch script but this time include `#!/bin/bash -x` at the top of your submission script. 
    * Now, the error file will produce debugging information for further inquiries to make into the program's behaviour and identifying where it failed.
    * If you software utilises OpenMPI or OpenBLAS, please check the related faq's and user guides to ensure you are following best practice with these pieces of software.
    * If nothing works, send the script file, JobId, error file, and other logs to [raapoi-help slack channel](https://uwrc.slack.com) to ask for help from the admins. They know the system and may have helped another user do something similar.


---

## Bug Report
* **_How do I ask for help?_**
    * If you need assistance, reach out on the [raapoi-help slack channel](https://uwrc.slack.com).
    * Good request for help will include:
    * A JobID that you tried with a short description of the problem like what are you trying to achieve and what went wrong.
    * If possible, attach your script and error file. 
   
    

---


    
 





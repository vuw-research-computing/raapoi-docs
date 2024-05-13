# Frequently Asked Questions

*  *I don't want to interfere with other people work, what does it mean "Other users are prevented from using resources you request, even if you don't use them"?*  
    * The system is shared, you will very rarely have full use of a node.  You need to be careful to request resources, leaving the extra space available to others.  For example, say you submitted a job to bigmem which asked for 800GB of ram and 10 CPUs.  Your job would end up on the node with 1000GB ram as only that one would fit it - if your job actually only used 300GB of ram - the extra 500GB of ram you requested would be "wasted" no one else could use it.  So, another user with a job requesting 600GB of ram would have to wait for your job to end even if there was space for it to run alongside yours.

    * The same issue occurs with CPU requests.   It can be very hard to accurately estimate memory and cpu needs before running your job.  If your job has a short run time (less than ~10 hours), you can just request more than you need and check the memory usage afterward to guide further jobs.  If your job has a long run time (several days), you should run a test job with a  short runtime (a few hours) to estimate your needs first. 

* *Allocating tasks, threads and MPI processes - or how to parse the meanings of **ntasks**, **cpus-per-task** and **nodes** in my sbatch scripts* From the [C.E.C.I hpc docs](https://support.ceci-hpc.be/doc/_contents/SubmittingJobs/SlurmFAQ.html#Q05)
    * you use mpi and do not care about where those cores are distributed: `--ntasks=16`
    * you want to launch 16 independent processes (no communication): `--ntasks=16`
    * you want those cores to spread across distinct nodes: `--ntasks=16` `--ntasks-per-node=1` or `--ntasks=16 --nodes=16`
    * you want 16 processes to spread across 8 nodes to have two processes per node: `--ntasks=16 --ntasks-per-node=2`
    * you want 16 processes to stay on the same node: `--ntasks=16 --ntasks-per-node=16`
    * you want one process that can use 16 cores for multithreading: -`-ntasks=1 --cpus-per-task=16`
    * you want 4 processes that can use 4 cores each for multithreading: `--ntasks=4 --cpus-per-task=4`

* *How do I increase the memory available to Java based applications?*
    * Add the command line option `-Xms<n>` where `<n>` should be replaced with the desired initial size of the memory allocation pool. For example, `-Xms512m` will result in an initial memory allocation pool of 512MB.
    * Add the command line option `-Xmx<n>` where `<n>` should be replaced with the desired maximum size of the memory allocation pool. For example, `-Xmx4096m` will set the maximum size of the memory allocation pool as 4096MB (i.e. 4GB).
    * The value supplied to these options obviously should not exceed the memory requested for a job through the srun/sbatch arguments.
    * See the [java documentation](https://docs.oracle.com/javase/6/docs/technotes/tools/windows/java.html) for more details.

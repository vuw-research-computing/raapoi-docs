# Managing Jobs
## Cancelling a Job

To cancel a job, first find the jobID, you can use the _vuw-myjobs_ (or _squeue_) command to see a list of your jobs, including jobIDs.  Once you have that you can use the _scancel_ command, eg

   `scancel 236789`

To cancel all of your jobs you can use the -u flag followed by your username:

   `scancel -u harrelwe`


## Viewing Job information

#### Job History

If you want to get a quick view of all the jobs completed within the last 5 days you can use the _vuw-job-history_ command, for example:

```
$ vuw-job-history

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

#### Job Reports

To view a report of your past jobs you can run _vuw-job-report_:

```
$ vuw-job-report 162711

JOB REPORT FOR JOB 162711
     JobName  Nodes    ReqMem   UsedMem(GB)  ReqCPUs CPUTime    State    Completed
test-schro        1       64Gn                   24  00:02.513  COMPLETED 2019-05-28T16:17:10
     batch        1       64Gn      0.15G        24  00:00.210  COMPLETED 2019-05-28T16:17:10
    extern        1       64Gn      0.15G        24  00:00.002  COMPLETED 2019-05-28T16:17:10
```

__NOTE:__ In this example you see that I requested 64 GigaBytes of memory but only used 0.15 GB.  This means that 63 GB of memory went unused, which was a waste of resources.

You can also get a report of your completed jobs using the _sacct_ command.  For example if I wanted to get a report on how much memory my job used I could do the following:

   `sacct --units=G --format="MaxVMSize" -j 2156`

* MaxVMSize will report the maximum virtual memory (RAM plus swap space) used by my job in GigBytes ( --units=G )
* -j 2156 shows the information for job ID 2156
* type _man sacct_ at a prompt in engaging to see the documentation on the _sacct_ command


## Viewing jobs in the Queue


To view your running jobs you can type _vuw-myjobs_  eg:


```
$ vuw-myjobs
JOBID PARTITION     NAME     USER ST       TIME  NODES NODELIST(REASON)
7921967 quicktest     bash harrelwe  R       0:12      1 c03n01
```

As you can see I have a single job running on the node c03n01 on the quicktest partition

You can see all the jobs in the queues by running the _vuw-alljobs_ command.  This will produce a very long list of jobs if the cluster is busy.

## Job Queuing (aka Why isn't my job running?)

When a partition is busy, jobs will be placed in a queue.  You can observe this
in the _vuw-myjobs_ and _vuw-alljobs_ commands.  The STATE of your job will be PENDING, this means it is waiting for resources or your job has been re-prioritized to allow other users access to run their jobs (this is called fair-share queueing).

The resource manager will list a reason the job is pending, these reasons can include:

* **Priority** - Your job priority has been reduced to allow other users access to the cluster.  If no other user with normal priority is also pending then your job will start once resources are available.  Possible reasons why your priority has been lowered can include:  the number of jobs you have run in the past 24-48 hours; the duration of the job and the amount of resources requested.  The Slurm manager uses fair-share queuing to ensure the best use of the cluster.  You can google fair-share queuing  if you want to know more
* **Resources**- There are insufficient resources to start your job.  Some combination of CPU, Memory, Time or other specialized resource are unavailable.  Once resources are freed up your job will begin to run.  
Time:   If you request more time than the max run-time of a partition, your job will be queued indefinitely (in other words:  it will never run).  Your time request must be less than or equal to the Partition Max Run-Time.  Also if a special reservation is placed on the cluster, for instance prior to a scheduled maintenance, this too will reduce the available time to run your job.  You can see Max Run-Time for our partitions described in this document.  CAD or ITS Staff will alert all users prior to any scheduled maintenance and advise them of the time restrictions.
* **QOSGrpCPULimit** - This is a Quality of Service configuration to limit the number of CPUs per user.   The QOSMax is the maximum that can be requested for any single job.  If a user requests more CPUs than the QOSMax for a single job then the job will not run.  If the user requests more than QOSMax in 2 or more jobs then the subsequent jobs will queue until the users running jobs complete.
* **PartitionTimeLimit** - This means you have requested more time than the maximum runtime of the partition.  This document contains information about the different partitions, including max run-time.  Typing _vuw-partitions_ will also show the max run-time for the partitions available to you.
* **ReqNodeNotAvail** - 99% of the time you will receive this code if you have asked for too much time. This frequently occurs when the cluster is about to go into maintenance and a reservation has been placed on the cluster, which reduces the maximum run-time of all jobs.  For example, if maintenance on the cluster is 1 week away, the maximum run-time on all jobs needs to be less than 1 week, regardless if the configured maximum run-time on a partition is greater than 1 week.  To request time you can use the --time parameter.  Another issue is if you request too much memory or a CPU configuration that does not exist on any node in a partition.  
* **Required node not available (down, drained or reserved)** - This is related to ReqNodeNotAvail, see above.

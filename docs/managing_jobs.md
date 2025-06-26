# Managing Jobs
## Cancelling a Job

To cancel a job, first find the jobID, you can use the _vuw-myjobs_ (or _squeue_) command to see a list of your jobs, including jobIDs.  Once you have that you can use the _scancel_ command, eg

   `scancel 236789`

To cancel all of your jobs you can use the -u flag followed by your username:

   `scancel -u <username>`

**Note:** Before cancelling your jobs, please make sure it runs for at least 2 mins, including the jobs submitted in error. 


---

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


---

## Viewing jobs in the Queue


To view your running jobs you can type `vuw-myjobs`  eg:


```
$ vuw-myjobs
               JOBID    PARTITION   CPUS  MIN_MEM   GPUs             RUN_TIME   TIME_LIMIT   PRIORITY      STATE             NODELIST JOB_NAME
      2039592_[9-10]     parallel      9     180G      0  2025-04-28T16:27:25  10-00:00:00      27365    PENDING          (Resources) MLMAP
     2042371_[11-20]     parallel      9     180G      0  2025-04-29T15:46:36  10-00:00:00      24560    PENDING           (Priority) MLMAP
     2042372_[11-20]     parallel      9     180G      0  2025-04-29T15:46:36  10-00:00:00      24560    PENDING           (Priority) MLMAP
           2039591_1     parallel     10     200G      0             11:17:24  10-00:00:00      24484    RUNNING             amd07n02 MLMAP
           2039591_2     parallel     10     200G      0             11:17:24  10-00:00:00      24484    RUNNING             amd06n03 MLMAP
           2039591_3     parallel     10     200G      0             11:17:24  10-00:00:00      24484    RUNNING             amd06n04 MLMAP

```

The `vuw-myjobs` command will show you all your jobs - _running_ and _pending_, the _partition_ they are in, the requested resources, the _priority_ of the job and the _state_ of the job.  For the jobs in _PENDING_ state, the reason for the job being in that state is also shown and an expectation of when the job will start.  

You can see all the jobs in the queues by running the _vuw-alljobs_ command.  This will produce a very long list of jobs if the cluster is busy.

!!! note
      By default, each user is allowed _1,024 CPU cores_, _2,048 GB of memory_ and _3 GPUs_ at any one time. Once you reach these limits, your jobs will be queued until resources are available.  These limits are set to ensure that all users have fair access to the cluster.  If you need more resources, please contact the [Rāpoi support team](support.md).


---

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

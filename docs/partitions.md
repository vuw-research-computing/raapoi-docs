# Partitions
## Using Partitions

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
c03n[02-04],c04n[02-04],c05n[01-02],c07n[01,03-04],c08n[01-04],c09n[01-04]

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

## Partition Descriptions

### Partition: quicktest

This partition is for quick tests of code, environment, software builds or
similar short-run jobs.  Since the max time limit is 1 hour it should not take
long for your job to run.  This can also be used for near-on-demand interactive
jobs.

* Maximum CPU available per task: 24
* Maximum memory available per task: 62G
* Maximum Runtime: 1 hour

<!--- ### Partition: gpu

This partition is for those jobs that require GPUs or those software that work with the CUDA platform and API (tensorflow, pytorch, MATLAB, etc)

* GPU nodes available: 2
* GPUs available per node: 3
* Maximum CPU available per task: 32
* Maximum memory available per task: 92G
* Maximum Runtime: 24 hours

_Note_:  To request GPU add the parameter, `--gres=gpu:X`  Where X is the number of GPUs required, typically 1:  `--gres=gpu:1` --->

### Partition: bigmem

This partition is primarily useful for jobs that require very large shared
memory (greater than 125 GB).  These are known as memory-bound jobs.

__NOTE:__ Please do not schedule jobs of less than 125GB of memory on the bigmem partition.

* Maximum CPU available per task: 48
* Maximum memory available per task: 1 TB (Note: maximum CPU for 1 TB is 40)
* Maximum Runtime: 10 days

### Partition: parallel

This partition is useful for parallel workflows, either loosely coupled or jobs
requiring MPI or other message passing protocols for tightly bound jobs.

* Maximum CPU available per task: 64
* Maximum memory available per task: 125G
* Maximum Runtime: 10 days

### Cluster Default Resources

Please note that if you do not specify CPU, Memory or Time in your job request
you will be given the cluster defaults which are:

* Default CPU: 2
* Default Memory: 2 GB
* Default Time: 1 hour

You can change these with the -c, --mem and --time parameters to the srun and sbatch commands.  Please see this documentation for more information about srun and sbatch.

# Partitions
## Using Partitions

A partition is a collection of compute nodes, think of it as a sub-cluster or
slice of the larger cluster.  Each partition has its own rules and
configurations.  

For example, the quicktest partition has a maximum job run-time of 5 hours, whereas the partition
bigmem has a maximum runtime of 10 days.  Partitions can also
limit who can run a job.  Currently any user can use any partition but there
may come a time when certain research groups purchase their own nodes and they are
given exclusive access.

To view the partitions available to use you can type the vuw-partitions
command, eg

```
<user>@raapoi-master:~$ vuw-partitions 

VUW CLUSTER PARTITIONS
PARTITION  AVAIL  TIMELIMIT  NODES  STATE NODELIST
quicktest*    up    5:00:00      1  down* itl03n02
quicktest*    up    5:00:00      4    mix itl02n[01-04]
quicktest*    up    5:00:00      1   idle itl03n01

PARTITION AVAIL  TIMELIMIT  NODES  STATE NODELIST
gpu          up 1-00:00:00      1    mix gpu02
gpu          up 1-00:00:00      2   idle gpu[01,03]

PARTITION AVAIL  TIMELIMIT  NODES  STATE NODELIST
bigmem       up 10-00:00:0      3    mix high[01-02,04]
bigmem       up 10-00:00:0      1  alloc high03

PARTITION AVAIL  TIMELIMIT  NODES  STATE NODELIST
parallel     up 10-00:00:0      1   resv spj01
parallel     up 10-00:00:0     24    mix amd01n[01-04],amd02n[01-04],amd03n[01-04],amd04n[01-04],amd05n[01-04],amd06n[01-04]
parallel     up 10-00:00:0      2  alloc amd07n[03-04]

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
similar short-run jobs.  Since the max time limit is 5 hours it should not take
long for your job to run.  This can also be used for near-on-demand interactive
jobs.  Note that unlike the other partitions, these nodes have intel cpus.

* Quicktest nodes available: 6
* Maximum CPU available per task: 64
* Maximum memory available per task: 128G
* Optimal cpu/mem ratio: 1 cpu/2G ram
* Minimum allocated cpus: 2 - Slurm won't split an SMT core between users/jobs
* Maximum Runtime: 5 hours

### Partition: gpu

This partition is for those jobs that require GPUs or those software that work with the CUDA platform and API (tensorflow, pytorch, MATLAB, etc)

* GPU nodes available: 3
* GPUs available per node: 2 (A100's)
* Maximum CPU available per task: 256
* Maximum memory available per task: 512G
* Optimal cpu/mem ratio: 1 cpu/2G ram
* Minimum allocated cpus: 2 - Slurm won't split an SMT core between users/jobs
* Maximum Runtime: 24 hours

_Note_:  To request GPU add the parameter, `--gres=gpu:X`  Where X is the number of GPUs required, typically 1:  `--gres=gpu:1` -

### Partition: bigmem

This partition is primarily useful for jobs that require very large shared
memory (greater than 125 GB).  These are known as memory-bound jobs.

__NOTE:__ Please do not schedule jobs of less than 125GB of memory on the bigmem partition.

* Bigmem nodes available: 4 (4x1024G ram)
* Maximum CPU available per task: 128
* Maximum memory available per task: 1 TB
* Optimal cpu/mem ratio: 1 cpu/8G ram - note jobs here often use much more ram than this.
* Minimum allocated cpus: 1 - These cpus are not currently SMT enabled.
* Maximum Runtime: 10 days

### Partition: parallel

This partition is useful for parallel workflows, either loosely coupled or jobs
requiring MPI or other message passing protocols for tightly bound jobs. The total number of CPU's in this partition is 6816 with 2GB ram per CPU.

*AMD nodes - amdXXnXX*

* AMD nodes available: 28
* Maximum CPU available per task: 256
* Maximum memory available per task: 512G
* Optimal cpu/mem ratio: 1 cpu/2G ram
* Minimum allocated cpus: 2 - Slurm won't split an SMT core between users/jobs
* Maximum Runtime: 10 days

### Cluster Default Resources

Please note that if you do not specify CPU, Memory or Time in your job request
you will be given the cluster defaults which are:

* Default CPU: 2
* Default Memory: 2 GB
* Default Time: 1 hour

You can change these with the -c, --mem and --time parameters to the srun and sbatch commands.  Please see this documentation for more information about srun and sbatch.

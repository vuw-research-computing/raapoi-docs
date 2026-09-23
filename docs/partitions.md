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
command, e.g.

```
<user>@raapoi-login:~$ vuw-partitions 

VUW CLUSTER PARTITIONS
PARTITION  AVAIL  TIMELIMIT  NODES  STATE NODELIST
quicktest*    up    5:00:00      1    mix amd01n04
quicktest*    up    5:00:00      3   idle amd01n[01-03]

PARTITION AVAIL  TIMELIMIT  NODES  STATE NODELIST
parallel     up 10-00:00:0      1  drain amd02n01
parallel     up 10-00:00:0      1   resv spj01
parallel     up 10-00:00:0     23    mix amd02n[02-04],amd03n[01-04],amd04n[01-04],amd05n[01-04],amd06n[01-04],amd07n[01-04]

PARTITION AVAIL  TIMELIMIT  NODES  STATE NODELIST
gpu          up 1-00:00:00      3    mix gpu[01-03]

PARTITION AVAIL  TIMELIMIT  NODES  STATE NODELIST
bigmem       up 10-00:00:0      3    mix high[02-04]
bigmem       up 10-00:00:0      1   idle high01

PARTITION AVAIL  TIMELIMIT  NODES  STATE NODELIST
longrun      up 30-00:00:0      2    mix bigtmp[01-02]

NOTE: This utility is a wrapper for the Slurm command:
      sinfo -p PARTITION

```      

Notice the STATE field, this describes the current condition of nodes within the
partition, the most common states are defined as:

* __idle__ - nodes in an idle state have no jobs running, all resources are available
for work
* __mix__ - nodes in a mixed state have some jobs running, but still have some
resources available for work
* __mixed+planned__ - nodes have some jobs running and additional jobs planned by Slurm
* __alloc__ - nodes in an alloc state are completely full, all resources are in use.
* __drain__ - nodes in a drain state have some running jobs, but no new jobs can be
run.  This is typically done before the node goes into maintenance
* __maint__ - node is in maintenance mode, no jobs can be submitted
* __resv__ - node is in a reservation.  A reservation is setup for future maintenance
or for special purposes such as temporary dedicated access
* __down__ - node is down, either for maitnenance or due to failure

Also notice the _TIMELIMIT_ field, this describes the maximum runtime of a
partition.  For example, the quicktest partition has a maximum runtime of 5
hours and the parallel partition has a max runtime of 10 days.

---

## Partition Descriptions

### Partition: quicktest

This partition is for quick tests of code, environment, software builds or
similar short-run jobs.  Since the max time limit is 5 hours it should not take
long for your job to run.  This can also be used for near-on-demand interactive
jobs.

* Quicktest nodes available: 4
* Maximum CPU available per task: 256
* Maximum memory available per task: 502G
* Optimal cpu/mem ratio: 1 cpu/2G ram
* Minimum allocated cpus: 2 - Slurm won't split an SMT core between users/jobs
* Maximum Runtime: 5 hours
* These nodes are connected to the InfiniBand network. 

### Partition: parallel

This partition is useful for parallel workflows, either loosely coupled or jobs
requiring MPI or other message passing protocols for tightly bound jobs. It has 24 AMD nodes (`amdXXnXX`) with 256 CPUs and 502GB RAM each, plus `spj01` with 128 CPUs and 250GB RAM.

*AMD nodes - amdXXnXX*

* AMD nodes available: 24
* Maximum CPU available per task: 256
* Maximum memory available per task: 502G
* Optimal cpu/mem ratio: 1 CPU/2G RAM
* Minimum allocated cpus: 2 - Slurm won't split an SMT core between users/jobs
* Maximum Runtime: 10 days
* These nodes are connected to the InfiniBand network.

### Partition: gpu

This partition is for those jobs that require GPUs or those software that work with the CUDA platform and API (tensorflow, pytorch, MATLAB, etc)

* GPU nodes available: 3
* GPUs available per node: 2 (NVIDIA A100-PCIE-40GB)
* Maximum CPU available per task: 256
* Maximum memory available per task: 502G
* Optimal cpu/mem ratio: 1 CPU/2G RAM
* Minimum allocated cpus: 2 - Slurm won't split an SMT core between users/jobs
* Maximum Runtime: 24 hours

_Note_:  To request GPU add the parameter, `--gres=gpu:X`  Where X is the number of GPUs required, typically 1:  `--gres=gpu:1` -

### Partition: bigmem

This partition is primarily useful for jobs that require very large shared
memory (greater than 125 GB).  These are known as memory-bound jobs.

__NOTE:__ Please do not schedule jobs of less than 125GB of memory on the bigmem partition.

* Bigmem nodes available: 4 (4 x 1024G RAM)
* Maximum CPU available per task: 128
* Maximum memory available per task: 1 TB
* Optimal cpu/mem ratio: 1 CPU/8G RAM - note jobs here often use much more ram than this.
* Minimum allocated cpus: 1 - These cpus are not currently SMT enabled.
* Maximum Runtime: 10 days

_Note_: The bigmem nodes also each have one NVIDIA Tesla T4 GPU. These are not as powerful as the A100's in the gpu nodes, but may still be of use at times when (a) the gpu partition is particularly busy or a gpu node is down, **and** (b) the bigmem node is being under-utilised. (i.e. please try to avoid using these gpus when there is a high demand for jobs requiring lots of memory in the bigmem partition, at other time, please go ahead.)

### Partition: longrun

This partition is useful for long running jobs (with modest resource requirements).
The total number of CPU's in this partition is 512 with ~2GB RAM per CPU. This partition has two nodes, ```bigtmp01``` and ```bigtmp02```, each with 25TB of local ```/tmp``` storage.

*AMD nodes - bigtmpXX*

* AMD nodes available: 2
* Maximum CPU available per task: 256
* Maximum memory available per task: 502G
* Optimal cpu/mem ratio: 1 CPU/2G RAM
* Minimum allocated cpus: 2 - Slurm won't split an SMT core between users/jobs
* Maximum Runtime: 30 days

---

## Cluster Default Resources

Please note that if you do not specify the Partition, CPU, Memory or Time in your job request 
(via `srun` or `sbatch`)
you will be assigned the corresponding cluster defaults.
The defaults are:

* Default Partition: quicktest
* Default CPUs: 2
* Default Memory: 2 GB
* Default Time: 1 hour

You can change these with the --partition , -c, --mem and --time parameters, respectively, to the srun and sbatch commands.  
Please see [this section](running_jobs.md) of the documentation for more information on how to run jobs using `srun` and `sbatch`.


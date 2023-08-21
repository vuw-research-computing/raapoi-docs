# Moderating a Slurm Cluster

This a list of common cluster moderator actions, provided as reference. Users without moderator privileges might find some of this of interest, but you won't be able to perform the actions that affect other users.

These commands will require you to be logged in with your moderator-specific account

## Dealing with badly behaved jobs

### Holding jobs

Users will occasionally run jobs which consume an unfair amount of resources, if a single user is causes problems, you can hold their jobs.  This won't stop their current jobs, but will prevent more from starting

```bash
# hold some jobs
scontrol hold jobid1,jobid2,etc

# Allow the jobs back onto the queue
scontrol requeue jobid1,jobid2,etc    ## previous step sets priority to zero so they won'ßt actually start now

# Release the jobs to run again
scontrol release jobid1,jobid2,etc
```

Alterativly you can reduce their priority to a low  setting

```bash
squeue -p gpu -u <username> -t pending --format "scontrol update jobid=%i nice=1000000" | sh
```

### Cancelling jobs

If a users jobs are causing too many problems, you can cancel their jobs.
Note this is drastic and can throw away many days of compute, it's best to try get hold of a user first. Get them to cancel their own jobs. 

If needed though:
```bash 
scancel <jobid>  # be careful to get the correct job id!

# to cancel all their running jobs on parallel
squeue -p parallel -u <username> -t running --format "scancel %i" | sh
```

### Limiting an unresponsive users resource allowance on Raapoi

#### Set maxjobs
```bash
sacctmgr modify user where name=bob set MaxJobs=2
```

After a few minutes you should be able to see the results on squeue
```bash
squeue -u bob -o "%i %r"

# returns something like
JOBID REASON
20582 AssocMaxJobsLimit
20583 Dependency
```

#### Limiting GPU resources

```bash
sacctmgr modify user bob set GrpTRES=cpu=-1,mem=-1,gres/gpu=4  # -1 means no restriction.

#check result
sacctmgr list assoc User=bob
```

#### Limiting CPU resources

```bash
sudo sacctmgr modify user <user> set GrpTRES=cpu=1026
```

## Using reservations

If a research group has a good need and the other moderators agree, you can give them a reservation that only they can use. This is usually done for a specific time period.  This is also one of the steps when we put the cluster into maintenance

Create a month-long reservation on amd01n01 and amd01n02
```bash
scontrol create reservationname=MyReservation starttime=2021-03-01T11:00:00 duration=30-00:00:00 user=user1,user2,user3 nodes=amd01n01,amd01n02
```

Users will use the reservation with
```bash
##SBATCH --reservation=MyReservation
```


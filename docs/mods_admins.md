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

## Building software with EasyBuild

Use a terminal multiplexer like screen, tmux or byobu to keep your ssh session alive and get a interactive session on a node. 

### Build a simple program

Here we will build a simple program called velvet - it's a genome assembler.

```bash
srun -c 10 --mem=10G -p parallel --time=6:00:00 --pty bash

# now on the node
module purge # just in case
module load EasyBuild

# Search for the package 
eb -S velvet

# Returns
* $CFGS1/v/Velvet/Velvet-1.2.10-GCC-8.3.0-mt-kmer_191.eb
* $CFGS1/v/Velvet/Velvet-1.2.10-GCC-11.2.0-mt-kmer_191.eb
* $CFGS1/v/Velvet/Velvet-1.2.10-foss-2018a-mt-kmer_191.eb
* $CFGS1/v/Velvet/Velvet-1.2.10-foss-2018b-mt-kmer_191.eb
* $CFGS1/v/Velvet/Velvet-1.2.10-intel-2017a-mt-kmer_37.eb

# We want to pick one that won't need to build a whole new toolchain if we can avoid it
# Let's have a look at what would get built with a Dry (D) run.  The r is for robot to
# find all the dependancies
eb -Dr Velvet-1.2.10-intel-2017a-mt-kmer_37.eb

# Partial return
* [x] $CFGS/m/M4/M4-1.4.17.eb (module: Core | M4/1.4.17)
* [x] $CFGS/b/Bison/Bison-3.0.4.eb (module: Core | Bison/3.0.4)
* [x] $CFGS/f/flex/flex-2.6.0.eb (module: Core | flex/2.6.0)
* [x] $CFGS/z/zlib/zlib-1.2.8.eb (module: Core | zlib/1.2.8)
* [ ] $CFGS/b/binutils/binutils-2.27.eb (module: Core | binutils/2.27)
* [ ] $CFGS/g/GCCcore/GCCcore-6.3.0.eb (module: Core | GCCcore/6.3.0)
* [ ] $CFGS/z/zlib/zlib-1.2.11-GCCcore-6.3.0.eb 
* [ ] $CFGS/h/help2man/help2man-1.47.4-GCCcore-6.3.0.eb 
* [ ] $CFGS/m/M4/M4-1.4.18-GCCcore-6.3.0.eb 
* [ ] $CFGS/b/Bison/Bison-3.0.4-GCCcore-6.3.0.eb 
* [ ] $CFGS/f/flex/flex-2.6.3-GCCcore-6.3.0.eb 
* [ ] $CFGS/i/icc/icc-2017.1.132-GCC-6.3.0-2.27.eb 
* [ ] $CFGS/i/ifort/ifort-2017.1.132-GCC-6.3.0-2.27.eb 
* [ ] $CFGS/i/iccifort/iccifort-2017.1.132-GCC-6.3.0-2.27.eb 
* [ ] $CFGS/i/impi/impi-2017.1.132-iccifort-2017.1.132-GCC-6.3.0-2.27.eb 
* [ ] $CFGS/i/iimpi/iimpi-2017a.eb (module: Core | iimpi/2017a)
* [ ] $CFGS/i/imkl/imkl-2017.1.132-iimpi-2017a.eb 
* [ ] $CFGS/i/intel/intel-2017a.eb (module: Core | intel/2017a)
* [ ] $CFGS/v/Velvet/Velvet-1.2.10-intel-2017a-mt-kmer_37.eb 

# Packages wiht a [x] are already built, [ ] will need to be built. This is a lot of building,, including a "new" compiler - intel-2017a.eb, let's avoid that and try another
eb -Dr Velvet-1.2.10-foss-2018b-mt-kmer_191.eb

# Partially returns, all [x] except for velvet
...
* [x] $CFGS/f/FFTW/FFTW-3.3.8-gompi-2018b.eb 
* [x] $CFGS/s/ScaLAPACK/ScaLAPACK-2.0.2-gompi-2018b-OpenBLAS-0.3.1.eb
* [x] $CFGS/f/foss/foss-2018b.eb (module: Core | foss/2018b)
* [ ] $CFGS/v/Velvet/Velvet-1.2.10-foss-2018b-mt-kmer_191.eb 

# To build this we would
eb -r --parallel=$SLURM_CPUS_PER_TASK Velvet-1.2.10-foss-2018b-mt-kmer_191.eb

```
Remember to close the interactive session when done to stop it consuming resources.

### Building a new toolchain

Below we ask for 10cpus, 10G memory and 6 hours.  Really long rebuilds might need more time and or cpu/memory.

```bash
srun -c 10 --mem=10G -p parallel --time=6:00:00 --pty bash

# now on the node
module purge # just in case
module load EasyBuild

# Search for the package 
eb -S foss-2022a

# There is a long output as that toochain gets used in many packages, but we can see:
$CFGS1/f/foss/foss-2022a.eb

# Check what will be built
# BE CAUTIOUS OF OPENMPI builds - the .eb file needs to be changed to use pmi2 rather than pmix each time!
eb -Dr foss-2022a.eb

#Trigger the build - this might take a long time, you could add more cpus or time if needed
eb -r --parallel=$SLURM_CPUS_PER_TASK foss-2022a.eb

```


### Rebuilding an existing package

This might be needed for some old packages after the move to Rocky 8

Below we ask for 10cpus, 10G memory and 6 hours.  Really long rebuilds might need more time and or cpu/memory.

```bash
srun -c 10 --mem=10G -p parallel --time=6:00:00 --pty bash

# now on the node
module purge # just in case
module load EasyBuild

# Example of finding the problem
ldd /home/software/apps/samtools/1.10/bin/samtools | grep found
        libncursesw.so.5 => not found
        libtinfo.so.5 => not found

# See the what got build for samtools
eb -Dr /home/software/EasyBuild/ebfiles_repo/SAMtools/SAMtools-1.10-GCC-8.3.0.eb

# There are a few options
ncurses-6.0.eb
ncurses-6.1-GCCcore-8.3.0.eb

# let's try one
eb -r --parallel=10 --rebuild ncurses-6.0.eb

# test once done
```


### Upgrading easybuild with Easybuild

Get an interactice session on a node, then

```bash
module load EasyBuild # will load latest version by default

eb --version # see version

eb --install-latest-eb-release  # upgrade - will create new module file for new version
```

## Building New Version of Schrodinger Suite

Schrödinger Suite releases new versions quarterly, it's good practice to keep up to date with the latest version of the software. To build the new version, first download the tar file from the Schrödinger website (www.schrodinger.com), then move the installation tar file to the directory `/home/software/src/Schrodinger` on Rāpoi.

### Quick Installation 

First, extract the tar file
```bash
tar -xvf Schrodinger_Download.tar
```

Change to the top-level directory of the download
```bash
cd Schrodinger_Download
```

Then, run the instalaation script
```bash
sh ./INSTALL
```

Answer y/n to prompts from the INSTALL script, then all packages should be installed.

**NOTE**
During installation, you will be asked to confirm the `installation directory`, this is `/home/software/apps/Schrodinger/2023-3`, '2023-3' should be replaced with the current version being installed. The `scratch directory` should be `/nfs/scratch/projects/tmp`.

The installation file will check for dependencies in the last stage, missing dependencies will be reported, and will need to be installed for Schrödinger Suite to run properly. Contact Rāpoi admin to install the missing dependencies.

### Modify the hosts file
Change directory to the installation folder

```bash
cd /home/software/apps/Schrodinger/2023-3
```

open the `schrodinger.hosts`file with `vi`, modify the contents to add hostnames. The hosts and settings can be found in the `schrodinger.hosts` file from the installation directory of older versions, such as `/home/software/apps/Schrodinger/2023-1`. Add all the remote hosts to the new host file. For example,

```bash
Name: parallel
Host: raapoi-login
Queue: SLURM2.1
Qargs: "-p parallel --mem-per-cpu=2G --time=5-00:00 --constraint=AMD"
processors: 1608
tmpdir: /nfs/scratch/projects/tmp
```

### Add new module file
Once installation is complete, add a new module file so that the new version can be loaded. Module files for existing Schrodinger versions can be found in `/home/software/tools/eb_modulefiles/all/Core/Schrodinger`. The module files are named with `.lua` extensions. Make a new module file by copying one of the older module files, for example,

```bash
cp 2023-1.lua 2023-3.lua
```

Then edit the new module file (in this case, `2023-3.lua`) to match the new version installed. Fields that will need to be updated include the `Whatis` section, and the `root`. For example:

```bash
local root="/home/software/apps/Schrodinger/2023-3"
```

You can check if the module has been properly installed by
```bash
module --ignore_cache avail
```
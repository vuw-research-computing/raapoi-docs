# Running Jobs
## Job Basics

_Rāpoi_ uses a scheduler and resource manager called Slurm that requires researchers to submit jobs for processing.  There are 2 main types of jobs: _batch_ and _interactive_.  More details about submitting these types of jobs are below, but in general interactive jobs allow a user to interact with the application, for example a researcher can start a MATLAB session and can type MATLAB commands at a prompt or within a GUI.  Batch jobs can work in the background and require no user interaction, they will start when resources are available and can be configured to email once a job completes.

### Job resources

Jobs require resources.  Basic resources are CPU, memory (aka RAM) and time.  If the researcher does not specify the number of CPUs, RAM and time, the defaults will be given (currently 2CPU, 2 GB RAM and 1 hour of runtime.)  Details on requesting the basic resources are included in the Batch and Interactive sections below.

Along with basic resources there can be other resources defined, such as GPU, license tokens, or even specific types of CPUs and CPU instruction sets.  Special resources can be requested using the parameters `--gres` or `--constraint`  For example, to request an Intel processor one can use the parameter: `--constraint="Intel"`

### Currently defined constraints

Below is a list of constraints that have need defined and a brief description:

* AMD - AMD processor
* IB - Infiniband network for tightly coupled and MPI processing
* Intel - Intel processor
* 10GE - 10 Gigabit Ethernet
* SSE41 - Streaming SIMD Extensions version 4.1
* AVX - Advanced Vector Extensions

For example, if you want to request a compute node with AMD processors you can add `--constraint="AMD"` in your submit script or srun request.

---


## Interactive jobs

One of the basic job submittal tools is the command _**srun**_. It is a useful command that let's us _**a)**_ work on a compute node (_a.k.a_ request an _interactive_ _session_) and _**b)**_ run your program written in _python_, _R_, etc.


Interactive sessions will take me to one of the compute nodes where I can perform tasks such as writing my program, debugging, and even data transfer.

_**a) To request an interactive session:**_
```
username@raapoi-login:~$ srun --pty bash
```
Depending on the node assigned by the scheduler (in this case - _amd01n01_), the prompt (_raapoi-login_) will change to:

```bash
username@amd01n01:~$ #start_your_applications_here
```


!!! tip
    You could playaround with _flags_ for the _srun_ command, e.g., `srun -J<Job_Name> --cpus-per-task=2 --mem=4G --time=0-00:10:00 --pty bash`. 


_**b) To run a small quick program**_


I can use `hello.py` program available on _Rāpoi_ to test a quick example on my own. I can then use the method to run my other programs that finish under 5 hours. 

```
 username@raapoi-login:~$ srun --mem=100M --time=00:10:00 --partition=quicktest python3 
 /home/software/tools/examples/python_venv/hello.py
```

To run my own python program, I can do: 

``` bash
 username@raapoi-login:~$ srun --mem=100M --time=00:10:00 --partition=quicktest python3 
 <your_program_name>
```


In case I use a programming language other than python, the method is slightly different. For example, say I want to start a job to run an interactive R session. Once logged into the cluster I can:

```bash
module purge                         # clean/reset your environment
module load config                   # reload utilities such as vuw-job-report
module load GCC/11.2.0 OpenMPI/4.1.1 # pre-requisites for the new R module
module load R/4.2.0
srun --pty --cpus-per-task=2 --mem=2G  --time=05:00:00 --partition=quicktest R
```


!!! info
    It may take a few seconds until the prompt appears when the cluster is busy. Please wait for at least 2 mins before interreputing this step to avoid any unexpected behaviour.
    
!!! note 
    MATLAB on X server should use `matlab -softwareopengl` to run the application.
    

---

## Batch jobs

To run a batch job (aka a job that runs unattended) you use the _sbatch_ command.  A simple example would look something like this:

`sbatch myjob.sl`

In this example the sbatch command runs the file myjob.sh, the contents of this file, also known as a "batch submit script" could look something like this:

```text
#!/bin/bash
#SBATCH --cpus-per-task=2
#SBATCH --mem=2G
#SBATCH --partition=parallel
#SBATCH --constraint=AVX
#SBATCH --output=slurm-%j.out
#SBATCH --error=slurm-%j.err
#SBATCH --mail-type=BEGIN,END,FAIL
#SBATCH --mail-user=me@email.com

module load python/3.6.8
python3 project1.py
```

This will request 2 CPUs and 4GB of memory (2GB per CPU) and a runtime of 3 days
12 hours.  We are requesting that this job be run on the parallel  partition, it
will then load the environment module for python version 3.6.3 and run a python
script called project1.py.  Any output from the script will be placed in your
home directory in a file named project1.out and any error information in a file called project1.err.  If you do not specify an output or error file, the default files will have the form of Slurm-jobID.o and Slurm-jobID.e and will be located in the directory from which you ran _sbatch_.

NOTE:  We have this example script available to copy on the cluster, you can type the following to copy it to your home directory:

`cp /home/software/tools/examples/batch/myjob.sh ~/myjob.sl`

The ~/ in front of the file is a short-cut to your home directory path.  You will want to edit this file accordingly.

For more information on the sbatch command, please use the manpages, eg: _man sbatch_


So what does this all mean?

The _module load_ commands will introduce the environment necessary to run a particular program, in this case R version 4.2.0
The _srun_ command will submit the job to the cluster.  The _srun_ command has many parameter available, some of the most common are in this example and explained below

* --pty - Required to run interactively
* --cpus-per-task=2 - requests 2 CPUs, can also use the -c flag, eg. -c 2
* --mem=2G - requests 2 GigaBytes (GB) of RAM.
* --time=08:00:00 - requests a runtime of up to 8 hours (format is DAYS-HOURS:MINUTES:SECONDS), this is important in case the cluster or partition has a limited run-time, for example if an outage window is approaching.  Keep in mind time is a resource along with CPU and Memory.  
* --partition=quicktest - requests a certain partition, in this case it requests the quicktest partition, see the [section on cluster partitions](partitions.md) for more information.
* R - the command you wish to run, this could also be matlab, python, etc. (just remember to load the module first)


For more information on the srun command, please use the manpages, eg: _man srun_



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
Please see [this section](partitions.md) of the documentation for more information on partitions.


## Example Gaussian Job Submission on HPC

Here is an example of submitting a Gaussian job on the HPC using Slurm. In this example, we will submit a Gaussian job using the `quicktest` partition, and request 1 task with 4 CPUs and 7GB of memory for a maximum run time of 1 hour. We will also load the `g16` module, which is required to run Gaussian on the HPC.

First, create a new directory and navigate to it:

```bash
mkdir gaussian_example
cd gaussian_example
```

### Get the example input file

The `test0397.com` file is an example input file for Gaussian. It contains instructions for Gaussian to perform a calculation on a molecule.

To run the example job using this input file, you should copy the `test0397.com` file from the Gaussian installation directory at `/home/software/apps/gaussian/g16/tests/com/test0397.com` to your working directory (`gaussian_example` in this case). 

To do that from the `gaussian_example` directory:
```bash
cp /home/software/apps/gaussian/g16/tests/com/test0397.com . # copy from location to . The dot means current directory
```

Have a look at the first few lines of the input file to see what it does.

```bash
head test0397.com  # the first 5 lines of test0397.com

#returns
!%nproc=4
#p rb3lyp/3-21g force test scf=novaracc

Gaussian Test Job 397:
Valinomycin force

0,1
O,-1.3754834437,-2.5956821046,3.7664927822
O,-0.3728418073,-0.530460483,3.8840401686
O,2.3301890394,0.5231526187,1.7996834334
```

The first line `!%nproc=4` specifies the number of processors that Gaussian will use to run the calculation, in this case, 4.

We will need to make sure that the number of processes used in this file matches the number of cpus we request from Slurm

### Slurm Submission

Next, create a submission script called `submit.sh` (using nano or similar) and add the following contents:

```bash
#!/bin/sh
#SBATCH --job-name=g16-test

# max run time
#SBATCH --time=1:00:00
#SBATCH --partition=quicktest

#SBATCH --output=_quicktest.out
#SBATCH --error=_quicktest.err

#SBATCH --ntasks=1
#SBATCH --cpus-per-task=4
#SBATCH --mem=7G

module load gaussian/g16

g16 test0397.com
```

In the submission script, we specify the following:

- `--job-name`: name of the job to appear in the queue.
- `--time`: maximum runtime for the job in `hh:mm:ss` format.
- `--partition`: the partition to run the job on.
- `--output`: specifies the name of the standard output file.
- `--error`: specifies the name of the standard error file.
- `--ntasks`: specifies the number of tasks the job will use.
- `--cpus-per-task`: specifies the number of CPUs per task.
- `--mem`: specifies the amount of memory to allocate for the job.

Submit the job to the queue using `sbatch`:

```bash
sbatch submit.sh
```

You can check the status of your job in the queue using `squeue`:

```bash
squeue -u <your_username>
```

Once the job is finished, you can check for the output files and see the contents of the standard output file using `cat`:

```bash
ls
cat _quicktest.out
```

The Gaussian output files (test0397.log, test0397.chk, etc.) will also be generated in the working directory. You can view the output file using the less command:

```bash
less test0397.log
```

Press `q` to Quit the less program

That's it! You have successfully submitted and run a Gaussian job on a HPC cluster using Slurm.
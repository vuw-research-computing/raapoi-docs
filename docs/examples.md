
# Examples

## Simple Bash Example - start here if new to HPC

In this example we will run a very simple bash script on the quicktest partition.  The bash script is very simple, it just prints the hostname - the node you're running on - and prints the date into a file.  It also sleeps for 1 minute - it just does this to give you a chance to see your job in the queue with ```squeue```

First lets create a sensible working directory

```bash
mkdir bash_example
cd bash_example
```
We'll use the text editor nano to create our bash script as well as our submission script.  In real life, you might find it easier to create your code and submission script on your local machine, then copy them over as nano is not a great editor for large projects.

Create and edit our simple bash script - this is our code we will run on the HPC
```bash
nano test.sh
```

Paste or type the following into the file
```bash
#!/bin/bash

hostname  #prints the host name to the terminal
date > date_when_job_ran.txt  #puts the content of the date command into a txt file
sleep 1m # do nothing for 1 minute.  Job will still be "running" 
```
press ctrl-O to save the text in nano, then ctrl-X to exit nano.

Using nano again create a file called submit.sh with the following content
```bash
#!/bin/bash
#
#SBATCH --job-name=bash_test
#SBATCH -o bash_test.out
#SBATCH -e bash_test.err
#
#SBATCH --partition=quicktest
#
#SBATCH --cpus-per-task=2 #Note: you are always allocated an even number of cpus
#SBATCH --mem=1G
#SBATCH --time=10:00

bash test.sh  #actually run our bash script, using bash
```

If you're familiar with bash scripts, the above is a bit weird.  The ```#SBATCH``` lines would normally be comments and hence not do anything, but Slurm will read those lines to determine how many resources to provide your job.  In this case we ask for the following:

 * quicktest partition (the default - so you don't technically need to ask for it). 
 * 1 cpu per task - we have one task, so we're asking for 1 cpu
 * 1 gig of memory.
 * a max runtime of 10 min

If your job uses more memory or time than requested, Slurm will immediately kill it.  If you use more CPU's than requested - your job will keep running, but your "cpus" will be shared bewteen the CPUs you actually requested. So if your job tried to use 10 CPUs but you only asked for one, it'll run extremely slowly - don't do this.

Our ```submit.sh``` script also names our job ```bash_test``` this is what the job will show up as in squeue. We ask for things printed out on the terminal to go to two seperate files.  Normal, non error, things that would be printed out on the terminal will be put into the text file ```bash_test.out```.  Errors will be printed into the text file ```bash_test.err```

Now submit your job to the Slurm queue.
```bash
sbatch submit.sh  

#See your job in the queue
squeue -u <your_username>

#When job is done see the new files
ls

#look at the content that would have been printed to the terminal if running locally
cat bash_test.out

# See the content of the file that your bash script created
cat date_when_job_ran.txt
```






{%
include-markdown "examples/Python_users_guide.md"
%}






{%
include-markdown "examples/R_users_guide.md"
%}






## Matlab GPU example

Matlab has various built-in routines which are GPU accelerated.  We will run a simple speed comparison between cpu and gpu tasks. In a sensible location create a file called ```matlab_gpu.m```  I used ```~/examples/matlab/cuda/matlab_gpu.m```.

```matlab
% Set an array which will calculate the Eigenvalues of
A=rand(1000);

% Copy the Array to the GPU memory - this process takes an erratic amount of time, so we will not time it.
Agpu=gpuArray(A);
tic
B=eig(Agpu);
t1=toc

% Let's compare the time with CPU
tic
B=eig(A);
t2=toc
```

We will also need a Slurm submission script; we'll call this ```matlab_gpu.sh```. Note that we will need to use the new Easybuild module files for our cuda libraries, so make sure to include the module use line ```module use /home/software/tools/eb_modulefiles/all/Core```

```bash
#!/bin/bash

#SBATCH --job-name=matlab-gpu-example
#SBATCH --output=out-gpu-example.out
#SBATCH --error=out-gpu-example.err
#SBATCH --time=00:05:00
#SBATCH --partition=gpu
#SBATCH --gres=gpu:1
#SBATCH --ntasks=2
#SBATCH --mem=60G

module use /home/software/tools/eb_modulefiles/all/Core
module load MATLAB/2024a
module load fosscuda/2020b

matlab -nodisplay -nosplash -nodesktop -r "run('matlab_gpu.m');exit;"
```

To submit this job to the Slurm queue ```sbatch matlab_gpu.sh```.  This job will take a few minutes to run - this is mostly the Matlab startup time.
Examine the queue for your job ```squeue -u $USER```.  When your job is done, inspect the output file.  You can use an editor like nano, vi or emacs, or you can just ```cat``` or ```less``` the file to see its contents on the terminal.

```bash
cat out-gpu-example.out
```
What do you notice about the output?  Surely GPUs should be faster than the CPU!  It takes time for the GPU to start processing your task, the CPU is able to start the task far more quickly.  So for short operations, the CPU can be faster than the GPU - remember to benchmark your code for optimal performance!  Just because you can use a GPU for your task doesn't mean it is necessarily faster!

To get a better idea of the advantage of the GPU let's increase the size of the array from ```1000``` to ```10000```

*matlab_gpu.m*
```Matlab
% Set an array which will calculate the Eigenvalues of
A=rand(10000);

% Copy the Array to the GPU memory - this process takes an erratic amount of time, so we will not time it.
Agpu=gpuArray(A);
tic
B=eig(Agpu);
t1=toc

% Let's compare the time with CPU
tic
B=eig(A);
t2=toc
```

To make things fairer for the CPU in this case, we will also allocate half the CPUs on the node to Matlab.  Half the CPUs, half the memory and half the GPUs, just to be fair.

*matlab_gpu.sh*
```
#!/bin/bash

#SBATCH --job-name=matlab-gpu-example
#SBATCH --output=out-gpu-example.out
#SBATCH --error=out-gpu-example.err
#SBATCH --time=00:05:00
#SBATCH --partition=gpu
#SBATCH --gres=gpu:1
#SBATCH --ntasks=128
#SBATCH --mem=256G

module use /home/software/tools/eb_modulefiles/all/Core
module load MATLAB/2024a
module load fosscuda/2020b

matlab -nodisplay -nosplash -nodesktop -r "run('matlab_gpu.m');exit;"
```

The output in my case was:
```bash
                            < M A T L A B (R) >
                  Copyright 1984-2024 The MathWorks, Inc.
                  R2024a (24.1.0.2537033) 64-bit (glnxa64)
                             February 21, 2024


For online documentation, see https://www.mathworks.com/support
For product information, visit www.mathworks.com.

 

t1 =

   62.0212


t2 =

  223.0818
```

So in thise case the GPU was considerably faster.  Matlab can do this a bit faster on the CPU if you give it **fewer** CPUs, the optimum appears to be around 20, but it still takes 177s.  Again, optimise your resource requests for your problem, less can sometimes be more, however the GPU easily wins  in this case.





## Job Arrays - running many similar jobs

Slurm makes it easy to run many jobs which are similar to each other.  This could be one piece of code running over many datasets in parallel or running a set of simulations with a different set of parameters for each run.

### Simple Bash Job Array example

The following code will run the submission script 16 times as resources become available (i.e. they will not neccesarily run at the same time).  It will just print out the Slurm array task ID and exit.

*submit.sh:*
```bash
#!/bin/bash

#SBATCH --job-name=test_array
#SBATCH --output=out_array_%A_%a.out
#SBATCH --error=out_array_%A_%a.err
#SBATCH --array=1-16
#SBATCH --time=00:00:20
#SBATCH --partition=parallel
#SBATCH --ntasks=1
#SBATCH --mem=1G

# Print the task id.
echo "My SLURM_ARRAY_TASK_ID: " $SLURM_ARRAY_TASK_ID

# Add lines here to run your computations.
```

Run the example with the standard
```bash
sbatch submit.sh
```

### A simple R job Array Example

As a slightly more practical example the following will run an R script 5 times as resources become available.  The R script takes as an input the ```$SLURM_ARRAY_TASK_ID``` which then selects a parameter ```alpha``` out of a lookup table.

This is one way you could run simulations or similar with a set parameters defined in a lookuop table in your code.

To make outputs more tidy and to help organisation, instead of dumping all the outputs into the directory with our code and submission script, we will separate the outputs into directories.  Dataframes saved from R will be saved to the output/ directory, and all output which would otherwise be printed to the commnd line (stdout and stderr) will be saved to the stdout/ directory. Both of these directories will need to be created before running the script.

*r_random_alpha.R:*
```R
# get the arguments supplied to R.  
# trailingOnly = TRUE gets the user supplied
# arguments, and for now we will only get the
# first user supplied argument
args <- commandArgs(trailingOnly = TRUE)
inputparam <- args[1]

# a vector with all our parameters.
alpha_vec <- c(2.5, 3.3, 5.1, 8.2, 10.9)
alpha <- alpha_vec[as.integer(inputparam)]

# Generate a random number between 0 and alpha 
# store it in dataframe with the coresponding 
# alpha value
randomnum <- runif(1, min=0, max=as.double(alpha))
df <- data.frame("alpha" = alpha, "random_num" = randomnum)

# Save the data frame to a file with the alpha value
# Note that the output/ folder will need to be 
# manually created first!
outputname <- paste("output/", "alpha_", alpha, ".Rda", sep="")
save(df,file=outputname)
```

Next create the submision script. Which we will run on the parallel partition rather than quicktest.

r_submit.sh:
```bash
#!/bin/bash
#SBATCH --job-name=test_R_array
#SBATCH --output=stdout/array_%A_%a.out
#SBATCH --error=stdout/array_%A_%a.err
#SBATCH --array=1-5
#SBATCH --time=00:00:20
#SBATCH --partition=parallel
#SBATCH --ntasks=1
#SBATCH --mem=1G

module purge
module load GCC/11.2.0 OpenMPI/4.1.1
module load R/4.2.0

# Print the task id.
Rscript r_random_alpha.R $SLURM_ARRAY_TASK_ID
```

Run the jobs with
```bash
sbatch r_submit.sh
```







{%
include-markdown "examples/Singularity.md"
%}




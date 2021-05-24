
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
#SBATCH --cpus-per-task=1
#SBATCH --mem=1G
#SBATCH --time=10:00

bash test.sh  #actually run our bash script, using bash
```

If you're familiar with bash scripts, the above is a bit weird.  The ```#SBATCH``` lines would normally be comments and hence not do anything, but Slurm will read those lines to determine how many resources to provide your job.  In this case we ask for the following:

 * quicktest partition (the default - so you don't technically need to ask for it). 
 * 1 cpu per task - we have one task, so we're asking for 1 cpu
 * 1 gig of memory.
 * a max runtime of 10 min

If your job uses more memory or time than requested, slurm will immediately kill it.  If you use more CPU's than requested - your job will keep running, but your "cpus" will be shared bewteen the CPUs you actually requested. So if your job tried to use 10 CPUs but you only asked for one, it'll run extremely slowly - don't do this.

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


## Simple Python program using virtualenv and pip

First we need to create a working directory and move there
```bash
mkdir python_test
cd python_test
```
Next we load the python 3 module and use python 3 to create a python virtualenv.  This way we can install pip packages which are not installed on the cluster
```bash
module load python/3.6.6
python3 -m venv mytest
```

Activate the `mytest` virtualenv and use pip to install the `webcolors` package
```bash
source mytest/bin/activate
pip install webcolors
```

Create the file test.py with the following contents using nano
```python
import webcolors
from random import randint
from socket import gethostname

colour_list = list(webcolors.css3_hex_to_names.items())
requested_colour = randint(0,len(colour_list))
colour_name = colour_list[requested_colour][1]

print("Random colour name:", colour_name, " on host: ", gethostname())
```

Alternatively download it with wget:
```bash
wget https://raw.githubusercontent.com/\
    vuw-research-computing/raapoi-tools/\
    master/examples/python_venv/test.py
```

Using nano create the submissions script called python_submit.sh with the following content - change `me@email.com` to your email address.
```bash
#!/bin/bash
#
#SBATCH --job-name=python_test
#SBATCH -o python_test.out
#SBATCH -e python_test.err
#
#SBATCH --cpus-per-task=1
#SBATCH --mem-per-cpu=1G
#SBATCH --time=10:00
#
#SBATCH --mail-type=BEGIN,END,FAIL
#SBATCH --mail-user=me@email.com

module load python/3.6.6

source mytest/bin/activate
python test.py
```

Alternatively download it with wget
```bash
wget https://raw.githubusercontent.com/\
    vuw-research-computing/raapoi-tools/\
    master/examples/python_venv/python_submit.sh
```

To submit your job to the Slurm scheduler
```bash
sbatch python_submit.sh
```

Check for your job on the queue with `squeue` though it might finish very fast.  The output files will appear in your working directory.


## Loading R packages & running a simple job

First login to Rāpoi and load the R/CRAN module:
```bash
module load R/CRAN      
```
(Note this will also load ```R/3.6```)


Then run R on the command line:


```bash
R
```

Test library existence:
```R
library(ggplot2)
```
This should load the package.
Metapackages like ```tidyverse``` currently don't load on Rāpoi, but components can be loaded individually: 

```R
> library(tidyr)
> library(dplyr)
```

Next create a bash submission script using your preferred text editor. An example submission script may look something like:
a file called r_submit.sh with:
```bash
#!/bin/bash
#
#SBATCH --job-name=r_test
#SBATCH -o r_test.out
#SBATCH -e r_test.err
#
#SBATCH --cpus-per-task=1
#SBATCH --mem-per-cpu=1G
#SBATCH --time=10:00
#
```

Save this to the current working directory, then run: 

```bash
module load R/CRAN
```

```bash
Rscript mytest.R
```


and then create another file with a test R script called ```mytest.R``` with:

```R
library(tidyr)
library(dplyr)
library(ggplot2)

sprintf("Hello World!")
```
then run it with the previously written bash script:  
```bash
sbatch r_submit.sh 
```
This submits a task that should execute quickly and create files in the directory from which it was run.
Examining ```r_test.out``` (with nano, cat or less) should print:
``` "Hello World"```

## Job Arrays - running many similar jobs

Slurm makes it easy to run many jobs which are similar to each other.  This could be one piece of code running over many datasets in parallel or running a set of simulations with a different set of parameters for each run.

### Simple Bash Job Array example

The following code will run the submission script 16 times as resources become available (i.e. they will not neccesarily run at the same time).  It will just print out the slurm array task ID and exit.

submit.sh:
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

To make outputs more tidy and to help organisation, instead of dumping all the outputs in to the directory with our code and submission script, we will seperate the outputs into directories.  Dataframes saved from R will be saved to output/ and all output which would otherwise be printed to the commnd line (stdout and stderr) will be saved to stdout/  Both of these directories will need to be created before running the script.

r_random_alpha.R:
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

module load R/CRAN

# Print the task id.
Rscript r_random_alpha.R $SLURM_ARRAY_TASK_ID
```

Run the jobs with
```bash
sbatch r_submit.sh
```

## Singularity

While there are many modules on Rāpoi, sometimes you might want to install your own packages in your own way.  Singularity allows you to do this.  If you are familiar with Docker, Singularity is similar, except you can't get root (or sudo) once your container is running on the Rāpoi.  However, you *can* have sudo rights locally on your own machine, setup your container however you like, then run it without sudo on the cluster.

### Singularity/Docker container example

Singularity allows you to use most (but not all!) docker images on Rāpoi.

On your local machine create the singularity definition file

input_args_example.def
```singularity
BootStrap: library
From: ubuntu:16.04

%runscript
    exec echo "$@"

%labels
    Author Andre
```

This will build an ubuntu 16.04 container that will eventually run on Rāpoi which runs Centos.  This container has a runscript which just echos back any arguments sent to the container when your start it up.

Build the container *locally* with sudo and singularity
```bash
sudo singularity build inputexample.sif input_args_example.def 
```

This will build an image that you can't modify any further and is immediately suitable to run on Rāpoi
Copy this file to Rāpoi via sftp
```bash
sftp <username>@raapoi.vuw.ac.nz
```

Create a submit script using singularity on the cluster

singularity_submit.sh
```bash
#!/bin/bash

#SBATCH --job-name=singularity_test
#SBATCH -o sing_test.out
#SBATCH -e sing_test.err
#SBATCH --time=00:00:20
#SBATCH --ntasks=1
#SBATCH --mem=1G

module load singularity

singularity run inputtest.sif "hello from a container"
```

Run the script with the usual
```bash
singularity_submit.sh
```

### Singularity/TensorFlow Example

tensor.def
```bash
Bootstrap: docker
From: tensorflow/tensorflow:latest-py3

%post
apt-get update && apt-get -y install wget build-essential 

%runscript
    exec python "$@"

```

compile this *locally* with sudo and singularity.
```bash
sudo singularity build tensorflow.sif tensor.def 
```

Create a quick tensorflow test code
tensortest.py
```python
import tensorflow as tf
mnist = tf.keras.datasets.mnist

(x_train, y_train),(x_test, y_test) = mnist.load_data()
x_train, x_test = x_train / 255.0, x_test / 255.0

model = tf.keras.models.Sequential([
  tf.keras.layers.Flatten(input_shape=(28, 28)),
  tf.keras.layers.Dense(512, activation=tf.nn.relu),
  tf.keras.layers.Dropout(0.2),
  tf.keras.layers.Dense(10, activation=tf.nn.softmax)
])
model.compile(optimizer='adam',
              loss='sparse_categorical_crossentropy',
              metrics=['accuracy'])

model.fit(x_train, y_train, epochs=5)
model.evaluate(x_test, y_test)
```

Copy your files to Rāpoi via sftp (or whatever you prefer)
```bash
sftp <username>@raapoi.vuw.ac.nz
cd <where you want to work>
put *   #put all files in your local directory onto Rāpoi
```

Lets quickly test the code via an interactive session on a node.  Note I find the tensorflow container only runs properly on intel nodes, which we don't have many of at the moment, I'll investigate this further.
```bash
srun --partition="parallel" --constraint="Intel" --pty bash

#now on the remote node - note you might need to wait if nodes are busy
module load singularity #load singularity
singularity shell tensorflow.sif 

#now inside the tensorflow container on the remote node
python tensortest.py 

#once that runs, exit the container
exit #exit the container
exit #exit the interactive session on the node
```



Create a submit script using singularity on the cluster

singularity_submit.sh
```bash
#!/bin/bash

#SBATCH --job-name=singularity_test
#SBATCH -o sing_test.out
#SBATCH -e sing_test.err
#SBATCH --time=00:10:00
#SBATCH --partition=parallel
#SBATCH --constraint=Intel
#SBATCH --ntasks=1
#SBATCH --mem=4G

module load singularity

#run the container with the runscript defined when we created it
singularity run tensorflow.sif tensortest.py 
```

### Singularity/MaxBin2 Example
In a sensible location, either in your home directory or on the scratch:

Get the maxbin2 container, there are a few places to get this, but will get the bioconda container as it is more recent than the one referenced on the official maxbin site.

```bash
module load module load singularity
singularity pull docker://quay.io/biocontainers/maxbin2:2.2.6--h14c3975_0
mv maxbin2_2.2.6--h14c3975_0.sif maxbin2_2.2.6.sif #rename for convenience
```

Download some test data

```bash
mkdir rawdata
curl https://downloads.jbei.org/data/microbial_communities/MaxBin/getfile.php?20x.scaffold > rawdata/20x.scaffold
curl https://downloads.jbei.org/data/microbial_communities/MaxBin/getfile.php?20x.abund > rawdata/20x.abund
```

Create an output data location
```bash
mkdir output
```



Create a submit script using singularity on the cluster

singularity_submit.sh
```bash
#!/bin/bash

#SBATCH --job-name=maxbin2_test
#SBATCH -o sing_test.out
#SBATCH -e sing_test.err
#SBATCH --time=00:10:00
#SBATCH --partition=parallel
#SBATCH --ntasks=4
#SBATCH --mem=4G

module load singularity

singularity exec maxbin2_2.2.6.sif run_MaxBin.pl -contig rawdata/20x.scaffold -abund rawdata/20x.abund -out output/20x.out -thread 4 
```

### Singularity/Sandbox Example

This lets you have root inside a container *locally* and make changes to it.  This is really handy for determining how to setuop your container.  While you can convert the sandbox container to one you can run on Rāpoi, I suggest you *don't do this*. Use the sandbox to figure out how you need to configure your container, what packages to install, config files to change etc. Then create a ```.def``` file that contains all the nessesary steps without the need to use the sandbox - this will make your work more reproducable and easier to share with others.


example.def
```bash
BootStrap: library
From: ubuntu:16.04

%post
apt-get update && apt-get -y install wget build-essential 

%runscript
    exec echo "$@"

%labels
    Author Andre
    
```

Compile this *locally* with sudo and singularity.  We are using the sandbox flag to create a writable *container directory* (```example/```) on our local machine where we have sudo rights.
```bash
sudo singularity build --sandbox example/ example.def
```

Now we can run the container we just built, but with sudo rights inside the container.  Your rights outside the container match the rights inside the container, so we need to do this with sudo.

```bash
sudo singularity shell --writable example/
```

Inside the container we now have root and can install packages and modify files in the root directories
```bash
Singularity example:~>  apt update
Singularity example:~>  apt install sqlite
Singularity example:~>  touch /test.txt  #create an empty file in root
Singularity example:~>  ls /
Singularity example:~>  exit   #exit container
```

To run the container on Rāpoi we convert it to the default immutable image with build.  We might need sudo for this as the prior use of sudo will have created a directory that your usual user can't see every file.

```bash
sudo singularity build new-example-sif example/
```
You could now copy the ```new-example-sif``` file to Rāpoi and run it there.  However a better workflow is to use this to experiment, to find out what changes you need to make to the image and what packages you need to install.  Once you've done that, I suggest starting afresh and putting *everything in the.def file*.  That way when you return to your project in 6 months, or hand it over to someone else, there is a clear record of how the image was built.

### Singularity/Custom Conda Container - idba example

In this example we'll build a singularity container using conda.  The example is building a container for idba - a genome assembler.  Idba is available in bioconda, but not as a bicontainer.  We'll build this container locally to match a local conda enviroment, then run it on the HPC and do an example assembly.

#### Locally

Make sure you have conda setup on your local machine, anaconda and miniconda are good choices.  Create a new conda enviroment and install idba

```bash
conda create --name idba
conda install -c bioconda idba
```

Export your conda enviroment, we will use this to build the containter.
```bash
conda env export > environment.yml
```

We will use a singularity definition, basing our build on a docker miniconda image.  There is a bunch of stuff in this file to make sure the conda enviroment is in the path. *[From stackoverflow](https://stackoverflow.com/questions/54678805/containerize-a-conda-environment-in-a-singularity-container)*

*idba.def*
```
Bootstrap: docker

From: continuumio/miniconda3

%files
    environment.yml

%environment
    PATH=/opt/conda/envs/$(head -1 environment.yml | cut -d' ' -f2)/bin:$PATH

%post
    echo ". /opt/conda/etc/profile.d/conda.sh" >> ~/.bashrc
    echo "source activate $(head -1 environment.yml | cut -d' ' -f2)" > ~/.bashrc
    /opt/conda/bin/conda env create -f environment.yml

%runscript
    exec "$@"
```

Build the image
```bash
sudo singularity build idba.img idba.def
```

Now copy the idba.img and enviroment.yml (technically the enviroment file is not needed, but not having it creates a warning) to somewhere sensible on Rāpoi.

#### On Rāpoi

Create a data directory, so we can seperate our inputs and outputs.  Download a paired end illumina read of Ecoli from S3 with wget.  The data comes from the [Illumina public data library](https://www.illumina.com/informatics/sequencing-data-analysis/data-examples.html)
```
mkdir data
cd data 
wget --content-disposition goo.gl/JDJTaz #sequence data
wget --content-disposition goo.gl/tt9fsn #sequence data
cd ..  #back to our project directory
```

The reads we have are paired end fastq files but idba requires a fasta file.  We can use a tool built into our container to convert them.  We'll do this on the Rāpoi login node as it is a fast task that doesn't need many resources.

```bash
module load singularity
singularity exec fq2fa --merge --filter data/MiSeq_Ecoli_MG1655_50x_R1.fastq data/MiSeq_Ecoli_MG1655_50x_R2.fastq data/read.fa
```

Create our sbatch submission script.  Note that this sequence doesn't need a lot of memory, so we'll use 1G. Too see your usage after the job has run use ```vuw-job-report <job-id>```

*idba_submit.sh*
```bash
#!/bin/bash

#SBATCH --job-name=idba_test
#SBATCH -o output.out
#SBATCH -e output.err
#SBATCH --time=00:10:00
#SBATCH --partition=quicktest
#SBATCH --ntasks=12
#SBATCH --mem=1G

module load singularity

singularity exec idba.img idba idba_ud -r data/read.fa -o output
```

Now we can submit our script to the queue with
```bash
sbatch idba_submit.sh 
```
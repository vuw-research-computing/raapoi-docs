
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

## Singularity/Docker container example

While there are many modules on Rāpooi, sometimes you might want to install your own packages in your own way.  Singularity allows you to do this.  If you are familiar with Docker, Singularity is similar, except you can't get root (or sudo) once your container is running on the Rāpooi.  However, you *can* have sudo rights locally on your own machine, setup your container however you like, then run it without sudo on the cluster.

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

This will build an ubuntu 16.04 container that will eventually run on Rāpooi which runs Centos.  This container has a runscript which just echos back any arguments sent to the container when your start it up.

Build the container *locally* with sudo and singularity
```bash
sudo singularity build inputexample.sif input_args_example.def 
```

This will build an image that you can't modify any further and is immediately suitable to run on Rāpooi
Copy this file to raapoi via sftp
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

module load singularity/3.2.1

singularity run inputtest.sif "hello from a container"
```

Run the script with the usual
```bash
singularity_submit.sh
```

## Singularity/TensorFlow Example

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

Copy your files to raapoi via sftp (or whatever you prefer)
```bash
sftp <username>@raapoi.vuw.ac.nz
cd <where you want to work>
put *   #put all files in your local directory onto raapoi
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

module load singularity/3.2.1

#run the container with the runscript defined when we created it
singularity run tensorflow.sif tensortest.py 
```

## Singularity/MaxBin2 Example
In a sensible location, either in your home directory or on the scratch:

Get the maxbin2 container, there are a few places to get this, but will get the bioconda container as it is more recent than the one referenced in the official maxbin site.

```bash
module load module load singularity/3.2.1
singularity pull docker://quay.io/biocontainers/maxbin2:2.2.6--h14c3975_0
mv maxbin2_2.2.6--h14c3975_0.sif maxbin2_2.2.6.sif #rename for convenience
```

Download some test data

```bash
mkdir rawdata
curl https://downloads.jbei.org/data/microbial_communities/MaxBin/getfile.php?20x.scaffold > rawdata/20x.scaffold
curl https://downloads.jbei.org/data/microbial_communities/MaxBin/getfile.php?20x.abund > rawdata/20x.abund
```

Create and output data location
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

module load singularity/3.2.1

singularity exec maxbin2_2.2.6.sif run_MaxBin.pl -contig rawdata/20x.scaffold -abund rawdata/20x.abund -out output/20x.out -thread 4 
```
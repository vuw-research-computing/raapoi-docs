## Singularity

While there are many modules on Rāpoi, sometimes you might want to install your own packages in your own way.  Singularity allows you to do this.  If you are familiar with Docker, Singularity is similar, except you can't get root (or sudo) once your container is running on the Rāpoi.  However, you *can* have sudo rights locally on your own machine, setup your container however you like, then run it without sudo on the cluster.

See also: [Using containers](containers.md)

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

In this example we'll build a singularity container using conda.  The example is building a container for idba - a genome assembler.  Idba is available in bioconda, but not as a biocontainer.  We'll build this container locally to match a local conda environment, then run it on the HPC and do an example assembly.

#### Locally

Make sure you have conda setup on your local machine, anaconda and miniconda are good choices.  Create a new conda environment and install idba

```bash
conda create --name idba
conda install -c bioconda idba
```

Export your conda environment, we will use this to build the container.
```bash
conda env export > environment.yml
```

We will use a singularity definition, basing our build on a docker miniconda image.  There is a bunch of stuff in this file to make sure the conda environment is in the path. *[From stackoverflow](https://stackoverflow.com/questions/54678805/containerize-a-conda-environment-in-a-singularity-container)*

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

Now copy the idba.img and environment.yml (technically the environment file is not needed, but not having it creates a warning) to somewhere sensible on Rāpoi.

#### On Rāpoi

Create a data directory, so we can separate our inputs and outputs.  Download a paired end illumina read of Ecoli from S3 with wget.  The data comes from the [Illumina public data library](https://www.illumina.com/informatics/sequencing-data-analysis/data-examples.html)
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

Create our sbatch submission script.  Note that this sequence doesn't need a lot of memory, so we'll use 1G. To see your usage after the job has run use ```vuw-job-report <job-id>```

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


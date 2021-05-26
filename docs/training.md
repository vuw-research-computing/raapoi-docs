# Training

These are longer worked examples.  If you have domain specific training you'd like to provide for your students or peers, contact Andre or Wes, or make a pull request against this repo.

## GPU example with neural style in pytorch

We'll do a quick python example using neural style implimented in pytorch. We will be using modules rather than conda/virtualenvs but there is nothing stopping you from loading the modules and creating a virtualenv/conda enviroment to install additioanl python packages

The code we use will come from the pytorch example git repo.

### Clone the pytorch example repo

In a sensible location, clone the rep.

```bash
git clone https://github.com/pytorch/examples.git
cd examples/fast_neural_style  # change to the example we will be running.
```

### Load the modules

We are using the new Easybuild based modules, to ensure we don't have conflicst with the old modules, it will be best to unuse them first and then use the new system.  At somepoint we may automatically add the new modules to your bashrc file - but currently you'll have to do this yourself or manually unuse and use the new module system

```bash
module unuse /home/software/tools/modulefiles/  #unuse the old module system
module use /home/software/tools/eb_modulefiles/all/Core #use the new module system
```

```bash
module load fosscuda/2020b
module load PyTorch/1.7.1
module load torchvision/0.8.2-PyTorch-1.7.1
module list #see all the dependancies we have loaded, in particular which version of python we're using now. Currently Python 3.8.6
```

### Optional: Setup a virtualenv

```bash
python3 -m venv env  # create a virtualenv folder called env. Note! This will likely only work with the python version listed above!
source env/bin/activate # activate the virtualenv
```

Now that we've activated the virtual enviroment, we can install any additional pacakges we need.  In thise case we don't need any additional packages

### Download some images to use as content as well as for training.

In your ```examples/fast_neural_style/``` directory.

```bash
# Download an image of an octopus to images/content-images. 
## CC BY-SA 3.0 H. Zell
wget https://upload.wikimedia.org/wikipedia/commons/0/0c/Octopus_vulgaris_02.JPG -P images/content-images/ 

# Download an image of The Great Wave off Kanagawa - public domain
wget https://upload.wikimedia.org/wikipedia/commons/a/a5/Tsunami_by_hokusai_19th_century.jpg -O images/style-images/wave.jpg
```

Depending on the GPU we are using, we may need to resize the image to ensure it first in memory.  On an RTX6000 we would need to resize the image to 70% of it's full size to fit in memroy.  Thankfully the GPUs on Rāpoi are A100's with 40GB of ram, so we can skip this step.

We will also need to download the pre-trained models for our initial inference runs.
```bash
python download_saved_models.py
```

### Style some images - inference

We'll initially just use pretrained models to generate styled images - this is known as model inference and is much less intensive than training the model, we'll do this on both CPU and GPU. 

submit_cpu.sh
```bash
#!/bin/bash

#SBATCH --job-name=pytorch_test
#SBATCH -o _test.out
#SBATCH -e _test.err
#SBATCH --time=00:15:00
#SBATCH --partition=parallel
#SBATCH --ntasks=12
#SBATCH --mem=6G

module unuse /home/software/tools/modulefiles/  #unuse the old module system
module use /home/software/tools/eb_modulefiles/all/Core #use the new module system
module load fosscuda/2020b
module load PyTorch/1.7.1
module load torchvision/0.8.2-PyTorch-1.7.1

#Optional
source env/bin/activate  #activate the virtualenv

# Run our job --cuda 0 means run on the CPU and we'll save the output image as test1.jpg
#
python neural_style/neural_style.py eval --content-image images/content-images/Octopus_vulgaris_02.JPG  --model saved_models/mosaic.pth --output-image ./test1.jpg --cuda 0
```

You can check how long the job took to run with ```vuw-job-history```.  The last lines are your last run job, in my case:

```bash
332281        COMPLETED pytorch_t+              00:02:36 
332281.batch  COMPLETED      batch      0.15G   00:02:36 
332281.exte+  COMPLETED     extern      0.15G   00:02:36
```

the job took 2:36.

Let's run the inference job again on GPU to see the speedup.

submit_gpu.sh
```bash
#!/bin/bash

#SBATCH --job-name=pytorch_test
#SBATCH -o _test.out
#SBATCH -e _test.err
#SBATCH --time=00:15:00
#SBATCH --partition=gpu
#SBATCH --gres=gpu:1
#SBATCH --ntasks=2
#SBATCH --mem=60G

module unuse /home/software/tools/modulefiles/  #unuse the old module system
module use /home/software/tools/eb_modulefiles/all/Core #use the new module system
module load fosscuda/2020b
module load PyTorch/1.7.1
module load torchvision/0.8.2-PyTorch-1.7.1

#optional
source env/bin/activate  #activate the virtualenv

# Run our job --cuda 1 means run on the GPU and we'll save the output image as test2.jpg
#
python neural_style/neural_style.py eval --content-image images/content-images/Octopus_vulgaris_02.JPG  --model saved_models/mosaic.pth --output-image ./test2.jpg --cuda 1
```

In this case ```vuw-job-history``` the job took:
```bash
692973        COMPLETED pytorch_t+              00:00:16 
692973.batch  COMPLETED      batch      0.15G   00:00:16 
692973.exte+  COMPLETED     extern      0.15G   00:00:16 
```

but the time varies a lot with short GPU runs, some are nearly 2 min long and some runs are 16s with the same data. The memory usage with pytorch is also hard to estimate, running ```vuw-job-report 332320``` shows:
```bash
Nodes: 1
Cores per node: 2
CPU Utilized: 00:00:07
CPU Efficiency: 43.75% of 00:00:16 core-walltime
Job Wall-clock time: 00:00:08
Memory Utilized: 1.38 MB
Memory Efficiency: 0.00% of 60.00 GB
```

The memory usage is very low, but there is a very brief spike in memory at the end of the run as the image is generated that ```vuw-job-report``` doesn't quite capture. 60G of memory is needed to ensure this completes - a good rule of thumb is to allocate at least as much system memory as GPU memory.  The A100's have 40G of ram.

### Train a new style - computationally expensive.

Training a new image style is where we will get the greatest speedup using a GPU.

We will use 13G of training images - [COCO 2014 Training images dataset ](http://cocodataset.org/#download). These images have already been downloaded and are accessable at ```/nfs/home/training/neural_style_data/train2014/```.  Note that training a new style will take about 1:15h on an A100 and two and a half hours on an RTX6000

```bash
#!/bin/bash

#SBATCH --job-name=pytorch_test
#SBATCH -o _test.out
#SBATCH -e _test.err
#SBATCH --time=03:00:00
#SBATCH --partition=gpu
#SBATCH --gres=gpu:1
#SBATCH --ntasks=2
#SBATCH --mem=60G

module unuse /home/software/tools/modulefiles/  #unuse the old module system
module use /home/software/tools/eb_modulefiles/all/Core #use the new module system
module load fosscuda/2020b
module load PyTorch/1.7.1
module load torchvision/0.8.2-PyTorch-1.7.1

#Optional
source env/bin/activate  #activate the virtualenv

# Run our job --cuda 1 means run on the GPU                                   
# style-weight and content-weight are just parameters adjusted to give better results
python neural_style/neural_style.py train \
        --dataset /nfs/home/training/neural_style_data/ \
        --style-image images/style-images/wave.jpg \
        --save-model-dir saved_models/style5e10_content_5e4 \
        --style-weight 5e10 \
        --content-weight 5e4 \
        --epochs 2 \
        --cuda 1
```

This will take a while, but should eventually complete. The A100 has enough memory to train on this image, with other GPUs you may need to scale down the style image to fit in the GPU memory.  Note: If you get an out of GPU memory error but it seems the GPU ha plenty of memory, it often means you ran out of system memory, try asking for more memory in slurm.

### Use our newly trained network


submit_gpu.sh
```bash
#!/bin/bash

#SBATCH --job-name=pytorch_test
#SBATCH -o _test.out
#SBATCH -e _test.err
#SBATCH --time=00:15:00
#SBATCH --partition=gpu
#SBATCH --gres=gpu:1
#SBATCH --ntasks=2
#SBATCH --mem=60G

module unuse /home/software/tools/modulefiles/  #unuse the old module system
module use /home/software/tools/eb_modulefiles/all/Core #use the new module system
module load fosscuda/2020b
module load PyTorch/1.7.1
module load torchvision/0.8.2-PyTorch-1.7.1

#Optional
source env/bin/activate  #activate the virtualenv

# Run our job --cuda 1 means run on the GPU and we'll save the output image as test2.jpg
#
python neural_style/neural_style.py eval \
    --content-image images/content-images/Octopus_vulgaris_02.JPG  \
    --model saved_models/style5e10_content_5e4 \
    --output-image ./test3.jpg --cuda 1
```

### Bonus content use a slurm task-array to find the optimum parameters.

In the above example we use parameters for style-weight and content-weight.  There are lots of possibilities for these parameters, we can use a task array and a parameter list to determine good values.   Note that actually running this example will consume a lot of resources and it is presented mostly to provide some information about task arrays.  Running this example will consume the whole GPU partition for about 12 hours.

First let's create a list of parameters to test, we could include these in the batch submision script, but I think it's clearer to seperate them out. If you're version controlling your submission script, it'll make it easier to see what are changes to parameters and what are changes to the script itself.

In the parameter list, the first column is style-weight parameters and the second is content-weight parameters
paramlist.txt
```bash
5e10 1e3
5e10 1e4
5e10 5e4
1e11 1e3
1e11 1e4
1e11 5e4
5e11 1e3
5e11 1e4
5e11 5e4
1e12 1e3
1e12 1e4
1e12 5e4
```

In our submision script we will parse these values with ```awk```.  Awk is a bit beyond the scope of this lesson, but it is handy shell tool for manipulating text. [Digital ocean has a nice primer on Akw](https://www.digitalocean.com/community/tutorials/how-to-use-the-awk-language-to-manipulate-text-in-linux)

submit_gpu_train_array
```bash
#!/bin/bash

#SBATCH --job-name=pytorch_test
#SBATCH -o _test.out
#SBATCH -e _test.err
#SBATCH --time=10:00:00
#SBATCH --partition=gpu
#SBATCH --gres=gpu:1
#SBATCH --ntasks=10
#SBATCH --mem=60G
#SBATCH --array=1-13

module unuse /home/software/tools/modulefiles/  #unuse the old module system
module use /home/software/tools/eb_modulefiles/all/Core #use the new module system
module load fosscuda/2020b
module load PyTorch/1.7.1
module load torchvision/0.8.2-PyTorch-1.7.1

#Optional
source env/bin/activate  #activate the virtualenv

# Run our job --cuda 1 means run on the GPU                                   
#
#awk -v var="$SLURM_ARRAY_TASK_ID" 'NR == var {print $1}' paramlist.txt 
style_weight=$(awk -v var="$SLURM_ARRAY_TASK_ID" 'NR == var {print $1}' paramlist.txt)
content_weight=$(awk -v var="$SLURM_ARRAY_TASK_ID" 'NR == var {print $2}' paramlist.txt)

echo $style_weight
echo $content_weight
python neural_style/neural_style.py train \
	--dataset nfs/home/training/neural_style_data/ \
	--style-image images/style-images/wave.jpg \
	--save-model-dir saved_models/test_params2_epoch2/style${style_weight}_content${content_weight} \
	--style-weight $style_weight \
	--content-weight $content_weight \
	--epochs 2 \
	--cuda 1
```

## Simple OpenMPI with Singularity using the hybrid approach.

The hybrid approach is one way of getting OpenMPI working with containers. It requires the OpenMPI version inside the container to match the OpenMPI outside the container (loaded via module loading)

First check what openMPI version we have on Rāpoi
cat 
On **Rāpoi** switch to our new modules
```bash
module unuse /home/software/tools/modulefiles # stop using the older modules
module use /home/software/tools/eb_modulefiles/all/Core #the new module files organised by compiler
module spider OpenMPI # search for openMPI - thre are several options, lets try
module spider OpenMPI/4.0.5  # we will use this one, which requires GCC/10.2.0
```

On your **local machine** we will create a very simple C openMPI program. Create this in a  sensible place.  I used ```~/projects/examples/singularity/openMPI```

```c
#include <mpi.h>
#include <stdio.h>
#include <stdlib.h>

int main (int argc, char **argv) {
        int rc;
        int size;
        int myrank;

        rc = MPI_Init (&argc, &argv);
        if (rc != MPI_SUCCESS) {
                fprintf (stderr, "MPI_Init() failed");
                return EXIT_FAILURE;
        }

        rc = MPI_Comm_size (MPI_COMM_WORLD, &size);
        if (rc != MPI_SUCCESS) {
                fprintf (stderr, "MPI_Comm_size() failed");
                goto exit_with_error;
        }

        rc = MPI_Comm_rank (MPI_COMM_WORLD, &myrank);
        if (rc != MPI_SUCCESS) {
                fprintf (stderr, "MPI_Comm_rank() failed");
                goto exit_with_error;
        }

        fprintf (stdout, "Hello, I am rank %d/%d", myrank, size);

        MPI_Finalize();

        return EXIT_SUCCESS;

 exit_with_error:
        MPI_Finalize();
        return EXIT_FAILURE;
}
```

In the same location as above create a singularity definition file, note that we choose to compile and install the same OpenMPI version as we will use on Rāpoi


```bash
Bootstrap: docker
From: ubuntu:latest

%files
    mpitest.c /opt

%environment
    export OMPI_DIR=/opt/ompi
    export SINGULARITY_OMPI_DIR=$OMPI_DIR
    export SINGULARITYENV_APPEND_PATH=$OMPI_DIR/bin
    export SINGULAIRTYENV_APPEND_LD_LIBRARY_PATH=$OMPI_DIR/lib

%post
    echo "Installing required packages..."
    apt-get update && apt-get install -y wget git bash gcc gfortran g++ make file

    echo "Installing Open MPI"
    export OMPI_DIR=/opt/ompi
    export OMPI_VERSION=4.0.5  #NOTE matching version to that on Raapoi
    export OMPI_URL="https://download.open-mpi.org/release/open-mpi/v4.0/openmpi-$OMPI_VERSION.tar.bz2"
    mkdir -p /tmp/ompi
    mkdir -p /opt
    # Download
    cd /tmp/ompi && wget -O openmpi-$OMPI_VERSION.tar.bz2 $OMPI_URL && tar -xjf openmpi-$OMPI_VERSION.tar.bz2
    # Compile and install
    cd /tmp/ompi/openmpi-$OMPI_VERSION && ./configure --prefix=$OMPI_DIR && make install
    # Set env variables so we can compile our application
    export PATH=$OMPI_DIR/bin:$PATH
    export LD_LIBRARY_PATH=$OMPI_DIR/lib:$LD_LIBRARY_PATH
    export MANPATH=$OMPI_DIR/share/man:$MANPATH

    echo "Compiling the MPI application..."
    cd /opt && mpicc -o mpitest mpitest.c
```

Now we build our container locally, giving it a sensible name.  We need ```OpenMPI-4.0.5``` to use this, so let's include that in the name
```bash
sudo singularity build test-openmpi-4.0.5.sif test-openmpi-4.0.5.def
```

Copy that file to Rāpoi somehow - Filezilla, rsync or similar.  I'll just use sftp for simplicity
```bash
sftp <username>@raapoi.vuw.ac.nz
put test-openmpi-4.0.5.sif
```
Now on **Rāpoi** copy that file to a sensible location, I used ```~/projects/examples/singularity/openMPI``` again

```bash
mv test-openmpi-4.0.5.sif ~/projects/examples/singularity/openMPI/
cd ~/projects/examples/singularity/openMPI/
```

In that location create a sbatch file

openmpi-test.sh
```bash
#!/bin/bash
#SBATCH --job-name=mpi_test
#SBATCH --time=00-00:02:00
#SBATCH --output=out_test.out
#SBATCH --error=out_test.err
#SBATCH --partition=parallel
#SBATCH --ntasks=2
#SBATCH --cpus-per-task=1
#SBATCH --tasks-per-node=1
#SBATCH --mem-per-cpu=1GB
#SBATCH --constraint="IB,AMD"
#SBATCH --nodes=2

module use /home/software/tools/eb_modulefiles/all/Core
module unuse /home/software/tools/modulefiles # to prevent conflicts with the old modules
module load GCC/10.2.0
module load OpenMPI/4.0.5
module load Singularity/3.7.3 # Note this is a new singularity build

CONPATH=$HOME/projects/examples/singularity/openMPI
mpirun -np 2 singularity exec $CONPATH/test-openmpi-4.0.5.sif /opt/mpitest
```

Submit that to slurm and see the output
```bash
sbatch openmpi-test.sh
squeue -u $USER  # see the job
cat out_test.out # examine the output after the job is done
```
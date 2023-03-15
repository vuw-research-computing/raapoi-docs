## Simple OpenMPI with Singularity using the hybrid approach.

The hybrid approach is one way of getting OpenMPI working with containers. It requires the OpenMPI version inside the container to match the OpenMPI outside the container (loaded via module loading).

First check what openMPI version we have on Rāpoi.

On **Rāpoi** switch to our new modules
```bash
module unuse /home/software/tools/modulefiles # stop using the older modules
module use /home/software/tools/eb_modulefiles/all/Core #the new module files organised by compiler
module spider OpenMPI # search for openMPI - thre are several options, lets try
module spider OpenMPI/4.0.5  # we will use this one, which requires GCC/10.2.0
```

On your **local machine** we will create a very simple C openMPI program. Create this in a sensible place.  I used ```~/projects/examples/singularity/openMPI```

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

In the same location as above create a singularity definition file, note that we choose to compile and install the same OpenMPI version as we will use on Rāpoi.


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

Now we build our container locally, giving it a sensible name.  We need ```OpenMPI-4.0.5``` to use this, so let's include that in the name.
```bash
sudo singularity build test-openmpi-4.0.5.sif test-openmpi-4.0.5.def
```

Copy that file to Rāpoi somehow - Filezilla, rsync or similar.  I'll just use sftp for simplicity.
```bash
sftp <username>@raapoi.vuw.ac.nz
put test-openmpi-4.0.5.sif
```
Now on **Rāpoi** copy that file to a sensible location, I used ```~/projects/examples/singularity/openMPI``` again.

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
#SBATCH --mem=1GB
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
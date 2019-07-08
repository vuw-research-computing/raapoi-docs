

Researchers can use Docker or Singularity containers within the cluster.  This is a great way to run difficult-to-compile applications or to share workflows among colleagues.

## Running an interactive container

User can run within a container interactively, this is great for testing code before running a job.  Here is an example of running within a docker container that has the blockchain software called BlockSci:

```
module load singularity
srun --pty -c 4 --mem=16G bash
singularity pull docker://tislaamo/blocksci
singularity shell blocksci.simg
```

Once you have typed the _singularity shell_ command you will be within the
container and can type the commands available from within the container such as
the BlockSci utility **blocksci_parser**

## Running a container in batch

Running a batch job with containers is similar to running a regular job, but will ultimately depend on how the container was created, so your mileage may vary.  Here is an example batch submit script that will run the BlockSci software that was created in an Ubuntu docker image, lets name the submit file runContainer.sh:

```
#SBATCH -J blockchain_parser
#SBATCH -c 4
#SBATCH --mem=16G
#SBATCH --mailtype=BEGIN,END,FAIL
#SBATCH --mail-user=myemail@email.net
#SBATCH --time=12:00:00

module load singularity
singularity exec blocksci.simg blocksci_parser --output-directory \ bitcoin-data update disk --coin-directory bitcoin
```

Now to run the file you can:

```
module load singularity
singularity pull docker://tislaamo/blocksci
sbatch runContainer.sh
```

Note that _singularity shell_ is primarily for interactive use and _singularity exec_ (or possibly _singularity run_) are for executing the applications that were built within the container directly.  It is important to know how the container was created to make effective use of the software.
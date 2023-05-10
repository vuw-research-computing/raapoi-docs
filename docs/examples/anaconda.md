## Using Anaconda/Miniconda/conda - idba

Many users use Anaconda/Miniconda to manage software stacks.  One way to do this is to use singularity containers with the conda environment inside - this allows the conda environment to load quickly as the many small conda files are inside a container which the file system sees as one file.

However, this is also an additional bit of complexity so many users just use conda outside of singularity.  You can install your own version of Anaconda/Miniconda to your home directory or scratch.  We have also got packaged versions of Anaconda/Miniconda installed with our module loading system.

Anaconda has many built in packages so we will use that in our examples, but Miniconda is available if you want a more minimal initial setup.

```bash
module load old-mod-system/Anaconda3/2020.11 
```

Let's create a new conda environment for this example, in a sensible location, I used ```~/examples/conda/idba```

```bash
conda create --name idba-example  # press y for the Proceed prompt if it looks correct
conda activate idba-example  #activate our example environment.
```

Conda environments are beyond the scope of this example, but they are a good way to contain all the dependencies and programs for a particular workflow, in this case, idba.

Install idba in our conda environment.  ** Note that best practise is to do the install on a compute node ** We'll just do it here on the login node for now - the code will run slower on the compute nodes as a result!

```bash
conda install -c bioconda idba
```

Idba is a genome assembler, we will use paired-end illumina reads of E. coli.  The data is available on an Amazon S3 bucket (a cloud storage location), and we can download it using wget.
```bash
mkdir data  # put our data in a sensible location
cd data
wget --content-disposition goo.gl/JDJTaz #sequence data
wget --content-disposition goo.gl/tt9fsn #sequence data
cd ..  #back to our project directory
```

The reads we have are paired-end fastq files but idba requires a fasta file. We can use a tool installed with idba to convert them. We'll do this on the Rāpoi login node as it is a fast task that doesn't need many resources.

```bash
fq2fa --merge --filter data/MiSeq_Ecoli_MG1655_50x_R1.fastq data/MiSeq_Ecoli_MG1655_50x_R2.fastq data/read.fa
```

To create our submission script we need to know the path to our conda enviroment. To get this:
```bash
conda env list
```
You'll need to find your ```idba-example``` environment, and next to it is the path you'll need for your submission script.  In my case:
```bash
# conda environments:
#
base                  *  /home/andre/anaconda3
idba-example          /home/andre/anaconda3/envs/idba-example  # We need this line, it'll be different for you!
```


Create our sbatch submission script. Note that this sequence doesn't need a lot of memory, so we'll use 3G. To see your usage after the job has run use ```vuw-job-report <job-id>```

*idba_submit.sh*
```bash
#!/bin/bash

#SBATCH --job-name=idba_test
#SBATCH -o _output.out
#SBATCH -e _output.err
#SBATCH --time=00:5:00
#SBATCH --partition=quicktest
#SBATCH --ntasks=12
#SBATCH --mem=3G

module load old-mod-system/Anaconda3/2020.11
eval "$(conda shell.bash hook)" # basically inits your conda - prevents errors like: CommandNotFoundError: Your shell has not been properly configured ...
conda activate /home/andre/anaconda3/envs/idba-example  # We will need to activate our conda enviroment on the remote node
idba idba_ud -r data/read.fa -o output
```

To submit our job
```bash
sbatch idba_submit.sh
```

To see our job running or queuing
```bash
squeue -u $USER
```
This job will take a few minutes to run, generally less than 5.
When the job is done we can see the output in the output folder.  We can also see the std output and std err in the files ```_output.out and _output.err```.  The quickest way to examine them is to ```cat``` the files when the run is done.
```bash
cat _output.out
```



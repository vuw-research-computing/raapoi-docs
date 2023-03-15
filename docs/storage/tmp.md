### Temp Disk Tips

This storage is very fast on the AMD nodes and GPU nodes.  It is your job to move data to the tmp space and clean it up when done.

There is very little management of this space and currently it is not visible to slurm for fair use scheduling - in other words someone else might have used up most of the temp space on the node!  This is generally not the case though.

A rough example of how you could use this in an sbatch script

```bash
#!/bin/bash
#
#SBATCH --job-name=bash_test
#
#SBATCH --partition=quicktest
#
#SBATCH --cpus-per-task=2 #Note: you are always allocated an even number of cpus
#SBATCH --mem=1G
#SBATCH --time=10:00

# Do the needed module loading for your use case
module load etc

#make a temporary directory with your usename so you don't tread on others
mkdir /tmp/<username>

#Copy dataset from scratch to the local tmp on the node (could also use rsync)
cp -r /nfs/scratch/<user>/dataset /tmp/<user>/dataset

Process data against /tmp/<user>/dataset
Lets say the data is output to /tmp/<user>/dataoutput/

# Copy data from output to your scratch - I suggest not overwriting your original dataset!
cp -r /tmp/<user>/dataoutput/* /nfs/scratch/<user>/dataset/dataoutput/

# Delete the data you copy to and created on tmp
rm -rf /tmp/<user>  #DANGER!!  
```
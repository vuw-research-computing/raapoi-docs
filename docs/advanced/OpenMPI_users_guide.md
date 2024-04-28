## OpenMPI users guide

### Which versions of OpenMPI are working on Rāpoi?

There are a number of versions of OpenMPI on Rāpoi, although many of these are old installations (prior to an OS update and changes to the module system) and *may* no longer work.
Generally speaking, your best bet is to try a version which appears when you search via `module spider OpenMPI` (noting that the capital 'O M P I' is important here).
A few examples of relatively recent version of OpenMPI which are available (as of April 2024) are `OpenMPI/4.1.1`, `OpenMPI/4.1.4` and `OpenMPI/4.1.6`.

Each of these OpenMPI modules has one or more of pre-requisite modules that need to be loaded first (generally a specific version of GCC compilers).
To find out what you need to load first for a specific version of Python you just need to check the output of `module spider OpenMPI/x.y.z` (with the appropriate values for x,y,z).
One of the examples below shows how to use `OpenMPI/4.1.6`.
In cases where your code utilises software from another module which also requires a specific GCC module, that will dictate which version of OpenMPI to load (i.e. whichever one depends on the same GCC version).
Otherwise, you are free to use any desired OpenMPI module. 



### Known issues and workarounds


There is a known issue with the communication/networking interfaces with several of the installations of OpenMPI. 
The error/warning messages occur sporadically, making it difficult to pin down and resolve, but it is likely there is a combination of internal and external factors that cause this (OpenMPI is a very complex beast).
The warning messages take the form:
```
Failed to modify UD QP to INIT on mlx5_0: Operation not permitted
```
A workaround is described below, this page will be updated in the future when a more permanent solution is found.

Exectute your mpi jobs using the additional arguments:
```bash
mpirun -mca pml ucx -mca btl '^uct,ofi' -mca mtl '^ofi' -np $SLURM_NTASKS <your executable>
```
This will ensure OpenMPI avoids trying to use the communication libraries which are problematic.
If your executable is launched without using mpirun (i.e. it implements its own wrapper/launcher), you will instead need to set the following environment variables:
```bash
export OMPI_MCA_btl='^uct,ofi'
export OMPI_MCA_pml='ucx'
export OMPI_MCA_mtl='^ofi'
```



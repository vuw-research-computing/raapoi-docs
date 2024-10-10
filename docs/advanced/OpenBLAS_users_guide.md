## OpenBLAS/FlexiBLAS users guide

### What is BLAS/OpenBLAS/FlexiBLAS?

BLAS stands for Basic Linear Algebra Subprograms and consists of a set routines that probide standard building blocks for numerical calculations involving linear algebra (e.g. addition and multiplication of vectors and matrices). 
Software such as OpenBLAS and FlexiBLAS (and other variants) provide optimised implementations and extensions of such routines. 

### How do I know if my software uses OpenBLAS/FlexiBLAS?

Linear algebra is fundamental to a broad range of numerical algorithms, so it is quite common for computationally intensive software to rely on some form of BLAS implementation. 
One way you can potentially find out if OpenBLAS or FlexiBLAS might be used by your software is to load the modules associated with your piece of software, then execute `module list` and see if OpenBLAS of FlexiBLAS are listed. 
If yes, it is not a guarantee they are being used, but is certainly a possibility. 


### Known issues

Some versions of OpenBLAS have been installed with OpenMP support. 
When using software that links to one of those versions it is essential that you set the environment variable `OMP_NUM_THREADS` to the desired number of threads. 
If you do not set this environment variable, then your software may hang when OpenBLAS is trying to determine how many threads are available to use (unless you allocate an obscene and uneccessary amount of memory). 
The recommended approach is to add the line `export OMP_NUM_THREADS=$SLURM_NTASKS` after you have loaded all of the modules you need. 
(You will need to change this accordingly if you are using some form of hybrid paralellism, and/or are setting a value of `--ncpus-per-task` which is more than one.)



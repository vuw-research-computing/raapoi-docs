## OpenBLAS/FlexiBLAS users guide

### What is BLAS/OpenBLAS/FlexiBLAS?

BLAS stands for Basic Linear Algebra Subprograms and consists of a set routines that probide standard building blocks for numerical calculations involving linear algebra (e.g. addition and multiplication of vectors and matrices). 
Software such as OpenBLAS and FlexiBLAS (and other variants) provide optimised implementations and extensions of such routines. 

### How do I know if my software uses OpenBLAS/FlexiBLAS?

Linear algebra is fundamental to a broad range of numerical algorithms, so it is quite common for computationally intensive software to rely on some form of BLAS implementation. 
One way you can potentially find out if OpenBLAS or FlexiBLAS might be used by your software is to load the modules associated with your piece of software, then execute `module list` and see if OpenBLAS of FlexiBLAS are listed. 
If yes, it is not a guarantee they are being used, but is certainly a possibility. 


### Known issues

Most versions of OpenBLAS have been installed with OpenMP support.
Some of the newer versions (0.3.20 onwards) have a bug which can cause OpenBLAS to hang on startup (unless you allocate an obscene and uneccessary amount of memory).
When using software that links to one of those versions it is essential that you: (i) set the environment variable `OMP_NUM_THREADS` to the desired number of threads, and/or (ii) set the environment varialbe `OMP_PROC_BIND=true` (or any other valid option, apart from `false`).  
The recommended approach is to add the line `export OMP_NUM_THREADS=$SLURM_NTASKS` and/or `export OMP_PROC_BIND=true` after you have loaded all of the modules you require. 
(You will need to change `export OMP_NUM_THREADS=$SLURM_NTASKS` accordingly if you are using some form of hybrid paralellism, and/or are setting a value of `--ncpus-per-task` which is more than one.)



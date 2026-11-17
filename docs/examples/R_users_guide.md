## R users guide

### Which versions of R are working on Rāpoi?

There are a number of versions of R on Rāpoi, although many of these are old installations (prior to an OS update and changes to the module system) and no longer work.
There are three relatively recent versions of R which currently work (as of November 2026), these are `R/4.4.2`, `R/4.5.2` and `R/4.6.1`.
Each of these modules has a couple of pre-requisite modules that need to be loaded prior to loading the R module. 
To find out what you need to load first you just need to check the output of `module spider R/x.y.z` (with the appropriate values for x,y,z).
The following example shows how to use `R/4.6.1`.

### Loading R packages & running a simple job

First login to Rāpoi and load the R module:
```bash
module purge               # clean/reset your environment
module load config         # reload utilities such as vuw-job-report
module load GCC/14.3.0     # pre-requisites for the new R module
module load OpenMPI/5.0.8
module load R/4.6.1   
```

Then run R on the command line:

```bash
R
```

Test library existence:
```R
> library(devtools)
```
This should load the package, and give some output like this:

```R
Loading required package: usethis
```

To quit R, type:

```R
> q()
```

Next create a bash submission script called ```r_submit.sh``` (or another name of your choice) using your preferred text editor, e.g. nano.

```bash
#!/bin/bash
#
#SBATCH --job-name=r_test
#SBATCH -o r_test.out
#SBATCH -e r_test.err
#
#SBATCH --cpus-per-task=2 #Note: you are always allocated an even number of cpus
#SBATCH --mem=1G
#SBATCH --time=10:00

module purge
module load GCC/14.3.0
module load OpenMPI/5.0.8
module load R/4.6.1   

Rscript mytest.R
```

Save this to the current working directory, and then create another file using your preferred text editor called ```mytest.R``` (or another name of your choice) containing the following R commands:
```R
library(devtools)

sprintf("Hello World!")
```
then run it with the previously written bash script:  
```bash
sbatch r_submit.sh 
```
This submits a task that should execute quickly and create files in the directory from which it was run.
Examine ```r_test.out```. You can use an editor like nano, vi or emacs, or you can just ```cat``` or ```less``` the file to see its contents on the terminal. You should see:
``` "Hello World"```

### Installing additional R packages/extensions in your local user directory

If you are in need of additional R packages which are not included in the R installation, you may intall them into your user directory.

Start by launching an R session
```bash
module purge
module load GCC/14.3.0
module load OpenMPI/5.0.8
module load R/4.6.1
mkdir -p "/nfs/scratch/${USER}/R_libs"          # Create this directory for the first time
export R_LIBS_USER=/nfs/scratch/${USER}/R_libs  # Export this directory path where R will install libraries for your user
export OMP_NUM_THREADS=1
R
```
Then, supposing you want to install a package from CRAN named "fortunes". 
If this is the first time you are attempting to install local packages for this R version then the steps look something like this.
```R
> library(fortunes) # confirm that fortunes is not already available
> install.packages("fortunes")
Installing package into ‘/nfs/scratch/admhashmi/R_libs’
(as ‘lib’ is unspecified)
--- Please select a CRAN mirror for use in this session ---
Secure CRAN mirrors

<long list of mirrors, the NZ mirror was number 49 in my list>
Selection: 49
trying URL 'https://cran.stat.auckland.ac.nz/src/contrib/fortunes_1.5-5.tar.gz'
Content type 'application/x-gzip' length 206140 bytes (201 KB)
==================================================
downloaded 201 KB

* installing *source* package ‘fortunes’ ...
** this is package ‘fortunes’ version ‘1.5-5’
** package ‘fortunes’ successfully unpacked and MD5 sums checked
** using staged installation
** R
** inst
** byte-compile and prepare package for lazy loading
** help
*** installing help indices
** building package indices
** installing vignettes
** testing if installed package can be loaded from temporary location
** testing if installed package can be loaded from final location
** testing if installed package keeps a record of temporary installation path
* DONE (fortunes)
```

In future, when you run this version of R, it should automatically check the local user directory created above for installed packages. Any other packages you install in future should automatically go into this directory as well (assuming you don't play around with `.libPaths()`).

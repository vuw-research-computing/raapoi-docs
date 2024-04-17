## R users guide

### Which versions of R are working on Rāpoi?

There are a number of versions of R on Rāpoi, although many of these are old installations (prior to an OS update and changes to the module system) and no longer work.
There are three relatively recent versions of R which currently work (as of April 2024), these are `R/3.6.3`, `R/4.1.0` and `R/4.2.0`.
Each of these modules has a couple of pre-requisite modules that need to be loaded prior to loading the R module. 
To find out what you need to load first you just need to check the output of `module spider R/x.y.z` (with the appropriate values for x,y,z).
The following example shows how to use `R/4.2.0`.

### Loading R packages & running a simple job

First login to Rāpoi and load the R module:
```bash
module purge                         # clean/reset your environment
module load config                   # reload utilities such as vuw-job-report
module load GCC/11.2.0 OpenMPI/4.1.1 # pre-requisites for the new R module
module load R/4.2.0   
```

Then run R on the command line:

```bash
R
```

Test library existence:
```R
> library(tidyverse)
```
This should load the package, and give some output like this:

```R
── Attaching packages ──────────────────────────── tidyverse 1.3.1 ──
✔ ggplot2 3.3.5     ✔ purrr   0.3.4
✔ tibble  3.1.6     ✔ dplyr   1.0.8
✔ tidyr   1.2.0     ✔ stringr 1.4.0
✔ readr   2.1.2     ✔ forcats 0.5.1
── Conflicts ─────────────────────────────── tidyverse_conflicts() ──
✖ dplyr::filter() masks stats::filter()
✖ dplyr::lag()    masks stats::lag()
```
(These conflicts are normal and can be ignored.)

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
module load GCC/11.2.0 OpenMPI/4.1.1
module load R/4.2.0   

Rscript mytest.R
```

Save this to the current working directory, and then create another file using your preferred text editor called ```mytest.R``` (or another name of your choice) containing the following R commands:
```R
library(tidyverse)

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
module load GCC/11.2.0 OpenMPI/4.1.1
module load R/4.2.0
R
```
Then, supposing you want to install a package from CRAN named "A3". 
If this is the first time you are attempting to install local packages for this R version then the steps look something like this.
```R
> library(A3) # confirm that foo is not already available
> install.packages('A3')
Warning in install.packages("A3") :
  'lib = "/home/software/EasyBuild/software/R/4.2.0-foss-2021b/lib64/R/library"' is not writable
Would you like to use a personal library instead? (yes/No/cancel) yes
Would you like to create a personal library
‘/nfs/home/<username>/R/x86_64-pc-linux-gnu-library/4.2’
to install packages into? (yes/No/cancel) yes
--- Please select a CRAN mirror for use in this session ---

Secure CRAN mirrors

<long list of mirrors, the NZ mirror was number 54 in my list>
Selection: 54
trying URL 'https://cran.stat.auckland.ac.nz/src/contrib/A3_1.0.0.tar.gz'
<additional output from the installation steps...>
```

In future, when you run this version of R, it should automatically check the local user directory created above for installed packages. Any other packages you install in future should automatically go into this directory as well (assuming you don't play around with `.libPaths()`).

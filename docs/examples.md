
## Simple Python program using virtualenv and pip

First we need to create a working directory and move there
```bash
mkdir python_test
cd python_test
```
Next we load the python 3 module and use python 3 to create a python virtualenv.  This way we can install pip packages which are not installed on the cluster
```bash
module load python/3.6.6
python3 -m venv mytest
```

Activate the `mytest` virtualenv and use pip to install the `webcolors` package
```bash
source mytest/bin/activate
pip install webcolors
```

Create the file test.py with the following contents using nano
```python
import webcolors
from random import randint
from socket import gethostname

colour_list = list(webcolors.css3_hex_to_names.items())
requested_colour = randint(0,len(colour_list))
colour_name = colour_list[requested_colour][1]

print("Random colour name:", colour_name, " on host: ", gethostname())
```

Alternatively download it with wget:
```bash
wget https://raw.githubusercontent.com/\
vuw-research-computing/raapoi-tools/\
master/examples/python_venv/test.py
```

Using nano create the submissions script called python_submit.sh with the following content - change `me@email.com` to your email address.
```bash
#!/bin/bash
#
#SBATCH --job-name=python_test
#SBATCH -o python_test.out
#SBATCH -e python_test.err
#
#SBATCH --cpus-per-task=1
#SBATCH --mem-per-cpu=1G
#SBATCH --time=10:00
#
#SBATCH --mail-type=BEGIN,END,FAIL
#SBATCH --mail-user=me@email.com

module load python/3.6.6

source mytest/bin/activate
python test.py
```

Alternatively download it with wget
```bash
wget https://raw.githubusercontent.com/\
vuw-research-computing/raapoi-tools/\
master/examples/python_venv/python_submit.sh
```

To submit your job to the Slurm scheduler
```bash
sbatch python_submit.sh
```

Check for your job on the queue with `squeue` though it might finish very fast.  The output files will appear in your working directory.


## Loading R packages & running a simple job

First login to Rāpoi and load the R/CRAN module:
```bash
module load R/CRAN      
```
(Note this will also load ```R/3.6```)


Then run R on the command line:


```bash
R
```

Test library existence:
```R
library(ggplot2)
```
This should load the package.
Metapackages like ```tidyverse``` currently don't load on Rāpoi, but components can be loaded individually: 

```R
> library(tidyr)
> library(dplyr)
```

Next create a bash submission script using your preferred text editor. An example submission script may look something like:
a file called r_submit.sh with:
```bash
#!/bin/bash
#
#SBATCH --job-name=r_test
#SBATCH -o r_test.out
#SBATCH -e r_test.err
#
#SBATCH --cpus-per-task=1
#SBATCH --mem-per-cpu=1G
#SBATCH --time=10:00
#
```

Save this to the current working directory, then run: 

```bash
module load R/CRAN
```

```bash
Rscript mytest.R
```


and then create another file with a test R script called ```mytest.R``` with:

```R
library(tidyr)
library(dplyr)
library(ggplot2)

sprintf("Hello World!")
```
then run it with the previously written bash script:  
```bash
sbatch r_submit.sh 
```
This submits a task that should execute quickly and create files in the directory from which it was run.
Examining ```r_test.out``` (with nano, cat or less) should print:
``` "Hello World"```
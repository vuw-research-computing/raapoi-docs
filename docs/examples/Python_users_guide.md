## Python users guide

### Which versions of Python are working on Rāpoi?

There are a number of versions of Python on Rāpoi, although many of these are old installations (prior to an OS update and changes to the module system) and *may* no longer work.
Generally speaking, your best bet is to try a version which appears when you search via `module spider Python` (noting that the capital 'P' in `Python` is important here).
A few examples of relatively recent version of Python which are available (as of April 2024) are `Python/3.9.5`, `Python/3.10.8` and `Python/3.11.5`.

Each of these Python modules has one or more of pre-requisite modules that need to be loaded first (generally a specific version of GCC compilers).
To find out what you need to load first for a specific version of Python you just need to check the output of `module spider Python/x.y.z` (with the appropriate values for x,y,z).
One of the examples below shows how to use `Python/3.9.5`.
In cases where your Python code needs to interact with software from another module which also requires a specific GCC module, that will dictate which version of Python to load (i.e. whichever one depends on the same GCC version).
Otherwise, you are free to use any desired Python module. 

The Python installations generally have a minimal number of packages/libraries installed.
If you require additional packages/libraries it is recommended to create a virtual environment and install any desired packages within that environment.
This is illustrated in the examples below using both virtualenv/pip and anaconda/conda. 

### Simple Python program using virtualenv and pip

First we need to create a working directory and move there
```bash
mkdir python_test
cd python_test
```
Next we load the python 3 module and use python 3 to create a python virtualenv.  This way we can install pip packages which are not installed on the cluster
```bash
module load GCCcore/10.3.0
module load Python/3.9.5
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

colour_list = list(webcolors.CSS3_HEX_TO_NAMES.items())
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
#SBATCH --cpus-per-task=2 #Note: you are always allocated an even number of cpus
#SBATCH --mem=1G
#SBATCH --time=10:00
#
#SBATCH --mail-type=BEGIN,END,FAIL
#SBATCH --mail-user=me@email.com

module load GCCcore/10.3.0
module load Python/3.9.5

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






{%
include-markdown "examples/anaconda.md"
%}






## Starting and Working with a Jupyter Notebook

Running Jupyter notebooks on Rāpoi is usually a two step processes.  First you start the jupyter server on a compute node - either via an interactive session or an sbatch job.  Then you connect to Rāpoi again via a new ssh session port forwarding the port selected by Jupter to your local machine for the web session.  There is a potentially simpler method at the end of this guide using firefox and tab containers.

The first step is getting and modifying a submission script.

Example submission scripts are included at `/home/software/vuwrc/examples/jupyter/` 
```bash 
notebook-bare.sh # The base notebook script - you manage your dependancies via pip
notebook-anaconda.sh # a version for if you prefer anaconda
R-notebook.sh # Using the R kernel instead
R-notebook-anaconda.sh # R kernel and managed by anaconda
```

All these scripts will need to be copied to your working directory and modified to suit your needs.

In each case you'll need to install your dependancies first - at a bare minimum you'll need Jupyter, installed either via pip or anaconda.

Note if you are intending to do anything needing GPU in your notebooks,  you'll need to do all these installs in the gpu or highmem nodes as you'll likely need the relavent CUDA modules loaded during the installs.

### notebook-bare.sh example
__Step 1:__ The best way to start jupyter is with a batch submit script.  We have created an example script.  You can copy this script from one available on the cluster, just type the following:
```
cp /home/software/vuwrc/examples/jupyter/notebook-bare.sh notebook.sh
```
If you are using Anaconda and have installed it in the default location you need to use the following submit file instead:
```
cp /home/software/vuwrc/examples/jupyter/notebook-anaconda.sh notebook-anaconda.sh
```

If you have any python dependancies you will need to install them before you run your script.  You will also have to install jupyter. Currenly you'll need to do that in an interactive session.  You only need to do this once.
```bash
srun -c 4  --mem=8G --partition=quicktest --time=0-01:00:00 --pty bash # get a 1 hour interactive session on quicktest
#prompt changes to something like 
#<username@itl02n02> you are now "on" a quicktest node

# Load required module for the python version in the notebook-bare.sh
module load gompi/2022a
module load Python/3.10.4-bare

python3 -m venv env # setup python virtual env in the env directory
pip install jupyterlab pandas plotnine # install dependancies - you *must* at least install jupyter

#exit the interactive session
exit
#prompt changes to something like 
#<username@raapoi-master> you are now back on the login/master node
```

This script is ready to run as is, but we recommend editing it to satisfy your own CPU, memory and time requirements.  Once you have edited the file you can run it thusly:

```
sbatch notebook.sh
```

or if using Anaconda:

```
sbatch notebook-anaconda.sh
```
__NOTE:__ If you copied the R notebook script, replace notebook.sh with R-notebook.sh

This will submit the file to run a job.  It may take some time for the job to
run, depending on how busy the cluster is at the time.  Once the job begins to
run you will see some information in the file called notebook-JOBID.out (JOBID
will be the actual jobid of this job, eg notebook-478903.out.  If you view this
file (users can type `cat notebook-JOBID.out` to view the file onscreen).  You will see a line such as:
```
    Or copy and paste one of these URLs:
        http://130.195.19.20:47033/lab?token=<some string of characters>
        http://127.0.0.1:47033/lab?token=<some string of characters>
```

The 2 important pieces of information here are the IP address, in this case *130.195.19.20* and the port number, *47033*.   These numbers should be different for you since the port number is random, although the IP Address may be the same since we have a limited number of compute nodes. Also notice after the ?token= you will see a random hash.  This hash is a security feature and allows you to connect to the notebook.  You will need to use these to view the notebook from your local machine.  

__Step 2:__ To start working with the notebook you will need to tunnel a ssh
session.  In your SSH tunnel you will use the cluster login node (raapoi.vuw.ac.nz)
to connect to the compute node (in the example above the compute node is at
address 130.195.19.20) and transfer all the traffic back and forth between your computer and the compute node).  

__Step 2a from a Mac:__

Open a new session window from Terminal.app or other terminal utility such as Xquartz and type the following:

```
ssh -L <PORT_NUMBER>:<IP_ADDRESS>:<PORT_NUMBER> username@raapoi.vuw.ac.nz
```

For example:

```
ssh -L 47033:130.195.19.20:47033 harrelwe@raapoi.vuw.ac.nz
```

Once you are at a prompt you can go to Step 3

__Step 2b: from Windows__

We recommend tunnelling using Git Bash, which is part of the [Git for Windows project](https://gitforwindows.org/) or [MobaXTerm](https://mobaxterm.mobatek.net/).  There are 2 methods for tunneling in Moba, one is command line, the other is GUI-based.

_Method 1 (Git Bash or MobaXterm):_
Command-line, start a local Git Bash or MobaXterm terminal (or try the GUI method, below)

From the command prompt type:
```
ssh -L <PORT_NUMBER>:<IP_ADDRESS>:<PORT_NUMBER> username@raapoi.vuw.ac.nz
```

For example:

```
ssh -L 47033:130.195.19.20:47033 harrelwe@raapoi.vuw.ac.nz
```

Once you are at a prompt you can go to Step 3


_Method 2 (MobaXterm):_
GUI-based, go to the Tunneling menu:

Now click on *New SSH Tunnel*

When you complete the creation of your tunnel click __Start all tunnels__.  Enter your password and reply "yes" to any questions asked about accepting hostkeys or opening firewalls.  You can safely exit the tunnel building menu.

__Step 3__

Now open your favorite web browser and then use the URL from your job output file and paste it in your browsers location bar, for example my full URL was:

  __http://130.195.19.20:47033/?token=badef11b1371945b314e2e89b9a182f68e39dc40783ed68e__

__Step 4__

One last thing you need to do is to replace the IP address with the word *localhost*.  This will allow your browser to follow the tunnel you just opened and connect to the notebook running on an engaging compute node, in my case my address line will now look like this:

  __http://localhost:47033/?token=badef11b1371945b314e2e89b9a182f68e39dc40783ed68e__

Now you can hit return and you should see your notebook running on an Engaging compute node.

__NOTE__: when you are done with your notebook, please remember to cancel your
job to free up the resources for others, hint: *scancel*

If you want more information on working with Jupyter, there is good
documentation, here: [Jupyter Notebooks](http://jupyter-notebook.readthedocs.io/en/latest/)

### Working with notebooks using Firefox tab containers

There is a perhaps simpler single step process of working with jupyter notebooks.  It relies on some nice features in [Firefox](https://www.mozilla.org/en-US/firefox/new/).  Firefox has tab containers - you can have categories of tabs that are basically independent from each other with separate web cookies but importantly for our case separate proxy settings.  You will also currenly need to get the firefox add-on [Container-Proxy](https://addons.mozilla.org/en-US/firefox/addon/container-proxy/) its [github page](https://github.com/bekh6ex/firefox-container-proxy)

Setup a tab container in Firefox called something like Raapoi.  Use the container-proxy extension to assign a proxy to that tab set.  I choose 9001, but you can use any fairly high port number - note it doens't matter if many people connect on the same port.

When you connect to raapoi, use ssh socks5 proxy settings.  In MacOS/linux/wsl2 
```bash
ssh -D 9001 <username>@raapoi.vuw.ac.nz
```

* putty: [Use Putty as a Socks Proxy](https://www.pwndefend.com/2022/06/25/how-to-use-putty-as-a-socks-proxy/)
* MobaXterm: [MobaXterm SOCKS5 Proxy - stackoverflow](https://stackoverflow.com/questions/63608784/mobaxterm-socks5-proxy-for-ssh)

In Firefox open a new tab by holding down the new tab + button and selecting your Raapoi tab container.  Any tabs opened with that container will have all their webtraffic directed via the Rāpoi login node.  Your laptop/desktop can't directly see all the compute nodes, but the login node can.  When you start a jupyter notebook and get the message:

```
    Or copy and paste one of these URLs:
        http://130.195.19.20:47033/lab?token=<some string of characters>
        http://127.0.0.1:47033/lab?token=<some string of characters>
```

You can just immediatly open `http://130.195.19.20:47033/lab?token=<some string of characters>` in your Raapoi container tab.

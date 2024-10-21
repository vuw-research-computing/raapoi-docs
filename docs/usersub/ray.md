## Ray 

Ray is a powerful distributed computing framework that can be used to accelerate High Performance Computing (HPC) workloads.

For this exercise, I'll need two terminal windows and a browser.

``` bash
# Terminal 1
# Start by requesting an interactive session
srun --cpus-per-task=4 --mem=8G --time=0-00:59:00 --pty bash

# Begin with a clear environment
module purge

# Create a python environment
module load gompi/2022a Python/3.10.4-bare
python -m venv env
source env/bin/activate

# Install Ray
pip install 'ray[default]'

# Verify installation
python -c "import ray;print(ray.__version__)";

# Start Ray head node by defining port,object_store_memory(osm),
# ncpus, dashboardhost; osm should be 80% of the requested mem 
# srun command. Here just using 20% 1.6G of 8G
PORT="6379"
OSM="1600000000"
NCPUS="4"
DBHOST="0.0.0.0"

ray start --head --port $PORT --object_store_memory $OSM --num-cpus $NCPUS --dashboard-host=$DBHOST

# A node name should be printed and
# Ray runtime started 
# with address to the dashboard
# In my case it was: 130.195.XX.XX:8265

# Terminal 2
# Now, leave this terminal running and 
# open a new terminal to that port and ip
# to start a tunnel
ssh -L 8265:130.195.XX.XX:8265 USERNAME@raapoi.vuw.ac.nz

# You should now open a browser to view your dashboard at
http://127.0.0.1:8265

# To submit a job
RAY_ADDRESS='http://130.195.XX.XX:8265' ray job submit --working-dir . -- python my_script.py

# To stop Ray
# Go back to terminal 1 and type
ray stop
```
## RStudio-Server

!!! Tip "**Pre-requisites:**"
    - [How to set up SSH Keys?](https://www.notion.so/Beginner-Workshop-1-Getting-started-with-Remote-HPC-system-1c7426d58abf8016ba1bc5116eb8e5bb?pvs=21)
    - [Request a Compute Node](https://www.notion.so/Request-a-Compute-Node-20d426d58abf80abbcabd81d04e95ef7?pvs=21)
    
    This tutorial assumes an interactive session requested via:
    `srun --mem=64G --cpus-per-task=8 -wamd01n04 --pty bash`

RStudio-Server on the cluster instructions modifies [[1]](https://rocker-project.org/use/singularity.html) and [[2]](https://www.c4.ucsf.edu/howto/rstudio-server.html)

**Step 1.** Create appropriate directories and pull singularity image to run RStudio-Server:

```bash
[user@amd01n04 ~]$ module purge; module load GCC/10.2.0 OpenMPI/4.0.5 Singularity/3.10.2
[user@amd01n04 ~]$ mkdir -p "$HOME/singularity-images"
[user@amd01n04 ~]$ singularity pull --dir="$HOME/singularity-images" --name=rstudio-server.sif docker://rocker/rstudio
```

**Step 2.** Create bind-mounts to later use inside the container.

```bash
[user@amd01n04 ~]$ workdir=$HOME/rstudio-server
[user@amd01n04 ~]$ mkdir -p "${workdir}"/{run,tmp,var/lib/rstudio-server}
[user@amd01n04 ~]$ chmod 700 "${workdir}"
[user@amd01n04 ~]$ cat > "${workdir}"/database.conf <<END
provider=sqlite
directory=/var/lib/rstudio-server
END
[user@amd01n04 ~]$ cat > "${workdir}"/rsession.sh <<END
#! /bin/sh
export OMP_NUM_THREADS=${SLURM_JOB_CPUS_PER_NODE}
export R_LIBS_USER=~/R/%p-library/%v-rocker-rstudio
exec /usr/lib/rstudio-server/bin/rsession "\${@}"
END
[user@amd01n04 ~]$ chmod +x "${workdir}"/rsession.sh
[user@amd01n04 ~]$ export SINGULARITY_BIND="${workdir}/run:/run,${workdir}/tmp:/tmp,${workdir}/database.conf:/etc/rstudio/database.conf,${workdir}/rsession.sh:/etc/rstudio/rsession.sh,${workdir}/var/lib/rstudio-server:/var/lib/rstudio-server"
[user@amd01n04 ~]$ export SINGULARITYENV_RSTUDIO_SESSION_TIMEOUT=0
[user@amd01n04 ~]$ export SINGULARITYENV_USER=$(id -un)
[user@amd01n04 ~]$ export SINGULARITYENV_PASSWORD=$(openssl rand -base64 15)
[user@amd01n04 ~]$ export PORT=$(/usr/bin/python3 -c 'import socket; s=socket.socket(); s.bind(("", 0)); print(s.getsockname()[1]); s.close()')
[user@amd01n04 ~]$ export IP=$(hostname -i)
```

**Step 3.** Run this to get instructions to connect to the RStudio-Server later.

```bash
[user@amd01n04 ~]$ cat 1>&2 <<END

A new instance of the RStudio Server was just launched. To access it,

1. SSH tunnel from your workstation using the following command from a terminal on your local workstation:

   IP="${IP}"; PORT="${PORT}"; ssh -L ${PORT}:${IP}:${PORT} ${SINGULARITYENV_USER}@raapoi.vuw.ac.nz

   and point your local web browser to <http://localhost:${PORT}>

2. Log in to RStudio Server using the following credentials:

   user: ${SINGULARITYENV_USER}
   password: ${SINGULARITYENV_PASSWORD}

When done, make sure to terminate everything by:

1. Exit the RStudio Session ("power" button in the top right corner of the RStudio window)

2. Cancel the job by issuing the following command on the login node:

      scancel -f ${SLURM_JOB_ID}
      
END
```

**Step 4.** Finally, start RStudio-Server

```bash
[user@amd01n04 ~]$ singularity exec --cleanenv --scratch /run,/tmp,/var/lib/rstudio-server --workdir ${workdir} ${HOME}/singularity-images/rstudio-server.sif rserver --www-port ${PORT} --auth-none=0 --auth-pam-helper-path=pam-helper --auth-stay-signed-in-days=30 --auth-timeout-minutes=0 --server-user=$(whoami) --rsession-path=/etc/rstudio/rsession.sh
```

**Step 5.** Inside a new terminal on your personal device, create a tunnel following the instructions of the above command.

```bash
[user@personal-device:~] IP="${IP}"; PORT="${PORT}"; ssh -L ${PORT}:${IP}:${PORT} ${SINGULARITYENV_USER}@raapoi.vuw.ac.nz
```

!!! Note
    Remember to note down output of the environment variables above.


Once the tunnel is set up successfully, go to your browser and with the port from the above output:

```bash
http://localhost:${PORT}/
```

For running this inside a batch script, submit the following via sbatch.

`submit.sl` 

```bash
#! /bin/bash
#SBATCH --time=00-01:00:00
#SBATCH --ntasks=2
#SBATCH --mem=8G
#SBATCH --output=rstudio-server.%j
#SBATCH --error=rstudio-server.%j.err
#SBATCH --export=NONE
# customize --output path as appropriate (to a directory readable only by the user!)

# Load Singularity module
module purge
module load GCC/10.2.0 OpenMPI/4.0.5 Singularity/3.10.2

# Create temporary directory to be populated with directories to bind-mount in the container
# where writable file systems are necessary. Adjust path as appropriate for your computing environment.
workdir=$HOME/rstudio-server

mkdir -p "${workdir}"/{run,tmp,var/lib/rstudio-server}
chmod 700 "${workdir}"
cat > "${workdir}"/database.conf <<END
provider=sqlite
directory=/var/lib/rstudio-server
END

# Set OMP_NUM_THREADS to prevent OpenBLAS (and any other OpenMP-enhanced
# libraries used by R) from spawning more threads than the number of processors
# allocated to the job.
#
# Set R_LIBS_USER to a path specific to Rocker/RStudio to avoid conflicts with
# personal libraries from any R installation in the host environment

cat > "${workdir}"/rsession.sh <<END
#! /bin/sh
export OMP_NUM_THREADS=${SLURM_JOB_CPUS_PER_NODE}
export R_LIBS_USER=~/R/%p-library/%v-rocker-rstudio
exec /usr/lib/rstudio-server/bin/rsession "\${@}"
END

chmod +x "${workdir}"/rsession.sh

export SINGULARITY_BIND="${workdir}/run:/run,${workdir}/tmp:/tmp,${workdir}/database.conf:/etc/rstudio/database.conf,${workdir}/rsession.sh:/etc/rstudio/rsession.sh,${workdir}/var/lib/rstudio-server:/var/lib/rstudio-server"

# Do not suspend idle sessions.
# Alternative to setting session-timeout-minutes=0 in /etc/rstudio/rsession.conf
# https://github.com/rstudio/rstudio/blob/v1.4.1106/src/cpp/server/ServerSessionManager.cpp#L126
export SINGULARITYENV_RSTUDIO_SESSION_TIMEOUT=0

export SINGULARITYENV_USER=$(id -un)
export SINGULARITYENV_PASSWORD=$(openssl rand -base64 15)
# Get unused socket per https://unix.stackexchange.com/a/132524
# Tiny race condition between the python & singularity commands
export PORT=$(/usr/bin/python3 -c 'import socket; s=socket.socket(); s.bind(("", 0)); print(s.getsockname()[1]); s.close()')
export IP=$(hostname -i)
cat 1>&2 <<END

A new instance of the RStudio Server was just launched. To access it,

1. SSH tunnel from your workstation using the following command from a terminal on your local workstation:

   IP="${IP}"; PORT="${PORT}"; ssh -L ${PORT}:${IP}:${PORT} ${SINGULARITYENV_USER}@raapoi.vuw.ac.nz

   and point your local web browser to <http://localhost:${PORT}>

2. Log in to RStudio Server using the following credentials:

   user: ${SINGULARITYENV_USER}
   password: ${SINGULARITYENV_PASSWORD}

When done, make sure to terminate everything by:

1. Exit the RStudio Session ("power" button in the top right corner of the RStudio window)

2. Cancel the job by issuing the following command on the login node:

      scancel -f ${SLURM_JOB_ID}
      
END

singularity exec "$HOME/singularity-images/rstudio-server.sif" \
    rserver --www-port "$PORT" \
            --auth-none=0 \
            --auth-pam-helper-path=pam-helper \
            --auth-stay-signed-in-days=30 \
            --auth-timeout-minutes=0 \
            --rsession-path=/etc/rstudio/rsession.sh \
            --server-user="$USER"
printf 'RStudio Server exited\n' 1>&2
```

Step 2. Once your job starts note the JOBID, read the output file for instructions to connect to the running RStudio-Server.

```bash
[user@raapoi-login:~]$ sbatch submit.sl; vuw-myjobs
Submitted batch job <job_id> 
[user@raapoi-login:~]$ cat rstudio-server.%j # %j is the job id 
```

For any help, please contact one of our support team members: [Support](../support.md)
## MATLAB GUI via X-Forwarding

!!! Warning "Friendly Reminder"
    *HPC is built to serve powerful computational work largely via commandline interface, kindly read our [FAQ section: Visualisation](../faq.md#visualisation). Please proceed only if you think non-GUI MATLAB is not an option. Kindly make sure your personal device has X-Server installed and running.*

### Installing and running an X Server on Windows

This tutorial explains how to install an X-Server on Windows. We will use the `VcXsrv`, a free X-server for this purpose.

**Steps:**

- Download the installer from here: [vcxsrv](https://sourceforge.net/projects/vcxsrv/files/latest/download)

- Run the installer.

    - Select `Full` under Installation Options and click `Next`
    - Select a target folder

To Run the Server:

- Open the `XLaunch` program (most likely on your desktop)

- Select `Multiple Windows` and click `Next`

- Select `Start no client` and click `Next`

- On the `Extra settings` window, click `Next`

- On the `Finish configuration` page click Finish

You have now started your X Server.

### Set up your console
In the `Git bash` or the windows command line (`cmd`) terminal, before you connect to an ssh server, you have to set the used display. Under normal circumstances, `VcXsrv` will start the `Xserver` as `display 0.0`. If for some reason the remote graphical user interface does not start later on, you can check, the actual display by right-clicking on the tray-icon of the `X Server` and select `Show log`. Search for `DISPLAY` in the log file, and you will find something like:

```
DISPLAY=127.0.0.1:0.0
```

In your terminal enter:

```
set DISPLAY=127.0.0.1:0.0
```

Now you are set up to connect to the server of your choice via:

```
ssh -Y <username>@raapoi.vuw.ac.nz
```

Notice, that on windows you will likely need the `-Y` flag for `X Server` connections, since it seems `-X` does not normally work.

### On _Rāpoi_

Once logged in allocate resources for an interactive session
```bash 
RAAPOI_USERNAME@raapoi-login:~$ srun --x11 -t0-01:00:00 -wamd01n01 --ntasks=8 --mem=32G --pty bash
```

!!! Tip
    Extend the time to maximum 5 hours with `-t0-04:59:59`

After your job starts and the prompt changes, run `matlab` and you'll see MATLAB GUI Window on your personal device.

```bash
RAAPOI_USERNAME@amd01n01:~$ module use /home/software/tools/eb_modulefiles/all/Core
RAAPOI_USERNAME@amd01n01:~$ module load MATLAB/2024a
RAAPOI_USERNAME@amd01n01:~$ module load fosscuda/2020b
RAAPOI_USERNAME@amd01n01:~$ matlab -softwareopengl
```

For any help, please contact one of our support team members: [Support](../support.md)
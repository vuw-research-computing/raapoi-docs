## Gview on Rāpoi

First, log in to Rāpoi using ``-X`` flag when using graphical applications on your local machine.

```bash
ssh -X <username>@raapoi.vuw.ac.nz
```


Then, from the login node. Get an interactive session by passing ``--x11`` flag.

```
<username>@raapoi-login:~$ srun --x11 --pty bash
```


Once a compute node has been allocated, load Gaussview module

```
<username>@amd01n01:~$ module load Gaussview/6.1
```


Launch gview and it should open graphical windows on your local device. 

```
<username>@amd01n01:~$ gview
```


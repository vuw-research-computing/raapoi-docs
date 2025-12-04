## VSCode 

!!! Tip
    Windows users are recommended to use [`Git Bash`](https://git-scm.com/downloads) for the following instructions to work. 

The instructions below should let users run their VSCode session on a compute node.

Pre-requisite installations: 

- A recent version of vscode.
- Extension `Remote - SSH` from [https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-ssh](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-ssh).

Step 1. On your local machine, create ssh keys _(existing ssh keys can also be used - no need to create new ones)_

```bash
user@local:~$ ssh-keygen
```

Follow the prompts on the terminal to generate ssh-key pair, and note the directory where keys are being saved. 

Step 2. Send the _public_ key to _Rāpoi_

```bash
user@local:~$ ssh-copy-id -i ~/path/to/public/key RAAPOI_USERNAME@raapoi.vuw.ac.nz
```

The path `~/path/to/public/key` should be the same as displayed when generating the ssh-key `~/.ssh/id_rsa.pub` in some cases. 

Step 3. Test the new keys 

```bash
user@local:~$ ssh -i ~/path/to/public/key RAAPOI_USERNAME@raapoi.vuw.ac.nz
```

Step 4. On your local machine, update `ssh config` file

Create `~/.ssh/config` file if it does not exist. Add hostname details to it:

```bash
Host VSCode_Compute
    User <YOUR_RAAPOI_USERNAME>
    HostName amd01n01
    ProxyJump raapoi_login

Host raapoi_login
    HostName raapoi.vuw.ac.nz
    User <YOUR_RAAPOI_USERNAME>

Host *
    ForwardAgent yes
    ForwardX11 yes
    ForwardX11Trusted yes
    IdentityFile ~/.ssh/id_rsa # Add your own private key path here
    AddKeysToAgent yes
    StrictHostKeyChecking no
    UserKnownHostsFile /dev/null
```

Step 5. On your local machine, open a terminal window and login to _Rāpoi_ normally 

```bash 
user@local:~$ ssh raapoi_login
```

Once logged in alloc resources for the VSCode session
```bash 
RAAPOI_USERNAME@raapoi-login:~$ srun -t0-01:00:00 -wamd01n01 --mem=4G --pty bash
```

!!! Tip
    Extend the time to maximum 5 hours with `-t0-04:59:59`

Step 6. Connect VSCode session 

Open VSCode window, and click on the bottom left corner that says `Open a Remote Window`, and then choose `Connect to Host` and then selecting `VSCode_Compute` as a host. 

Once a connection is established, your VSCode session should be running on a compute node now. 

!!! Tip

    To speed up VSCode, there are steps mentioned in [the official VSCode docs](https://code.visualstudio.com/docs/configure/settings). Below is just a part of it: 

    Once connected, update VSCode's `/nfs/home/$USER/.vscode-server/data/Machine/settings.json`, and add the following lines to it:

    ```bash
    {
        "files.watcherExclude": {
            "**":true,
        },
        "files.exclude": {
            "**/.*": true,
        },
        "search.followSymlinks":false,
        "search.exclude": {
            "**":true,
        },
        "terminal.integrated.inheritEnv": false,
    }
    ```

    


Step 7. To close VSCode session. 

Go to `File` > `Close Remote Connection`

!!! Tip
    The instructions above assume that the node amd01n01 is up and has sufficient resources available. There may be times when this is not the case and you need to adapt these steps to access cpus on a different node. As a workaround you'll need to modify steps 4 and 5 to point towards a different node. If you do, you should make a note to revert these changes to utilise amd01n01 once it is available again.


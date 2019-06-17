# Storage and quotas

Currently users have 2 main storage areas:

* __/nfs/home/USERNAME__ - This is your Home Directory, each user has a *50 GB* quota limit.

* __/nfs/scratch/USERNAME__ - This is you scratch space, each user has a *5 TB* quota limit 

To view your current quota and usage use the _vuw-quota_ command, for example:

```
harrelwe@raapoi-master:~$ vuw-quota 

Quotas for home directory

                       Storage  Usage (GB)  Quota (GB)     % Used 
            /nfs/home/harrelwe        0.10       50.00      0.20%
```
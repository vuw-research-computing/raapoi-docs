# Storage and quotas

Currently users have 2 main storage areas:

* __/nfs/home/USERNAME__ - This is your Home Directory, each user has a *50 GB* quota limit.

* __/nfs/scratch/USERNAME__ - This is you scratch space, each user has a *5 TB* quota limit

Note: Home directory quotas cannot be increased, however if you need more space in your scratch folder let us know.

To view your current quota and usage use the _vuw-quota_ command, for example:

```
harrelwe@raapoi-master:~$ vuw-quota

Quotas for home directory

                       Storage  Usage (GB)  Quota (GB)     % Used
              /nfs/home/harrelwe       17.21       50.00     34.43%


Quota for scratch directory

                       Storage  Usage (GB)  Quota (GB)     % Used

           /nfs/scratch/harrelwe      133.39     5000.00      2.67%
```
